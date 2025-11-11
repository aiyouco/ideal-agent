# Framework Testing e Benchmarking

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Filosofia di Testing](#filosofia-di-testing)
3. [Piramide dei Test](#piramide-dei-test)
4. [Unit Testing](#unit-testing)
5. [Integration Testing](#integration-testing)
6. [End-to-End Testing](#end-to-end-testing)
7. [Performance Benchmarking](#performance-benchmarking)
8. [Metriche di Valutazione](#metriche-di-valutazione)
9. [Infrastruttura di Test](#infrastruttura-di-test)
10. [Testing Continuo](#testing-continuo)

---

## Panoramica

Il Framework Testing e Benchmarking garantisce che l'architettura dell'agente sia validata a tutti i livelli - dai singoli componenti all'esecuzione completa dei task - con misurazione sistematica delle prestazioni.

### Obiettivi di Design

1. **Copertura Completa**: Testare tutti i componenti e punti di integrazione
2. **Feedback Veloce**: Test unitari in secondi, suite completa in minuti
3. **Deterministici**: I test producono risultati consistenti
4. **Realistici**: Test di integrazione usano scenari simili alla produzione
5. **Misurabili**: Metriche quantitative per qualità e prestazioni
6. **Automatizzati**: Integrazione pipeline CI/CD
7. **Mantenibili**: I test sono asset di codice, non debito tecnico

### Ambito del Testing

```
Test Unitari (70% dei test)
├─ Comportamento componenti
├─ Casi limite
├─ Condizioni di errore
└─ Caratteristiche prestazioni

Test di Integrazione (20% dei test)
├─ Interazioni tra componenti
├─ Flusso dati
├─ Gestione stato
└─ Dipendenze esterne

Test End-to-End (10% dei test)
├─ Workflow task completi
├─ Scenari utente
├─ Ragionamento multi-step
└─ Ambiente reale
```

---

## Filosofia di Testing

### Qualità dei Test

**Principi FIRST**:
- **Fast**: Test unitari <100ms, integrazione <5s, E2E <60s
- **Independent**: Nessuno stato condiviso tra test
- **Repeatable**: Stesso input → Stesso output
- **Self-validating**: Chiaro pass/fail (nessuna ispezione manuale)
- **Timely**: Scritto prima o con il codice (TDD/BDD)

### Struttura Test

```typescript
describe('Component/Feature', () => {
  // Setup
  beforeEach(() => {
    // Arrange: Crea fixture di test
  })

  // Cleanup
  afterEach(() => {
    // Pulisci risorse
  })

  it('should behave correctly in normal case', () => {
    // Arrange: Configura dati test
    // Act: Esegui operazione
    // Assert: Verifica risultati
  })

  it('should handle edge case', () => {
    // ...
  })

  it('should fail gracefully on error', () => {
    // ...
  })
})
```

### Gestione Dati Test

```typescript
interface TestFixture {
  // Dati test predefiniti
  goals: Goal[]
  plans: Plan[]
  episodes: Episode[]
  memories: Memory[]

  // Factory
  createGoal(overrides?: Partial<Goal>): Goal
  createPlan(overrides?: Partial<Plan>): Plan
  createEpisode(overrides?: Partial<Episode>): Episode
}

class TestFixtureFactory {
  createBasicGoal(): Goal {
    return {
      id: 'test-goal-123',
      description: 'Test goal',
      type: GoalType.ACHIEVEMENT,
      status: GoalStatus.PENDING,
      successCriteria: [],
      constraints: [],
      createdAt: Date.now(),
      completedAt: null
    }
  }

  createComplexGoal(): Goal {
    return {
      id: 'test-goal-456',
      description: 'Complex multi-step goal',
      type: GoalType.ACHIEVEMENT,
      status: GoalStatus.PENDING,
      successCriteria: [
        { type: 'observable', description: 'Observable outcome' }
      ],
      constraints: [
        { type: 'resource', maxCost: 1.0 }
      ],
      createdAt: Date.now(),
      completedAt: null
    }
  }
}
```

---

## Piramide dei Test

### Distribuzione dei Test

```
     ▲
    ╱ ╲
   ╱E2E╲         10% (100 test)
  ╱─────╲        - Workflow completi
 ╱   I   ╲       20% (200 test)
╱─────────╲      - Integrazione componenti
╲   Unit  ╱      70% (700 test)
 ╲───────╱       - Logica componenti
  ───────

Totale: ~1000 test
Unit: ~10 min
Integrazione: ~20 min
E2E: ~30 min
Suite Completa: ~60 min
```

### Perché Questa Distribuzione?

1. **Test unitari** sono:
   - Più veloci da eseguire
   - Più facili da debuggare
   - Più specifici (individuano fallimenti)
   - Meno costosi da mantenere

2. **Test di integrazione** sono:
   - Più lenti ma validano interazioni
   - Catturano disallineamenti interfacce
   - Testano flussi dati realistici

3. **Test E2E** sono:
   - Più lenti ma validano sistema completo
   - Catturano comportamenti emergenti
   - Validano scenari utente

---

## Unit Testing

### Strategia Test Componenti

```typescript
// Esempio: Test Unitari Goal Manager
describe('GoalManager', () => {
  let goalManager: GoalManager
  let mockMemory: MockMemorySystem

  beforeEach(() => {
    mockMemory = new MockMemorySystem()
    goalManager = new GoalManager(mockMemory)
  })

  describe('decompose', () => {
    it('should decompose complex goal into subgoals', async () => {
      // Arrange
      const complexGoal = fixtures.createComplexGoal()

      // Act
      const subgoals = await goalManager.decompose(complexGoal)

      // Assert
      expect(subgoals).toHaveLength(3)
      expect(subgoals[0].parent).toBe(complexGoal.id)
      expect(subgoals.every(sg => sg.status === GoalStatus.PENDING)).toBe(true)
    })

    it('should not decompose simple goal', async () => {
      // Arrange
      const simpleGoal = fixtures.createSimpleGoal()

      // Act
      const subgoals = await goalManager.decompose(simpleGoal)

      // Assert
      expect(subgoals).toHaveLength(1)
      expect(subgoals[0]).toBe(simpleGoal)
    })

    it('should handle decomposition failure', async () => {
      // Arrange
      const invalidGoal = fixtures.createInvalidGoal()

      // Act & Assert
      await expect(goalManager.decompose(invalidGoal))
        .rejects
        .toThrow(DecompositionError)
    })
  })

  describe('isAchieved', () => {
    it('should return true when all criteria met', async () => {
      // Arrange
      const goal = fixtures.createGoalWithCriteria()
      const successState = fixtures.createSuccessState()

      // Act
      const achieved = await goalManager.isAchieved(goal, successState)

      // Assert
      expect(achieved).toBe(true)
    })

    it('should return false when criteria not met', async () => {
      // Arrange
      const goal = fixtures.createGoalWithCriteria()
      const failureState = fixtures.createFailureState()

      // Act
      const achieved = await goalManager.isAchieved(goal, failureState)

      // Assert
      expect(achieved).toBe(false)
    })
  })
})
```

### Strategia di Mocking

```typescript
// Mock LLM per test deterministici
class MockLLM implements LanguageModel {
  private responses: Map<string, string> = new Map()

  setResponse(promptPattern: string, response: string): void {
    this.responses.set(promptPattern, response)
  }

  async generate(prompt: string): Promise<string> {
    // Trova risposta corrispondente
    for (const [pattern, response] of this.responses) {
      if (prompt.includes(pattern)) {
        return response
      }
    }

    throw new Error(`No mock response for prompt: ${prompt}`)
  }
}

// Mock Memory per stato controllato
class MockMemorySystem implements MemorySystem {
  private storage: Map<UUID, Memory> = new Map()

  seed(memories: Memory[]): void {
    for (const memory of memories) {
      this.storage.set(memory.id, memory)
    }
  }

  async retrieve(cue: MemoryCue): Promise<Memory[]> {
    // Recupero semplificato per testing
    return Array.from(this.storage.values())
      .filter(m => this.matches(m, cue))
      .slice(0, cue.limit)
  }

  async store(memory: Memory): Promise<void> {
    this.storage.set(memory.id, memory)
  }
}

// Mock Tools per esecuzione controllata
class MockToolRegistry implements ToolRegistry {
  private tools: Map<string, MockTool> = new Map()

  registerMock(name: string, behavior: ToolBehavior): void {
    this.tools.set(name, {
      name,
      execute: behavior.execute,
      shouldFail: behavior.shouldFail || false,
      latency: behavior.latency || 0
    })
  }

  async execute(tool: string, params: any): Promise<ToolResult> {
    const mockTool = this.tools.get(tool)
    if (!mockTool) throw new Error(`Tool ${tool} not registered`)

    // Simula latenza
    if (mockTool.latency > 0) {
      await sleep(mockTool.latency)
    }

    // Simula fallimento
    if (mockTool.shouldFail) {
      throw new Error(`Mock tool ${tool} configured to fail`)
    }

    return mockTool.execute(params)
  }
}
```

### Property-Based Testing

```typescript
import { fc } from 'fast-check'

describe('GoalManager - Property Tests', () => {
  it('decomposed subgoals should cover original goal', async () => {
    await fc.assert(
      fc.asyncProperty(
        fc.record({
          description: fc.string(),
          complexity: fc.integer(1, 10)
        }),
        async (goalSpec) => {
          const goal = createGoalFromSpec(goalSpec)
          const subgoals = await goalManager.decompose(goal)

          // Proprietà: Unione degli scope dei subgoal copre obiettivo originale
          const coverage = computeCoverage(goal, subgoals)
          expect(coverage).toBeGreaterThanOrEqual(0.95)
        }
      )
    )
  })

  it('achievement detection should be monotonic', async () => {
    await fc.assert(
      fc.asyncProperty(
        fc.record({
          goal: fc.constantFrom(...fixtures.goals),
          states: fc.array(fc.anything(), { minLength: 2 })
        }),
        async ({ goal, states }) => {
          // Proprietà: Se raggiunto allo stato[i], deve essere raggiunto allo stato[j] per j > i
          let wasAchieved = false

          for (const state of states) {
            const achieved = await goalManager.isAchieved(goal, state)

            if (wasAchieved) {
              // Una volta raggiunto, deve rimanere raggiunto
              expect(achieved).toBe(true)
            }

            wasAchieved = achieved
          }
        }
      )
    )
  })
})
```

---

## Integration Testing

### Design Test di Integrazione

```typescript
describe('Reasoning Engine Integration', () => {
  let agent: Agent
  let memorySystem: MemorySystem
  let toolRegistry: ToolRegistry
  let llm: LanguageModel

  beforeEach(async () => {
    // Usa implementazioni reali ma ambiente controllato
    memorySystem = new MemorySystem(testConfig.memory)
    toolRegistry = new ToolRegistry(testConfig.tools)
    llm = new LanguageModel(testConfig.llm)

    agent = new Agent({
      memory: memorySystem,
      tools: toolRegistry,
      llm: llm,
      config: testConfig.agent
    })
  })

  afterEach(async () => {
    await agent.cleanup()
  })

  it('should execute simple task end-to-end', async () => {
    // Arrange
    const task = fixtures.createSimpleTask()
    const budget = fixtures.createStandardBudget()

    // Act
    const result = await agent.execute(task, budget)

    // Assert
    expect(result.success).toBe(true)
    expect(result.resourcesUsed.tokens).toBeLessThan(budget.maxTokens)
    expect(result.duration).toBeLessThan(budget.maxWallTime)
  })

  it('should handle tool failures gracefully', async () => {
    // Arrange
    const task = fixtures.createTaskRequiringTool('flaky-tool')

    // Configura tool per fallire prima volta, successo seconda volta
    toolRegistry.configureTool('flaky-tool', {
      failurePattern: [true, false]
    })

    // Act
    const result = await agent.execute(task, budget)

    // Assert
    expect(result.success).toBe(true)
    expect(result.retriedOperations).toContain('tool:flaky-tool')
  })

  it('should store and retrieve from memory', async () => {
    // Arrange
    const task1 = fixtures.createTask('Learn about X')
    const task2 = fixtures.createTask('What did I learn about X?')

    // Act
    await agent.execute(task1, budget)
    const result2 = await agent.execute(task2, budget)

    // Assert
    expect(result2.output).toContain('X')
    expect(result2.usedMemories).toHaveLength(1)
    expect(result2.usedMemories[0].episodeId).toBe(task1.episodeId)
  })
})
```

### Pattern Test di Integrazione

```typescript
// Pattern: Test basato su stato
class StateMachine {
  async execute(input: Input): Promise<Output> {
    // ...
  }
}

it('should transition states correctly', async () => {
  const machine = new StateMachine(initialState)

  // Verifica stato iniziale
  expect(machine.getState()).toEqual(expectedInitialState)

  // Esegui operazione
  await machine.execute(input)

  // Verifica transizione stato
  expect(machine.getState()).toEqual(expectedFinalState)
})

// Pattern: Test interazione
it('should call dependencies in correct order', async () => {
  const mockMemory = createMock<MemorySystem>()
  const mockTools = createMock<ToolRegistry>()

  const agent = new Agent({ memory: mockMemory, tools: mockTools })

  await agent.execute(task, budget)

  // Verifica ordine chiamate
  expect(mockMemory.retrieve).toHaveBeenCalledBefore(mockTools.execute)
})

// Pattern: Test flusso dati
it('should propagate data correctly', async () => {
  const pipeline = new Pipeline([
    stage1,  // Input: A, Output: B
    stage2,  // Input: B, Output: C
    stage3   // Input: C, Output: D
  ])

  const output = await pipeline.execute(inputA)

  expect(output).toEqual(expectedD)
  expect(stage1.receivedInput).toEqual(inputA)
  expect(stage2.receivedInput).toEqual(stage1.output)
  expect(stage3.receivedInput).toEqual(stage2.output)
})
```

---

## End-to-End Testing

### Scenari Test E2E

```typescript
describe('Agent E2E Scenarios', () => {
  // Usa LLM reale ma in modalità deterministica
  const llm = new LanguageModel({ temperature: 0 })

  it('Scenario: Code Generation', async () => {
    const task = {
      description: 'Create a Python function to calculate fibonacci numbers',
      successCriteria: [
        { type: 'file_created', path: 'fibonacci.py' },
        { type: 'tests_pass', count: 5 }
      ]
    }

    const result = await agent.execute(task)

    expect(result.success).toBe(true)
    expect(fileExists('fibonacci.py')).toBe(true)
    expect(await runTests('fibonacci.py')).toBe('pass')
  })

  it('Scenario: Research & Analysis', async () => {
    const task = {
      description: 'Research the latest developments in quantum computing',
      successCriteria: [
        { type: 'sources_used', minCount: 5 },
        { type: 'summary_length', minLength: 500 }
      ]
    }

    const result = await agent.execute(task)

    expect(result.success).toBe(true)
    expect(result.sourcesUsed.length).toBeGreaterThanOrEqual(5)
    expect(result.summary.length).toBeGreaterThanOrEqual(500)
  })

  it('Scenario: Complex Multi-Step Task', async () => {
    const task = {
      description: 'Build a todo app with backend and frontend',
      successCriteria: [
        { type: 'files_created', minCount: 10 },
        { type: 'server_running', port: 3000 },
        { type: 'ui_accessible', url: 'http://localhost:3000' }
      ]
    }

    const result = await agent.execute(task, {
      maxTokens: 500000,
      maxWallTime: 600000  // 10 minuti
    })

    expect(result.success).toBe(true)
    expect(result.filesCreated.length).toBeGreaterThanOrEqual(10)
  })
})
```

### Simulazione Ambiente

```typescript
class TestEnvironment {
  private fileSystem: MockFileSystem
  private network: MockNetwork
  private processes: MockProcessManager

  async setup(): Promise<void> {
    this.fileSystem = new MockFileSystem()
    this.network = new MockNetwork()
    this.processes = new MockProcessManager()
  }

  async teardown(): Promise<void> {
    await this.fileSystem.cleanup()
    await this.network.cleanup()
    await this.processes.cleanup()
  }

  // Simula operazioni file
  fileExists(path: string): boolean {
    return this.fileSystem.exists(path)
  }

  readFile(path: string): string {
    return this.fileSystem.read(path)
  }

  // Simula rete
  mockAPIResponse(url: string, response: any): void {
    this.network.setResponse(url, response)
  }

  // Simula processi
  mockCommandOutput(command: string, output: string): void {
    this.processes.setOutput(command, output)
  }
}
```

---

## Performance Benchmarking

### Framework Benchmark

```typescript
interface BenchmarkConfig {
  name: string
  warmupRuns: number
  measurementRuns: number
  timeout: number
}

class BenchmarkRunner {
  async runBenchmark(
    config: BenchmarkConfig,
    operation: () => Promise<any>
  ): Promise<BenchmarkResult> {

    // Fase warmup (preparazione cache)
    for (let i = 0; i < config.warmupRuns; i++) {
      await operation()
    }

    // Fase misurazione
    const measurements: Measurement[] = []

    for (let i = 0; i < config.measurementRuns; i++) {
      const start = performance.now()
      const startMemory = process.memoryUsage()

      try {
        await operation()

        const duration = performance.now() - start
        const endMemory = process.memoryUsage()

        measurements.push({
          duration,
          memoryDelta: endMemory.heapUsed - startMemory.heapUsed,
          success: true
        })

      } catch (error) {
        measurements.push({
          duration: performance.now() - start,
          memoryDelta: 0,
          success: false,
          error: error.message
        })
      }
    }

    return this.analyzeResults(config.name, measurements)
  }

  private analyzeResults(
    name: string,
    measurements: Measurement[]
  ): BenchmarkResult {

    const durations = measurements
      .filter(m => m.success)
      .map(m => m.duration)

    return {
      name,
      runs: measurements.length,
      successCount: measurements.filter(m => m.success).length,

      duration: {
        min: Math.min(...durations),
        max: Math.max(...durations),
        mean: mean(durations),
        median: median(durations),
        p95: percentile(durations, 0.95),
        p99: percentile(durations, 0.99),
        stddev: standardDeviation(durations)
      },

      memory: {
        avgDelta: mean(measurements.map(m => m.memoryDelta))
      }
    }
  }
}
```

### Benchmark Standard

```typescript
const STANDARD_BENCHMARKS = {
  // Benchmark latenza
  'simple_query_latency': {
    operation: () => agent.execute(simpleQuery),
    target: { p95: 2000 },  // <2s
    warmupRuns: 10,
    measurementRuns: 100
  },

  'complex_task_latency': {
    operation: () => agent.execute(complexTask),
    target: { p95: 300000 },  // <5min
    warmupRuns: 3,
    measurementRuns: 20
  },

  // Benchmark throughput
  'concurrent_tasks': {
    operation: () => executeConcurrent(10),
    target: { throughput: 100 },  // 100 task/sec
    warmupRuns: 5,
    measurementRuns: 50
  },

  // Benchmark memoria
  'memory_retrieval': {
    operation: () => memory.retrieve(commonQuery),
    target: { p95: 100 },  // <100ms
    warmupRuns: 50,
    measurementRuns: 1000
  },

  // Benchmark costo
  'cost_per_task': {
    operation: () => agent.execute(typicalTask),
    target: { mean: 0.10 },  // <$0.10
    warmupRuns: 5,
    measurementRuns: 50
  }
}
```

### Test di Regressione

```typescript
class RegressionTester {
  async checkRegression(
    benchmark: string,
    current: BenchmarkResult,
    baseline: BenchmarkResult
  ): Promise<RegressionReport> {

    const regressions: Regression[] = []

    // Controlla regressione latenza p95 (soglia 20%)
    if (current.duration.p95 > baseline.duration.p95 * 1.2) {
      regressions.push({
        metric: 'p95_latency',
        baseline: baseline.duration.p95,
        current: current.duration.p95,
        degradation: (current.duration.p95 - baseline.duration.p95) / baseline.duration.p95,
        threshold: 0.2,
        severity: 'high'
      })
    }

    // Controlla regressione costo medio (soglia 50%)
    if (current.cost.mean > baseline.cost.mean * 1.5) {
      regressions.push({
        metric: 'mean_cost',
        baseline: baseline.cost.mean,
        current: current.cost.mean,
        degradation: (current.cost.mean - baseline.cost.mean) / baseline.cost.mean,
        threshold: 0.5,
        severity: 'medium'
      })
    }

    // Controlla regressione tasso successo (soglia 5%)
    if (current.successRate < baseline.successRate * 0.95) {
      regressions.push({
        metric: 'success_rate',
        baseline: baseline.successRate,
        current: current.successRate,
        degradation: (baseline.successRate - current.successRate) / baseline.successRate,
        threshold: 0.05,
        severity: 'critical'
      })
    }

    return {
      benchmark,
      passed: regressions.length === 0,
      regressions
    }
  }
}
```

---

## Metriche di Valutazione

### Metriche Successo Task

```typescript
interface TaskMetrics {
  // Successo binario
  success: boolean

  // Successo parziale
  goalsAchieved: number
  goalsFailed: number
  completionRate: number  // 0-1

  // Metriche qualità
  outputQuality: number   // 0-1 (valutazione umana o automatica)
  confidence: number      // Confidenza auto-valutata dell'agente
  calibration: number     // Quanto bene la confidenza corrisponde al successo reale

  // Metriche efficienza
  tokensUsed: number
  llmCallsUsed: number
  toolCallsUsed: number
  duration: number
  cost: number

  // Confronto con ottimale
  tokensOptimal: number
  tokensEfficiency: number  // tokensOptimal / tokensUsed

  // Qualità ragionamento
  planQuality: number      // Quanto era buono il piano
  executionQuality: number // Quanto bene è stato eseguito
  recoveryQuality: number  // Quanto bene sono stati gestiti gli errori
}
```

### Valutazione Qualità

```typescript
async function evaluateQuality(
  output: any,
  expected: any,
  criteria: EvaluationCriteria
): Promise<number> {

  const scores: number[] = []

  // Correttezza
  if (criteria.includeCorrectness) {
    const correctness = await evaluateCorrectness(output, expected)
    scores.push(correctness)
  }

  // Completezza
  if (criteria.includeCompleteness) {
    const completeness = evaluateCompleteness(output, expected)
    scores.push(completeness)
  }

  // Coerenza
  if (criteria.includeCoherence) {
    const coherence = await evaluateCoherence(output)
    scores.push(coherence)
  }

  // Efficienza
  if (criteria.includeEfficiency) {
    const efficiency = evaluateEfficiency(output, criteria.resourceUsage)
    scores.push(efficiency)
  }

  // Media ponderata
  return mean(scores)
}

async function evaluateCorrectness(
  output: any,
  expected: any
): Promise<number> {
  // Corrispondenza esatta
  if (deepEqual(output, expected)) {
    return 1.0
  }

  // Similarità semantica (usando embeddings)
  const outputEmb = await embeddings.embed(JSON.stringify(output))
  const expectedEmb = await embeddings.embed(JSON.stringify(expected))
  const similarity = cosineSimilarity(outputEmb, expectedEmb)

  return similarity
}
```

### Metriche di Calibrazione

```typescript
class CalibrationEvaluator {
  async evaluateCalibration(
    episodes: Episode[]
  ): Promise<CalibrationMetrics> {

    // Raggruppa per bin di confidenza
    const bins = [0, 0.2, 0.4, 0.6, 0.8, 1.0]
    const binned: Map<number, Episode[]> = new Map()

    for (const episode of episodes) {
      const bin = bins.find(b => episode.confidence <= b) || 1.0
      const binEpisodes = binned.get(bin) || []
      binEpisodes.push(episode)
      binned.set(bin, binEpisodes)
    }

    // Calcola tasso successo reale per bin
    const calibration: CalibrationBin[] = []

    for (const [bin, binEpisodes] of binned) {
      const actualSuccessRate = binEpisodes.filter(e => e.outcome.success).length / binEpisodes.length

      calibration.push({
        confidenceBin: bin,
        predictedSuccessRate: bin,
        actualSuccessRate,
        count: binEpisodes.length,
        error: Math.abs(bin - actualSuccessRate)
      })
    }

    // Errore calibrazione complessivo
    const calibrationError = mean(calibration.map(c => c.error))

    return {
      bins: calibration,
      overallError: calibrationError,
      wellCalibrated: calibrationError < 0.1  // <10% errore
    }
  }
}
```

---

## Infrastruttura di Test

### Test Harness

```typescript
class TestHarness {
  private fixtures: TestFixtureFactory
  private mocks: MockFactory
  private assertions: AssertionLibrary

  createTestAgent(config?: Partial<AgentConfig>): Agent {
    return new Agent({
      llm: this.mocks.createLLM(config?.llm),
      memory: this.mocks.createMemory(config?.memory),
      tools: this.mocks.createTools(config?.tools),
      ...config
    })
  }

  async runInSandbox<T>(
    operation: () => Promise<T>
  ): Promise<T> {
    const sandbox = await this.createSandbox()

    try {
      return await sandbox.execute(operation)
    } finally {
      await sandbox.cleanup()
    }
  }

  async compareWithBaseline(
    testName: string,
    result: any
  ): Promise<ComparisonResult> {
    const baseline = await this.loadBaseline(testName)
    return this.assertions.compare(result, baseline)
  }
}
```

### Libreria Asserzioni

```typescript
class AgentAssertions {
  // Successo task
  assertTaskSucceeded(result: TaskResult): void {
    expect(result.success).toBe(true)
    expect(result.outcome.type).toBe(OutcomeType.SUCCESS)
  }

  // Utilizzo risorse
  assertWithinBudget(result: TaskResult, budget: ResourceBudget): void {
    expect(result.resourcesUsed.tokens).toBeLessThanOrEqual(budget.maxTokens)
    expect(result.resourcesUsed.cost).toBeLessThanOrEqual(budget.maxCost)
    expect(result.duration).toBeLessThanOrEqual(budget.maxWallTime)
  }

  // Operazioni memoria
  assertMemoryStored(memoryId: UUID): void {
    const exists = await memory.exists(memoryId)
    expect(exists).toBe(true)
  }

  assertMemoryRetrieved(result: TaskResult, expectedMemories: UUID[]): void {
    const retrieved = result.memoriesUsed.map(m => m.id)
    expect(retrieved).toEqual(expect.arrayContaining(expectedMemories))
  }

  // Tracing
  assertTraceComplete(trace: Trace): void {
    expect(trace.spans.length).toBeGreaterThan(0)
    expect(trace.spans.every(s => s.endTime > s.startTime)).toBe(true)
  }

  // Qualità
  assertQuality(result: any, minQuality: number): void {
    const quality = evaluateQuality(result)
    expect(quality).toBeGreaterThanOrEqual(minQuality)
  }
}
```

---

## Testing Continuo

### Pipeline CI/CD

```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run unit tests
        run: npm run test:unit
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - name: Run integration tests
        run: npm run test:integration

  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    steps:
      - name: Run E2E tests
        run: npm run test:e2e

  benchmarks:
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests]
    steps:
      - name: Run benchmarks
        run: npm run benchmark
      - name: Check regression
        run: npm run benchmark:regression
```

### Copertura Test

```typescript
interface CoverageReport {
  // Copertura righe
  lines: {
    total: number
    covered: number
    percentage: number
  }

  // Copertura branch
  branches: {
    total: number
    covered: number
    percentage: number
  }

  // Copertura funzioni
  functions: {
    total: number
    covered: number
    percentage: number
  }

  // Copertura per file
  files: Map<string, FileCoverage>
}

// Target copertura
const COVERAGE_TARGETS = {
  lines: 80,      // 80% copertura righe
  branches: 70,   // 70% copertura branch
  functions: 85   // 85% copertura funzioni
}
```

---

## Conclusione

Il Framework Testing e Benchmarking fornisce validazione completa attraverso:

1. **Testing Multi-Livello**: Unit (70%), Integrazione (20%), E2E (10%)
2. **Benchmark Prestazioni**: Misurazione sistematica e rilevamento regressioni
3. **Metriche Qualità**: Correttezza, completezza, efficienza
4. **Testing Deterministico**: Mocking e fixture per riproducibilità
5. **Testing Continuo**: Integrazione CI/CD automatizzata
6. **Tracciamento Copertura**: Target >80% copertura codice

Questo garantisce che l'architettura dell'agente sia completamente validata, performante e production-ready.

---

## Riferimenti

1. Fowler, M. (2012). Testing Strategies in Continuous Delivery.
2. Google Testing Blog. Test sizes and the testing pyramid.
3. Documentazione Jest (2024). Framework testing JavaScript.
4. Documentazione pytest (2024). Framework testing Python.
5. Documentazione JMeter (2024). Testing prestazioni.
6. Benchmark.js. Libreria benchmarking JavaScript.
