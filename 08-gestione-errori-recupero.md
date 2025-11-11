# Sistema di Gestione Errori e Recupero

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Classificazione Errori](#classificazione-errori)
3. [Strategie di Retry](#strategie-di-retry)
4. [Backoff Esponenziale](#backoff-esponenziale)
5. [Modelli di Fallback](#modelli-di-fallback)
6. [Circuit Breaker](#circuit-breaker)
7. [Degradazione Graduale](#degradazione-graduale)
8. [Pattern di Recupero Errori](#pattern-di-recupero-errori)
9. [Rilevamento Fallimenti](#rilevamento-fallimenti)
10. [Alberi Decisionali di Recupero](#alberi-decisionali-di-recupero)
11. [Specifiche delle Interfacce](#specifiche-delle-interfacce)

---

## Panoramica

Il Sistema di Gestione Errori e Recupero garantisce che l'agente rimanga operativo e faccia progressi anche in presenza di fallimenti. Fornisce classificazione sistematica degli errori, logica di retry intelligente e strategie di degradazione graduale.

### Obiettivi di Design

1. **Resilienza**: Continuare a operare nonostante i fallimenti
2. **Trasparenza**: Comunicazione chiara degli errori
3. **Efficienza**: Evitare sprechi di risorse su retry futili
4. **Apprendimento**: Migliorare dai fallimenti passati
5. **Sicurezza**: Non peggiorare mai la situazione durante il recupero
6. **Prevedibilità**: Comportamento di recupero deterministico

### Principi di Gestione Errori

```
Principio 1: Fallire Velocemente, Recuperare Intelligentemente
- Rilevare i fallimenti rapidamente
- Classificare gli errori accuratamente
- Ritentare intelligentemente
- Fallback con grazia

Principio 2: Prevenire Fallimenti a Cascata
- Isolare i fallimenti
- Usare circuit breaker
- Implementare timeout
- Limitare i tentativi di retry

Principio 3: Preservare il Progresso
- Checkpoint frequenti
- Abilitare rollback
- Riprendere dal checkpoint
- Non perdere il lavoro completato

Principio 4: Comunicare Chiaramente
- Tipi di errore strutturati
- Contesto ricco degli errori
- Messaggi user-friendly
- Suggerimenti azionabili
```

---

## Classificazione Errori

### Tassonomia degli Errori

```typescript
enum ErrorCategory {
  // Errori ritentabili (temporanei, riprovare)
  NETWORK = 'network',              // Problemi di connettività di rete
  TIMEOUT = 'timeout',              // Operazione ha superato il limite di tempo
  RATE_LIMIT = 'rate_limit',        // Limite di rate API raggiunto
  RESOURCE_EXHAUSTED = 'resource_exhausted',  // Risorse esaurite temporaneamente
  SERVICE_UNAVAILABLE = 'service_unavailable',  // Servizio temporaneamente non disponibile

  // Errori non ritentabili (permanenti, non ritentare)
  INVALID_INPUT = 'invalid_input',  // Dati di richiesta errati
  UNAUTHORIZED = 'unauthorized',    // Autenticazione fallita
  FORBIDDEN = 'forbidden',          // Permesso negato
  NOT_FOUND = 'not_found',         // Risorsa non esiste
  INVALID_STATE = 'invalid_state',  // Sistema in stato errato

  // Errori sconosciuti (investigare prima di ritentare)
  UNKNOWN = 'unknown',
  INTERNAL =  'internal',           // Errore interno del sistema
}

interface ClassifiedError {
  // Errore originale
  error: Error

  // Classificazione
  category: ErrorCategory
  retriable: boolean
  severity: ErrorSeverity

  // Guida al recupero
  suggestedAction: RecoveryAction
  retryAfter?: number  // millisecondi
  maxRetries?: number

  // Contesto
  component: string
  operation: string
  context: Record<string, any>

  // Tracing
  traceId: UUID
  spanId: UUID
}

enum ErrorSeverity {
  LOW = 'low',          // Problema minore, può continuare
  MEDIUM = 'medium',    // Problema significativo, può degradare
  HIGH = 'high',        // Problema maggiore, percorso critico affetto
  CRITICAL = 'critical' // Fallimento a livello sistema
}

enum RecoveryAction {
  RETRY = 'retry',                    // Riprovare l'operazione
  RETRY_WITH_BACKOFF = 'retry_with_backoff',  // Retry con ritardo
  FALLBACK = 'fallback',              // Usare approccio alternativo
  SKIP = 'skip',                      // Saltare questa operazione
  ABORT = 'abort',                    // Rinunciare al task
  ESCALATE = 'escalate',              // Richiedere intervento umano
  CIRCUIT_BREAK = 'circuit_break'     // Interrompere chiamate al servizio fallito
}
```

### Classificatore di Errori

```typescript
class ErrorClassifier {
  classify(error: Error, context: OperationContext): ClassifiedError {
    // Controlla tipi di errore noti
    if (error instanceof NetworkError) {
      return this.classifyNetworkError(error, context)
    }

    if (error instanceof TimeoutError) {
      return this.classifyTimeoutError(error, context)
    }

    if (error instanceof ValidationError) {
      return this.classifyValidationError(error, context)
    }

    if (error instanceof PermissionError) {
      return this.classifyPermissionError(error, context)
    }

    // Controlla pattern di errore
    const pattern = this.matchErrorPattern(error)
    if (pattern) {
      return this.classifyByPattern(error, pattern, context)
    }

    // Default a sconosciuto
    return {
      error,
      category: ErrorCategory.UNKNOWN,
      retriable: false,  // Default conservativo
      severity: ErrorSeverity.MEDIUM,
      suggestedAction: RecoveryAction.ESCALATE,
      component: context.component,
      operation: context.operation,
      context: context.data,
      traceId: context.traceId,
      spanId: context.spanId
    }
  }

  private classifyNetworkError(
    error: NetworkError,
    context: OperationContext
  ): ClassifiedError {
    // Errori di rete sono solitamente ritentabili
    return {
      error,
      category: ErrorCategory.NETWORK,
      retriable: true,
      severity: ErrorSeverity.MEDIUM,
      suggestedAction: RecoveryAction.RETRY_WITH_BACKOFF,
      retryAfter: 1000,  // 1 secondo
      maxRetries: 3,
      component: context.component,
      operation: context.operation,
      context: { ...context.data, errorCode: error.code },
      traceId: context.traceId,
      spanId: context.spanId
    }
  }

  private classifyTimeoutError(
    error: TimeoutError,
    context: OperationContext
  ): ClassifiedError {
    // I timeout potrebbero avere successo con timeout più lungo o retry
    return {
      error,
      category: ErrorCategory.TIMEOUT,
      retriable: true,
      severity: ErrorSeverity.MEDIUM,
      suggestedAction: RecoveryAction.RETRY_WITH_BACKOFF,
      retryAfter: 2000,  // 2 secondi
      maxRetries: 2,
      component: context.component,
      operation: context.operation,
      context: {
        ...context.data,
        timeoutMs: error.timeout,
        elapsedMs: error.elapsed
      },
      traceId: context.traceId,
      spanId: context.spanId
    }
  }

  private classifyValidationError(
    error: ValidationError,
    context: OperationContext
  ): ClassifiedError {
    // Errori di validazione sono permanenti (input errato)
    return {
      error,
      category: ErrorCategory.INVALID_INPUT,
      retriable: false,
      severity: ErrorSeverity.HIGH,
      suggestedAction: RecoveryAction.ABORT,
      component: context.component,
      operation: context.operation,
      context: {
        ...context.data,
        validationErrors: error.errors
      },
      traceId: context.traceId,
      spanId: context.spanId
    }
  }
}
```

---

## Strategie di Retry

### Politica di Retry

```typescript
interface RetryPolicy {
  // Tentativi massimi
  maxAttempts: number

  // Strategia di backoff
  backoff: BackoffStrategy

  // Condizioni
  retryIf: (error: ClassifiedError) => boolean

  // Timeout per tentativo
  timeoutPerAttempt?: number

  // Timeout complessivo
  overallTimeout?: number

  // Callback
  onRetry?: (attempt: number, error: ClassifiedError) => void
  onMaxAttempts?: (error: ClassifiedError) => void
}

enum BackoffStrategy {
  CONSTANT = 'constant',        // Stesso ritardo ogni volta
  LINEAR = 'linear',            // Aumento lineare
  EXPONENTIAL = 'exponential',  // Crescita esponenziale
  FIBONACCI = 'fibonacci',      // Sequenza di Fibonacci
  JITTERED = 'jittered'        // Jitter casuale aggiunto
}

// Politiche di esempio
const RETRY_POLICIES = {
  // Retry veloce per errori transitori
  fast: {
    maxAttempts: 3,
    backoff: BackoffStrategy.CONSTANT,
    constantDelay: 100,
    retryIf: (e) => e.category === ErrorCategory.NETWORK
  },

  // Retry standard con backoff
  standard: {
    maxAttempts: 5,
    backoff: BackoffStrategy.EXPONENTIAL,
    initialDelay: 1000,
    maxDelay: 30000,
    retryIf: (e) => e.retriable
  },

  // Retry aggressivo per operazioni critiche
  aggressive: {
    maxAttempts: 10,
    backoff: BackoffStrategy.JITTERED,
    initialDelay: 500,
    maxDelay: 60000,
    overallTimeout: 300000,  // 5 minuti
    retryIf: (e) => e.retriable && e.severity >= ErrorSeverity.HIGH
  },

  // Retry conservativo (minimizzare spreco di risorse)
  conservative: {
    maxAttempts: 2,
    backoff: BackoffStrategy.LINEAR,
    initialDelay: 2000,
    retryIf: (e) =>
      e.retriable &&
      e.category !== ErrorCategory.RATE_LIMIT &&
      e.severity <= ErrorSeverity.MEDIUM
  }
}
```

### Esecutore Retry

```typescript
class RetryExecutor {
  async execute<T>(
    operation: () => Promise<T>,
    policy: RetryPolicy,
    context: OperationContext
  ): Promise<T> {

    let attempt = 0
    let lastError: ClassifiedError

    while (attempt < policy.maxAttempts) {
      attempt++

      try {
        // Imposta timeout per questo tentativo
        if (policy.timeoutPerAttempt) {
          return await withTimeout(
            operation(),
            policy.timeoutPerAttempt
          )
        }

        return await operation()

      } catch (error) {
        // Classifica errore
        lastError = this.classifier.classify(error, context)

        // Controlla se dovrebbe ritentare
        if (!policy.retryIf(lastError)) {
          throw lastError
        }

        // Controlla se fuori tentativi
        if (attempt >= policy.maxAttempts) {
          if (policy.onMaxAttempts) {
            policy.onMaxAttempts(lastError)
          }
          throw lastError
        }

        // Calcola ritardo di backoff
        const delay = this.calculateDelay(
          policy.backoff,
          attempt,
          policy
        )

        // Notifica retry
        if (policy.onRetry) {
          policy.onRetry(attempt, lastError)
        }

        // Log retry
        logger.warn('operation_retry', {
          operation: context.operation,
          attempt,
          maxAttempts: policy.maxAttempts,
          delayMs: delay,
          error: lastError.category
        })

        // Attendi prima di ritentare
        await sleep(delay)
      }
    }

    throw lastError!
  }

  private calculateDelay(
    strategy: BackoffStrategy,
    attempt: number,
    policy: RetryPolicy
  ): number {
    let delay: number

    switch (strategy) {
      case BackoffStrategy.CONSTANT:
        delay = policy.constantDelay || 1000
        break

      case BackoffStrategy.LINEAR:
        delay = (policy.initialDelay || 1000) * attempt
        break

      case BackoffStrategy.EXPONENTIAL:
        delay = (policy.initialDelay || 1000) * Math.pow(2, attempt - 1)
        break

      case BackoffStrategy.FIBONACCI:
        delay = this.fibonacci(attempt) * (policy.initialDelay || 1000)
        break

      case BackoffStrategy.JITTERED:
        const base = (policy.initialDelay || 1000) * Math.pow(2, attempt - 1)
        delay = base + Math.random() * base * 0.1  // ±10% jitter
        break
    }

    // Applica cap massimo di ritardo
    if (policy.maxDelay) {
      delay = Math.min(delay, policy.maxDelay)
    }

    return delay
  }

  private fibonacci(n: number): number {
    if (n <= 1) return 1
    let a = 1, b = 1
    for (let i = 2; i < n; i++) {
      [a, b] = [b, a + b]
    }
    return b
  }
}
```

---

## Backoff Esponenziale

### Implementazione Backoff

```typescript
class ExponentialBackoff {
  constructor(
    private initialDelay: number = 1000,
    private maxDelay: number = 60000,
    private multiplier: number = 2,
    private jitter: boolean = true
  ) {}

  async execute<T>(
    operation: () => Promise<T>,
    maxAttempts: number = 5,
    shouldRetry: (error: Error) => boolean = () => true
  ): Promise<T> {

    let attempt = 0

    while (true) {
      try {
        return await operation()
      } catch (error) {
        attempt++

        if (attempt >= maxAttempts || !shouldRetry(error)) {
          throw error
        }

        const delay = this.calculateDelay(attempt)
        await sleep(delay)
      }
    }
  }

  private calculateDelay(attempt: number): number {
    // Ritardo base con crescita esponenziale
    let delay = Math.min(
      this.initialDelay * Math.pow(this.multiplier, attempt - 1),
      this.maxDelay
    )

    // Aggiungi jitter per evitare thundering herd
    if (this.jitter) {
      const jitterAmount = delay * 0.1  // ±10%
      delay += (Math.random() * 2 - 1) * jitterAmount
    }

    return Math.floor(delay)
  }
}

// Utilizzo
const backoff = new ExponentialBackoff(1000, 30000, 2, true)

const result = await backoff.execute(
  async () => makeAPICall(),
  5,
  (error) => error instanceof NetworkError
)
```

### Jitter Decorrelato

```typescript
// Backoff con jitter decorrelato raccomandato da AWS
class DecorrelatedJitterBackoff {
  private prevDelay: number = 0

  calculateDelay(
    baseDelay: number,
    maxDelay: number
  ): number {
    const temp = Math.min(maxDelay, baseDelay * 3)
    this.prevDelay = Math.floor(
      baseDelay + Math.random() * (temp - baseDelay)
    )
    return this.prevDelay
  }
}

// Ritardi più uniformemente distribuiti rispetto all'esponenziale puro
```

---

## Modelli di Fallback

### Catena di Fallback

```typescript
interface FallbackChain<T> {
  primary: () => Promise<T>
  fallbacks: Array<() => Promise<T>>
  options: FallbackOptions
}

interface FallbackOptions {
  // Prova fallback se il primario fallisce
  tryFallbacksOn: ErrorCategory[]

  // Massimo fallback da provare
  maxFallbacks?: number

  // Preferisci dati cached/stale
  preferStale?: boolean

  // Callback
  onPrimaryFailure?: (error: ClassifiedError) => void
  onFallbackUsed?: (index: number) => void
  onAllFailed?: (errors: ClassifiedError[]) => void
}

class FallbackExecutor {
  async execute<T>(chain: FallbackChain<T>): Promise<T> {
    const errors: ClassifiedError[] = []

    // Prova primario
    try {
      return await chain.primary()
    } catch (error) {
      const classified = classifier.classify(error, context)
      errors.push(classified)

      if (chain.options.onPrimaryFailure) {
        chain.options.onPrimaryFailure(classified)
      }

      // Controlla se dovrebbe provare fallback
      if (!this.shouldTryFallbacks(classified, chain.options)) {
        throw classified
      }
    }

    // Prova fallback in ordine
    const maxFallbacks = chain.options.maxFallbacks || chain.fallbacks.length

    for (let i = 0; i < Math.min(maxFallbacks, chain.fallbacks.length); i++) {
      try {
        const result = await chain.fallbacks[i]()

        if (chain.options.onFallbackUsed) {
          chain.options.onFallbackUsed(i)
        }

        logger.warn('fallback_used', {
          fallbackIndex: i,
          primaryError: errors[0].category
        })

        return result

      } catch (error) {
        const classified = classifier.classify(error, context)
        errors.push(classified)
      }
    }

    // Tutti i tentativi falliti
    if (chain.options.onAllFailed) {
      chain.options.onAllFailed(errors)
    }

    throw new AllFallbacksFailedError(errors)
  }

  private shouldTryFallbacks(
    error: ClassifiedError,
    options: FallbackOptions
  ): boolean {
    return options.tryFallbacksOn.includes(error.category)
  }
}
```

### Pattern di Fallback Comuni

```typescript
// Pattern 1: Fallback modello (costoso → economico)
const fallbackChain = {
  primary: () => llm.call('gpt-4', prompt),
  fallbacks: [
    () => llm.call('gpt-3.5-turbo', prompt),
    () => llm.call('claude-3-haiku', prompt),
    () => cache.getStale(promptHash)
  ],
  options: {
    tryFallbacksOn: [ErrorCategory.TIMEOUT, ErrorCategory.RATE_LIMIT]
  }
}

// Pattern 2: Fallback servizio (preferito → alternativo)
const searchFallback = {
  primary: () => googleSearch(query),
  fallbacks: [
    () => bingSearch(query),
    () => duckDuckGoSearch(query),
    () => localCache.search(query)
  ],
  options: {
    tryFallbacksOn: [
      ErrorCategory.SERVICE_UNAVAILABLE,
      ErrorCategory.RATE_LIMIT
    ]
  }
}

// Pattern 3: Fallback qualità (alta qualità → qualità accettabile)
const generationFallback = {
  primary: () => generateWithValidation(prompt),
  fallbacks: [
    () => generateWithRelaxedValidation(prompt),
    () => generateBestEffort(prompt),
    () => useTemplate()
  ],
  options: {
    tryFallbacksOn: [ErrorCategory.TIMEOUT, ErrorCategory.INVALID_STATE]
  }
}
```

---

## Circuit Breaker

### Pattern Circuit Breaker

```typescript
enum CircuitState {
  CLOSED = 'closed',      // Operazione normale
  OPEN = 'open',          // Fallimento, rifiuta chiamate
  HALF_OPEN = 'half_open' // Test di recupero
}

interface CircuitBreakerConfig {
  // Soglia di fallimento
  failureThreshold: number      // es. 5 fallimenti
  failureThresholdPercentage?: number  // es. 50%

  // Finestre temporali
  windowSize: number           // Finestra rolling (ms)
  openTimeout: number          // Quanto rimanere aperto (ms)
  halfOpenRequests: number     // Richieste da provare in half-open

  // Callback
  onOpen?: () => void
  onClose?: () => void
  onHalfOpen?: () => void
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED
  private failures: number = 0
  private successes: number = 0
  private lastFailureTime: number = 0
  private halfOpenAttempts: number = 0

  constructor(
    private name: string,
    private config: CircuitBreakerConfig
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    // Controlla stato circuit
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.transitionToHalfOpen()
      } else {
        throw new CircuitBreakerOpenError(
          `Circuit breaker ${this.name} is OPEN`
        )
      }
    }

    try {
      const result = await operation()
      this.recordSuccess()
      return result

    } catch (error) {
      this.recordFailure()
      throw error
    }
  }

  private recordSuccess(): void {
    this.successes++

    if (this.state === CircuitState.HALF_OPEN) {
      this.halfOpenAttempts++

      if (this.halfOpenAttempts >= this.config.halfOpenRequests) {
        this.transitionToClosed()
      }
    }
  }

  private recordFailure(): void {
    this.failures++
    this.lastFailureTime = Date.now()

    if (this.state === CircuitState.HALF_OPEN) {
      this.transitionToOpen()
    } else if (this.state === CircuitState.CLOSED) {
      if (this.failures >= this.config.failureThreshold) {
        this.transitionToOpen()
      }
    }
  }

  private shouldAttemptReset(): boolean {
    return Date.now() - this.lastFailureTime >= this.config.openTimeout
  }

  private transitionToOpen(): void {
    this.state = CircuitState.OPEN

    logger.error('circuit_breaker_opened', {
      breaker: this.name,
      failures: this.failures,
      threshold: this.config.failureThreshold
    })

    if (this.config.onOpen) {
      this.config.onOpen()
    }
  }

  private transitionToHalfOpen(): void {
    this.state = CircuitState.HALF_OPEN
    this.halfOpenAttempts = 0

    logger.info('circuit_breaker_half_open', {
      breaker: this.name
    })

    if (this.config.onHalfOpen) {
      this.config.onHalfOpen()
    }
  }

  private transitionToClosed(): void {
    this.state = CircuitState.CLOSED
    this.failures = 0
    this.successes = 0
    this.halfOpenAttempts = 0

    logger.info('circuit_breaker_closed', {
      breaker: this.name
    })

    if (this.config.onClose) {
      this.config.onClose()
    }
  }

  getState(): CircuitState {
    return this.state
  }

  getMetrics(): CircuitBreakerMetrics {
    return {
      state: this.state,
      failures: this.failures,
      successes: this.successes,
      lastFailureTime: this.lastFailureTime
    }
  }
}

// Utilizzo
const llmCircuitBreaker = new CircuitBreaker('llm_api', {
  failureThreshold: 5,
  windowSize: 60000,  // 1 minuto
  openTimeout: 30000,  // 30 secondi
  halfOpenRequests: 3
})

const result = await llmCircuitBreaker.execute(async () => {
  return await llm.call(prompt)
})
```

---

## Degradazione Graduale

### Strategie di Degradazione

```typescript
enum DegradationLevel {
  FULL = 'full',              // Tutte le funzionalità disponibili
  REDUCED = 'reduced',        // Alcune funzionalità disabilitate
  MINIMAL = 'minimal',        // Solo funzionalità core
  EMERGENCY = 'emergency'     // Minimo indispensabile
}

interface DegradationStrategy {
  level: DegradationLevel

  // Feature flag
  features: {
    complexReasoning: boolean
    memoryRetrieval: boolean
    toolExecution: boolean
    parallelExecution: boolean
    streaming: boolean
  }

  // Limiti di risorse
  limits: {
    maxTokens: number
    maxLLMCalls: number
    maxToolCalls: number
    maxDuration: number
  }

  // Fallback
  fallbacks: {
    useCachedResults: boolean
    useSimpleModels: boolean
    skipNonCritical: boolean
  }
}

const DEGRADATION_STRATEGIES: Record<DegradationLevel, DegradationStrategy> = {
  [DegradationLevel.FULL]: {
    level: DegradationLevel.FULL,
    features: {
      complexReasoning: true,
      memoryRetrieval: true,
      toolExecution: true,
      parallelExecution: true,
      streaming: true
    },
    limits: {
      maxTokens: 100000,
      maxLLMCalls: 100,
      maxToolCalls: 50,
      maxDuration: 300000
    },
    fallbacks: {
      useCachedResults: false,
      useSimpleModels: false,
      skipNonCritical: false
    }
  },

  [DegradationLevel.REDUCED]: {
    level: DegradationLevel.REDUCED,
    features: {
      complexReasoning: true,
      memoryRetrieval: true,
      toolExecution: true,
      parallelExecution: false,
      streaming: true
    },
    limits: {
      maxTokens: 50000,
      maxLLMCalls: 50,
      maxToolCalls: 20,
      maxDuration: 120000
    },
    fallbacks: {
      useCachedResults: true,
      useSimpleModels: false,
      skipNonCritical: true
    }
  },

  [DegradationLevel.MINIMAL]: {
    level: DegradationLevel.MINIMAL,
    features: {
      complexReasoning: false,
      memoryRetrieval: false,
      toolExecution: true,
      parallelExecution: false,
      streaming: false
    },
    limits: {
      maxTokens: 10000,
      maxLLMCalls: 10,
      maxToolCalls: 5,
      maxDuration: 30000
    },
    fallbacks: {
      useCachedResults: true,
      useSimpleModels: true,
      skipNonCritical: true
    }
  },

  [DegradationLevel.EMERGENCY]: {
    level: DegradationLevel.EMERGENCY,
    features: {
      complexReasoning: false,
      memoryRetrieval: false,
      toolExecution: false,
      parallelExecution: false,
      streaming: false
    },
    limits: {
      maxTokens: 1000,
      maxLLMCalls: 1,
      maxToolCalls: 0,
      maxDuration: 5000
    },
    fallbacks: {
      useCachedResults: true,
      useSimpleModels: true,
      skipNonCritical: true
    }
  }
}
```

### Decisione di Degradazione

```typescript
class DegradationManager {
  private currentLevel: DegradationLevel = DegradationLevel.FULL

  async determineLevel(metrics: SystemMetrics): Promise<DegradationLevel> {
    // Controlla vari indicatori di salute
    const healthScore = this.calculateHealthScore(metrics)

    if (healthScore >= 0.9) {
      return DegradationLevel.FULL
    } else if (healthScore >= 0.7) {
      return DegradationLevel.REDUCED
    } else if (healthScore >= 0.4) {
      return DegradationLevel.MINIMAL
    } else {
      return DegradationLevel.EMERGENCY
    }
  }

  private calculateHealthScore(metrics: SystemMetrics): number {
    const factors = {
      // Tasso di errori (0-1, più alto è meglio)
      errorRate: 1 - metrics.errorRate,

      // Latenza (0-1, più alto è meglio)
      latency: Math.max(0, 1 - metrics.p99Latency / 10000),

      // Disponibilità risorse (0-1)
      resources: Math.min(
        metrics.cpuAvailable / 100,
        metrics.memoryAvailable / 100
      ),

      // Salute dipendenze (0-1)
      dependencies: metrics.healthyDependencies / metrics.totalDependencies
    }

    // Media ponderata
    return (
      0.3 * factors.errorRate +
      0.3 * factors.latency +
      0.2 * factors.resources +
      0.2 * factors.dependencies
    )
  }

  async degrade(to: DegradationLevel): Promise<void> {
    const from = this.currentLevel

    if (from === to) return

    logger.warn('system_degradation', {
      from,
      to,
      reason: 'health_score_threshold'
    })

    this.currentLevel = to

    // Applica strategia di degradazione
    await this.applyStrategy(DEGRADATION_STRATEGIES[to])

    // Emetti evento
    events.emit('degradation', { from, to })
  }

  private async applyStrategy(strategy: DegradationStrategy): Promise<void> {
    // Aggiorna feature flag
    featureFlags.update(strategy.features)

    // Aggiorna limiti di risorse
    resourceManager.setLimits(strategy.limits)

    // Configura fallback
    fallbackManager.configure(strategy.fallbacks)
  }
}
```

---

## Pattern di Recupero Errori

### Pattern: Checkpoint e Ripresa

```typescript
interface Checkpoint {
  id: UUID
  timestamp: number
  state: State
  progress: Progress
}

class CheckpointManager {
  async checkpoint(state: State): Promise<Checkpoint> {
    const checkpoint: Checkpoint = {
      id: generateUUID(),
      timestamp: Date.now(),
      state: cloneDeep(state),
      progress: state.progress
    }

    await this.storage.save(checkpoint)

    return checkpoint
  }

  async resume(checkpointId: UUID): Promise<State> {
    const checkpoint = await this.storage.load(checkpointId)

    if (!checkpoint) {
      throw new Error(`Checkpoint ${checkpointId} not found`)
    }

    logger.info('resuming_from_checkpoint', {
      checkpointId,
      age: Date.now() - checkpoint.timestamp
    })

    return checkpoint.state
  }
}

// Utilizzo
class ResilientExecutor {
  async execute(task: Task): Promise<Result> {
    let checkpoint: Checkpoint | null = null

    try {
      // Crea checkpoint prima di iniziare
      checkpoint = await checkpointManager.checkpoint(task.initialState)

      // Esegui task
      return await this.executeInternal(task)

    } catch (error) {
      // Prova a recuperare dal checkpoint
      if (checkpoint && this.canRecover(error)) {
        logger.warn('recovering_from_checkpoint', {
          checkpointId: checkpoint.id,
          error: error.message
        })

        const state = await checkpointManager.resume(checkpoint.id)
        return await this.executeFrom(task, state)
      }

      throw error
    }
  }
}
```

### Pattern: Transazioni Compensative

```typescript
interface Transaction {
  forward: () => Promise<void>
  compensate: () => Promise<void>
}

class SAGA {
  private transactions: Transaction[] = []
  private completed: number = 0

  add(transaction: Transaction): void {
    this.transactions.push(transaction)
  }

  async execute(): Promise<void> {
    try {
      // Esegui transazioni forward
      for (let i = 0; i < this.transactions.length; i++) {
        await this.transactions[i].forward()
        this.completed = i + 1
      }

    } catch (error) {
      // Compensa in ordine inverso
      logger.error('saga_failed_compensating', {
        completed: this.completed,
        total: this.transactions.length
      })

      for (let i = this.completed - 1; i >= 0; i--) {
        try {
          await this.transactions[i].compensate()
        } catch (compensateError) {
          logger.error('compensation_failed', {
            transaction: i,
            error: compensateError
          })
        }
      }

      throw error
    }
  }
}

// Utilizzo
const saga = new SAGA()

saga.add({
  forward: async () => {
    await database.insert(record)
  },
  compensate: async () => {
    await database.delete(record.id)
  }
})

saga.add({
  forward: async () => {
    await cache.set(key, value)
  },
  compensate: async () => {
    await cache.delete(key)
  }
})

await saga.execute()
```

---

## Rilevamento Fallimenti

### Health Check

```typescript
interface HealthCheck {
  name: string
  check: () => Promise<HealthStatus>
  interval: number
  timeout: number
}

enum HealthStatus {
  HEALTHY = 'healthy',
  DEGRADED = 'degraded',
  UNHEALTHY = 'unhealthy'
}

class HealthMonitor {
  private checks: Map<string, HealthCheck> = new Map()
  private statuses: Map<string, HealthStatus> = new Map()

  register(check: HealthCheck): void {
    this.checks.set(check.name, check)
    this.scheduleCheck(check)
  }

  private scheduleCheck(check: HealthCheck): void {
    setInterval(async () => {
      try {
        const status = await withTimeout(
          check.check(),
          check.timeout
        )

        this.statuses.set(check.name, status)

        if (status !== HealthStatus.HEALTHY) {
          logger.warn('health_check_failed', {
            check: check.name,
            status
          })
        }

      } catch (error) {
        this.statuses.set(check.name, HealthStatus.UNHEALTHY)

        logger.error('health_check_error', {
          check: check.name,
          error: error.message
        })
      }
    }, check.interval)
  }

  getOverallHealth(): HealthStatus {
    const statuses = Array.from(this.statuses.values())

    if (statuses.every(s => s === HealthStatus.HEALTHY)) {
      return HealthStatus.HEALTHY
    }

    if (statuses.some(s => s === HealthStatus.UNHEALTHY)) {
      return HealthStatus.UNHEALTHY
    }

    return HealthStatus.DEGRADED
  }
}

// Registra health check
healthMonitor.register({
  name: 'llm_api',
  check: async () => {
    const start = Date.now()
    await llm.healthCheck()
    const latency = Date.now() - start

    if (latency < 1000) return HealthStatus.HEALTHY
    if (latency < 5000) return HealthStatus.DEGRADED
    return HealthStatus.UNHEALTHY
  },
  interval: 30000,  // 30 secondi
  timeout: 10000    // 10 secondi
})
```

---

## Alberi Decisionali di Recupero

### Albero Decisionale

```typescript
async function recoverFromError(
  error: ClassifiedError,
  context: OperationContext
): Promise<RecoveryDecision> {

  // Livello 1: Controlla categoria errore
  if (!error.retriable) {
    return {
      action: RecoveryAction.ABORT,
      reason: 'Non-retriable error category'
    }
  }

  // Livello 2: Controlla budget retry
  if (context.retryAttempts >= context.maxRetries) {
    return {
      action: RecoveryAction.FALLBACK,
      reason: 'Retry budget exhausted'
    }
  }

  // Livello 3: Controlla stato circuit breaker
  const breakerState = circuitBreaker.getState()
  if (breakerState === CircuitState.OPEN) {
    return {
      action: RecoveryAction.CIRCUIT_BREAK,
      reason: 'Circuit breaker is OPEN'
    }
  }

  // Livello 4: Controlla severità errore
  if (error.severity === ErrorSeverity.CRITICAL) {
    return {
      action: RecoveryAction.ESCALATE,
      reason: 'Critical error requires human intervention'
    }
  }

  // Livello 5: Controlla disponibilità risorse
  const resources = await resourceManager.getAvailability()
  if (resources.available < 0.2) {  // < 20%
    return {
      action: RecoveryAction.FALLBACK,
      reason: 'Insufficient resources for retry'
    }
  }

  // Livello 6: Controlla pattern errore
  const pattern = await errorPatternDetector.detect(error, context)
  if (pattern === 'infinite_loop') {
    return {
      action: RecoveryAction.ABORT,
      reason: 'Infinite loop detected'
    }
  }

  // Default: Retry con backoff
  return {
    action: RecoveryAction.RETRY_WITH_BACKOFF,
    reason: 'Retriable error, retry with backoff',
    retryAfter: error.retryAfter || 1000
  }
}

interface RecoveryDecision {
  action: RecoveryAction
  reason: string
  retryAfter?: number
}
```

---

## Specifiche delle Interfacce

### Interfacce Core

```typescript
interface ErrorHandler {
  handle(error: Error, context: OperationContext): Promise<RecoveryResult>
}

interface RecoveryResult {
  recovered: boolean
  action: RecoveryAction
  result?: any
  error?: Error
}

interface OperationContext {
  operation: string
  component: string
  retryAttempts: number
  maxRetries: number
  traceId: UUID
  spanId: UUID
  data: Record<string, any>
}
```

---

## Conclusione

Il Sistema di Gestione Errori e Recupero fornisce una gestione completa dei fallimenti attraverso:
- **Classificazione Sistematica degli Errori**: Categorizzare e comprendere i fallimenti
- **Logica di Retry Intelligente**: Retry con strategie appropriate
- **Circuit Breaker**: Prevenire fallimenti a cascata
- **Degradazione Graduale**: Mantenere funzionalità core sotto stress
- **Pattern di Recupero**: Checkpoint, ripresa, compensazione
- **Rilevamento Fallimenti**: Monitoraggio proattivo della salute

Questo garantisce che l'agente rimanga resiliente e continui a fare progressi anche in presenza di fallimenti.

---

## Riferimenti

1. Release It! (Nygard, 2018). Pattern per software production-ready.
2. AWS Architecture Blog. Backoff esponenziale e jitter.
3. Netflix Hystrix. Pattern circuit breaker.
4. Martin Fowler. Patterns of Enterprise Application Architecture.
5. Microsoft Azure Architecture. Guida retry per servizi specifici.
