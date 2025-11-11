# Sistema di Osservabilità e Tracciamento

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Principi di Osservabilità](#principi-di-osservabilita)
3. [I Tre Pilastri](#i-tre-pilastri)
4. [Tracciamento Distribuito](#tracciamento-distribuito)
5. [Logging Strutturato](#logging-strutturato)
6. [Raccolta Metriche](#raccolta-metriche)
7. [Interfacce di Debugging](#interfacce-di-debugging)
8. [Meccanismi di Replay](#meccanismi-di-replay)
9. [Contratti Dati](#contratti-dati)
10. [Architettura di Implementazione](#architettura-di-implementazione)
11. [Caratteristiche di Prestazione](#caratteristiche-di-prestazione)

---

## Panoramica

Il Sistema di Osservabilità e Tracciamento fornisce visibilità completa sul comportamento dell'agente, abilitando debugging, ottimizzazione delle prestazioni, auditing per conformità e comprensione del sistema.

### Obiettivi di Design

1. **Visibilità Completa**: Ogni azione, decisione e cambio di stato è osservabile
2. **Overhead Basso**: <5% impatto sulle prestazioni
3. **Real-time**: Monitoraggio e alerting live
4. **Interrogabile**: Ricche capacità di query su tracce, log e metriche
5. **Actionable**: Segnali chiari per debugging e ottimizzazione
6. **Conforme**: Soddisfare requisiti normativi per auditing
7. **Riproducibile**: Abilitare replay esatto di qualsiasi esecuzione

### Stack di Osservabilità

```
┌────────────────────────────────────────────┐
│       Sistema di Osservabilità             │
│                                            │
│  ┌──────────────────────────────────┐    │
│  │     Livello Instrumentazione     │    │
│  │  • Auto-instrumentazione         │    │
│  │  • Instrumentazione manuale      │    │
│  │  • Propagazione contesto         │    │
│  └──────────┬───────────────────────┘    │
│             │                             │
│  ┌──────────▼───────────────────────┐    │
│  │      Livello Raccolta            │    │
│  │  • Trace collector               │    │
│  │  • Log aggregator                │    │
│  │  • Metrics scraper               │    │
│  └──────────┬───────────────────────┘    │
│             │                             │
│  ┌──────────▼───────────────────────┐    │
│  │      Livello Storage             │    │
│  │  • Trace store (Jaeger/Tempo)    │    │
│  │  • Log store (Elasticsearch)     │    │
│  │  • Metrics store (Prometheus)    │    │
│  └──────────┬───────────────────────┘    │
│             │                             │
│  ┌──────────▼───────────────────────┐    │
│  │      Livello Query e Analisi     │    │
│  │  • Analisi tracce (Jaeger UI)    │    │
│  │  • Ricerca log (Kibana)          │    │
│  │  • Dashboard (Grafana)           │    │
│  │  • Alerting (Prometheus)         │    │
│  └──────────────────────────────────┘    │
└────────────────────────────────────────────┘
```

---

## Principi di Osservabilità

### Principio 1: Osservabile di Default

Ogni componente emette telemetria automaticamente - nessuna configurazione speciale richiesta.

```typescript
// Male: Osservabilità opzionale
class Component {
  execute() {
    // Nessuna telemetria
    return doWork()
  }
}

// Bene: Osservabilità integrata
class Component {
  @traced()
  @logged()
  @metered()
  execute() {
    return doWork()
  }
}
```

### Principio 2: Dati Strutturati

Tutta la telemetria è strutturata, tipizzata e validata con schema.

```typescript
// Male: Log non strutturato
console.log('User John did thing at ' + new Date())

// Bene: Log strutturato
logger.info('user_action', {
  user_id: 'john',
  action: 'thing',
  timestamp: Date.now(),
  context: { ... }
})
```

### Principio 3: Propagazione Contesto

Il contesto di esecuzione fluisce attraverso tutti i componenti automaticamente.

```typescript
interface ExecutionContext {
  requestId: UUID
  userId: UUID
  sessionId: UUID
  traceId: UUID
  spanId: UUID
  parentSpanId: UUID | null
}
```

### Principio 4: Sampling per Scala

Gli eventi ad alto volume vengono campionati intelligentemente.

```typescript
interface SamplingStrategy {
  // Campiona sempre errori
  alwaysSampleErrors: true

  // Campiona 1% dei casi di successo
  successSampleRate: 0.01

  // Campiona 100% delle richieste lente (>1s)
  alwaysSampleSlow: true

  // Sampling adattivo basato sul carico
  adaptiveSampling: {
    targetRate: 1000,  // tracce/sec
    currentLoad: () => getLoad()
  }
}
```

### Principio 5: Correlazione

Tutta la telemetria per una singola richiesta è correlata via ID.

```
Request ID: req_123
├─ Trace ID: trace_456
│  ├─ Span: reasoning_engine
│  │  ├─ Span: goal_decomposition
│  │  └─ Span: plan_generation
│  └─ Span: tool_execution
│     └─ Span: api_call
├─ Log: [log1, log2, log3] (tutti taggati con req_123)
└─ Metriche: [m1, m2, m3] (tutti etichettati con req_123)
```

---

## I Tre Pilastri

### 1. Log

**Scopo**: Eventi discreti con contesto

**Quando Loggare**:
- Transizioni di stato
- Errori e warning
- Operazioni significative
- Punti di decisione
- Interazioni utente

**Livelli di Log**:
```typescript
enum LogLevel {
  DEBUG = 0,    // Verboso, solo development
  INFO = 1,     // Operazioni normali
  WARN = 2,     // Qualcosa di insolito ma non errore
  ERROR = 3,    // Errore occorso ma recuperato
  FATAL = 4     // Errore irrecuperabile
}
```

**Struttura Log**:
```typescript
interface LogEntry {
  // Identificazione
  id: UUID
  timestamp: number
  level: LogLevel

  // Contesto
  requestId: UUID
  traceId: UUID
  spanId: UUID
  userId: UUID

  // Contenuto
  message: string
  event: string
  data: Record<string, any>

  // Sorgente
  component: string
  function: string
  file: string
  line: number

  // Tag
  tags: string[]
  labels: Record<string, string>
}
```

### 2. Metriche

**Scopo**: Misurazioni aggregabili nel tempo

**Tipi di Metriche**:
```typescript
enum MetricType {
  COUNTER = 'counter',       // Monotonicamente crescente (richieste)
  GAUGE = 'gauge',          // Valore punto nel tempo (uso memoria)
  HISTOGRAM = 'histogram',   // Distribuzione (latenze)
  SUMMARY = 'summary'        // Sommario statistico (percentili)
}
```

**Metriche Chiave**:
```typescript
interface AgentMetrics {
  // Metriche richieste
  requests_total: Counter
  requests_duration_seconds: Histogram
  requests_in_flight: Gauge

  // Metriche LLM
  llm_calls_total: Counter
  llm_tokens_total: Counter
  llm_latency_seconds: Histogram
  llm_errors_total: Counter

  // Metriche strumenti
  tool_executions_total: Counter
  tool_duration_seconds: Histogram
  tool_errors_total: Counter

  // Metriche memoria
  memory_retrievals_total: Counter
  memory_cache_hit_rate: Gauge
  memory_active_memories: Gauge

  // Metriche risorse
  cost_dollars_total: Counter
  cpu_usage_percent: Gauge
  memory_usage_bytes: Gauge

  // Metriche qualità
  task_success_rate: Gauge
  user_satisfaction: Histogram
  confidence_scores: Histogram
}
```

### 3. Tracce

**Scopo**: Ciclo di vita richiesta e dipendenze

**Struttura Traccia**:
```typescript
interface Trace {
  traceId: UUID
  rootSpanId: UUID
  spans: Span[]
  startTime: number
  endTime: number
  duration: number
  status: TraceStatus
}

interface Span {
  spanId: UUID
  traceId: UUID
  parentSpanId: UUID | null
  name: string
  startTime: number
  endTime: number
  duration: number
  status: SpanStatus
  attributes: Record<string, any>
  events: SpanEvent[]
  links: SpanLink[]
}

enum SpanStatus {
  OK = 'ok',
  ERROR = 'error',
  UNSET = 'unset'
}

interface SpanEvent {
  timestamp: number
  name: string
  attributes: Record<string, any>
}
```

---

## Tracciamento Distribuito

### Propagazione Traccia

```typescript
interface TracePropagation {
  // Estrae contesto da richiesta in entrata
  extract(carrier: any): SpanContext | null

  // Inietta contesto in richiesta in uscita
  inject(context: SpanContext, carrier: any): void
}

interface SpanContext {
  traceId: UUID
  spanId: UUID
  traceFlags: number
  traceState: string
}

// Standard W3C Trace Context
// traceparent: 00-{trace-id}-{span-id}-{flags}
// Esempio: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

### Creazione Span

```typescript
class Tracer {
  startSpan(name: string, options?: SpanOptions): Span {
    const span = new Span({
      spanId: generateSpanId(),
      traceId: this.currentTrace.traceId,
      parentSpanId: this.currentSpan?.spanId,
      name,
      startTime: Date.now(),
      attributes: options?.attributes || {}
    })

    this.activeSpans.push(span)
    return span
  }

  endSpan(span: Span): void {
    span.endTime = Date.now()
    span.duration = span.endTime - span.startTime
    this.activeSpans = this.activeSpans.filter(s => s !== span)
    this.collector.collect(span)
  }
}

// Uso
const span = tracer.startSpan('goal_decomposition')
try {
  const result = await decomposeGoal(goal)
  span.setStatus(SpanStatus.OK)
  span.setAttribute('subgoals_count', result.length)
  return result
} catch (error) {
  span.setStatus(SpanStatus.ERROR)
  span.recordException(error)
  throw error
} finally {
  tracer.endSpan(span)
}
```

### Instrumentazione Automatica

```typescript
// Decorator per tracing automatico
function traced(name?: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value

    descriptor.value = async function (...args: any[]) {
      const spanName = name || `${target.constructor.name}.${propertyKey}`
      const span = tracer.startSpan(spanName)

      try {
        const result = await original.apply(this, args)
        span.setStatus(SpanStatus.OK)
        return result
      } catch (error) {
        span.setStatus(SpanStatus.ERROR)
        span.recordException(error)
        throw error
      } finally {
        tracer.endSpan(span)
      }
    }

    return descriptor
  }
}

// Uso
class ReasoningEngine {
  @traced()
  async decompose(goal: Goal): Promise<Goal[]> {
    // Automaticamente tracciato
    return this.goalManager.decompose(goal)
  }
}
```

### Span Critici

Span chiave da tracciare:

```typescript
const CRITICAL_SPANS = [
  // Ciclo di vita richiesta
  'request',
  'authentication',
  'authorization',

  // Ragionamento
  'goal_decomposition',
  'planning',
  'plan_execution',
  'reflection',

  // Memoria
  'memory_retrieval',
  'memory_storage',

  // Strumenti
  'tool_discovery',
  'tool_execution',

  // Chiamate LLM
  'llm_call',
  'prompt_construction',
  'response_parsing'
]
```

---

## Logging Strutturato

### Formato Log

```typescript
interface StructuredLog {
  // Campi standard (sempre presenti)
  timestamp: string        // ISO 8601
  level: LogLevel
  message: string

  // Contesto (sempre presente)
  request_id: UUID
  trace_id: UUID
  span_id: UUID
  user_id: UUID

  // Sorgente (sempre presente)
  component: string
  function: string

  // Dati specifici dell'evento
  event: string
  data: Record<string, any>

  // Opzionale
  error?: {
    type: string
    message: string
    stack: string
  }
  tags?: string[]
  labels?: Record<string, string>
}
```

### Esempi di Log

```typescript
// Log info
logger.info('goal_decomposed', {
  goal_id: 'goal_123',
  subgoal_count: 3,
  strategy: 'hierarchical'
})

// Log errore
logger.error('tool_execution_failed', {
  tool_id: 'tool_456',
  error_type: 'TimeoutError',
  duration_ms: 5000
}, error)

// Log warn
logger.warn('memory_cache_miss', {
  query: 'find similar episodes',
  fallback: 'vector search'
})
```

### Aggregazione Log

```typescript
interface LogAggregator {
  // Raccoglie log da tutte le fonti
  collect(log: StructuredLog): void

  // Batch e forward
  async flush(): Promise<void>

  // Filtra e campiona
  shouldSample(log: StructuredLog): boolean

  // Arricchisce con contesto
  enrich(log: StructuredLog): StructuredLog
}

class ElasticsearchLogAggregator implements LogAggregator {
  private buffer: StructuredLog[] = []
  private bufferSize = 1000

  collect(log: StructuredLog): void {
    const enriched = this.enrich(log)

    if (this.shouldSample(enriched)) {
      this.buffer.push(enriched)

      if (this.buffer.length >= this.bufferSize) {
        this.flush()
      }
    }
  }

  async flush(): Promise<void> {
    if (this.buffer.length === 0) return

    await this.elasticsearch.bulk({
      index: 'agent-logs',
      body: this.buffer.flatMap(log => [
        { index: { _index: `agent-logs-${getDate()}` } },
        log
      ])
    })

    this.buffer = []
  }

  shouldSample(log: StructuredLog): boolean {
    // Logga sempre errori
    if (log.level >= LogLevel.ERROR) return true

    // Campiona debug/info in base al rate
    return Math.random() < this.sampleRate
  }

  enrich(log: StructuredLog): StructuredLog {
    return {
      ...log,
      environment: process.env.NODE_ENV,
      hostname: os.hostname(),
      process_id: process.pid,
      version: getVersion()
    }
  }
}
```

---

## Raccolta Metriche

### Raccolta Metriche

```typescript
interface MetricsCollector {
  // Counter
  incrementCounter(name: string, value: number, labels?: Labels): void

  // Gauge
  setGauge(name: string, value: number, labels?: Labels): void

  // Histogram
  recordHistogram(name: string, value: number, labels?: Labels): void

  // Summary
  recordSummary(name: string, value: number, labels?: Labels): void
}

type Labels = Record<string, string>

// Uso
const metricsCollector = new PrometheusCollector()

// Incrementa counter
metricsCollector.incrementCounter('llm_calls_total', 1, {
  model: 'gpt-4',
  status: 'success'
})

// Imposta gauge
metricsCollector.setGauge('memory_usage_bytes', getMemoryUsage())

// Registra histogram
metricsCollector.recordHistogram('request_duration_seconds', duration, {
  endpoint: '/api/agent/execute',
  status: '200'
})
```

### Metriche Standard

```typescript
class StandardMetrics {
  // Metriche richieste
  private requestsTotal = new Counter({
    name: 'agent_requests_total',
    help: 'Total number of agent requests',
    labelNames: ['endpoint', 'status']
  })

  private requestDuration = new Histogram({
    name: 'agent_request_duration_seconds',
    help: 'Request duration in seconds',
    labelNames: ['endpoint'],
    buckets: [0.1, 0.5, 1, 2, 5, 10, 30]
  })

  // Metriche LLM
  private llmCallsTotal = new Counter({
    name: 'agent_llm_calls_total',
    help: 'Total LLM calls',
    labelNames: ['model', 'status']
  })

  private llmTokens = new Counter({
    name: 'agent_llm_tokens_total',
    help: 'Total tokens used',
    labelNames: ['model', 'type']  // type: prompt, completion
  })

  private llmLatency = new Histogram({
    name: 'agent_llm_latency_seconds',
    help: 'LLM call latency',
    labelNames: ['model'],
    buckets: [0.5, 1, 2, 5, 10, 20]
  })

  // Metriche strumenti
  private toolExecutions = new Counter({
    name: 'agent_tool_executions_total',
    help: 'Tool executions',
    labelNames: ['tool', 'status']
  })

  // Metriche memoria
  private memoryRetrievals = new Counter({
    name: 'agent_memory_retrievals_total',
    help: 'Memory retrievals',
    labelNames: ['type', 'cache_hit']
  })

  // Metriche costi
  private costTotal = new Counter({
    name: 'agent_cost_dollars_total',
    help: 'Total cost in dollars',
    labelNames: ['category']  // llm, tools, infrastructure
  })
}
```

### Esposizione Metriche

```typescript
// Esposizione Prometheus
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType)
  res.end(await register.metrics())
})

// Esempio output:
/*
# HELP agent_requests_total Total number of agent requests
# TYPE agent_requests_total counter
agent_requests_total{endpoint="/api/agent/execute",status="success"} 1234
agent_requests_total{endpoint="/api/agent/execute",status="error"} 56

# HELP agent_request_duration_seconds Request duration in seconds
# TYPE agent_request_duration_seconds histogram
agent_request_duration_seconds_bucket{endpoint="/api/agent/execute",le="0.1"} 100
agent_request_duration_seconds_bucket{endpoint="/api/agent/execute",le="0.5"} 500
agent_request_duration_seconds_bucket{endpoint="/api/agent/execute",le="1"} 900
agent_request_duration_seconds_sum{endpoint="/api/agent/execute"} 1234.5
agent_request_duration_seconds_count{endpoint="/api/agent/execute"} 1234
*/
```

---

## Interfacce di Debugging

### Interfaccia Query Tracce

```typescript
interface TraceQuery {
  traceId?: UUID
  spanName?: string
  serviceName?: string
  minDuration?: number
  maxDuration?: number
  tags?: Record<string, string>
  startTime?: number
  endTime?: number
  limit?: number
}

interface TraceQueryService {
  async findTraces(query: TraceQuery): Promise<Trace[]>
  async getTrace(traceId: UUID): Promise<Trace | null>
  async getSpan(spanId: UUID): Promise<Span | null>
}

// Uso
const traces = await traceQuery.findTraces({
  spanName: 'llm_call',
  minDuration: 5000,  // >5s
  tags: { model: 'gpt-4' },
  startTime: Date.now() - 3600000,  // Ultima ora
  limit: 100
})
```

### Debugging Live

```typescript
interface LiveDebugger {
  // Collega a esecuzione in corso
  async attach(requestId: UUID): Promise<DebugSession>

  // Step through esecuzione
  async stepOver(): Promise<void>
  async stepInto(): Promise<void>
  async continue(): Promise<void>

  // Ispeziona stato
  async getState(): Promise<State>
  async getMemory(): Promise<Memory>
  async getVariables(): Promise<Variables>

  // Modifica stato
  async setVariable(name: string, value: any): Promise<void>
  async injectEvent(event: Event): Promise<void>
}
```

### Debugging Time-Travel

```typescript
interface TimeTravelDebugger {
  // Carica esecuzione storica
  async load(traceId: UUID): Promise<Execution>

  // Naviga timeline
  async gotoTime(timestamp: number): Promise<void>
  async gotoSpan(spanId: UUID): Promise<void>

  // Ispeziona stato storico
  async getStateAt(timestamp: number): Promise<State>

  // Replay con modifiche
  async replayFrom(timestamp: number, modifications?: Modifications): Promise<Execution>
}
```

---

## Meccanismi di Replay

### Replay Deterministico

```typescript
interface ReplayEngine {
  // Registra esecuzione
  async record(execution: Execution): Promise<Recording>

  // Replay esecuzione
  async replay(recording: Recording): Promise<Execution>

  // Replay con modifiche
  async replayWith(
    recording: Recording,
    changes: ReplayChanges
  ): Promise<Execution>
}

interface Recording {
  // Metadata esecuzione
  traceId: UUID
  startTime: number
  endTime: number

  // Stato iniziale
  initialState: State

  // Eventi in ordine
  events: RecordedEvent[]

  // Input esterni
  llmResponses: Map<UUID, LLMResponse>
  toolOutputs: Map<UUID, ToolOutput>
  randomSeeds: Map<UUID, number>
}

interface RecordedEvent {
  timestamp: number
  type: EventType
  data: any
}
```

### Strategia di Replay

```typescript
async function replay(recording: Recording): Promise<Execution> {
  // 1. Ripristina stato iniziale
  const state = cloneDeep(recording.initialState)

  // 2. Mock dipendenze esterne
  const llmMock = createLLMMock(recording.llmResponses)
  const toolMock = createToolMock(recording.toolOutputs)
  const randomMock = createRandomMock(recording.randomSeeds)

  // 3. Crea agente con mock
  const agent = new Agent({
    llm: llmMock,
    tools: toolMock,
    random: randomMock,
    initialState: state
  })

  // 4. Replay eventi
  for (const event of recording.events) {
    await agent.handleEvent(event)
  }

  // 5. Ritorna esecuzione finale
  return agent.getExecution()
}
```

### Replay con Modifiche

```typescript
interface ReplayChanges {
  // Cambia risposte LLM
  llmResponses?: Map<UUID, LLMResponse>

  // Cambia output strumenti
  toolOutputs?: Map<UUID, ToolOutput>

  // Cambia stato iniziale
  initialState?: Partial<State>

  // Inietta nuovi eventi
  injectEvents?: RecordedEvent[]

  // Salta eventi
  skipEvents?: UUID[]
}

// Esempio: Replay con risposta LLM diversa
const newRecording = await replayEngine.replayWith(originalRecording, {
  llmResponses: new Map([
    ['llm_call_123', alternativeLLMResponse]
  ])
})
```

---

## Contratti Dati

### Contratto Traccia

```typescript
interface TraceContract {
  // Campi richiesti
  traceId: UUID
  rootSpanId: UUID
  startTime: number

  // Gerarchia span
  spans: SpanContract[]
}

interface SpanContract {
  // Campi richiesti
  spanId: UUID
  traceId: UUID
  name: string
  startTime: number
  endTime: number

  // Attributi standard
  attributes: {
    'component': string              // Nome componente
    'operation': string              // Nome operazione
    'request.id': UUID              // ID richiesta
    'user.id': UUID                 // ID utente
    'resource.used.tokens'?: number  // Token usati
    'resource.used.cost'?: number    // Costo in dollari
    'error'?: boolean               // Ha avuto errore
    'error.type'?: string           // Tipo errore
    'error.message'?: string        // Messaggio errore
  }
}
```

### Contratto Log

```typescript
interface LogContract {
  // Campi richiesti
  timestamp: string    // ISO 8601
  level: LogLevel
  message: string

  // Campi contesto
  request_id: UUID
  trace_id: UUID
  span_id: UUID
  user_id: UUID

  // Campi sorgente
  component: string
  function: string

  // Campi evento
  event: string
  data: Record<string, any>
}
```

### Contratto Metriche

```typescript
interface MetricContract {
  // Campi richiesti
  name: string
  type: MetricType
  value: number
  timestamp: number

  // Etichette
  labels: Record<string, string>

  // Metadata
  unit?: string
  help?: string
}
```

---

## Architettura di Implementazione

### Architettura Componenti

```typescript
class ObservabilitySystem {
  private tracer: Tracer
  private logger: Logger
  private metrics: MetricsCollector

  constructor(config: ObservabilityConfig) {
    this.tracer = new Tracer(config.tracing)
    this.logger = new Logger(config.logging)
    this.metrics = new MetricsCollector(config.metrics)
  }

  // Ottieni versioni instrumentate dei componenti
  instrument<T>(component: T): T {
    return createProxy(component, {
      beforeMethod: (name, args) => {
        const span = this.tracer.startSpan(name)
        this.logger.debug(`${name}_started`, { args })
        return { span, startTime: Date.now() }
      },

      afterMethod: (name, result, context) => {
        const duration = Date.now() - context.startTime
        this.tracer.endSpan(context.span)
        this.logger.debug(`${name}_completed`, { duration })
        this.metrics.recordHistogram(`${name}_duration`, duration)
      },

      onError: (name, error, context) => {
        context.span.recordException(error)
        this.logger.error(`${name}_failed`, { error })
        this.metrics.incrementCounter(`${name}_errors_total`, 1)
      }
    })
  }
}
```

### Punti di Integrazione

```typescript
// Reasoning Engine
class ReasoningEngine {
  constructor(
    private observability: ObservabilitySystem
  ) {}

  async execute(goal: Goal): Promise<Episode> {
    const tracer = this.observability.tracer
    const logger = this.observability.logger
    const metrics = this.observability.metrics

    const span = tracer.startSpan('reasoning_engine.execute')
    const startTime = Date.now()

    try {
      logger.info('execution_started', { goal_id: goal.id })

      const episode = await this.executeInternal(goal)

      const duration = Date.now() - startTime
      metrics.recordHistogram('execution_duration_seconds', duration / 1000)
      metrics.incrementCounter('executions_total', 1, { status: 'success' })

      logger.info('execution_completed', {
        goal_id: goal.id,
        duration_ms: duration,
        success: episode.outcome.success
      })

      span.setStatus(SpanStatus.OK)
      return episode

    } catch (error) {
      metrics.incrementCounter('executions_total', 1, { status: 'error' })
      logger.error('execution_failed', { goal_id: goal.id }, error)
      span.recordException(error)
      span.setStatus(SpanStatus.ERROR)
      throw error

    } finally {
      tracer.endSpan(span)
    }
  }
}
```

---

## Caratteristiche di Prestazione

### Analisi Overhead

| Operazione | Overhead | Impatto |
|-----------|----------|---------|
| Creazione span | ~10μs | Trascurabile |
| Attributo span | ~1μs | Trascurabile |
| Voce log | ~50μs | Basso |
| Registrazione metrica | ~5μs | Trascurabile |
| Export traccia | ~1ms | Basso (batch) |
| Export log | ~1ms | Basso (batch) |

**Overhead Totale**: <5% del tempo di esecuzione

### Strategie di Sampling

```typescript
interface SamplingConfig {
  // Campiona sempre
  alwaysSample: {
    errors: true,
    slowRequests: { threshold: 1000 },  // >1s
    expensiveRequests: { threshold: 1.0 }  // >$1
  }

  // Sampling probabilistico
  probabilistic: {
    successRate: 0.01,  // 1% delle richieste di successo
    fastRate: 0.001     // 0.1% delle richieste veloci
  }

  // Rate limiting
  rateLimit: {
    maxTracesPerSecond: 1000
  }

  // Sampling adattivo
  adaptive: {
    targetLoad: 1000,  // tracce/sec
    adjustInterval: 60000  // 1 minuto
  }
}
```

### Requisiti Storage

```typescript
interface StorageEstimates {
  traces: {
    avgTraceSize: '50 KB',
    retention: '30 giorni',
    dailyVolume: '10 GB',
    totalStorage: '300 GB'
  }

  logs: {
    avgLogSize: '500 bytes',
    retention: '30 giorni',
    dailyVolume: '50 GB',
    totalStorage: '1.5 TB'
  }

  metrics: {
    avgMetricSize: '100 bytes',
    retention: '1 anno',
    dailyVolume: '1 GB',
    totalStorage: '365 GB'
  }
}
```

---

## Conclusione

Il Sistema di Osservabilità e Tracciamento fornisce visibilità completa sul comportamento dell'agente mantenendo overhead basso. Caratteristiche chiave:

- **Tre Pilastri**: Log, metriche e tracce forniscono viste complementari
- **Tracciamento Distribuito**: Tracciamento richieste end-to-end attraverso i componenti
- **Dati Strutturati**: Tutta la telemetria è tipizzata e interrogabile
- **Capacità di Replay**: Replay deterministico di qualsiasi esecuzione
- **Overhead Basso**: <5% impatto prestazioni
- **Basato su Standard**: OpenTelemetry, Prometheus, W3C Trace Context

Questo sistema abilita debugging, ottimizzazione delle prestazioni, auditing per conformità e comprensione profonda del sistema - tutti essenziali per un agente universale di livello produttivo.

---

## Riferimenti

1. OpenTelemetry Specification (2024). Distributed tracing standard.
2. W3C Trace Context (2024). Context propagation standard.
3. Prometheus Documentation (2024). Metrics collection and querying.
4. Jaeger Documentation (2024). Distributed tracing backend.
5. Elastic Stack Documentation (2024). Log aggregation and search.
6. Grafana Documentation (2024). Metrics visualization.
7. Google SRE Book (2016). Observability and monitoring best practices.
