# Architettura Sicurezza e Safety

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Modello delle Minacce](#modello-delle-minacce)
3. [Principi di Sicurezza](#principi-di-sicurezza)
4. [Validazione Input](#validazione-input)
5. [Filtraggio Output](#filtraggio-output)
6. [Sandboxing](#sandboxing)
7. [Rate Limiting](#rate-limiting)
8. [Audit Logging](#audit-logging)
9. [Autenticazione e Autorizzazione](#autenticazione-e-autorizzazione)
10. [Gestione Segreti](#gestione-segreti)
11. [Guardrail di Safety](#guardrail-di-safety)
12. [Conformità](#conformità)

---

## Panoramica

L'Architettura Sicurezza e Safety garantisce che l'agente operi in modo sicuro e protetto, proteggendo utenti, sistemi e dati da danni. Sicurezza e safety sono integrate fin dall'inizio, non aggiunte come ripensamento.

### Obiettivi di Design

1. **Defense in Depth**: Livelli multipli di sicurezza
2. **Fail Secure**: I fallimenti di sicurezza fermano l'operazione, non bypassano
3. **Least Privilege**: Permessi minimi di default
4. **Zero Trust**: Verificare tutto, non fidarsi di nulla
5. **Auditable**: Tutti gli eventi di sicurezza sono registrati
6. **Compliant**: Soddisfare requisiti normativi (GDPR, SOC 2, ecc.)

### Architettura di Sicurezza

```
┌────────────────────────────────────────┐
│      Livelli di Sicurezza (Defense)    │
│                                        │
│  Layer 1: Validazione Input           │
│  ├─ Enforcement schema                │
│  ├─ Rilevamento prompt injection      │
│  └─ Limiti dimensione                 │
│                                        │
│  Layer 2: Autenticazione               │
│  ├─ Verifica identità utente          │
│  ├─ Validazione API key               │
│  └─ Verifica token JWT                │
│                                        │
│  Layer 3: Autorizzazione               │
│  ├─ Controllo accesso basato su ruoli │
│  ├─ Permessi a livello risorsa        │
│  └─ Allowlist operazioni              │
│                                        │
│  Layer 4: Isolamento Esecuzione        │
│  ├─ Esecuzione tool sandboxed         │
│  ├─ Limiti risorse                    │
│  └─ Restrizioni rete                  │
│                                        │
│  Layer 5: Filtraggio Output            │
│  ├─ Rilevamento dati sensibili        │
│  ├─ Filtraggio contenuti dannosi      │
│  └─ Validazione formato               │
│                                        │
│  Layer 6: Audit e Monitoraggio         │
│  ├─ Tutte le operazioni registrate    │
│  ├─ Rilevamento anomalie              │
│  └─ Alert sicurezza                   │
└────────────────────────────────────────┘
```

---

## Modello delle Minacce

### Categorie di Minacce

```typescript
enum ThreatCategory {
  // Minacce basate su input
  PROMPT_INJECTION = 'prompt_injection',
  DATA_POISONING = 'data_poisoning',
  ADVERSARIAL_INPUT = 'adversarial_input',

  // Minacce accesso
  UNAUTHORIZED_ACCESS = 'unauthorized_access',
  PRIVILEGE_ESCALATION = 'privilege_escalation',

  // Minacce esecuzione
  CODE_INJECTION = 'code_injection',
  COMMAND_INJECTION = 'command_injection',
  RESOURCE_EXHAUSTION = 'resource_exhaustion',

  // Minacce dati
  DATA_EXFILTRATION = 'data_exfiltration',
  DATA_CORRUPTION = 'data_corruption',
  PRIVACY_VIOLATION = 'privacy_violation',

  // Minacce supply chain
  COMPROMISED_DEPENDENCY = 'compromised_dependency',
  MALICIOUS_TOOL = 'malicious_tool'
}

interface Threat {
  category: ThreatCategory
  description: string
  likelihood: 'low' | 'medium' | 'high'
  impact: 'low' | 'medium' | 'high' | 'critical'
  mitigations: Mitigation[]
}

const THREAT_MODEL: Threat[] = [
  {
    category: ThreatCategory.PROMPT_INJECTION,
    description: 'Attacker crafts input to manipulate agent behavior',
    likelihood: 'high',
    impact: 'high',
    mitigations: [
      'Input validation and sanitization',
      'Prompt structure isolation',
      'Output filtering',
      'Behavioral monitoring'
    ]
  },

  {
    category: ThreatCategory.DATA_EXFILTRATION,
    description: 'Agent leaks sensitive data in outputs',
    likelihood: 'medium',
    impact: 'critical',
    mitigations: [
      'Output filtering for PII',
      'Data classification',
      'Access controls',
      'Audit logging'
    ]
  },

  {
    category: ThreatCategory.RESOURCE_EXHAUSTION,
    description: 'Attacker causes agent to consume excessive resources',
    likelihood: 'high',
    impact: 'medium',
    mitigations: [
      'Rate limiting',
      'Resource budgets',
      'Timeout enforcement',
      'Circuit breakers'
    ]
  }
]
```

### Confini di Fiducia

```
┌─────────────────────────────────────┐
│  Fidato                             │
│  • Codice core agente               │
│  • Provider LLM verificati          │
│  • Infrastruttura gestita           │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│  Non Fidato (Deve Validare)         │
│  • Input utenti                     │
│  • API esterne                      │
│  • Tool terze parti                 │
│  • Codice fornito dall'utente       │
│  • Risposte rete                    │
└─────────────────────────────────────┘
```

---

## Principi di Sicurezza

### 1. Defense in Depth

Livelli di sicurezza multipli e indipendenti:

```typescript
async function executeSecurely(request: Request): Promise<Response> {
  // Layer 1: Valida input
  const validated = await inputValidator.validate(request)

  // Layer 2: Autentica
  const authenticated = await authenticator.authenticate(validated)

  // Layer 3: Autorizza
  const authorized = await authorizer.authorize(authenticated)

  // Layer 4: Esegui in sandbox
  const result = await sandbox.execute(authorized)

  // Layer 5: Filtra output
  const filtered = await outputFilter.filter(result)

  // Layer 6: Log audit
  await auditLog.record(authenticated, authorized, filtered)

  return filtered
}
```

### 2. Fail Secure

I fallimenti di sicurezza fermano l'operazione:

```typescript
try {
  const validated = await validator.validate(input)
} catch (error) {
  // Validazione sicurezza fallita → FERMA, non continuare
  logger.error('security_validation_failed', { error })
  throw new SecurityError('Input validation failed')
  // NON ripiegare su input non validato
}
```

### 3. Least Privilege

Concedi permessi minimi necessari:

```typescript
interface Permission {
  resource: string
  actions: Action[]
  conditions?: Condition[]
}

// Esempio: Accesso file read-only
const readOnlyFilePermission: Permission = {
  resource: 'file:/data/*',
  actions: ['read'],
  conditions: [
    { type: 'path_prefix', value: '/data/' }
  ]
}

// Esempio: Accesso API limitato
const limitedAPIPermission: Permission = {
  resource: 'api:external/*',
  actions: ['GET'],
  conditions: [
    { type: 'rate_limit', value: '10/minute' }
  ]
}
```

### 4. Zero Trust

Verifica ogni richiesta, ogni volta:

```typescript
async function processRequest(request: Request): Promise<Response> {
  // Verifica sempre autenticazione
  const user = await verifyAuthentication(request.token)

  // Controlla sempre autorizzazione
  const allowed = await checkAuthorization(user, request.operation)
  if (!allowed) {
    throw new ForbiddenError()
  }

  // Valida sempre input
  const validated = await validateInput(request.data)

  // Controlla sempre rate limit
  await enforceRateLimit(user.id, request.operation)

  // Esegui
  return await execute(validated, user)
}
```

---

## Validazione Input

### Architettura Validazione

```typescript
interface InputValidator {
  // Validazione schema
  async validateSchema(
    input: any,
    schema: JSONSchema
  ): Promise<ValidationResult>

  // Sanitizzazione
  sanitize(input: string): string

  // Rilevamento prompt injection
  async detectPromptInjection(input: string): Promise<boolean>

  // Limiti dimensione
  checkSizeLimits(input: any): boolean
}

class ComprehensiveInputValidator implements InputValidator {
  async validateSchema(input: any, schema: JSONSchema): Promise<ValidationResult> {
    // Usa validatore JSON Schema (es. Ajv)
    const validator = new Ajv()
    const validate = validator.compile(schema)
    const valid = validate(input)

    if (!valid) {
      return {
        valid: false,
        errors: validate.errors || []
      }
    }

    return { valid: true, errors: [] }
  }

  sanitize(input: string): string {
    // Rimuovi caratteri di controllo
    let sanitized = input.replace(/[\x00-\x1F\x7F]/g, '')

    // Escape HTML
    sanitized = sanitized
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')

    // Escape SQL (se si salva in DB)
    sanitized = sanitized.replace(/['";\\]/g, '\\$&')

    return sanitized
  }

  async detectPromptInjection(input: string): Promise<boolean> {
    const indicators = [
      // Ignora istruzioni precedenti
      /ignore (previous|above|all) instructions?/i,
      /disregard (previous|above|all) instructions?/i,

      // Prompt sistema
      /system:?/i,
      /<\|system\|>/,

      // Leak prompt
      /print (your|the) (system )?(prompt|instructions)/i,
      /reveal (your|the) (system )?(prompt|instructions)/i,

      // Hijacking ruolo
      /you are now/i,
      /act as (if you are|a)/i,
      /pretend (you are|to be)/i
    ]

    for (const pattern of indicators) {
      if (pattern.test(input)) {
        logger.warn('prompt_injection_detected', {
          pattern: pattern.source,
          input: input.substring(0, 100)
        })
        return true
      }
    }

    // Usa rilevatore basato su ML (più sofisticato)
    const mlDetection = await this.mlDetector.detect(input)
    if (mlDetection.score > THRESHOLD) {
      logger.warn('ml_injection_detected', {
        score: mlDetection.score,
        confidence: mlDetection.confidence
      })
      return true
    }

    return false
  }

  checkSizeLimits(input: any): boolean {
    const size = JSON.stringify(input).length

    if (size > MAX_INPUT_SIZE) {
      logger.warn('input_size_exceeded', {
        size,
        limit: MAX_INPUT_SIZE
      })
      return false
    }

    return true
  }
}
```

### Pipeline Validazione Input

```typescript
async function validateInput(input: UserInput): Promise<ValidatedInput> {
  // 1. Controllo dimensione
  if (!validator.checkSizeLimits(input)) {
    throw new ValidationError('Input exceeds size limit')
  }

  // 2. Validazione schema
  const schemaResult = await validator.validateSchema(
    input,
    INPUT_SCHEMA
  )
  if (!schemaResult.valid) {
    throw new ValidationError('Schema validation failed', schemaResult.errors)
  }

  // 3. Sanitizzazione
  const sanitized = {
    ...input,
    query: validator.sanitize(input.query),
    context: validator.sanitize(input.context || '')
  }

  // 4. Rilevamento prompt injection
  const isInjection = await validator.detectPromptInjection(sanitized.query)
  if (isInjection) {
    // Log e rifiuta
    await auditLog.record({
      event: 'prompt_injection_blocked',
      userId: input.userId,
      input: sanitized.query.substring(0, 100)
    })

    throw new SecurityError('Potential prompt injection detected')
  }

  // 5. Controllo policy contenuti
  const policyViolation = await contentPolicy.check(sanitized.query)
  if (policyViolation) {
    throw new PolicyViolationError(policyViolation.reason)
  }

  return sanitized
}
```

---

## Filtraggio Output

### Filtro Output

```typescript
interface OutputFilter {
  // Filtra dati sensibili
  async filterSensitiveData(output: string): Promise<string>

  // Filtra contenuti dannosi
  async filterHarmfulContent(output: string): Promise<string>

  // Oscura PII
  async redactPII(output: string): Promise<string>

  // Valida formato
  validateFormat(output: any, schema: JSONSchema): ValidationResult
}

class ComprehensiveOutputFilter implements OutputFilter {
  async filterSensitiveData(output: string): Promise<string> {
    let filtered = output

    // Rileva e oscura API key
    const apiKeyPattern = /\b[A-Za-z0-9]{32,}\b/g
    filtered = filtered.replace(apiKeyPattern, '[REDACTED_API_KEY]')

    // Rileva e oscura password
    const passwordPattern = /password[:\s]*[^\s]*/gi
    filtered = filtered.replace(passwordPattern, 'password: [REDACTED]')

    // Rileva e oscura token
    const tokenPattern = /token[:\s]*[^\s]*/gi
    filtered = filtered.replace(tokenPattern, 'token: [REDACTED]')

    return filtered
  }

  async redactPII(output: string): Promise<string> {
    let redacted = output

    // Indirizzi email
    const emailPattern = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g
    redacted = redacted.replace(emailPattern, '[EMAIL_REDACTED]')

    // Numeri telefono (formato US)
    const phonePattern = /\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g
    redacted = redacted.replace(phonePattern, '[PHONE_REDACTED]')

    // SSN (formato US)
    const ssnPattern = /\b\d{3}-\d{2}-\d{4}\b/g
    redacted = redacted.replace(ssnPattern, '[SSN_REDACTED]')

    // Numeri carte di credito
    const ccPattern = /\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g
    redacted = redacted.replace(ccPattern, '[CC_REDACTED]')

    // Usa modello NER per rilevamento più sofisticato
    const nerResult = await this.nerModel.detectPII(redacted)
    for (const entity of nerResult.entities) {
      if (entity.type === 'PERSON' || entity.type === 'ADDRESS') {
        redacted = redacted.replace(entity.text, `[${entity.type}_REDACTED]`)
      }
    }

    return redacted
  }

  async filterHarmfulContent(output: string): Promise<string> {
    // Usa API moderazione contenuti
    const moderation = await moderationAPI.moderate(output)

    if (moderation.flagged) {
      logger.warn('harmful_content_detected', {
        categories: moderation.categories,
        severity: moderation.severity
      })

      // Rifiuta interamente se molto dannoso
      if (moderation.severity === 'high') {
        throw new ContentPolicyViolationError(
          'Output contains harmful content'
        )
      }

      // Oscura sezioni dannose specifiche
      let filtered = output
      for (const span of moderation.flaggedSpans) {
        filtered = filtered.replace(
          span.text,
          '[CONTENT_REMOVED]'
        )
      }

      return filtered
    }

    return output
  }
}
```

### Validazione Output

```typescript
async function validateOutput(
  output: any,
  expectedSchema: JSONSchema,
  safety: SafetyConfig
): Promise<ValidatedOutput> {

  // 1. Validazione schema
  const schemaValid = await validator.validateSchema(output, expectedSchema)
  if (!schemaValid.valid) {
    throw new OutputValidationError('Output schema invalid', schemaValid.errors)
  }

  // 2. Filtra dati sensibili
  let filtered = output
  if (safety.filterSensitiveData) {
    filtered = await outputFilter.filterSensitiveData(JSON.stringify(filtered))
    filtered = JSON.parse(filtered)
  }

  // 3. Oscura PII
  if (safety.redactPII) {
    filtered = await outputFilter.redactPII(JSON.stringify(filtered))
    filtered = JSON.parse(filtered)
  }

  // 4. Filtra contenuti dannosi
  if (safety.filterHarmful) {
    filtered = await outputFilter.filterHarmfulContent(JSON.stringify(filtered))
    filtered = JSON.parse(filtered)
  }

  // 5. Controllo dimensione
  const size = JSON.stringify(filtered).length
  if (size > safety.maxOutputSize) {
    throw new OutputTooLargeError(size, safety.maxOutputSize)
  }

  return {
    output: filtered,
    filtered: filtered !== output,
    filterActions: getFilterActions()
  }
}
```

---

## Sandboxing

### Livelli Sandbox

```typescript
enum SandboxLevel {
  NONE = 'none',              // Nessun isolamento (fiducia)
  PROCESS = 'process',        // Processo separato
  CONTAINER = 'container',    // Container Docker
  VM = 'vm'                   // Macchina virtuale
}

interface SandboxConfig {
  level: SandboxLevel

  // Restrizioni filesystem
  filesystem: {
    allowedPaths: string[]
    readOnly: boolean
    maxFileSize: number
  }

  // Restrizioni rete
  network: {
    allowed: boolean
    allowedHosts: string[]
    allowedPorts: number[]
  }

  // Limiti risorse
  resources: {
    maxCPU: number      // Core CPU
    maxMemory: number   // Byte
    maxProcesses: number
    maxTime: number     // Millisecondi
  }

  // Restrizioni system call
  syscalls: {
    allowList: string[]
    denyList: string[]
  }
}
```

### Esecutore Sandbox

```typescript
class SandboxExecutor {
  async execute(
    code: string,
    language: string,
    config: SandboxConfig
  ): Promise<ExecutionResult> {

    const sandbox = await this.createSandbox(config)

    try {
      // Scrivi codice in sandbox
      await sandbox.writeFile('main.' + language, code)

      // Compila se necessario
      if (needsCompilation(language)) {
        await sandbox.compile('main.' + language)
      }

      // Esegui con monitoraggio
      const monitor = new ResourceMonitor()
      const result = await Promise.race([
        sandbox.run('main.' + language, monitor),
        timeout(config.resources.maxTime)
      ])

      // Controlla utilizzo risorse
      const usage = monitor.getUsage()
      if (usage.cpu > config.resources.maxCPU) {
        throw new ResourceLimitError('CPU limit exceeded')
      }
      if (usage.memory > config.resources.maxMemory) {
        throw new ResourceLimitError('Memory limit exceeded')
      }

      return result

    } finally {
      await sandbox.destroy()
    }
  }

  private async createSandbox(config: SandboxConfig): Promise<Sandbox> {
    switch (config.level) {
      case SandboxLevel.PROCESS:
        return new ProcessSandbox(config)

      case SandboxLevel.CONTAINER:
        return new ContainerSandbox(config)

      case SandboxLevel.VM:
        return new VMSandbox(config)

      default:
        throw new Error(`Unsupported sandbox level: ${config.level}`)
    }
  }
}
```

### Esecuzione Tool Sicura

```typescript
async function executeToolSecurely(
  tool: Tool,
  parameters: BoundParameters,
  permissions: Permission[]
): Promise<ToolResult> {

  // 1. Controlla permessi
  const allowed = await checkPermission(tool, permissions)
  if (!allowed) {
    throw new ForbiddenError(`Tool ${tool.name} not permitted`)
  }

  // 2. Valida parametri
  await validator.validateSchema(parameters, tool.inputSchema)

  // 3. Crea sandbox
  const sandbox = await createSandbox({
    level: tool.sandboxLevel,
    filesystem: tool.fileSystemAccess,
    network: tool.networkAccess,
    resources: tool.resourceLimits
  })

  try {
    // 4. Esegui in sandbox
    const result = await sandbox.execute(
      tool.executor,
      parameters
    )

    // 5. Valida output
    await validator.validateSchema(result.output, tool.outputSchema)

    // 6. Filtra output
    const filtered = await outputFilter.filter(result.output)

    return {
      ...result,
      output: filtered
    }

  } finally {
    await sandbox.destroy()
  }
}
```

---

## Rate Limiting

### Rate Limiter

```typescript
interface RateLimiter {
  // Controlla se richiesta permessa
  async allow(
    key: string,
    limit: RateLimit
  ): Promise<boolean>

  // Consuma token
  async consume(
    key: string,
    tokens: number
  ): Promise<void>

  // Ottieni utilizzo corrente
  async getUsage(key: string): Promise<Usage>
}

interface RateLimit {
  requests: number
  per: number  // millisecondi
  burst?: number
}

class TokenBucketRateLimiter implements RateLimiter {
  private buckets: Map<string, Bucket> = new Map()

  async allow(key: string, limit: RateLimit): Promise<boolean> {
    const bucket = this.getBucket(key, limit)

    // Riempi token basandosi sul tempo trascorso
    const now = Date.now()
    const elapsed = now - bucket.lastRefill
    const tokensToAdd = Math.floor(elapsed / limit.per * limit.requests)

    bucket.tokens = Math.min(
      bucket.capacity,
      bucket.tokens + tokensToAdd
    )
    bucket.lastRefill = now

    // Controlla se ci sono token disponibili
    if (bucket.tokens < 1) {
      logger.warn('rate_limit_exceeded', {
        key,
        limit: limit.requests,
        per: limit.per
      })
      return false
    }

    return true
  }

  async consume(key: string, tokens: number = 1): Promise<void> {
    const bucket = this.getBucket(key, DEFAULT_LIMIT)
    bucket.tokens -= tokens
  }

  private getBucket(key: string, limit: RateLimit): Bucket {
    if (!this.buckets.has(key)) {
      this.buckets.set(key, {
        capacity: limit.burst || limit.requests,
        tokens: limit.burst || limit.requests,
        lastRefill: Date.now()
      })
    }
    return this.buckets.get(key)!
  }
}

// Utilizzo
const rateLimiter = new TokenBucketRateLimiter()

// Rate limit per utente
const userKey = `user:${userId}`
const allowed = await rateLimiter.allow(userKey, {
  requests: 100,
  per: 60000  // 100 richieste al minuto
})

if (!allowed) {
  throw new RateLimitExceededError()
}

await rateLimiter.consume(userKey)
```

### Rate Limiting Adattivo

```typescript
class AdaptiveRateLimiter {
  private baseLimits: Map<string, RateLimit> = new Map()

  async allow(key: string): Promise<boolean> {
    const baseLimit = this.baseLimits.get(key) || DEFAULT_LIMIT

    // Adatta in base al carico sistema
    const systemLoad = await this.getSystemLoad()
    const adjustedLimit = this.adjustForLoad(baseLimit, systemLoad)

    // Adatta in base al comportamento utente
    const userBehavior = await this.getUserBehavior(key)
    const finalLimit = this.adjustForBehavior(adjustedLimit, userBehavior)

    return await this.limiter.allow(key, finalLimit)
  }

  private adjustForLoad(limit: RateLimit, load: number): RateLimit {
    // Riduci limiti quando sistema sotto carico pesante
    if (load > 0.8) {
      return {
        requests: Math.floor(limit.requests * 0.5),
        per: limit.per
      }
    }

    return limit
  }

  private adjustForBehavior(
    limit: RateLimit,
    behavior: UserBehavior
  ): RateLimit {
    // Aumenta limiti per utenti buoni
    if (behavior.trustScore > 0.8) {
      return {
        requests: Math.floor(limit.requests * 1.5),
        per: limit.per
      }
    }

    // Diminuisci per utenti sospetti
    if (behavior.trustScore < 0.3) {
      return {
        requests: Math.floor(limit.requests * 0.5),
        per: limit.per
      }
    }

    return limit
  }
}
```

---

## Audit Logging

### Schema Audit Log

```typescript
interface AuditLogEntry {
  // Identificazione
  id: UUID
  timestamp: number

  // Attore
  userId: UUID
  sessionId: UUID
  ipAddress: string
  userAgent: string

  // Azione
  action: string
  resource: string
  operation: string

  // Risultato
  success: boolean
  statusCode: number

  // Contesto
  requestId: UUID
  traceId: UUID

  // Dati (possono essere oscurati)
  inputHash: string     // Hash dell'input, non input reale
  outputHash: string    // Hash dell'output, non output reale

  // Sicurezza
  authMethod: string
  permissions: Permission[]

  // Metadata
  duration: number
  resourcesUsed: ResourceUsage
}
```

### Audit Logger

```typescript
class AuditLogger {
  async record(entry: AuditLogEntry): Promise<void> {
    // 1. Valida entry
    await this.validate(entry)

    // 2. Arricchisci con contesto
    const enriched = this.enrich(entry)

    // 3. Cifra campi sensibili
    const encrypted = await this.encrypt(enriched)

    // 4. Scrivi su log immutabile
    await this.writeToLog(encrypted)

    // 5. Indicizza per query
    await this.indexEntry(encrypted)

    // 6. Controlla per alert
    await this.checkForAlerts(encrypted)
  }

  private async encrypt(entry: AuditLogEntry): Promise<EncryptedEntry> {
    // Cifra campi PII
    const encrypted = { ...entry }

    if (entry.userId) {
      encrypted.userId = await this.crypto.encrypt(entry.userId)
    }

    if (entry.ipAddress) {
      encrypted.ipAddress = await this.crypto.encrypt(entry.ipAddress)
    }

    return encrypted
  }

  private async writeToLog(entry: EncryptedEntry): Promise<void> {
    // Scrivi su log append-only (immutabile)
    await this.appendOnlyLog.append(entry)

    // Scrivi anche su indice ricercabile
    await this.searchIndex.index(entry)
  }

  private async checkForAlerts(entry: AuditLogEntry): Promise<void> {
    // Autenticazione fallita
    if (entry.action === 'authenticate' && !entry.success) {
      await this.incrementFailedAuth(entry.userId)

      const failedAttempts = await this.getFailedAuthCount(entry.userId)
      if (failedAttempts > THRESHOLD) {
        await this.raiseAlert('repeated_auth_failures', {
          userId: entry.userId,
          attempts: failedAttempts
        })
      }
    }

    // Operazioni sensibili
    if (SENSITIVE_OPERATIONS.includes(entry.operation)) {
      await this.raiseAlert('sensitive_operation', {
        userId: entry.userId,
        operation: entry.operation
      })
    }
  }
}
```

### Query Audit

```typescript
interface AuditQuery {
  // Chi
  userId?: UUID

  // Cosa
  action?: string
  resource?: string

  // Quando
  startTime: number
  endTime: number

  // Risultato
  success?: boolean

  // Limite
  limit: number
}

class AuditLogQuery {
  async query(query: AuditQuery): Promise<AuditLogEntry[]> {
    // Costruisci query
    const dbQuery = this.buildQuery(query)

    // Esegui contro indice
    const results = await this.searchIndex.search(dbQuery)

    // Decifra PII se utente autorizzato
    if (await this.authorizedToViewPII()) {
      return await Promise.all(
        results.map(r => this.decrypt(r))
      )
    }

    return results
  }

  async exportForCompliance(
    startDate: Date,
    endDate: Date
  ): Promise<ComplianceReport> {
    const entries = await this.query({
      startTime: startDate.getTime(),
      endTime: endDate.getTime(),
      limit: Infinity
    })

    return {
      periodStart: startDate,
      periodEnd: endDate,
      totalOperations: entries.length,
      byUser: this.groupByUser(entries),
      byOperation: this.groupByOperation(entries),
      failuresByType: this.groupFailures(entries),
      sensitiveOperations: this.filterSensitive(entries)
    }
  }
}
```

---

## Autenticazione e Autorizzazione

### Autenticazione

```typescript
interface Authenticator {
  // Verifica credenziali
  async authenticate(credentials: Credentials): Promise<AuthResult>

  // Verifica token
  async verifyToken(token: string): Promise<TokenPayload>

  // Refresh token
  async refreshToken(refreshToken: string): Promise<Tokens>

  // Revoca token
  async revokeToken(token: string): Promise<void>
}

interface AuthResult {
  success: boolean
  userId: UUID
  sessionId: UUID
  tokens: Tokens
  permissions: Permission[]
}

interface Tokens {
  accessToken: string   // Breve durata (15 min)
  refreshToken: string  // Lunga durata (30 giorni)
  expiresAt: number
}

class JWTAuthenticator implements Authenticator {
  async authenticate(credentials: Credentials): Promise<AuthResult> {
    // Verifica username/password
    const user = await this.userStore.findByUsername(credentials.username)

    if (!user) {
      throw new AuthenticationError('User not found')
    }

    const passwordValid = await this.crypto.verify(
      credentials.password,
      user.passwordHash
    )

    if (!passwordValid) {
      // Log tentativo fallito
      await auditLog.record({
        action: 'authenticate',
        userId: user.id,
        success: false
      })

      throw new AuthenticationError('Invalid password')
    }

    // Crea sessione
    const session = await this.sessionStore.create(user.id)

    // Genera token
    const tokens = await this.generateTokens(user, session)

    // Ottieni permessi
    const permissions = await this.permissionStore.get(user.id)

    // Log autenticazione riuscita
    await auditLog.record({
      action: 'authenticate',
      userId: user.id,
      sessionId: session.id,
      success: true
    })

    return {
      success: true,
      userId: user.id,
      sessionId: session.id,
      tokens,
      permissions
    }
  }

  async verifyToken(token: string): Promise<TokenPayload> {
    try {
      const payload = jwt.verify(token, this.publicKey)

      // Controlla se revocato
      const revoked = await this.revokedTokens.has(token)
      if (revoked) {
        throw new AuthenticationError('Token revoked')
      }

      return payload

    } catch (error) {
      throw new AuthenticationError('Invalid token')
    }
  }
}
```

### Autorizzazione

```typescript
interface Authorizer {
  // Controlla se l'utente può eseguire operazione
  async authorize(
    user: User,
    operation: Operation,
    resource: Resource
  ): Promise<boolean>

  // Ottieni permessi utente
  async getPermissions(userId: UUID): Promise<Permission[]>
}

class RBACAuthorizer implements Authorizer {
  async authorize(
    user: User,
    operation: Operation,
    resource: Resource
  ): Promise<boolean> {

    // Ottieni ruoli utente
    const roles = await this.roleStore.getUserRoles(user.id)

    // Ottieni permessi per ruoli
    const permissions = await Promise.all(
      roles.map(role => this.permissionStore.getRolePermissions(role))
    )

    const allPermissions = permissions.flat()

    // Controlla se qualche permesso consente operazione
    for (const perm of allPermissions) {
      if (this.permissionAllows(perm, operation, resource)) {
        // Log successo autorizzazione
        await auditLog.record({
          action: 'authorize',
          userId: user.id,
          operation: operation.name,
          resource: resource.id,
          success: true
        })

        return true
      }
    }

    // Log fallimento autorizzazione
    await auditLog.record({
      action: 'authorize',
      userId: user.id,
      operation: operation.name,
      resource: resource.id,
      success: false
    })

    return false
  }

  private permissionAllows(
    permission: Permission,
    operation: Operation,
    resource: Resource
  ): boolean {
    // Controlla corrispondenza risorsa
    if (!this.resourceMatches(permission.resource, resource.id)) {
      return false
    }

    // Controlla azione consentita
    if (!permission.actions.includes(operation.action)) {
      return false
    }

    // Controlla condizioni
    if (permission.conditions) {
      for (const condition of permission.conditions) {
        if (!condition.evaluate(operation, resource)) {
          return false
        }
      }
    }

    return true
  }
}
```

---

## Gestione Segreti

### Store Segreti

```typescript
interface SecretsManager {
  // Salva segreto
  async storeSecret(
    name: string,
    value: string,
    metadata?: SecretMetadata
  ): Promise<void>

  // Recupera segreto
  async getSecret(name: string): Promise<string>

  // Elimina segreto
  async deleteSecret(name: string): Promise<void>

  // Ruota segreto
  async rotateSecret(name: string): Promise<string>

  // Lista segreti (solo nomi, non valori)
  async listSecrets(): Promise<SecretInfo[]>
}

interface SecretMetadata {
  description: string
  expiresAt?: number
  rotationInterval?: number
  tags: string[]
}

class EncryptedSecretsManager implements SecretsManager {
  async storeSecret(
    name: string,
    value: string,
    metadata?: SecretMetadata
  ): Promise<void> {
    // Cifra valore
    const encrypted = await this.crypto.encrypt(value, this.masterKey)

    // Salva valore cifrato
    await this.storage.set(name, {
      encrypted,
      metadata,
      createdAt: Date.now(),
      version: 1
    })

    // Log audit
    await auditLog.record({
      action: 'secret_stored',
      resource: name,
      success: true
    })

    // Pianifica rotazione se necessario
    if (metadata?.rotationInterval) {
      await this.scheduleRotation(name, metadata.rotationInterval)
    }
  }

  async getSecret(name: string): Promise<string> {
    const stored = await this.storage.get(name)

    if (!stored) {
      throw new Error(`Secret ${name} not found`)
    }

    // Controlla scadenza
    if (stored.metadata?.expiresAt && stored.metadata.expiresAt < Date.now()) {
      throw new Error(`Secret ${name} expired`)
    }

    // Decifra
    const decrypted = await this.crypto.decrypt(stored.encrypted, this.masterKey)

    // Log audit (NON includere valore segreto)
    await auditLog.record({
      action: 'secret_retrieved',
      resource: name,
      success: true
    })

    return decrypted
  }

  async rotateSecret(name: string): Promise<string> {
    // Genera nuovo segreto
    const newValue = await this.generateSecret()

    // Salva con versione incrementata
    const stored = await this.storage.get(name)
    const encrypted = await this.crypto.encrypt(newValue, this.masterKey)

    await this.storage.set(name, {
      encrypted,
      metadata: stored.metadata,
      createdAt: Date.now(),
      version: stored.version + 1
    })

    // Log audit
    await auditLog.record({
      action: 'secret_rotated',
      resource: name,
      oldVersion: stored.version,
      newVersion: stored.version + 1,
      success: true
    })

    return newValue
  }
}
```

---

## Guardrail di Safety

### Policy di Safety

```typescript
interface SafetyPolicy {
  // Operazioni pericolose richiedono conferma
  requireConfirmation: string[]  // nomi operazioni

  // Operazioni mai consentite
  blockedOperations: string[]

  // Restrizioni contenuti output
  contentPolicy: ContentPolicy

  // Limiti risorse (safety)
  maxResourcesPerTask: ResourceBudget

  // Limiti operazione autonoma
  maxAutonomousSteps: number
}

const DEFAULT_SAFETY_POLICY: SafetyPolicy = {
  requireConfirmation: [
    'delete_file',
    'drop_database',
    'send_email',
    'make_purchase',
    'execute_privileged_command'
  ],

  blockedOperations: [
    'format_disk',
    'delete_all',
    'disable_security'
  ],

  contentPolicy: {
    blockHate: true,
    blockViolence: true,
    blockSexual: true,
    blockSelfHarm: true,
    blockMalware: true
  },

  maxResourcesPerTask: {
    maxTokens: 100000,
    maxLLMCalls: 100,
    maxWallTime: 300000,
    maxCost: 10.0,
    maxDepth: 5
  },

  maxAutonomousSteps: 50
}
```

### Enforcer Safety

```typescript
class SafetyEnforcer {
  async enforceBeforeOperation(
    operation: Operation,
    policy: SafetyPolicy
  ): Promise<void> {

    // 1. Controlla se operazione bloccata
    if (policy.blockedOperations.includes(operation.name)) {
      logger.error('blocked_operation_attempted', {
        operation: operation.name
      })
      throw new OperationBlockedError(
        `Operation ${operation.name} is blocked by safety policy`
      )
    }

    // 2. Controlla se richiesta conferma
    if (policy.requireConfirmation.includes(operation.name)) {
      const confirmed = await this.requestUserConfirmation(operation)

      if (!confirmed) {
        throw new OperationCancelledError(
          'User did not confirm operation'
        )
      }
    }

    // 3. Controlla limite step autonomi
    if (operation.step > policy.maxAutonomousSteps) {
      throw new SafetyLimitError(
        'Maximum autonomous steps exceeded'
      )
    }
  }

  async enforceAfterOperation(
    result: OperationResult,
    policy: SafetyPolicy
  ): Promise<void> {

    // Controlla policy contenuti
    const contentViolation = await this.checkContentPolicy(
      result.output,
      policy.contentPolicy
    )

    if (contentViolation) {
      logger.error('content_policy_violation', {
        category: contentViolation.category,
        severity: contentViolation.severity
      })

      // Oscura o rifiuta
      if (contentViolation.severity === 'high') {
        throw new ContentPolicyViolationError(
          'Output violates content policy'
        )
      }
    }
  }

  private async requestUserConfirmation(
    operation: Operation
  ): Promise<boolean> {
    // Mostra all'utente cosa farà l'operazione
    const preview = await this.generateOperationPreview(operation)

    // Richiedi conferma
    return await userInterface.confirm({
      title: 'Confirm Operation',
      message: `The agent wants to ${operation.name}`,
      details: preview,
      options: ['Confirm', 'Cancel']
    })
  }
}
```

---

## Conformità

### Conformità GDPR

```typescript
interface GDPRCompliance {
  // Diritto di accesso
  async exportUserData(userId: UUID): Promise<UserDataExport>

  // Diritto alla cancellazione
  async deleteUserData(userId: UUID): Promise<DeletionReport>

  // Diritto di rettifica
  async updateUserData(userId: UUID, updates: any): Promise<void>

  // Diritto alla portabilità dati
  async exportInStandardFormat(userId: UUID): Promise<PortableData>

  // Record elaborazione dati
  async getProcessingRecords(
    startDate: Date,
    endDate: Date
  ): Promise<ProcessingRecord[]>
}

async function deleteUserData(userId: UUID): Promise<DeletionReport> {
  const report: DeletionReport = {
    userId,
    startTime: Date.now(),
    deletedItems: []
  }

  // Elimina da tutti gli store

  // 1. Episodi
  const episodes = await episodicMemory.getByUser(userId)
  for (const episode of episodes) {
    await episodicMemory.delete(episode.id)
    report.deletedItems.push({ type: 'episode', id: episode.id })
  }

  // 2. Memorie semantiche create dall'utente
  const facts = await semanticMemory.getByUser(userId)
  for (const fact of facts) {
    await semanticMemory.delete(fact.id)
    report.deletedItems.push({ type: 'fact', id: fact.id })
  }

  // 3. Log audit (mantieni versione anonimizzata per conformità)
  await auditLog.anonymizeUser(userId)

  // 4. Profilo utente
  await userStore.delete(userId)
  report.deletedItems.push({ type: 'user', id: userId })

  report.endTime = Date.now()
  report.duration = report.endTime - report.startTime

  // Audit della cancellazione
  await auditLog.record({
    action: 'gdpr_deletion',
    userId: '[DELETED]',
    success: true,
    metadata: { itemsDeleted: report.deletedItems.length }
  })

  return report
}
```

---

## Conclusione

L'Architettura Sicurezza e Safety fornisce protezione completa attraverso:

1. **Defense in Depth**: Sei livelli di sicurezza dall'input all'output
2. **Modellazione Minacce**: Identificazione sistematica di minacce e mitigazioni
3. **Validazione Input**: Enforcement schema e rilevamento injection
4. **Filtraggio Output**: Oscuramento PII e rimozione contenuti dannosi
5. **Sandboxing**: Esecuzione isolata con limiti risorse
6. **Rate Limiting**: Prevenzione abuso ed esaurimento risorse
7. **Audit Logging**: Record immutabile di tutti gli eventi di sicurezza
8. **Autenticazione e Autorizzazione**: RBAC con least privilege
9. **Gestione Segreti**: Storage cifrato e rotazione
10. **Guardrail Safety**: Conferma utente per azioni critiche
11. **Conformità**: GDPR, SOC 2 e altri requisiti normativi

Questo garantisce che l'agente operi in modo sicuro, protetto e conforme alle normative.

---

## Riferimenti

1. OWASP Top 10 (2024). Rischi sicurezza applicazioni web.
2. NIST Cybersecurity Framework (2024). Standard di sicurezza.
3. Regolamento GDPR (UE) 2016/679. Requisiti protezione dati.
4. Criteri SOC 2 Trust Services. Framework di conformità.
5. AWS Security Best Practices (2024). Pattern sicurezza cloud.
6. Microsoft Security Development Lifecycle. Pratiche sviluppo sicuro.
