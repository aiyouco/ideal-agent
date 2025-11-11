# Specifiche delle Interfacce e Contratti API

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Interfacce Principali](#interfacce-principali)
3. [Specifica API REST](#specifica-api-rest)
4. [API WebSocket](#api-websocket)
5. [Interfacce Interne](#interfacce-interne)
6. [Contratti Dati](#contratti-dati)
7. [Risposte di Errore](#risposte-di-errore)

---

## Panoramica

Questo documento definisce tutte le interfacce pubbliche e interne per il sistema dell'agente ideale. Tutte le interfacce seguono questi principi:

1. **Versionate**: Le modifiche che rompono la compatibilità incrementano la versione principale
2. **Retrocompatibili**: I vecchi client continuano a funzionare
3. **Auto-descrittive**: Documentazione OpenAPI/JSON Schema
4. **Type-Safe**: Fortemente tipizzate con validazione
5. **Osservabili**: Tutte le chiamate tracciate e registrate

---

## Interfacce Principali

### Interfaccia ReasoningEngine

```typescript
/**
 * Interfaccia principale per il motore di ragionamento dell'agente
 */
interface ReasoningEngine {
  /**
   * Esegue un obiettivo entro un budget di risorse
   * @param goal - Obiettivo da raggiungere
   * @param budget - Vincoli di risorse
   * @param context - Contesto di esecuzione opzionale
   * @returns Episode contenente i risultati dell'esecuzione
   * @throws ResourceBudgetExceededError se il budget è esaurito
   * @throws SecurityError se il controllo di sicurezza fallisce
   */
  execute(
    goal: Goal,
    budget: ResourceBudget,
    context?: ExecutionContext
  ): Promise<Episode>

  /**
   * Ottiene lo stato corrente del ragionamento
   * @returns Snapshot dello stato corrente
   */
  getState(): ReasoningState

  /**
   * Mette in pausa l'esecuzione (può essere ripresa successivamente)
   */
  pause(): void

  /**
   * Riprende l'esecuzione in pausa
   */
  resume(): void

  /**
   * Interrompe l'esecuzione immediatamente
   * @param reason - Motivo dell'interruzione
   */
  abort(reason: string): void

  /**
   * Si iscrive ai cambiamenti di stato
   * @param event - Nome dell'evento
   * @param handler - Gestore dell'evento
   */
  on(event: ReasoningEvent, handler: EventHandler): void
}

enum ReasoningEvent {
  STATE_CHANGED = 'state_changed',
  GOAL_ADDED = 'goal_added',
  GOAL_COMPLETED = 'goal_completed',
  STEP_EXECUTED = 'step_executed',
  ERROR_OCCURRED = 'error_occurred',
  BUDGET_WARNING = 'budget_warning'
}

type EventHandler = (data: any) => void | Promise<void>
```

### Interfaccia MemorySystem

```typescript
/**
 * Interfaccia per i sistemi di memoria dell'agente
 */
interface MemorySystem {
  /** Memoria di lavoro (capacità limitata) */
  readonly workingMemory: WorkingMemory

  /** Memoria episodica (esperienze) */
  readonly episodicMemory: EpisodicMemory

  /** Memoria semantica (fatti e concetti) */
  readonly semanticMemory: SemanticMemory

  /** Memoria procedurale (abilità e pattern) */
  readonly proceduralMemory: ProceduralMemory

  /**
   * Recupero unificato attraverso tutti i tipi di memoria
   * @param cue - Indizio per il recupero della memoria
   * @returns Memorie rilevanti ordinate per attivazione
   */
  retrieve(cue: MemoryCue): Promise<Memory[]>

  /**
   * Memorizza una nuova memoria
   * @param memory - Memoria da memorizzare
   */
  store(memory: Memory): Promise<void>

  /**
   * Consolida le memorie episodiche in semantiche/procedurali
   * @returns Report delle attività di consolidamento
   */
  consolidate(): Promise<ConsolidationReport>

  /**
   * Rimuove le memorie a bassa attivazione
   * @param threshold - Soglia di attivazione per la rimozione
   * @returns ID delle memorie rimosse
   */
  forget(threshold: number): Promise<UUID[]>

  /**
   * Ottiene le statistiche della memoria
   * @returns Statistiche di utilizzo e prestazioni
   */
  getStats(): Promise<MemoryStats>
}
```

### Interfaccia ToolRegistry

```typescript
/**
 * Interfaccia per la gestione degli strumenti
 */
interface ToolRegistry {
  /**
   * Registra un nuovo strumento
   * @param tool - Definizione dello strumento
   * @throws ToolAlreadyExistsError se l'ID dello strumento esiste già
   * @throws InvalidToolError se la definizione dello strumento non è valida
   */
  register(tool: Tool): Promise<void>

  /**
   * Deregistra uno strumento
   * @param toolId - Identificatore dello strumento
   */
  unregister(toolId: UUID): Promise<void>

  /**
   * Ottiene uno strumento per ID
   * @param toolId - Identificatore dello strumento
   * @returns Strumento o null se non trovato
   */
  getTool(toolId: UUID): Promise<Tool | null>

  /**
   * Cerca strumenti per capacità
   * @param capabilities - Capacità richieste
   * @returns Strumenti corrispondenti alle capacità
   */
  searchByCapability(capabilities: Capability[]): Promise<Tool[]>

  /**
   * Cerca strumenti per query semantica
   * @param query - Query in linguaggio naturale
   * @param limit - Numero massimo di risultati
   * @returns Strumenti rilevanti ordinati per rilevanza
   */
  search(query: string, limit: number): Promise<Tool[]>

  /**
   * Esegue uno strumento
   * @param toolId - Strumento da eseguire
   * @param parameters - Parametri vincolati
   * @param context - Contesto di esecuzione
   * @returns Risultato dell'esecuzione dello strumento
   */
  execute(
    toolId: UUID,
    parameters: BoundParameters,
    context: ExecutionContext
  ): Promise<ToolResult>
}
```

---

## Specifica API REST

### URL di Base

```
Production: https://api.agent.example.com/v1
Staging: https://staging-api.agent.example.com/v1
Development: http://localhost:3000/v1
```

### Autenticazione

Tutte le richieste richiedono token Bearer:

```http
Authorization: Bearer <JWT_TOKEN>
```

### Header Comuni

```http
Content-Type: application/json
Accept: application/json
X-Request-ID: <UUID>
X-Client-Version: <VERSION>
```

---

### Endpoint

#### POST /tasks

Esegue un'attività.

**Richiesta**:
```json
{
  "goal": {
    "description": "Create a Python function to calculate fibonacci",
    "type": "achievement",
    "successCriteria": [
      {
        "type": "test_passes",
        "description": "All unit tests pass"
      }
    ],
    "constraints": [
      {
        "type": "resource",
        "maxCost": 1.0
      }
    ]
  },
  "budget": {
    "maxTokens": 100000,
    "maxLLMCalls": 50,
    "maxWallTime": 300000,
    "maxCost": 5.0,
    "maxDepth": 5
  },
  "context": {
    "files": [],
    "environment": {},
    "preferences": {}
  }
}
```

**Risposta** (200 OK):
```json
{
  "taskId": "task_abc123",
  "status": "completed",
  "episode": {
    "id": "episode_xyz789",
    "goal": { ... },
    "outcome": {
      "success": true,
      "type": "success",
      "result": { ... }
    },
    "resourcesUsed": {
      "tokens": 45000,
      "llmCalls": 23,
      "wallTime": 145000,
      "cost": 2.34
    },
    "duration": 145000,
    "trace": {
      "traceId": "trace_123",
      "spans": [ ... ]
    }
  }
}
```

**Risposta** (4xx/5xx Error):
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Goal description is required",
    "details": {
      "field": "goal.description",
      "reason": "missing_required_field"
    },
    "requestId": "req_456",
    "timestamp": 1699564800000
  }
}
```

---

#### GET /tasks/{taskId}

Ottiene lo stato e i risultati dell'attività.

**Risposta** (200 OK):
```json
{
  "taskId": "task_abc123",
  "status": "completed",
  "createdAt": 1699564800000,
  "completedAt": 1699564945000,
  "episode": { ... }
}
```

**Stati**: `pending`, `executing`, `completed`, `failed`, `cancelled`

---

#### POST /tasks/{taskId}/cancel

Annulla l'attività in esecuzione.

**Risposta** (200 OK):
```json
{
  "taskId": "task_abc123",
  "status": "cancelled",
  "partialResult": { ... }
}
```

---

#### GET /memory/retrieve

Recupera memorie.

**Parametri Query**:
```
query: string (richiesto)
type: episodic|semantic|procedural (opzionale)
limit: number (default: 10, max: 100)
minActivation: number (0-1, opzionale)
```

**Risposta** (200 OK):
```json
{
  "memories": [
    {
      "id": "mem_123",
      "type": "episodic",
      "content": { ... },
      "activation": 0.85,
      "similarity": 0.92,
      "timestamp": 1699564800000
    }
  ],
  "retrievalTime": 45,
  "cacheHit": false
}
```

---

#### POST /memory/store

Memorizza una nuova memoria.

**Richiesta**:
```json
{
  "type": "episodic",
  "content": {
    "episode": { ... }
  },
  "tags": ["coding", "python"],
  "metadata": {}
}
```

**Risposta** (201 Created):
```json
{
  "memoryId": "mem_789",
  "stored": true,
  "activation": 1.0
}
```

---

#### GET /tools

Elenca gli strumenti disponibili.

**Parametri Query**:
```
category: string (opzionale)
capability: string (opzionale)
limit: number (default: 50)
```

**Risposta** (200 OK):
```json
{
  "tools": [
    {
      "id": "tool_123",
      "name": "file_reader",
      "description": "Read file contents",
      "category": "file_system",
      "capabilities": [
        {
          "type": "read",
          "description": "Read text files"
        }
      ],
      "inputSchema": { ... },
      "outputSchema": { ... }
    }
  ],
  "total": 42
}
```

---

#### POST /tools/{toolId}/execute

Esegue uno strumento.

**Richiesta**:
```json
{
  "parameters": {
    "path": "/path/to/file.txt"
  },
  "timeout": 30000,
  "retryPolicy": {
    "maxAttempts": 3,
    "backoff": "exponential"
  }
}
```

**Risposta** (200 OK):
```json
{
  "executionId": "exec_456",
  "success": true,
  "output": {
    "content": "File contents here..."
  },
  "duration": 125,
  "resourcesUsed": {
    "cpu": 0.1,
    "memory": 1048576
  }
}
```

---

#### GET /traces/{traceId}

Ottiene la traccia di esecuzione.

**Risposta** (200 OK):
```json
{
  "traceId": "trace_123",
  "startTime": 1699564800000,
  "endTime": 1699564945000,
  "duration": 145000,
  "status": "ok",
  "spans": [
    {
      "spanId": "span_1",
      "name": "goal_decomposition",
      "startTime": 1699564800100,
      "endTime": 1699564802300,
      "duration": 2200,
      "attributes": {
        "subgoals_count": 3
      }
    }
  ]
}
```

---

#### GET /health

Endpoint di controllo dello stato.

**Risposta** (200 OK):
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 3600000,
  "components": {
    "reasoning": "healthy",
    "memory": "healthy",
    "tools": "healthy",
    "llm": "healthy"
  },
  "timestamp": 1699564800000
}
```

---

## API WebSocket

### Connessione

```
wss://api.agent.example.com/v1/stream
```

### Autenticazione

```json
{
  "type": "auth",
  "token": "Bearer <JWT_TOKEN>"
}
```

### Esecuzione Attività (Streaming)

**Client → Server**:
```json
{
  "type": "execute",
  "requestId": "req_123",
  "goal": { ... },
  "budget": { ... }
}
```

**Server → Client** (Aggiornamenti di Progresso):
```json
{
  "type": "progress",
  "requestId": "req_123",
  "status": "executing",
  "progress": {
    "stepsCompleted": 2,
    "stepsTotal": 5,
    "percentComplete": 40
  },
  "currentStep": "Generating code..."
}
```

**Server → Client** (Output in Streaming):
```json
{
  "type": "output",
  "requestId": "req_123",
  "chunk": "def fibonacci(n):\n",
  "sequence": 1,
  "isLast": false
}
```

**Server → Client** (Completamento):
```json
{
  "type": "complete",
  "requestId": "req_123",
  "episode": { ... },
  "trace": { ... }
}
```

**Server → Client** (Errore):
```json
{
  "type": "error",
  "requestId": "req_123",
  "error": {
    "code": "EXECUTION_FAILED",
    "message": "Tool execution timeout",
    "retriable": true
  }
}
```

---

## Interfacce Interne

### Interfaccia LanguageModel

```typescript
/**
 * Interfaccia per i provider di modelli linguistici
 */
interface LanguageModel {
  /**
   * Genera un completamento
   * @param prompt - Prompt di input
   * @param options - Opzioni di generazione
   * @returns Testo generato
   */
  generate(
    prompt: string,
    options?: GenerationOptions
  ): Promise<string>

  /**
   * Genera con streaming
   * @param prompt - Prompt di input
   * @param onToken - Callback per ogni token
   * @param options - Opzioni di generazione
   * @returns Testo generato finale
   */
  generateStream(
    prompt: string,
    onToken: (token: string) => void,
    options?: GenerationOptions
  ): Promise<string>

  /**
   * Incorpora testo
   * @param text - Testo da incorporare
   * @returns Vettore di embedding
   */
  embed(text: string): Promise<number[]>

  /**
   * Conta i token nel testo
   * @param text - Testo da tokenizzare
   * @returns Conteggio dei token
   */
  countTokens(text: string): Promise<number>
}

interface GenerationOptions {
  temperature?: number      // 0-2, default 0.7
  maxTokens?: number       // Max tokens da generare
  topP?: number            // 0-1, nucleus sampling
  frequencyPenalty?: number // -2 a 2
  presencePenalty?: number  // -2 a 2
  stop?: string[]          // Sequenze di stop
  seed?: number            // Per riproducibilità
  responseFormat?: 'text' | 'json'
}
```

### Interfaccia WorkingMemory

```typescript
/**
 * Interfaccia per la memoria di lavoro (finestra di contesto)
 */
interface WorkingMemory {
  /** Capacità massima in token */
  readonly maxTokens: number

  /** Utilizzo corrente dei token */
  readonly currentTokens: number

  /**
   * Aggiunge contenuto alla memoria di lavoro
   * @param chunk - Contenuto da aggiungere
   * @throws CapacityExceededError se troppo grande
   */
  add(chunk: Chunk): void

  /**
   * Rimuove contenuto dalla memoria di lavoro
   * @param chunkId - ID del chunk da rimuovere
   */
  remove(chunkId: UUID): void

  /**
   * Cancella tutto il contenuto
   */
  clear(): void

  /**
   * Ottiene tutti i contenuti
   * @returns Tutti i chunk nella memoria di lavoro
   */
  getContents(): Chunk[]

  /**
   * Stima i token per il contenuto
   * @param content - Contenuto da stimare
   * @returns Conteggio stimato dei token
   */
  estimateTokens(content: any): number
}

interface Chunk {
  id: UUID
  type: ChunkType
  content: any
  tokens: number
  activation: number
  protected: boolean
}

enum ChunkType {
  SYSTEM = 'system',
  GOAL = 'goal',
  OBSERVATION = 'observation',
  REASONING = 'reasoning',
  CONTEXT = 'context',
  SCRATCHPAD = 'scratchpad'
}
```

### Interfaccia ToolExecutor

```typescript
/**
 * Interfaccia per l'esecuzione degli strumenti
 */
interface ToolExecutor {
  /**
   * Inizializza l'esecutore
   */
  initialize(): Promise<void>

  /**
   * Esegue lo strumento
   * @param parameters - Parametri validati
   * @param environment - Ambiente di esecuzione
   * @returns Output dello strumento
   * @throws ToolExecutionError in caso di fallimento
   */
  execute(
    parameters: Record<string, any>,
    environment: ExecutionEnvironment
  ): Promise<any>

  /**
   * Pulisce le risorse
   */
  cleanup(): Promise<void>

  /**
   * Controlla se lo strumento è pronto
   * @returns true se pronto per l'esecuzione
   */
  isReady(): Promise<boolean>

  /**
   * Ottiene le capacità dell'esecutore
   * @returns Lista delle capacità
   */
  getCapabilities(): Capability[]
}
```

---

## Contratti Dati

### Contratto Goal

```typescript
/**
 * Struttura dati Goal
 */
interface Goal {
  /** Identificatore univoco */
  id: UUID

  /** Descrizione dell'obiettivo (linguaggio naturale) */
  description: string

  /** Tipo di obiettivo */
  type: GoalType

  /** Obiettivo padre (se sottobiettivo) */
  parent: UUID | null

  /** Obiettivi figlio */
  children: UUID[]

  /** Stato corrente */
  status: GoalStatus

  /** Criteri di successo */
  successCriteria: Criterion[]

  /** Vincoli */
  constraints: Constraint[]

  /** Metadati */
  createdAt: number
  completedAt: number | null
  priority?: number
  tags?: string[]
}

enum GoalType {
  ACHIEVEMENT = 'achievement',
  MAINTENANCE = 'maintenance',
  AVOIDANCE = 'avoidance',
  EXPLORATION = 'exploration'
}

enum GoalStatus {
  PENDING = 'pending',
  ACTIVE = 'active',
  BLOCKED = 'blocked',
  COMPLETED = 'completed',
  FAILED = 'failed',
  ABANDONED = 'abandoned'
}

interface Criterion {
  type: CriterionType
  description: string
  value?: any
}

enum CriterionType {
  STATE_PROPERTY = 'state_property',
  OBSERVABLE = 'observable',
  USER_CONFIRMATION = 'user_confirmation',
  TEST_PASSES = 'test_passes',
  INVARIANT = 'invariant'
}
```

### Contratto Episode

```typescript
/**
 * Episode (registro completo dell'esecuzione)
 */
interface Episode {
  /** Identificatore univoco */
  id: UUID

  /** Obiettivo perseguito */
  goal: Goal

  /** Piano generato */
  plan: Plan

  /** Risultati dell'esecuzione */
  execution: ExecutionResult

  /** Risultato finale */
  outcome: Outcome

  /** Durata in millisecondi */
  duration: number

  /** Risorse consumate */
  resourcesUsed: ResourceUsage

  /** Traccia di esecuzione */
  trace: ExecutionTrace

  /** Riflessione (se eseguita) */
  reflection: Reflection | null

  /** Metadati */
  timestamp: number
  tags?: string[]
  embedding?: number[]
}

interface Outcome {
  success: boolean
  type: OutcomeType
  result?: any
  error?: Error
  confidence?: number
}

enum OutcomeType {
  SUCCESS = 'success',
  PARTIAL_SUCCESS = 'partial_success',
  FAILURE = 'failure',
  TIMEOUT = 'timeout',
  BUDGET_EXCEEDED = 'budget_exceeded',
  CANCELLED = 'cancelled'
}
```

### Contratto Plan

```typescript
/**
 * Struttura dati Plan
 */
interface Plan {
  /** Identificatore univoco */
  id: UUID

  /** Obiettivo associato */
  goalId: UUID

  /** Passi del piano */
  steps: PlanStep[]

  /** Dipendenze tra i passi */
  dependencies: Dependency[]

  /** Utilizzo stimato delle risorse */
  estimatedCost: ResourceEstimate

  /** Piani di contingenza */
  contingencies: ContingencyPlan[]

  /** Strategia utilizzata */
  strategy: Strategy

  /** Metadati */
  createdAt: number
  version?: number
}

interface PlanStep {
  id: UUID
  name: string
  type: StepType
  action: Action
  preconditions: Condition[]
  postconditions: Condition[]
  expectedDuration: number
  dependencies: UUID[]
  alternatives: PlanStep[]
}

enum StepType {
  REASONING = 'reasoning',
  TOOL_USE = 'tool_use',
  SUBGOAL = 'subgoal',
  DECISION = 'decision',
  VALIDATION = 'validation',
  WAIT = 'wait'
}

interface Dependency {
  from: UUID  // Passo che deve completarsi per primo
  to: UUID    // Passo che dipende da esso
  type: DependencyType
}

enum DependencyType {
  REQUIRED = 'required',    // Deve completarsi prima
  OPTIONAL = 'optional',    // Preferito ma non richiesto
  DATA = 'data'            // I dati fluiscono da → a
}
```

### Contratto Tool

```typescript
/**
 * Definizione dello strumento
 */
interface Tool {
  /** Identificatore univoco */
  id: UUID

  /** Nome dello strumento */
  name: string

  /** Versione (semantic versioning) */
  version: string

  /** Descrizione */
  description: string

  /** Categoria */
  category: ToolCategory

  /** Schema di input (JSON Schema) */
  inputSchema: JSONSchema

  /** Schema di output (JSON Schema) */
  outputSchema: JSONSchema

  /** Capacità fornite */
  capabilities: Capability[]

  /** Requisiti necessari */
  requirements: Requirement[]

  /** Proprietà dello strumento */
  properties: ToolProperties

  /** Implementazione dell'esecutore */
  executor: ToolExecutor

  /** Documentazione */
  documentation: string
  examples: ToolExample[]

  /** Caratteristiche di performance */
  estimatedLatency: number
  estimatedCost: number
  reliability: number
}

interface ToolProperties {
  idempotent: boolean
  deterministic: boolean
  sideEffects: boolean
  stateful: boolean
  async: boolean
  streaming: boolean
  cacheable: boolean
  retriable: boolean
  timeout: number
  rateLimit: RateLimit
}

interface ToolExample {
  description: string
  input: Record<string, any>
  output: any
  explanation?: string
}
```

### Contratto Memory

```typescript
/**
 * Interfaccia base della memoria
 */
interface Memory {
  /** Identificatore univoco */
  id: UUID

  /** Tipo di memoria */
  type: MemoryType

  /** Contenuto (specifico del tipo) */
  content: any

  /** Livello di attivazione (0-1) */
  activation: number

  /** Statistiche di accesso */
  accessCount: number
  lastAccess: number

  /** Timestamp di creazione */
  timestamp: number

  /** Tag per la categorizzazione */
  tags?: string[]

  /** Embedding per similarità */
  embedding?: number[]
}

enum MemoryType {
  EPISODIC = 'episodic',
  SEMANTIC = 'semantic',
  PROCEDURAL = 'procedural'
}
```

---

## Risposte di Errore

### Struttura Standard degli Errori

```typescript
interface ErrorResponse {
  error: {
    /** Codice errore (leggibile dalla macchina) */
    code: ErrorCode

    /** Messaggio di errore (leggibile dall'umano) */
    message: string

    /** Dettagli aggiuntivi */
    details?: Record<string, any>

    /** ID richiesta per il tracciamento */
    requestId: UUID

    /** Timestamp */
    timestamp: number

    /** Stack trace (solo in sviluppo) */
    stack?: string
  }
}

enum ErrorCode {
  // Errori client (4xx)
  INVALID_INPUT = 'INVALID_INPUT',
  UNAUTHORIZED = 'UNAUTHORIZED',
  FORBIDDEN = 'FORBIDDEN',
  NOT_FOUND = 'NOT_FOUND',
  CONFLICT = 'CONFLICT',
  RATE_LIMITED = 'RATE_LIMITED',

  // Errori server (5xx)
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
  TIMEOUT = 'TIMEOUT',
  BUDGET_EXCEEDED = 'BUDGET_EXCEEDED',

  // Errori specifici dell'agente
  EXECUTION_FAILED = 'EXECUTION_FAILED',
  PLANNING_FAILED = 'PLANNING_FAILED',
  TOOL_UNAVAILABLE = 'TOOL_UNAVAILABLE',
  MEMORY_ERROR = 'MEMORY_ERROR'
}
```

### Codici di Stato HTTP

| Codice | Significato | Caso d'Uso |
|--------|-------------|------------|
| 200 | OK | Richiesta riuscita |
| 201 | Created | Risorsa creata |
| 202 | Accepted | Attività asincrona accettata |
| 400 | Bad Request | Input non valido |
| 401 | Unauthorized | Autenticazione non valida/mancante |
| 403 | Forbidden | Permessi insufficienti |
| 404 | Not Found | La risorsa non esiste |
| 409 | Conflict | Conflitto di risorse |
| 429 | Too Many Requests | Rate limit raggiunto |
| 500 | Internal Error | Errore del server |
| 503 | Service Unavailable | Temporaneamente non disponibile |
| 504 | Gateway Timeout | Timeout della richiesta |

---

## Specifica OpenAPI

### Schema API Completo

```yaml
openapi: 3.1.0
info:
  title: Ideal Agent API
  version: 1.0.0
  description: REST API for universal agent system

servers:
  - url: https://api.agent.example.com/v1
    description: Production
  - url: https://staging-api.agent.example.com/v1
    description: Staging

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    Goal:
      type: object
      required: [description, type]
      properties:
        id:
          type: string
          format: uuid
        description:
          type: string
          minLength: 1
          maxLength: 10000
        type:
          type: string
          enum: [achievement, maintenance, avoidance, exploration]
        successCriteria:
          type: array
          items:
            $ref: '#/components/schemas/Criterion'
        constraints:
          type: array
          items:
            $ref: '#/components/schemas/Constraint'

    ResourceBudget:
      type: object
      properties:
        maxTokens:
          type: integer
          minimum: 1000
          maximum: 1000000
        maxLLMCalls:
          type: integer
          minimum: 1
          maximum: 1000
        maxWallTime:
          type: integer
          description: Milliseconds
          minimum: 1000
          maximum: 3600000
        maxCost:
          type: number
          minimum: 0.01
          maximum: 1000.00
        maxDepth:
          type: integer
          minimum: 1
          maximum: 10

    Episode:
      type: object
      required: [id, goal, outcome, duration, resourcesUsed]
      properties:
        id:
          type: string
          format: uuid
        goal:
          $ref: '#/components/schemas/Goal'
        plan:
          $ref: '#/components/schemas/Plan'
        outcome:
          $ref: '#/components/schemas/Outcome'
        duration:
          type: integer
          description: Duration in milliseconds
        resourcesUsed:
          $ref: '#/components/schemas/ResourceUsage'
        trace:
          $ref: '#/components/schemas/ExecutionTrace'

paths:
  /tasks:
    post:
      summary: Execute a task
      operationId: executeTask
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [goal, budget]
              properties:
                goal:
                  $ref: '#/components/schemas/Goal'
                budget:
                  $ref: '#/components/schemas/ResourceBudget'
                context:
                  type: object
      responses:
        200:
          description: Task completed successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  taskId:
                    type: string
                    format: uuid
                  status:
                    type: string
                    enum: [completed, failed]
                  episode:
                    $ref: '#/components/schemas/Episode'
        400:
          description: Invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        401:
          description: Unauthorized
        429:
          description: Rate limited
        500:
          description: Internal error

  /tasks/{taskId}:
    get:
      summary: Get task status
      operationId: getTaskStatus
      security:
        - BearerAuth: []
      parameters:
        - name: taskId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        200:
          description: Task found
          content:
            application/json:
              schema:
                type: object
                properties:
                  taskId:
                    type: string
                  status:
                    type: string
                  episode:
                    $ref: '#/components/schemas/Episode'
        404:
          description: Task not found
```

---

## Strategia di Versionamento

### Versionamento API

**Versionamento URL**: `/v1/`, `/v2/`, ecc.

**Compatibilità delle Versioni**:
- **Major**: Modifiche che rompono la compatibilità (incremento nell'URL)
- **Minor**: Aggiunte retrocompatibili (nessun cambio URL)
- **Patch**: Correzioni di bug (nessun cambio URL)

**Politica di Deprecazione**:
1. Annunciare la deprecazione con 6 mesi di preavviso
2. Marcare gli endpoint deprecati con header: `Deprecation: true`
3. Fornire guida alla migrazione
4. Rimuovere solo dopo il periodo di preavviso

### Versionamento delle Interfacce

```typescript
interface VersionedInterface {
  /** Versione dell'interfaccia (semver) */
  version: string

  /** Deprecata? */
  deprecated?: {
    since: string
    removeIn: string
    migration: string
  }
}

// Esempio
const ReasoningEngineV1: VersionedInterface = {
  version: '1.0.0',
  deprecated: {
    since: '1.5.0',
    removeIn: '2.0.0',
    migration: 'See migration guide at /docs/v2-migration'
  }
}
```

---

## Protocol Buffers (Alternativa)

Per comunicazioni interne ad alte prestazioni:

```protobuf
syntax = "proto3";

package agent.v1;

// Goal message
message Goal {
  string id = 1;
  string description = 2;
  GoalType type = 3;
  repeated Criterion success_criteria = 4;
  repeated Constraint constraints = 5;
  int64 created_at = 6;
}

enum GoalType {
  GOAL_TYPE_UNSPECIFIED = 0;
  ACHIEVEMENT = 1;
  MAINTENANCE = 2;
  AVOIDANCE = 3;
  EXPLORATION = 4;
}

// gRPC service
service AgentService {
  rpc Execute(ExecuteRequest) returns (ExecuteResponse);
  rpc GetStatus(GetStatusRequest) returns (GetStatusResponse);
  rpc Cancel(CancelRequest) returns (CancelResponse);
  rpc StreamExecute(StreamExecuteRequest) returns (stream ExecuteProgress);
}

message ExecuteRequest {
  Goal goal = 1;
  ResourceBudget budget = 2;
  map<string, string> context = 3;
}

message ExecuteResponse {
  string task_id = 1;
  Episode episode = 2;
}
```

---

## Conclusione

Questo documento fornisce specifiche complete delle interfacce tra cui:

1. **Interfacce Principali**: Definizioni TypeScript per tutti i componenti principali
2. **API REST**: Endpoint HTTP con formati di richiesta/risposta
3. **API WebSocket**: Protocollo di comunicazione in streaming
4. **Interfacce Interne**: Contratti componente-a-componente
5. **Contratti Dati**: Definizioni precise delle strutture
6. **Gestione Errori**: Risposte di errore standardizzate
7. **Versionamento**: Strategia di evoluzione dell'API

Tutte le interfacce sono:
- ✅ Fortemente tipizzate
- ✅ Auto-documentanti
- ✅ Versionate
- ✅ Retrocompatibili (all'interno della versione principale)
- ✅ Osservabili (tutte le chiamate tracciate)

Queste specifiche abilitano:
- Sviluppo indipendente frontend/backend
- Integrazioni di terze parti
- Generazione di librerie client
- Testing e validazione API
- Testing basato su contratti

---

## Riferimenti

1. OpenAPI Specification 3.1.0. https://spec.openapis.org/oas/latest.html
2. JSON Schema 2020-12. https://json-schema.org/
3. Protocol Buffers. https://protobuf.dev/
4. gRPC Documentation. https://grpc.io/docs/
5. Semantic Versioning 2.0.0. https://semver.org/
