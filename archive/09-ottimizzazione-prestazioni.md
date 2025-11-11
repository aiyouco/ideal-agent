# Ottimizzazione delle Prestazioni

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Principi di Prestazione](#principi-di-prestazione)
3. [Livelli di Caching](#livelli-di-caching)
4. [Esecuzione Parallela](#esecuzione-parallela)
5. [Protocolli di Streaming](#protocolli-di-streaming)
6. [Routing dei Modelli](#routing-dei-modelli)
7. [Ottimizzazione dei Prompt](#ottimizzazione-dei-prompt)
8. [Gestione delle Risorse](#gestione-delle-risorse)
9. [Benchmark delle Prestazioni](#benchmark-delle-prestazioni)
10. [Pattern di Ottimizzazione](#pattern-di-ottimizzazione)

---

## Panoramica

L'ottimizzazione delle prestazioni è una preoccupazione di prima classe nell'architettura dell'agente ideale. Questo documento specifica strategie concrete, algoritmi e benchmark per raggiungere prestazioni production-grade.

### Obiettivi di Prestazione

| Metrica | Target | Obiettivo Ambizioso |
|--------|--------|---------------------|
| Latenza query semplice (p95) | <2s | <1s |
| Latenza task medio (p95) | <30s | <15s |
| Latenza task complesso (p95) | <5min | <2min |
| Throughput | 100 task/sec | 500 task/sec |
| Costo per task semplice | <$0.10 | <$0.01 |
| Recupero memoria (p95) | <100ms | <50ms |
| Tasso hit cache | >60% | >80% |

### Filosofia di Ottimizzazione

```
1. Misurare Prima
   - Profilare prima di ottimizzare
   - Identificare colli di bottiglia
   - Quantificare l'impatto

2. Ottimizzare la Cosa Giusta
   - Concentrarsi sul percorso critico
   - Principio di Pareto (80/20)
   - Non sovra-ottimizzare

3. I Trade-off sono Espliciti
   - Velocità vs. Qualità
   - Costo vs. Prestazioni
   - Latenza vs. Throughput

4. Mantenere Osservabilità
   - Le ottimizzazioni sono tracciabili
   - Regressioni prestazionali rilevate
   - Test A/B per validazione
```

---

## Principi di Prestazione

### Principio 1: Percorso Veloce per Casi Comuni

Ottimizzare per il caso dell'80%, non per il 20% dei casi limite.

```typescript
async function execute(task: Task): Promise<Result> {
  // Percorso veloce: Task semplice con hit cache
  if (isSimple(task)) {
    const cached = await cache.get(task.hash)
    if (cached) {
      return cached  // ~10ms
    }

    // Esecuzione semplice senza pianificazione completa
    return await simpleExecute(task)  // ~1s
  }

  // Percorso lento: Task complesso con ragionamento completo
  return await fullExecute(task)  // ~30s-5min
}
```

### Principio 2: Valutazione Lazy

Calcolare solo ciò che è necessario, quando è necessario.

```typescript
// Male: Valutazione eager
async function plan(goal: Goal): Promise<Plan> {
  const allPossibleSteps = await generateAllSteps(goal)  // Costoso
  const plan = selectBestSteps(allPossibleSteps)
  return plan
}

// Bene: Valutazione lazy
async function plan(goal: Goal): Promise<Plan> {
  const plan = new Plan()

  while (!plan.isComplete()) {
    const nextStep = await generateNextStep(plan, goal)  // Genera su richiesta
    plan.addStep(nextStep)
  }

  return plan
}
```

### Principio 3: Esecuzione Speculativa

Prevedere e precalcolare le necessità probabili.

```typescript
class SpeculativeExecutor {
  async execute(task: Task): Promise<Result> {
    // Avvia esecuzione principale
    const mainPromise = this.executeMain(task)

    // Avvia speculativamente passaggi successivi probabili
    const predictions = await this.predictNextSteps(task)
    const speculativePromises = predictions.map(p =>
      this.executeSpeculative(p)
    )

    // Attendi risultato principale
    const result = await mainPromise

    // Cancella lavoro speculativo non utilizzato
    await this.cancelUnused(speculativePromises, result)

    return result
  }
}
```

### Principio 4: Batching

Raggruppare operazioni simili per ammortizzare l'overhead.

```typescript
// Male: Chiamate individuali
for (const item of items) {
  await llm.embed(item)  // N chiamate separate
}

// Bene: Chiamata batch
const embeddings = await llm.embedBatch(items)  // 1 chiamata
```

---

## Livelli di Caching

### Cache Multi-Livello

```
┌────────────────────────────────────┐
│  L1: Cache In-Memory (accesso ms)  │
│  • Dati hot                        │
│  • Dimensione piccola (100MB-1GB)  │
│  • Eviction LRU                    │
└──────────────┬─────────────────────┘
               │ Miss
┌──────────────▼─────────────────────┐
│  L2: Cache Redis (accesso 1-10ms)  │
│  • Dati warm                       │
│  • Dimensione media (10-100GB)     │
│  • Scadenza TTL                    │
└──────────────┬─────────────────────┘
               │ Miss
┌──────────────▼─────────────────────┐
│  L3: Cache Storage (10-100ms)      │
│  • Dati cold                       │
│  • Dimensione grande (100GB-1TB)   │
│  • TTL lungo                       │
└────────────────────────────────────┘
```

### Strategia di Cache

```typescript
interface CacheStrategy {
  // Cosa cachare
  cacheable: (item: any) => boolean

  // Generazione chiave cache
  keyGenerator: (item: any) => string

  // Politica TTL
  ttl: (item: any) => number

  // Invalidazione
  invalidateOn: Event[]

  // Riscaldamento
  warmOnStartup: boolean
}

// Cache risposta LLM
const LLM_CACHE_STRATEGY: CacheStrategy = {
  cacheable: (call: LLMCall) =>
    call.temperature === 0 &&  // Deterministico
    call.prompt.length < 10000,  // Non troppo grande,

  keyGenerator: (call: LLMCall) =>
    hash(call.model, call.prompt, call.parameters),

  ttl: (call: LLMCall) =>
    3600000,  // 1 ora

  invalidateOn: ['model_update'],

  warmOnStartup: false
}

// Cache recupero memoria
const MEMORY_CACHE_STRATEGY: CacheStrategy = {
  cacheable: (query: MemoryCue) =>
    query.limit <= 10,  // Set di risultati piccoli

  keyGenerator: (query: MemoryCue) =>
    hash(query.query, query.filters, query.limit),

  ttl: (query: MemoryCue) =>
    60000,  // 1 minuto (la memoria cambia frequentemente)

  invalidateOn: ['memory_update', 'memory_delete'],

  warmOnStartup: true  // Riscalda con query comuni
}

// Cache risultati tool
const TOOL_CACHE_STRATEGY: CacheStrategy = {
  cacheable: (execution: ToolExecution) =>
    execution.tool.properties.idempotent &&
    execution.tool.properties.cacheable &&
    !execution.tool.properties.sideEffects,

  keyGenerator: (execution: ToolExecution) =>
    hash(execution.tool.id, execution.parameters),

  ttl: (execution: ToolExecution) =>
    execution.tool.properties.cacheTimeout || 300000,  // 5 min default

  invalidateOn: [],

  warmOnStartup: false
}
```

### Implementazione Cache

```typescript
class MultiLevelCache {
  private l1: Map<string, CacheEntry> = new Map()
  private l2: RedisClient
  private l3: StorageBackend

  async get<T>(key: string): Promise<T | null> {
    // Prova L1
    const l1Entry = this.l1.get(key)
    if (l1Entry && !l1Entry.isExpired()) {
      metrics.incrementCounter('cache_hit', 1, { level: 'l1' })
      return l1Entry.value as T
    }

    // Prova L2
    const l2Value = await this.l2.get(key)
    if (l2Value) {
      metrics.incrementCounter('cache_hit', 1, { level: 'l2' })
      // Promuovi a L1
      this.l1.set(key, { value: l2Value, expiresAt: Date.now() + 60000 })
      return l2Value as T
    }

    // Prova L3
    const l3Value = await this.l3.get(key)
    if (l3Value) {
      metrics.incrementCounter('cache_hit', 1, { level: 'l3' })
      // Promuovi a L2 e L1
      await this.l2.set(key, l3Value, 3600)
      this.l1.set(key, { value: l3Value, expiresAt: Date.now() + 60000 })
      return l3Value as T
    }

    // Cache miss
    metrics.incrementCounter('cache_miss', 1)
    return null
  }

  async set<T>(key: string, value: T, ttl: number): Promise<void> {
    // Scrivi su tutti i livelli
    this.l1.set(key, { value, expiresAt: Date.now() + Math.min(ttl, 60000) })
    await this.l2.set(key, value, Math.floor(ttl / 1000))
    await this.l3.set(key, value, ttl)
  }

  async invalidate(key: string): Promise<void> {
    this.l1.delete(key)
    await this.l2.del(key)
    await this.l3.delete(key)
  }
}
```

### Riscaldamento Cache

```typescript
async function warmCache(strategies: CacheStrategy[]): Promise<void> {
  for (const strategy of strategies) {
    if (!strategy.warmOnStartup) continue

    // Carica query/pattern comuni
    const warmingSet = await getWarmingSet(strategy)

    for (const item of warmingSet) {
      const key = strategy.keyGenerator(item)
      const value = await computeValue(item)
      const ttl = strategy.ttl(item)

      await cache.set(key, value, ttl)
    }
  }

  logger.info('cache_warmed', {
    strategies: strategies.length,
    itemsCached: warmingSet.length
  })
}
```

---

## Esecuzione Parallela

### Analisi Dipendenze

```typescript
interface DependencyGraph {
  nodes: Map<UUID, Node>
  edges: Map<UUID, Edge[]>

  // Trova nodi indipendenti (possono eseguire in parallelo)
  getIndependentNodes(): UUID[]

  // Ordinamento topologico (ordine di esecuzione)
  topologicalSort(): UUID[]

  // Percorso critico (catena di dipendenze più lunga)
  criticalPath(): UUID[]
}

// Costruisci grafo dipendenze dal piano
function buildDependencyGraph(plan: Plan): DependencyGraph {
  const graph = new DependencyGraph()

  // Aggiungi nodi (passi del piano)
  for (const step of plan.steps) {
    graph.addNode(step.id, step)
  }

  // Aggiungi archi (dipendenze)
  for (const dep of plan.dependencies) {
    graph.addEdge(dep.from, dep.to)
  }

  return graph
}
```

### Esecutore Parallelo

```typescript
class ParallelExecutor {
  async executePlan(plan: Plan): Promise<ExecutionResult> {
    const graph = buildDependencyGraph(plan)
    const completed = new Set<UUID>()
    const results = new Map<UUID, StepResult>()

    while (completed.size < plan.steps.length) {
      // Trova passi pronti per l'esecuzione (dipendenze soddisfatte)
      const ready = graph.nodes
        .filter(node =>
          !completed.has(node.id) &&
          node.dependencies.every(dep => completed.has(dep))
        )

      if (ready.length === 0) {
        throw new Error('Circular dependency or blocked')
      }

      // Esegui passi pronti in parallelo
      const executions = ready.map(node =>
        this.executeStep(node.step, results)
      )

      const stepResults = await Promise.allSettled(executions)

      // Elabora risultati
      for (let i = 0; i < stepResults.length; i++) {
        const result = stepResults[i]
        const node = ready[i]

        if (result.status === 'fulfilled') {
          results.set(node.id, result.value)
          completed.add(node.id)
        } else {
          // Gestisci fallimento
          const recovery = await this.handleStepFailure(
            node.step,
            result.reason
          )

          if (recovery.action === 'abort') {
            throw new Error(`Step ${node.id} failed: ${result.reason}`)
          }
        }
      }
    }

    return createExecutionResult(results)
  }
}
```

### Chiamate LLM Parallele

```typescript
class ParallelLLMExecutor {
  async callParallel(calls: LLMCall[]): Promise<LLMResponse[]> {
    // Raggruppa per modello (per batching)
    const byModel = groupBy(calls, c => c.model)

    // Esegui le chiamate di ogni modello in parallelo
    const results = await Promise.all(
      Object.entries(byModel).map(([model, modelCalls]) =>
        this.callModel(model, modelCalls)
      )
    )

    // Appiattisci risultati
    return results.flat()
  }

  async callModel(model: string, calls: LLMCall[]): Promise<LLMResponse[]> {
    // Controlla se il modello supporta batching
    if (this.supportsBatch(model)) {
      return await this.batchCall(model, calls)
    }

    // Altrimenti, chiamate individuali parallele (con limite di concorrenza)
    const limiter = new ConcurrencyLimiter(MAX_CONCURRENT_CALLS)

    return await limiter.map(calls, async call => {
      return await this.singleCall(model, call)
    })
  }
}
```

---

## Protocolli di Streaming

### Architettura Streaming

```typescript
interface StreamingExecutor {
  // Esegui con risposta streaming
  async stream(
    operation: Operation,
    onChunk: (chunk: Chunk) => void
  ): Promise<StreamResult>

  // Cancella stream
  cancel(streamId: UUID): void
}

interface Chunk {
  streamId: UUID
  sequence: number
  data: any
  isLast: boolean
  metadata?: Record<string, any>
}

interface StreamResult {
  streamId: UUID
  totalChunks: number
  duration: number
  cancelled: boolean
}
```

### Streaming LLM

```typescript
async function streamLLM(
  prompt: string,
  onToken: (token: string) => void
): Promise<string> {

  const stream = await llm.createStream(prompt)
  let fullResponse = ''

  for await (const token of stream) {
    fullResponse += token
    onToken(token)

    // Consenti terminazione anticipata
    if (shouldStop(fullResponse)) {
      stream.cancel()
      break
    }
  }

  return fullResponse
}

// Caso d'uso: Aggiornamenti UI progressivi
async function executeWithStreaming(task: Task) {
  const ui = getUserInterface()

  await streamLLM(
    task.prompt,
    (token) => ui.appendToken(token)  // Aggiorna UI in tempo reale
  )
}
```

### Benefici dello Streaming

1. **Latenza Percepita Inferiore**: Gli utenti vedono progressi immediatamente
2. **Terminazione Anticipata**: Fermarsi quando si raggiunge qualità sufficiente
3. **Raffinamento Progressivo**: Migliorare l'output incrementalmente
4. **Efficienza Memoria**: Elaborare chunk senza caricare tutto

---

## Routing dei Modelli

### Strategia di Routing

```typescript
interface ModelRouter {
  // Seleziona il modello migliore per il task
  selectModel(task: Task, constraints: Constraints): ModelChoice

  // Ottieni tutti i modelli disponibili
  getAvailableModels(): Model[]

  // Aggiorna statistiche prestazioni modello
  recordPerformance(model: string, metrics: Metrics): void
}

interface ModelChoice {
  model: string
  confidence: number
  reasoning: string
  estimatedCost: number
  estimatedLatency: number
}

class IntelligentModelRouter implements ModelRouter {
  selectModel(task: Task, constraints: Constraints): ModelChoice {
    const complexity = this.assessComplexity(task)
    const urgency = constraints.maxLatency
    const budget = constraints.maxCost

    // Task molto semplice → usa modello più piccolo/economico
    if (complexity < 0.3) {
      return {
        model: 'gpt-3.5-turbo',
        confidence: 0.9,
        reasoning: 'Simple task, fast model sufficient',
        estimatedCost: 0.001,
        estimatedLatency: 500
      }
    }

    // Task complesso + budget limitato → usa modello medio
    if (complexity > 0.7 && budget < 0.10) {
      return {
        model: 'claude-3.5-sonnet',
        confidence: 0.7,
        reasoning: 'Complex but budget-constrained',
        estimatedCost: 0.05,
        estimatedLatency: 2000
      }
    }

    // Task complesso + budget sufficiente → usa modello migliore
    if (complexity > 0.7 && budget >= 0.10) {
      return {
        model: 'gpt-4',
        confidence: 0.95,
        reasoning: 'Complex task, premium model justified',
        estimatedCost: 0.20,
        estimatedLatency: 3000
      }
    }

    // Default: modello bilanciato
    return {
      model: 'claude-3.5-sonnet',
      confidence: 0.8,
      reasoning: 'Balanced choice for typical task',
      estimatedCost: 0.03,
      estimatedLatency: 1500
    }
  }

  private assessComplexity(task: Task): number {
    const factors = {
      // Lunghezza descrizione task
      descriptionLength: task.description.length / 1000,

      // Numero di vincoli
      constraints: task.constraints.length / 10,

      // Difficoltà storica
      historicalDifficulty: this.getHistoricalDifficulty(task),

      // Capacità richieste
      capabilities: task.requiredCapabilities.length / 5
    }

    // Normalizza a 0-1
    return Math.min(
      1.0,
      (factors.descriptionLength +
       factors.constraints +
       factors.historicalDifficulty +
       factors.capabilities) / 4
    )
  }
}
```

### Cascata di Modelli

Usa modelli a cascata: prova prima l'economico, ripiegare sul costoso se necessario.

```typescript
async function cascadeModels(prompt: string): Promise<string> {
  const models = [
    { name: 'gpt-3.5-turbo', cost: 0.001, quality: 0.7 },
    { name: 'claude-3.5-sonnet', cost: 0.03, quality: 0.85 },
    { name: 'gpt-4', cost: 0.20, quality: 0.95 }
  ]

  for (const model of models) {
    const response = await llm.call(model.name, prompt)

    // Controlla se la qualità è sufficiente
    const quality = await assessQuality(response)

    if (quality >= THRESHOLD) {
      logger.info('model_cascade_success', {
        model: model.name,
        cost: model.cost,
        quality
      })
      return response
    }

    logger.warn('model_cascade_continue', {
      model: model.name,
      quality,
      threshold: THRESHOLD
    })
  }

  // Restituisci il miglior tentativo se tutti falliscono la soglia
  return responses[responses.length - 1]
}
```

---

## Ottimizzazione dei Prompt

### Riduzione Token

```typescript
class PromptOptimizer {
  async optimize(prompt: string): Promise<OptimizedPrompt> {
    return {
      // Rimuovi spazi bianchi non necessari
      trimmed: this.trim(prompt),

      // Comprimi esempi
      compressed: this.compressExamples(prompt),

      // Usa abbreviazioni
      abbreviated: this.useAbbreviations(prompt),

      // Rimuovi ridondanza
      deduplicated: this.removeDuplicates(prompt),

      // Prompt ottimizzato finale
      optimized: this.apply(prompt)
    }
  }

  private apply(prompt: string): string {
    let optimized = prompt

    // Taglia spazi bianchi
    optimized = optimized.trim().replace(/\s+/g, ' ')

    // Sostituisci pattern verbosi
    const replacements = [
      ['Please provide', 'Provide'],
      ['I would like you to', ''],
      ['Could you please', ''],
      ['Thank you', '']
    ]

    for (const [from, to] of replacements) {
      optimized = optimized.replace(new RegExp(from, 'g'), to)
    }

    // Comprimi esempi di codice (rimuovi commenti/righe vuote)
    optimized = this.compressCodeBlocks(optimized)

    return optimized
  }

  private compressCodeBlocks(prompt: string): string {
    return prompt.replace(/```(\w+)\n([\s\S]*?)```/g, (match, lang, code) => {
      // Rimuovi commenti
      const noComments = code.replace(/\/\/.*$/gm, '')
                            .replace(/\/\*[\s\S]*?\*\//g, '')

      // Rimuovi righe vuote
      const noBlankLines = noComments.replace(/^\s*\n/gm, '')

      // Minifica spaziatura
      const minified = noBlankLines.replace(/\s+/g, ' ')

      return '```' + lang + '\n' + minified + '\n```'
    })
  }
}
```

### Selezione Few-Shot

```typescript
class FewShotSelector {
  async selectExamples(
    task: Task,
    library: Example[],
    maxTokens: number
  ): Promise<Example[]> {

    // Embedding task
    const taskEmbedding = await embeddings.embed(task.description)

    // Punteggia tutti gli esempi per rilevanza
    const scored = await Promise.all(
      library.map(async ex => ({
        example: ex,
        similarity: cosineSimilarity(taskEmbedding, ex.embedding),
        tokens: await countTokens(ex.text)
      }))
    )

    // Ordina per similarità
    scored.sort((a, b) => b.similarity - a.similarity)

    // Seleziona esempi top entro budget token
    const selected: Example[] = []
    let totalTokens = 0

    for (const item of scored) {
      if (totalTokens + item.tokens <= maxTokens) {
        selected.push(item.example)
        totalTokens += item.tokens
      }
    }

    return selected
  }
}
```

### Caching Template Prompt

```typescript
class PromptTemplateCache {
  private templates: Map<string, CompiledTemplate> = new Map()

  compile(template: string): CompiledTemplate {
    const cached = this.templates.get(template)
    if (cached) return cached

    // Parse template (identifica variabili)
    const parsed = parseTemplate(template)

    // Compila a funzione
    const compiled: CompiledTemplate = (vars: Record<string, any>) => {
      return fillTemplate(parsed, vars)
    }

    this.templates.set(template, compiled)
    return compiled
  }

  render(templateKey: string, vars: Record<string, any>): string {
    const template = this.templates.get(templateKey)
    if (!template) {
      throw new Error(`Template ${templateKey} not found`)
    }

    return template(vars)
  }
}

// Utilizzo
const template = cache.compile(`
  Task: {{task}}
  Context: {{context}}
  Examples: {{examples}}
`)

const prompt = template({
  task: task.description,
  context: formatContext(context),
  examples: formatExamples(examples)
})
```

---

## Gestione delle Risorse

### Budget delle Risorse

```typescript
interface ResourceBudget {
  maxTokens: number
  maxLLMCalls: number
  maxWallTime: number
  maxCost: number
  maxDepth: number

  // Rimanenti
  tokensRemaining: number
  llmCallsRemaining: number
  wallTimeRemaining: number
  costRemaining: number
  depthRemaining: number
}

class ResourceManager {
  private budgets: Map<UUID, ResourceBudget> = new Map()

  allocate(requestId: UUID, budget: ResourceBudget): void {
    this.budgets.set(requestId, budget)
  }

  consume(
    requestId: UUID,
    usage: ResourceUsage
  ): void {
    const budget = this.budgets.get(requestId)
    if (!budget) throw new Error('No budget allocated')

    budget.tokensRemaining -= usage.tokens
    budget.llmCallsRemaining -= usage.llmCalls
    budget.costRemaining -= usage.cost
    budget.wallTimeRemaining -= usage.duration

    // Controlla se superato
    if (this.isExceeded(budget)) {
      throw new ResourceBudgetExceededError(budget)
    }
  }

  isExceeded(budget: ResourceBudget): boolean {
    return (
      budget.tokensRemaining <= 0 ||
      budget.llmCallsRemaining <= 0 ||
      budget.wallTimeRemaining <= 0 ||
      budget.costRemaining <= 0
    )
  }

  getRemaining(requestId: UUID): ResourceBudget {
    return this.budgets.get(requestId)!
  }
}
```

### Allocazione Dinamica del Budget

```typescript
function allocateBudget(
  goal: Goal,
  totalBudget: ResourceBudget
): Map<UUID, ResourceBudget> {

  // Scomponi obiettivo in subtask
  const subtasks = decomposeGoal(goal)

  // Stima costo per subtask
  const estimates = subtasks.map(st => ({
    subtask: st,
    estimatedCost: estimateCost(st)
  }))

  const totalEstimated = estimates.reduce((sum, e) => sum + e.estimatedCost, 0)

  // Alloca proporzionalmente (con buffer)
  const BUFFER = 0.2  // 20% buffer
  const allocations = new Map<UUID, ResourceBudget>()

  for (const { subtask, estimatedCost } of estimates) {
    const proportion = estimatedCost / totalEstimated

    allocations.set(subtask.id, {
      maxTokens: Math.floor(totalBudget.maxTokens * proportion * (1 - BUFFER)),
      maxLLMCalls: Math.floor(totalBudget.maxLLMCalls * proportion * (1 - BUFFER)),
      maxWallTime: Math.floor(totalBudget.maxWallTime * proportion * (1 - BUFFER)),
      maxCost: totalBudget.maxCost * proportion * (1 - BUFFER),
      maxDepth: totalBudget.maxDepth - 1,

      tokensRemaining: Math.floor(totalBudget.maxTokens * proportion * (1 - BUFFER)),
      llmCallsRemaining: Math.floor(totalBudget.maxLLMCalls * proportion * (1 - BUFFER)),
      wallTimeRemaining: Math.floor(totalBudget.maxWallTime * proportion * (1 - BUFFER)),
      costRemaining: totalBudget.maxCost * proportion * (1 - BUFFER),
      depthRemaining: totalBudget.maxDepth - 1
    })
  }

  return allocations
}
```

---

## Benchmark delle Prestazioni

### Suite di Benchmark

```typescript
interface Benchmark {
  name: string
  category: BenchmarkCategory
  task: Task
  expectedDuration: number
  expectedCost: number
  successCriteria: Criterion[]
}

enum BenchmarkCategory {
  SIMPLE = 'simple',
  MEDIUM = 'medium',
  COMPLEX = 'complex',
  EDGE_CASE = 'edge_case'
}

const BENCHMARK_SUITE: Benchmark[] = [
  // Benchmark semplici
  {
    name: 'simple_query',
    category: BenchmarkCategory.SIMPLE,
    task: { description: 'What is 2+2?' },
    expectedDuration: 1000,
    expectedCost: 0.001,
    successCriteria: [{ type: 'correctness', value: '4' }]
  },

  // Benchmark medi
  {
    name: 'code_generation',
    category: BenchmarkCategory.MEDIUM,
    task: { description: 'Write a Python function to sort a list' },
    expectedDuration: 10000,
    expectedCost: 0.05,
    successCriteria: [
      { type: 'syntax_valid', value: true },
      { type: 'tests_pass', value: true }
    ]
  },

  // Benchmark complessi
  {
    name: 'multi_step_analysis',
    category: BenchmarkCategory.COMPLEX,
    task: { description: 'Analyze codebase and suggest improvements' },
    expectedDuration: 120000,
    expectedCost: 0.50,
    successCriteria: [
      { type: 'suggestions_count', value: '>5' },
      { type: 'actionable', value: true }
    ]
  }
]
```

### Esecuzione Benchmark

```typescript
async function runBenchmarks(
  suite: Benchmark[],
  agent: Agent
): Promise<BenchmarkResults> {

  const results: BenchmarkResult[] = []

  for (const benchmark of suite) {
    logger.info('benchmark_start', { name: benchmark.name })

    const start = Date.now()

    try {
      // Esegui task
      const result = await agent.execute(benchmark.task)

      const duration = Date.now() - start

      // Valuta contro criteri
      const success = await evaluateCriteria(
        result,
        benchmark.successCriteria
      )

      results.push({
        benchmark: benchmark.name,
        success,
        duration,
        cost: result.resourcesUsed.cost,
        metrics: {
          expectedDuration: benchmark.expectedDuration,
          actualDuration: duration,
          expectedCost: benchmark.expectedCost,
          actualCost: result.resourcesUsed.cost,
          speedup: benchmark.expectedDuration / duration,
          costRatio: result.resourcesUsed.cost / benchmark.expectedCost
        }
      })

    } catch (error) {
      results.push({
        benchmark: benchmark.name,
        success: false,
        duration: Date.now() - start,
        cost: 0,
        error: error.message
      })
    }
  }

  return {
    results,
    summary: summarizeResults(results)
  }
}
```

---

## Pattern di Ottimizzazione

### Pattern 1: Memoizzazione

```typescript
function memoize<T extends (...args: any[]) => any>(
  fn: T,
  keyGenerator: (...args: Parameters<T>) => string
): T {
  const cache = new Map<string, ReturnType<T>>()

  return ((...args: Parameters<T>) => {
    const key = keyGenerator(...args)

    if (cache.has(key)) {
      metrics.incrementCounter('memoization_hit', 1)
      return cache.get(key)!
    }

    const result = fn(...args)
    cache.set(key, result)
    metrics.incrementCounter('memoization_miss', 1)

    return result
  }) as T
}

// Utilizzo
const decomposeGoal = memoize(
  (goal: Goal) => actuallyDecomposeGoal(goal),
  (goal: Goal) => goal.id
)
```

### Pattern 2: Request Coalescing

```typescript
class RequestCoalescer {
  private pending: Map<string, Promise<any>> = new Map()

  async coalesce<T>(
    key: string,
    operation: () => Promise<T>
  ): Promise<T> {

    // Se la richiesta è già in corso, attendi
    if (this.pending.has(key)) {
      metrics.incrementCounter('request_coalesced', 1)
      return await this.pending.get(key)!
    }

    // Avvia nuova richiesta
    const promise = operation()
    this.pending.set(key, promise)

    try {
      const result = await promise
      return result
    } finally {
      this.pending.delete(key)
    }
  }
}

// Utilizzo: Richieste simultanee multiple per stessa chiamata LLM
const coalescer = new RequestCoalescer()

const result = await coalescer.coalesce(
  promptHash,
  () => llm.call(prompt)
)
```

### Pattern 3: Prefetching

```typescript
class Prefetcher {
  async prefetch(current: State, predictions: Prediction[]): Promise<void> {
    // Predici operazioni successive probabili
    const predicted = predictions.map(p => ({
      operation: p.operation,
      probability: p.probability
    }))

    // Ordina per probabilità
    predicted.sort((a, b) => b.probability - a.probability)

    // Prefetch top-k
    const topK = predicted.slice(0, 3)

    await Promise.all(
      topK.map(async p => {
        try {
          // Esegui speculativamente in background
          const result = await p.operation()

          // Cache risultato
          await cache.set(p.operation.key, result, 60000)

          metrics.incrementCounter('prefetch_success', 1)
        } catch (error) {
          metrics.incrementCounter('prefetch_failure', 1)
        }
      })
    )
  }
}
```

### Pattern 4: Caricamento Lazy

```typescript
class LazyLoader<T> {
  private value: T | null = null
  private loading: Promise<T> | null = null

  constructor(
    private loader: () => Promise<T>
  ) {}

  async get(): Promise<T> {
    // Restituisci valore cached
    if (this.value !== null) {
      return this.value
    }

    // Restituisci richiesta in corso
    if (this.loading !== null) {
      return await this.loading
    }

    // Avvia caricamento
    this.loading = this.loader()

    try {
      this.value = await this.loading
      return this.value
    } finally {
      this.loading = null
    }
  }

  clear(): void {
    this.value = null
    this.loading = null
  }
}

// Utilizzo
const knowledgeGraph = new LazyLoader(() => loadKnowledgeGraph())

// Carica solo quando acceduto per la prima volta
const graph = await knowledgeGraph.get()
```

### Pattern 5: Connection Pooling

```typescript
class ConnectionPool<T> {
  private available: T[] = []
  private inUse: Set<T> = new Set()

  constructor(
    private factory: () => Promise<T>,
    private maxSize: number,
    private minSize: number
  ) {
    this.initialize()
  }

  private async initialize(): Promise<void> {
    for (let i = 0; i < this.minSize; i++) {
      const conn = await this.factory()
      this.available.push(conn)
    }
  }

  async acquire(): Promise<T> {
    // Usa connessione disponibile
    if (this.available.length > 0) {
      const conn = this.available.pop()!
      this.inUse.add(conn)
      return conn
    }

    // Crea nuova se sotto il limite
    if (this.inUse.size < this.maxSize) {
      const conn = await this.factory()
      this.inUse.add(conn)
      return conn
    }

    // Attendi che una connessione diventi disponibile
    return await this.waitForConnection()
  }

  release(conn: T): void {
    this.inUse.delete(conn)
    this.available.push(conn)
  }

  async destroy(): Promise<void> {
    // Chiudi tutte le connessioni
    for (const conn of [...this.available, ...this.inUse]) {
      await this.closeConnection(conn)
    }
  }
}

// Utilizzo con connessioni database
const dbPool = new ConnectionPool(
  () => createDatabaseConnection(),
  maxSize: 20,
  minSize: 5
)

const conn = await dbPool.acquire()
try {
  await conn.query('SELECT ...')
} finally {
  dbPool.release(conn)
}
```

---

## Monitoraggio delle Prestazioni

### Profiler di Prestazioni

```typescript
class PerformanceProfiler {
  async profile<T>(
    name: string,
    operation: () => Promise<T>
  ): Promise<ProfiledResult<T>> {

    const profile: PerformanceProfile = {
      name,
      startTime: Date.now(),
      startMemory: process.memoryUsage(),
      startCPU: process.cpuUsage()
    }

    try {
      const result = await operation()

      profile.endTime = Date.now()
      profile.endMemory = process.memoryUsage()
      profile.endCPU = process.cpuUsage()

      profile.duration = profile.endTime - profile.startTime
      profile.memoryDelta = profile.endMemory.heapUsed - profile.startMemory.heapUsed
      profile.cpuDelta = {
        user: profile.endCPU.user - profile.startCPU.user,
        system: profile.endCPU.system - profile.startCPU.system
      }

      // Registra metriche
      metrics.recordHistogram(`${name}_duration_ms`, profile.duration)
      metrics.recordHistogram(`${name}_memory_bytes`, profile.memoryDelta)

      return { result, profile }

    } catch (error) {
      profile.error = error
      throw error
    }
  }
}
```

### Rilevamento Regressioni Prestazioni

```typescript
class RegressionDetector {
  async checkRegression(
    benchmark: string,
    current: Metrics,
    baseline: Metrics
  ): Promise<RegressionReport> {

    const regressions: Regression[] = []

    // Controlla regressione latenza
    if (current.p95Latency > baseline.p95Latency * 1.2) {  // 20% più lento
      regressions.push({
        metric: 'p95_latency',
        baseline: baseline.p95Latency,
        current: current.p95Latency,
        degradation: (current.p95Latency - baseline.p95Latency) / baseline.p95Latency,
        severity: 'high'
      })
    }

    // Controlla regressione costo
    if (current.avgCost > baseline.avgCost * 1.5) {  // 50% più costoso
      regressions.push({
        metric: 'avg_cost',
        baseline: baseline.avgCost,
        current: current.avgCost,
        degradation: (current.avgCost - baseline.avgCost) / baseline.avgCost,
        severity: 'medium'
      })
    }

    // Controlla regressione tasso successo
    if (current.successRate < baseline.successRate * 0.95) {  // 5% peggiore
      regressions.push({
        metric: 'success_rate',
        baseline: baseline.successRate,
        current: current.successRate,
        degradation: (baseline.successRate - current.successRate) / baseline.successRate,
        severity: 'critical'
      })
    }

    return {
      benchmark,
      regressions,
      overallStatus: regressions.length === 0 ? 'pass' : 'fail'
    }
  }
}
```

---

## Conclusione

L'ottimizzazione delle prestazioni si ottiene attraverso una strategia completa:

1. **Caching Multi-Livello**: Memoria, Redis, storage (target tasso hit 60-80%)
2. **Esecuzione Parallela**: Operazioni indipendenti eseguite concorrentemente
3. **Streaming**: Output progressivo per reattività
4. **Routing Modelli**: Modello giusto per il task (trade-off costo-qualità)
5. **Ottimizzazione Prompt**: Minimizzare token mantenendo qualità
6. **Gestione Risorse**: Budget espliciti e allocazione
7. **Monitoraggio Continuo**: Rilevamento regressioni prestazioni

**Risultati Attesi**:
- 50-80% riduzione latenza (vs. implementazione naive)
- 60-80% riduzione costi (tramite caching e routing)
- 5-10x aumento throughput (tramite parallelizzazione)
- Qualità mantenuta (>90% tasso successo)

Tutte le ottimizzazioni sono tracciabili, testabili e mantenibili - coerenti con i nostri principi core.

---

## Riferimenti

1. Google SRE Book (2016). Best practice per ottimizzazione prestazioni.
2. High Performance Browser Networking (Grigorik, 2013). Ottimizzazione latenza.
3. Documentazione Redis (2024). Strategie di caching.
4. Documentazione OpenAI (2024). Caratteristiche prestazioni modelli.
5. AWS Well-Architected Framework. Pilastro efficienza prestazioni.
