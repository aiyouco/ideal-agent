# Specifica del Motore di Ragionamento

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice dei Contenuti

1. [Panoramica](#panoramica)
2. [Principi Architetturali](#principi-architetturali)
3. [Componenti Principali](#componenti-principali)
4. [Loop di Ragionamento](#loop-di-ragionamento)
5. [Gestione degli Obiettivi](#gestione-degli-obiettivi)
6. [Sistema di Pianificazione](#sistema-di-pianificazione)
7. [Metacognizione](#metacognizione)
8. [Auto-Riflessione](#auto-riflessione)
9. [Specifiche delle Interfacce](#specifiche-delle-interfacce)
10. [Strutture Dati](#strutture-dati)
11. [Algoritmi](#algoritmi)
12. [Caratteristiche di Performance](#caratteristiche-di-performance)

---

## Panoramica

Il Motore di Ragionamento è il nucleo cognitivo dell'agente ideale. Orchestra tutte le decisioni di alto livello, la pianificazione, la gestione degli obiettivi e le capacità di auto-miglioramento.

### Obiettivi di Design

1. **Sistematico**: Seguire cicli di ragionamento ben definiti, non elaborazione ad hoc
2. **Tracciabile**: Ogni decisione deve essere giustificata e registrata
3. **Adattivo**: Imparare dall'esperienza e regolare le strategie
4. **Efficiente**: Minimizzare calcoli non necessari mantenendo la qualità
5. **Limitato**: Limiti rigidi sulle risorse (tempo, token, costo)
6. **Testabile**: I componenti possono essere testati in isolamento

### Capacità Principali

- **Decomposizione degli Obiettivi**: Scomporre obiettivi complessi in sottoobiettivi gestibili
- **Generazione di Piani**: Creare piani strutturati per raggiungere obiettivi
- **Esecuzione dei Piani**: Eseguire piani con monitoraggio e adattamento
- **Gestione degli Impasse**: Riconoscere e risolvere blocchi nel ragionamento
- **Metacognizione**: Monitorare e controllare i propri processi di ragionamento
- **Auto-Riflessione**: Imparare dal successo e dal fallimento
- **Gestione delle Risorse**: Operare entro budget computazionali

---

## Principi Architetturali

### 1. Separazione delle Responsabilità

```
┌─────────────────────────────────────────┐
│  Architettura Motore di Ragionamento    │
│                                         │
│  ┌─────────────┐                       │
│  │  Controllo  │ ← Cosa fare dopo      │
│  └──────┬──────┘                       │
│         │                              │
│  ┌──────▼──────┐                       │
│  │ Pianificaz. │ ← Come farlo          │
│  └──────┬──────┘                       │
│         │                              │
│  ┌──────▼──────┐                       │
│  │  Esecuzione │ ← Farlo davvero       │
│  └──────┬──────┘                       │
│         │                              │
│  ┌──────▼──────┐                       │
│  │ Valutazione │ ← Ha funzionato?      │
│  └──────┬──────┘                       │
│         │                              │
│  ┌──────▼──────┐                       │
│  │ Riflessione │ ← Cosa abbiamo        │
│  └─────────────┘   imparato?          │
└─────────────────────────────────────────┘
```

### 2. Macchina a Stati Esplicita

Tutti gli stati di ragionamento sono espliciti e le transizioni sono ben definite:

```
Stati:
- IDLE: In attesa di nuovo obiettivo
- ANALYZING: Comprensione dell'obiettivo
- PLANNING: Creazione piano di esecuzione
- EXECUTING: Esecuzione passi del piano
- MONITORING: Controllo progresso esecuzione
- REPLANNING: Aggiustamento piano
- EVALUATING: Valutazione risultati
- REFLECTING: Apprendimento dall'episodio
- IMPASSE: Bloccato, creazione sottoobiettivo

Le transizioni sono deterministiche dato stato + input
```

### 3. Calcolo Limitato

Ogni operazione ha limiti espliciti di risorse:

```typescript
interface ResourceBudget {
  maxTokens: number        // Token totali per il ragionamento
  maxLLMCalls: number      // Invocazioni LLM totali
  maxWallTime: number      // Limite tempo reale (ms)
  maxCost: number          // Limite costo monetario
  maxDepth: number         // Profondità ricorsione/sottoobiettivi
}
```

### 4. Tracciabilità per Design

Tutte le decisioni producono tracce strutturate:

```typescript
interface ReasoningTrace {
  id: UUID
  timestamp: number
  state: ReasoningState
  input: any
  output: any
  reasoning: StructuredThought[]
  metadata: {
    tokensUsed: number
    latency: number
    confidence: number
  }
}
```

### 5. Testabilità Prioritaria

Ogni componente ha interfacce ben definite che possono essere simulate:

```typescript
// Testabile: Può iniettare LLM finto
interface ReasoningEngine {
  constructor(llm: LanguageModel, memory: MemorySystem)
}

// Testabile: Può verificare transizioni di stato
interface StateMachine {
  getState(): State
  transition(event: Event): State
}
```

---

## Componenti Principali

### Diagramma dei Componenti

```
┌──────────────────────────────────────────────────┐
│              Motore di Ragionamento              │
│                                                  │
│  ┌─────────────────────────────────────────┐   │
│  │        Gestore Obiettivi                │   │
│  │  • Stack obiettivi                      │   │
│  │  • Decomposizione obiettivi             │   │
│  │  • Rilevamento completamento obiettivi  │   │
│  └─────────────┬───────────────────────────┘   │
│                │                                │
│  ┌─────────────▼───────────────────────────┐   │
│  │        Motore di Pianificazione         │   │
│  │  • Decomposizione task                  │   │
│  │  • Analisi dipendenze                   │   │
│  │  • Generazione piano                    │   │
│  │  • Ottimizzazione piano                 │   │
│  └─────────────┬───────────────────────────┘   │
│                │                                │
│  ┌─────────────▼───────────────────────────┐   │
│  │       Motore di Esecuzione              │   │
│  │  • Esecuzione passi                     │   │
│  │  • Monitoraggio progresso               │   │
│  │  • Gestione errori                      │   │
│  │  • Capacità rollback                    │   │
│  └─────────────┬───────────────────────────┘   │
│                │                                │
│  ┌─────────────▼───────────────────────────┐   │
│  │      Monitor Metacognitivo              │   │
│  │  • Tracciamento progresso               │   │
│  │  • Stima confidenza                     │   │
│  │  • Selezione strategia                  │   │
│  │  • Rilevamento impasse                  │   │
│  └─────────────┬───────────────────────────┘   │
│                │                                │
│  ┌─────────────▼───────────────────────────┐   │
│  │      Motore di Riflessione              │   │
│  │  • Analisi episodio                     │   │
│  │  • Estrazione pattern                   │   │
│  │  • Aggiornamento conoscenza             │   │
│  │  • Adattamento strategia                │   │
│  └─────────────────────────────────────────┘   │
└──────────────────────────────────────────────────┘
```

### 1. Gestore Obiettivi

**Responsabilità**: Mantenere e decomporre obiettivi

```typescript
interface GoalManager {
  // Stack obiettivi (gerarchico)
  goalStack: Goal[]

  // Obiettivo attivo corrente
  getCurrentGoal(): Goal | null

  // Push nuovo obiettivo nello stack
  pushGoal(goal: Goal): void

  // Pop obiettivo completato
  popGoal(): Goal | null

  // Decomporre obiettivo complesso in sottoobiettivi
  async decompose(goal: Goal): Promise<Goal[]>

  // Verificare se obiettivo è raggiunto
  async isAchieved(goal: Goal, state: State): Promise<boolean>

  // Ottenere gerarchia obiettivi per contesto
  getGoalContext(): GoalContext
}

interface Goal {
  id: UUID
  description: string
  type: GoalType
  parent: UUID | null
  children: UUID[]
  status: GoalStatus
  constraints: Constraint[]
  successCriteria: Criterion[]
  createdAt: number
  completedAt: number | null
}

enum GoalType {
  ACHIEVEMENT = 'achievement',    // Fare qualcosa
  MAINTENANCE = 'maintenance',    // Mantenere qualcosa vero
  AVOIDANCE = 'avoidance',       // Non fare qualcosa
  EXPLORATION = 'exploration'     // Imparare qualcosa
}

enum GoalStatus {
  PENDING = 'pending',
  ACTIVE = 'active',
  BLOCKED = 'blocked',
  COMPLETED = 'completed',
  FAILED = 'failed',
  ABANDONED = 'abandoned'
}
```

#### Algoritmo di Decomposizione Obiettivi

```typescript
async function decomposeGoal(goal: Goal): Promise<Goal[]> {
  // 1. Analizzare complessità obiettivo
  const complexity = await analyzeComplexity(goal)

  // Se abbastanza semplice, nessuna decomposizione
  if (complexity < THRESHOLD) {
    return [goal]
  }

  // 2. Identificare strategia di decomposizione
  const strategy = selectDecompositionStrategy(goal)

  // 3. Generare sottoobiettivi basati su strategia
  const subgoals = await generateSubgoals(goal, strategy)

  // 4. Validare decomposizione
  await validateDecomposition(goal, subgoals)

  // 5. Restituire sottoobiettivi ordinati
  return orderSubgoals(subgoals)
}

enum DecompositionStrategy {
  SEQUENTIAL = 'sequential',      // A poi B poi C
  PARALLEL = 'parallel',          // A e B e C
  CONDITIONAL = 'conditional',    // Se X allora A altrimenti B
  ITERATIVE = 'iterative',        // Ripetere A fino a condizione
  HIERARCHICAL = 'hierarchical'   // Decomposizione nidificata
}
```

### 2. Motore di Pianificazione

**Responsabilità**: Creare piani di esecuzione strutturati

```typescript
interface PlanningEngine {
  // Generare piano per obiettivo
  async generatePlan(
    goal: Goal,
    context: Context
  ): Promise<Plan>

  // Ottimizzare piano (ridurre passi, parallelizzare, ecc.)
  async optimizePlan(plan: Plan): Promise<Plan>

  // Validare fattibilità piano
  async validatePlan(plan: Plan): Promise<ValidationResult>

  // Ripianificare durante esecuzione (adattivo)
  async replan(
    originalPlan: Plan,
    currentState: State,
    failure: Failure
  ): Promise<Plan>
}

interface Plan {
  id: UUID
  goalId: UUID
  steps: PlanStep[]
  dependencies: Dependency[]
  estimatedCost: ResourceEstimate
  contingencies: ContingencyPlan[]
  createdAt: number
}

interface PlanStep {
  id: UUID
  name: string
  type: StepType
  action: Action
  preconditions: Condition[]
  postconditions: Condition[]
  expectedDuration: number
  estimatedCost: ResourceEstimate
  dependencies: UUID[]  // ID dei passi che devono completarsi prima
  alternatives: PlanStep[]  // Passi di fallback
}

enum StepType {
  REASONING = 'reasoning',        // Pensare a qualcosa
  TOOL_USE = 'tool_use',          // Eseguire uno strumento
  SUBGOAL = 'subgoal',           // Perseguire sottoobiettivo
  DECISION = 'decision',          // Prendere una scelta
  VALIDATION = 'validation',      // Verificare assunzione
  WAIT = 'wait'                   // Attendere evento esterno
}

interface Action {
  type: string
  parameters: Record<string, any>
  timeout: number
  retryPolicy: RetryPolicy
}
```

#### Algoritmo di Pianificazione

```typescript
async function generatePlan(goal: Goal, context: Context): Promise<Plan> {
  // 1. Recuperare piani passati rilevanti
  const similarPlans = await memory.retrieveSimilarPlans(goal)

  // 2. Generare piano candidato usando LLM
  const prompt = constructPlanningPrompt(goal, context, similarPlans)
  const response = await llm.generate(prompt)
  const plan = parsePlan(response)

  // 3. Analizzare dipendenze
  const dependencyGraph = buildDependencyGraph(plan.steps)

  // 4. Validare coerenza logica
  await validatePlanLogic(plan, dependencyGraph)

  // 5. Stimare risorse
  plan.estimatedCost = await estimateResources(plan)

  // 6. Generare contingenze
  plan.contingencies = await generateContingencies(plan)

  // 7. Ottimizzare se necessario
  if (context.optimizationLevel === 'high') {
    plan = await optimizePlan(plan)
  }

  return plan
}

// Tecniche di ottimizzazione piano
async function optimizePlan(plan: Plan): Promise<Plan> {
  // Parallelizzare passi indipendenti
  plan = parallelizeIndependentSteps(plan)

  // Rimuovere passi ridondanti
  plan = removeRedundantSteps(plan)

  // Cachare risultati riutilizzabili
  plan = identifyCacheableSteps(plan)

  // Raggruppare operazioni simili
  plan = batchSimilarOperations(plan)

  return plan
}
```

### 3. Motore di Esecuzione

**Responsabilità**: Eseguire passi del piano e monitorare il progresso

```typescript
interface ExecutionEngine {
  // Eseguire piano completo
  async executePlan(
    plan: Plan,
    budget: ResourceBudget
  ): Promise<ExecutionResult>

  // Eseguire singolo passo
  async executeStep(
    step: PlanStep,
    context: ExecutionContext
  ): Promise<StepResult>

  // Monitorare progresso esecuzione
  getProgress(): ExecutionProgress

  // Gestire fallimento passo
  async handleFailure(
    step: PlanStep,
    error: Error
  ): Promise<FailureResolution>

  // Pausa/Riprendi esecuzione
  pause(): void
  resume(): void

  // Interrompere esecuzione
  abort(reason: string): void
}

interface ExecutionContext {
  plan: Plan
  currentStep: UUID
  state: State
  memory: MemorySystem
  tools: ToolRegistry
  budget: ResourceBudget
  trace: ExecutionTrace
}

interface ExecutionResult {
  success: boolean
  completedSteps: PlanStep[]
  failedSteps: PlanStep[]
  finalState: State
  resourcesUsed: ResourceUsage
  trace: ExecutionTrace
  outcome: Outcome
}

interface StepResult {
  success: boolean
  output: any
  stateChanges: StateChange[]
  resourcesUsed: ResourceUsage
  duration: number
  error: Error | null
}
```

#### Algoritmo di Esecuzione

```typescript
async function executePlan(
  plan: Plan,
  budget: ResourceBudget
): Promise<ExecutionResult> {

  const context: ExecutionContext = {
    plan,
    currentStep: plan.steps[0].id,
    state: getCurrentState(),
    memory: getMemorySystem(),
    tools: getToolRegistry(),
    budget,
    trace: createTrace()
  }

  const results: StepResult[] = []

  // Costruire ordine di esecuzione (ordinamento topologico delle dipendenze)
  const executionOrder = topologicalSort(plan.steps, plan.dependencies)

  for (const step of executionOrder) {
    // Verificare budget
    if (budgetExceeded(context.budget)) {
      return createPartialResult(results, 'budget_exceeded')
    }

    // Verificare precondizioni
    if (!await checkPreconditions(step, context.state)) {
      // Tentare di stabilire precondizioni
      const established = await establishPreconditions(step, context)
      if (!established) {
        return createPartialResult(results, 'precondition_failed')
      }
    }

    // Eseguire passo
    try {
      const result = await executeStep(step, context)
      results.push(result)

      // Aggiornare stato
      applyStateChanges(context.state, result.stateChanges)

      // Aggiornare budget
      updateBudget(context.budget, result.resourcesUsed)

      // Registrare traccia
      context.trace.addStep(step, result)

      // Verificare se continuare
      if (!result.success && !step.alternatives.length) {
        return createPartialResult(results, 'step_failed')
      }

    } catch (error) {
      // Gestire fallimento
      const resolution = await handleFailure(step, error)

      if (resolution.type === 'retry') {
        // Riprovare con parametri aggiustati
        continue
      } else if (resolution.type === 'skip') {
        // Saltare e continuare
        continue
      } else {
        // Interrompere
        return createPartialResult(results, 'unrecoverable_error')
      }
    }
  }

  return createSuccessResult(results, context)
}
```

### 4. Monitor Metacognitivo

**Responsabilità**: Monitorare e controllare il processo di ragionamento

```typescript
interface MetacognitiveMonitor {
  // Valutare progresso corrente
  async assessProgress(
    goal: Goal,
    plan: Plan,
    execution: ExecutionProgress
  ): Promise<ProgressAssessment>

  // Stimare confidenza nell'approccio corrente
  async estimateConfidence(
    context: Context
  ): Promise<number>  // 0-1

  // Rilevare se bloccato (impasse)
  detectImpasse(
    history: History
  ): ImpasseType | null

  // Selezionare strategia di ragionamento appropriata
  async selectStrategy(
    goal: Goal,
    context: Context
  ): Promise<Strategy>

  // Decidere se continuare, ripianificare o interrompere
  async makeControlDecision(
    assessment: ProgressAssessment
  ): Promise<ControlDecision>
}

interface ProgressAssessment {
  percentComplete: number
  stepsCompleted: number
  stepsRemaining: number
  successProbability: number
  estimatedTimeRemaining: number
  resourcesRemaining: ResourceBudget
  confidence: number
  issues: Issue[]
}

enum ImpasseType {
  NO_APPLICABLE_ACTION = 'no_applicable_action',
  TIE = 'tie',                            // Azioni multiple ugualmente buone
  CONFLICT = 'conflict',                  // Requisiti contraddittori
  RESOURCE_EXHAUSTED = 'resource_exhausted',
  INFINITE_LOOP = 'infinite_loop',
  EXTERNAL_DEPENDENCY = 'external_dependency'
}

enum Strategy {
  GREEDY = 'greedy',              // Veloce, prima azione ragionevole
  REACT = 'react',                // Loop pensiero-azione-osservazione
  TREE_SEARCH = 'tree_search',    // Esplorare percorsi multipli
  PLAN_EXECUTE = 'plan_execute',  // Pianificare in anticipo, poi eseguire
  ITERATIVE = 'iterative',        // Raffinare attraverso iterazioni
  COLLABORATIVE = 'collaborative'  // Approccio multi-agente
}

interface ControlDecision {
  action: ControlAction
  reason: string
  confidence: number
}

enum ControlAction {
  CONTINUE = 'continue',
  REPLAN = 'replan',
  ESCALATE = 'escalate',      // Creare sottoobiettivo
  ABORT = 'abort',
  SWITCH_STRATEGY = 'switch_strategy',
  REQUEST_HELP = 'request_help'
}
```

#### Algoritmo di Rilevamento Impasse

```typescript
function detectImpasse(history: History): ImpasseType | null {
  // Verificare loop infiniti
  if (detectLoopInHistory(history, LOOP_THRESHOLD)) {
    return ImpasseType.INFINITE_LOOP
  }

  // Verificare se nessuna azione è applicabile
  if (history.last().applicableActions.length === 0) {
    return ImpasseType.NO_APPLICABLE_ACTION
  }

  // Verificare se bloccato su azioni paritarie
  if (detectTie(history, TIE_THRESHOLD)) {
    return ImpasseType.TIE
  }

  // Verificare requisiti contraddittori
  if (detectConflict(history.last().state)) {
    return ImpasseType.CONFLICT
  }

  // Verificare esaurimento risorse
  if (history.last().budget.isExhausted()) {
    return ImpasseType.RESOURCE_EXHAUSTED
  }

  return null
}

// Rilevamento loop: Abbiamo visto questo stato prima?
function detectLoopInHistory(
  history: History,
  threshold: number
): boolean {
  const recentStates = history.getRecentStates(threshold)
  const currentState = history.last().state

  return recentStates.some(state =>
    statesSimilar(state, currentState, SIMILARITY_THRESHOLD)
  )
}
```

#### Algoritmo di Selezione Strategia

```typescript
async function selectStrategy(
  goal: Goal,
  context: Context
): Promise<Strategy> {

  // Fattori da considerare
  const factors = {
    goalComplexity: analyzeComplexity(goal),
    timeConstraint: context.budget.maxWallTime,
    costConstraint: context.budget.maxCost,
    confidenceRequired: context.confidenceThreshold,
    explorationValue: assessExplorationValue(goal),
    pastPerformance: await memory.getStrategyPerformance(goal)
  }

  // Logica decisionale
  if (factors.timeConstraint < 1000) {
    // Tempo molto stretto: usare strategia più veloce
    return Strategy.GREEDY
  }

  if (factors.goalComplexity > 0.8 && factors.costConstraint > 1000) {
    // Complesso e possiamo permetterci la ricerca
    return Strategy.TREE_SEARCH
  }

  if (factors.confidenceRequired > 0.9) {
    // Serve alta confidenza: pianificare con attenzione
    return Strategy.PLAN_EXECUTE
  }

  // Verificare prestazioni passate
  const bestStrategy = findBestStrategy(factors.pastPerformance)
  if (bestStrategy && bestStrategy.successRate > 0.7) {
    return bestStrategy.strategy
  }

  // Default: approccio reattivo
  return Strategy.REACT
}
```

### 5. Motore di Riflessione

**Responsabilità**: Imparare dall'esperienza

```typescript
interface ReflectionEngine {
  // Riflettere su episodio completato
  async reflect(
    episode: Episode
  ): Promise<Reflection>

  // Estrarre pattern da episodi multipli
  async extractPatterns(
    episodes: Episode[]
  ): Promise<Pattern[]>

  // Aggiornare preferenze strategia
  async updateStrategies(
    reflection: Reflection
  ): Promise<void>

  // Consolidare memorie episodiche
  async consolidateMemories(): Promise<void>
}

interface Episode {
  id: UUID
  goal: Goal
  plan: Plan
  execution: ExecutionResult
  outcome: Outcome
  duration: number
  resourcesUsed: ResourceUsage
  trace: ExecutionTrace
  timestamp: number
}

interface Reflection {
  episodeId: UUID
  successes: Success[]
  failures: Failure[]
  insights: Insight[]
  recommendations: Recommendation[]
  patternsObserved: Pattern[]
  confidence: number
}

interface Insight {
  type: InsightType
  description: string
  evidence: Evidence[]
  generalizability: number  // 0-1, quanto ampiamente applicabile
  confidence: number
}

enum InsightType {
  STRATEGY_EFFECTIVE = 'strategy_effective',
  STRATEGY_INEFFECTIVE = 'strategy_ineffective',
  PATTERN_DISCOVERED = 'pattern_discovered',
  ASSUMPTION_VIOLATED = 'assumption_violated',
  TOOL_LIMITATION = 'tool_limitation',
  OPTIMIZATION_OPPORTUNITY = 'optimization_opportunity'
}
```

#### Algoritmo di Riflessione

```typescript
async function reflect(episode: Episode): Promise<Reflection> {

  // 1. Classificare risultato
  const outcome = classifyOutcome(episode)

  // 2. Analizzare cosa ha funzionato
  const successes = await analyzeSuccesses(episode)

  // 3. Analizzare cosa non ha funzionato
  const failures = await analyzeFailures(episode)

  // 4. Generare insight usando LLM
  const prompt = constructReflectionPrompt(episode, successes, failures)
  const response = await llm.generate(prompt)
  const insights = parseInsights(response)

  // 5. Estrarre pattern
  const patterns = await extractPatterns([episode])

  // 6. Generare raccomandazioni
  const recommendations = generateRecommendations(insights, patterns)

  // 7. Memorizzare riflessione per uso futuro
  await memory.storeReflection({
    episodeId: episode.id,
    successes,
    failures,
    insights,
    recommendations,
    patternsObserved: patterns,
    confidence: computeConfidence(insights)
  })

  // 8. Aggiornare utilità strategie
  await updateStrategyUtilities(episode, insights)

  return {
    episodeId: episode.id,
    successes,
    failures,
    insights,
    recommendations,
    patternsObserved: patterns,
    confidence: computeConfidence(insights)
  }
}

// Analizzare cosa ha funzionato bene
async function analyzeSuccesses(episode: Episode): Promise<Success[]> {
  const successes: Success[] = []

  // Identificare passi efficienti
  for (const step of episode.execution.completedSteps) {
    if (step.efficiency > 0.8) {  // Molto efficiente
      successes.push({
        type: 'efficient_step',
        description: `Il passo ${step.name} è stato molto efficiente`,
        metrics: { efficiency: step.efficiency }
      })
    }
  }

  // Identificare strategie efficaci
  if (episode.outcome.success) {
    successes.push({
      type: 'effective_strategy',
      description: `La strategia ${episode.plan.strategy} ha avuto successo`,
      metrics: {
        successRate: 1.0,
        duration: episode.duration,
        cost: episode.resourcesUsed.cost
      }
    })
  }

  return successes
}

// Analizzare cosa è andato storto
async function analyzeFailures(episode: Episode): Promise<Failure[]> {
  const failures: Failure[] = []

  // Identificare passi falliti
  for (const step of episode.execution.failedSteps) {
    failures.push({
      type: 'step_failure',
      description: step.error.message,
      causes: await diagnoseCauses(step),
      impact: assessImpact(step, episode)
    })
  }

  // Identificare risorse sprecate
  const wastedEffort = identifyWastedEffort(episode)
  if (wastedEffort.amount > THRESHOLD) {
    failures.push({
      type: 'wasted_resources',
      description: wastedEffort.description,
      causes: wastedEffort.causes,
      impact: wastedEffort.amount
    })
  }

  return failures
}
```

---

## Loop di Ragionamento

### Loop di Ragionamento Principale

Il loop di ragionamento principale integra tutti i componenti:

```typescript
async function reasoningLoop(
  initialGoal: Goal,
  budget: ResourceBudget
): Promise<Episode> {

  // Inizializzare
  const goalManager = new GoalManager()
  const planner = new PlanningEngine()
  const executor = new ExecutionEngine()
  const monitor = new MetacognitiveMonitor()
  const reflector = new ReflectionEngine()

  goalManager.pushGoal(initialGoal)

  const trace = createExecutionTrace()

  // Loop principale
  while (goalManager.hasActiveGoals() && !budget.isExhausted()) {

    const currentGoal = goalManager.getCurrentGoal()

    // 1. Verificare se obiettivo è già raggiunto
    if (await goalManager.isAchieved(currentGoal, getCurrentState())) {
      goalManager.popGoal()
      continue
    }

    // 2. Valutare progresso
    const progress = await monitor.assessProgress(
      currentGoal,
      getCurrentPlan(),
      getExecutionProgress()
    )

    // 3. Verificare impasse
    const impasse = monitor.detectImpasse(trace.history)
    if (impasse) {
      await handleImpasse(impasse, goalManager, planner)
      continue
    }

    // 4. Prendere decisione di controllo
    const decision = await monitor.makeControlDecision(progress)

    switch (decision.action) {
      case ControlAction.CONTINUE:
        // Continuare esecuzione piano corrente
        break

      case ControlAction.REPLAN:
        // Generare nuovo piano
        const newPlan = await planner.replan(
          getCurrentPlan(),
          getCurrentState(),
          getLastFailure()
        )
        setCurrentPlan(newPlan)
        break

      case ControlAction.ESCALATE:
        // Creare sottoobiettivo
        const subgoals = await goalManager.decompose(currentGoal)
        subgoals.forEach(sg => goalManager.pushGoal(sg))
        break

      case ControlAction.ABORT:
        // Rinunciare all'obiettivo corrente
        goalManager.popGoal()
        break

      case ControlAction.SWITCH_STRATEGY:
        // Cambiare strategia di ragionamento
        const newStrategy = await monitor.selectStrategy(
          currentGoal,
          getContext()
        )
        await applyStrategy(newStrategy)
        break
    }

    // 5. Eseguire passo/i successivo/i
    if (decision.action === ControlAction.CONTINUE) {
      const plan = getCurrentPlan()
      const result = await executor.executeStep(
        plan.getNextStep(),
        getExecutionContext()
      )

      trace.addStepResult(result)
      updateBudget(budget, result.resourcesUsed)
    }
  }

  // Creare episodio
  const episode = createEpisode(initialGoal, trace, budget)

  // Riflettere
  const reflection = await reflector.reflect(episode)
  episode.reflection = reflection

  // Memorizzare episodio
  await memory.storeEpisode(episode)

  return episode
}
```

### Gestione Impasse

```typescript
async function handleImpasse(
  impasse: ImpasseType,
  goalManager: GoalManager,
  planner: PlanningEngine
): Promise<void> {

  switch (impasse) {
    case ImpasseType.NO_APPLICABLE_ACTION:
      // Creare sottoobiettivo per trovare azione applicabile
      const discoverGoal = new Goal({
        description: "Scoprire azioni applicabili per stato corrente",
        type: GoalType.EXPLORATION
      })
      goalManager.pushGoal(discoverGoal)
      break

    case ImpasseType.TIE:
      // Creare sottoobiettivo per rompere parità
      const disambiguateGoal = new Goal({
        description: "Disambiguare tra azioni paritarie",
        type: GoalType.ACHIEVEMENT
      })
      goalManager.pushGoal(disambiguateGoal)
      break

    case ImpasseType.CONFLICT:
      // Creare sottoobiettivo per risolvere conflitto
      const resolveGoal = new Goal({
        description: "Risolvere requisiti in conflitto",
        type: GoalType.ACHIEVEMENT
      })
      goalManager.pushGoal(resolveGoal)
      break

    case ImpasseType.INFINITE_LOOP:
      // Uscire dal loop provando alternativa
      const currentPlan = getCurrentPlan()
      const alternativePlan = await planner.generateAlternativePlan(
        currentPlan,
        "avoid_loop"
      )
      setCurrentPlan(alternativePlan)
      break

    case ImpasseType.RESOURCE_EXHAUSTED:
      // Non può continuare, deve interrompere o richiedere più risorse
      const decision = await decideOnResourceExhaustion()
      if (decision === 'abort') {
        goalManager.popGoal()
      } else {
        await requestAdditionalResources()
      }
      break
  }
}
```

---

## Gestione degli Obiettivi

### Gerarchia degli Obiettivi

Gli obiettivi sono organizzati gerarchicamente:

```
Obiettivo di Alto Livello: Costruire app todo
  ├─ [ ] Progettare modello dati
  │   ├─ [x] Definire schema task
  │   └─ [ ] Definire schema utente
  ├─ [ ] Implementare backend
  │   ├─ [ ] Configurare database
  │   ├─ [ ] Creare endpoint API
  │   └─ [ ] Scrivere test
  └─ [ ] Implementare frontend
      ├─ [ ] Creare componenti
      └─ [ ] Stilizzare interfaccia
```

### Rilevamento Raggiungimento Obiettivi

```typescript
async function isAchieved(goal: Goal, state: State): Promise<boolean> {
  // Verificare tutti i criteri di successo
  for (const criterion of goal.successCriteria) {
    const satisfied = await evaluateCriterion(criterion, state)
    if (!satisfied) {
      return false
    }
  }

  // Tutti i criteri soddisfatti
  return true
}

interface Criterion {
  type: CriterionType
  description: string
  evaluator: (state: State) => Promise<boolean>
}

enum CriterionType {
  STATE_PROPERTY = 'state_property',      // Lo stato ha una certa proprietà
  OBSERVABLE = 'observable',              // Può osservare risultato
  USER_CONFIRMATION = 'user_confirmation', // L'utente conferma
  TEST_PASSES = 'test_passes',           // I test passano
  INVARIANT = 'invariant'                // L'invariante è valido
}
```

---

## Sistema di Pianificazione

### Pianificazione Hierarchical Task Network (HTN)

Il sistema di pianificazione usa decomposizione HTN:

```typescript
interface Task {
  name: string
  type: 'primitive' | 'compound'
  parameters: Parameter[]
}

interface Method {
  name: string
  task: Task
  preconditions: Condition[]
  subtasks: Task[]
  ordering: Ordering
}

// Algoritmo di pianificazione HTN
async function htnPlanning(
  task: Task,
  state: State,
  methods: Method[]
): Promise<Plan> {

  if (task.type === 'primitive') {
    // Caso base: task primitivo diventa azione
    return new Plan([taskToAction(task)])
  }

  // Trovare metodi applicabili
  const applicable = methods.filter(m =>
    m.task.name === task.name &&
    m.preconditions.every(c => c.evaluate(state))
  )

  if (applicable.length === 0) {
    throw new Error(`Nessun metodo applicabile per task ${task.name}`)
  }

  // Selezionare miglior metodo
  const method = await selectBestMethod(applicable, state)

  // Pianificare ricorsivamente per subtask
  const subplans = await Promise.all(
    method.subtasks.map(st => htnPlanning(st, state, methods))
  )

  // Combinare subplan secondo ordinamento
  return combinePlans(subplans, method.ordering)
}
```

---

## Metacognizione

### Stima della Confidenza

```typescript
async function estimateConfidence(context: Context): Promise<number> {
  const factors = {
    // Quanto bene comprendiamo l'obiettivo?
    goalClarity: assessGoalClarity(context.goal),

    // Quanto è completo il nostro piano?
    planCompleteness: assessPlanCompleteness(context.plan),

    // Quanta esperienza rilevante abbiamo?
    experienceLevel: await assessExperience(context.goal),

    // Stiamo facendo progressi?
    progressRate: assessProgressRate(context.history),

    // Quanto sono affidabili i nostri strumenti?
    toolReliability: assessToolReliability(context.tools),

    // Abbiamo abbastanza risorse?
    resourceSufficiency: assessResources(context.budget)
  }

  // Combinazione pesata
  const confidence =
    0.2 * factors.goalClarity +
    0.2 * factors.planCompleteness +
    0.2 * factors.experienceLevel +
    0.2 * factors.progressRate +
    0.1 * factors.toolReliability +
    0.1 * factors.resourceSufficiency

  return confidence
}
```

---

## Auto-Riflessione

### Template Prompt di Riflessione

```typescript
const REFLECTION_PROMPT = `
Stai riflettendo sulle tue prestazioni nel completare un compito.

OBIETTIVO: {goal.description}

PIANO:
{plan.summary}

ESECUZIONE:
- Passi completati: {execution.completedSteps.length}
- Passi falliti: {execution.failedSteps.length}
- Durata: {execution.duration}ms
- Costo: {execution.cost} token

RISULTATO: {outcome.success ? 'SUCCESSO' : 'FALLIMENTO'}

Rifletti su:
1. Quali strategie e approcci hanno funzionato bene? Perché?
2. Quali strategie e approcci non hanno funzionato? Perché no?
3. Quali pattern o insight puoi estrarre?
4. Cosa faresti diversamente la prossima volta?
5. Cosa può essere generalizzato ad altri compiti?

Fornisci riflessione strutturata in formato JSON:
{
  "successi": [...],
  "fallimenti": [...],
  "insight": [...],
  "raccomandazioni": [...]
}
`
```

---

## Specifiche delle Interfacce

### Interfacce Principali

```typescript
// Interfaccia principale
interface ReasoningEngine {
  // Eseguire obiettivo con budget
  async execute(
    goal: Goal,
    budget: ResourceBudget,
    context?: Context
  ): Promise<Episode>

  // Ottenere stato corrente
  getState(): ReasoningState

  // Iscriversi agli eventi
  on(event: string, handler: EventHandler): void

  // Pausa/riprendi
  pause(): void
  resume(): void
}

// Configurazione
interface ReasoningEngineConfig {
  llm: LanguageModel
  memory: MemorySystem
  tools: ToolRegistry
  strategies: StrategyConfig
  limits: ResourceLimits
  observability: ObservabilityConfig
}

// Contesto
interface Context {
  goal: Goal
  state: State
  plan: Plan | null
  history: History
  budget: ResourceBudget
  memory: MemorySystem
  tools: ToolRegistry
}
```

---

## Strutture Dati

Tutte le strutture dati usano formati serializzabili JSON per la tracciabilità:

```typescript
// Esempio: Serializzazione Goal
{
  "id": "uuid-1234",
  "description": "Creare un web server",
  "type": "achievement",
  "status": "active",
  "createdAt": 1699564800000,
  "successCriteria": [
    {
      "type": "test_passes",
      "description": "Il server risponde alle richieste HTTP"
    }
  ],
  "constraints": [
    {
      "type": "resource",
      "maxCost": 1000
    }
  ]
}
```

---

## Algoritmi

### Riepilogo Algoritmi Principali

1. **Decomposizione Obiettivi**: Decomposizione ricorsiva basata su HTN
2. **Pianificazione**: Ricerca in avanti con euristiche
3. **Esecuzione**: Ordinamento topologico del grafo delle dipendenze
4. **Rilevamento Impasse**: Pattern matching sulla storia
5. **Selezione Strategia**: Multi-armed bandit con contesto
6. **Riflessione**: Analisi basata su LLM con output strutturato

---

## Caratteristiche di Performance

### Complessità Temporale

| Operazione | Caso Migliore | Caso Medio | Caso Peggiore |
|-----------|-----------|--------------|------------|
| Decomposizione Goal | O(1) | O(d) | O(d×b^d) |
| Generazione Piano | O(n) | O(n log n) | O(n²) |
| Esecuzione Passo | O(1) | O(1) | O(t) timeout |
| Rilevamento Impasse | O(1) | O(h) | O(h²) |
| Riflessione | O(1) | O(n) | O(n²) |

Dove:
- d = profondità decomposizione
- b = fattore di ramificazione
- n = numero di passi del piano
- h = lunghezza storia
- t = durata timeout

### Complessità Spaziale

| Componente | Uso Spazio |
|-----------|-------------|
| Stack Obiettivi | O(d) |
| Piano | O(n) |
| Traccia Esecuzione | O(n×m) |
| Memoria di Lavoro | O(k) limitata da finestra di contesto |

### Budget delle Risorse

Valori predefiniti raccomandati:

```typescript
const DEFAULT_BUDGET: ResourceBudget = {
  maxTokens: 100000,      // 100K token
  maxLLMCalls: 100,       // 100 chiamate LLM
  maxWallTime: 300000,    // 5 minuti
  maxCost: 10.00,         // $10
  maxDepth: 5             // 5 livelli di ricorsione
}
```

---

## Conclusione

Il Motore di Ragionamento fornisce una base completa, tracciabile e testabile per la cognizione degli agenti. Combina insight dalle architetture cognitive (SOAR, ACT-R) con la potenza degli LLM per creare un sistema che può:

- Decomporre e raggiungere sistematicamente obiettivi complessi
- Pianificare ed eseguire con monitoraggio e adattamento
- Rilevare e risolvere impasse attraverso subgoaling
- Imparare dall'esperienza attraverso riflessione
- Operare entro risorse computazionali limitate

Tutti i componenti hanno interfacce ben definite e possono essere testati indipendentemente, garantendo che il sistema soddisfi i requisiti di performance, tracciabilità e testabilità.

---

## Riferimenti

1. Laird, J. E. (2012). The Soar Cognitive Architecture. MIT Press.
2. Ghallab, M., Nau, D., & Traverso, P. (2016). Automated Planning and Acting. Cambridge University Press.
3. Russell, S., & Norvig, P. (2020). Artificial Intelligence: A Modern Approach (4th ed.). Pearson.
4. Yao, S., et al. (2022). ReAct: Synergizing Reasoning and Acting in Language Models. ICLR 2023.
5. Shinn, N., et al. (2023). Reflexion: Language Agents with Verbal Reinforcement Learning. NeurIPS 2023.
