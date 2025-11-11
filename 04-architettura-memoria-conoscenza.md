# Architettura della Memoria e della Conoscenza

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Principi Architetturali](#principi-architetturali)
3. [Tipi di Memoria](#tipi-di-memoria)
4. [Memoria di Lavoro](#memoria-di-lavoro)
5. [Memoria Episodica](#memoria-episodica)
6. [Memoria Semantica](#memoria-semantica)
7. [Memoria Procedurale](#memoria-procedurale)
8. [Grafi di Conoscenza](#grafi-di-conoscenza)
9. [Archivi Vettoriali](#archivi-vettoriali)
10. [Operazioni sulla Memoria](#operazioni-sulla-memoria)
11. [Gestione della Memoria](#gestione-della-memoria)
12. [Specifiche delle Interfacce](#specifiche-delle-interfacce)
13. [Schemi dei Dati](#schemi-dei-dati)
14. [Caratteristiche di Prestazione](#caratteristiche-di-prestazione)

---

## Panoramica

L'Architettura della Memoria e della Conoscenza fornisce all'agente uno storage e un recupero delle informazioni persistente, strutturato ed efficiente. Ispirata alle architetture cognitive e ai moderni sistemi di apprendimento automatico, combina molteplici tipi di memoria con punti di forza complementari.

### Obiettivi di Design

1. **Molteplici Tipi di Memoria**: Supportare diversi tipi di informazioni (episodi, fatti, competenze)
2. **Recupero Efficiente**: Accesso rapido alle informazioni rilevanti basato sul contesto
3. **Capacità Limitata**: La memoria di lavoro ha limiti fissi (vincolo realistico)
4. **Basato su Attivazione**: Le memorie hanno livelli di attivazione dinamici (recency, frequenza, rilevanza)
5. **Consolidamento**: Trasferimento da storage a breve termine a lungo termine
6. **Dimenticanza**: Rimozione o archiviazione di informazioni irrilevanti
7. **Tracciabilità**: Tutte le operazioni sulla memoria vengono registrate
8. **Persistenza**: Le memorie sopravvivono attraverso le sessioni

### Panoramica dell'Architettura della Memoria

```
┌─────────────────────────────────────────────────┐
│         Sistema di Memoria e Conoscenza         │
│                                                 │
│  ┌───────────────────────────────────────┐    │
│  │  Memoria di Lavoro (Limitata)         │    │
│  │  • Finestra di contesto (8K-128K token)│   │
│  │  • Focus corrente                      │    │
│  │  • Obiettivi attivi                    │    │
│  │  • Osservazioni recenti                │    │
│  └──────────┬────────────────────────────┘    │
│             │                                  │
│  ┌──────────▼──────────────────────────────┐  │
│  │  Motore di Recupero della Memoria      │  │
│  │  • Recupero basato su attivazione      │  │
│  │  • Ricerca per similarità              │  │
│  │  • Spreading activation                │  │
│  └──────────┬──────────────────────────────┘  │
│             │                                  │
│    ┌────────┴─────────┬──────────┬──────────┐ │
│    │                  │          │          │ │
│  ┌─▼──────────┐  ┌───▼─────┐  ┌▼────────┐ ┌▼────────┐
│  │ Memoria    │  │Memoria  │  │Memoria   │ │Archivio │
│  │ Episodica  │  │Semantica│  │Procedurale│ │Vettoriale│
│  │(Episodi)   │  │(Fatti)  │  │(Abilità) │ │(Embed)  │
│  └────────────┘  └─────────┘  └──────────┘ └─────────┘
│       │              │             │            │      │
│  ┌────▼──────────────▼─────────────▼────────────▼───┐ │
│  │         Livello di Storage Persistente           │ │
│  │  • Vector DB (embeddings)                       │ │
│  │  • Graph DB (conoscenza)                        │ │
│  │  • Document DB (episodi)                        │ │
│  │  • Key-Value Store (cache)                     │ │
│  └──────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## Principi Architetturali

### 1. Separazione per Tipo

Diversi tipi di memoria hanno caratteristiche diverse:

| Tipo di Memoria | Capacità | Velocità di Accesso | Struttura | Volatilità |
|-----------------|----------|---------------------|-----------|------------|
| Lavoro | Limitata (contesto) | Istantanea | Piatta | Volatile |
| Episodica | Illimitata | Media | Temporale | Persistente |
| Semantica | Illimitata | Veloce | Grafo | Persistente |
| Procedurale | Illimitata | Veloce | Template | Persistente |
| Vettoriale | Illimitata | Veloce | Vettori densi | Persistente |

### 2. Recupero Basato su Attivazione

Tutte le memorie hanno valori di attivazione che determinano la probabilità di recupero:

```
Activation = BaseLevel + SpreadingActivation + Noise

BaseLevel = log(∑ t_i^-d)  dove t_i = tempo dall'accesso i
SpreadingActivation = ∑ W_j × S_ij  dove W_j = attivazione della sorgente
Noise = N(0, σ)
```

Attivazione alta → Alta probabilità di recupero
Attivazione bassa → Bassa probabilità di recupero (eventuale dimenticanza)

### 3. Storage Ibrido

Combinare più backend di storage per prestazioni ottimali:

- **Vector DB**: Per ricerca per similarità semantica (es., Pinecone, Weaviate)
- **Graph DB**: Per conoscenza strutturata (es., Neo4j, Memgraph)
- **Document DB**: Per storage degli episodi (es., MongoDB, PostgreSQL)
- **Key-Value**: Per caching veloce (es., Redis)

### 4. Consolidamento della Memoria

Trasferire informazioni dalla memoria di lavoro alla memoria a lungo termine:

```
Memoria di Lavoro (Breve termine)
       ↓
   Ripetizione/Uso
       ↓
Memoria a Lungo Termine (Persistente)
       ↓
   Consolidamento
       ↓
Semantica/Procedurale (Generalizzata)
```

### 5. Meccanismi di Dimenticanza

Non tutte le memorie devono persistere per sempre:

- **Decadimento**: L'attivazione diminuisce nel tempo
- **Interferenza**: Memorie simili competono
- **Archiviazione**: Spostamento di vecchie memorie in cold storage
- **Cancellazione**: Rimozione di memorie sotto la soglia

---

## Tipi di Memoria

### Confronto tra Tipi di Memoria

```
┌────────────────────────────────────────────────────┐
│  "Cosa è successo?"    →  Memoria Episodica        │
│  "Cos'è X?"            →  Memoria Semantica        │
│  "Come faccio X?"      →  Memoria Procedurale      │
│  "Cosa sto pensando?"  →  Memoria di Lavoro        │
│  "Cosa è simile?"      →  Archivio Vettoriale      │
└────────────────────────────────────────────────────┘
```

---

## Memoria di Lavoro

### Panoramica

La memoria di lavoro contiene il focus corrente dell'agente - obiettivi, osservazioni recenti, tracce di ragionamento. Ha limiti di capacità stretti (finestra di contesto).

### Architettura

```typescript
interface WorkingMemory {
  // Content slots
  slots: {
    goals: Goal[]                    // Obiettivi attivi
    observations: Observation[]      // Percezioni recenti
    reasoning: ReasoningTrace[]      // Pensieri correnti
    context: ContextChunk[]          // Memoria recuperata rilevante
  }

  // Capacità
  maxTokens: number                  // es., 128000
  currentTokens: number

  // Operazioni
  add(chunk: Chunk): void
  remove(chunkId: UUID): void
  clear(): void
  getContents(): Chunk[]
  estimateTokens(chunk: Chunk): number
}
```

### Gestione del Contenuto

La memoria di lavoro utilizza **eviction basata su attivazione**:

```typescript
interface Chunk {
  id: UUID
  content: any
  tokens: number
  activation: number
  lastAccess: number
  accessCount: number
  protected: boolean  // Non può essere rimosso
}

function addToWorkingMemory(chunk: Chunk): void {
  // Controlla capacità
  while (currentTokens + chunk.tokens > maxTokens) {
    evictLowestActivation()
  }

  // Aggiungi chunk
  slots.push(chunk)
  currentTokens += chunk.tokens

  // Aggiorna attivazione
  chunk.activation = computeActivation(chunk)
}

function evictLowestActivation(): void {
  // Trova chunk non protetto con attivazione più bassa
  const evictable = slots.filter(c => !c.protected)
  const lowest = evictable.reduce((min, c) =>
    c.activation < min.activation ? c : min
  )

  // Rimuovi
  slots = slots.filter(c => c.id !== lowest.id)
  currentTokens -= lowest.tokens
}
```

### Slot della Memoria di Lavoro

Slot diversi per diversi tipi di contenuto:

```typescript
interface WorkingMemorySlots {
  // Slot di sistema (sempre protetto)
  system: {
    instructions: string
    capabilities: string[]
    constraints: string[]
  }

  // Slot obiettivi (alta priorità)
  goals: {
    current: Goal
    stack: Goal[]
    maxGoals: 5
  }

  // Slot osservazioni (percezioni recenti)
  observations: {
    recent: Observation[]
    maxObservations: 10
    windowSize: 5  // minuti
  }

  // Slot ragionamento (pensiero corrente)
  reasoning: {
    traces: ReasoningTrace[]
    maxTraces: 5
  }

  // Slot contesto recuperato (dalla memoria a lungo termine)
  context: {
    retrieved: Memory[]
    maxTokens: 50000  // ~50% della memoria di lavoro
  }

  // Slot scratchpad (spazio di lavoro temporaneo)
  scratchpad: {
    content: any
    maxTokens: 10000
  }
}
```

### Caratteristiche di Prestazione

- **Tempo di Accesso**: O(1) - tutto in memoria
- **Capacità**: Fissa (finestra di contesto)
- **Persistenza**: Volatile (cancellata tra le sessioni)
- **Recupero**: Accesso diretto, nessuna ricerca

---

## Memoria Episodica

### Panoramica

La memoria episodica memorizza **esperienze** - sequenze di stati, azioni e risultati. Risponde a "Cosa è successo quando ho provato X?"

### Schema Dati

```typescript
interface Episode {
  id: UUID

  // Informazioni temporali
  startTime: number
  endTime: number
  duration: number

  // Obiettivo e piano
  goal: Goal
  plan: Plan

  // Esecuzione
  steps: EpisodeStep[]
  outcome: Outcome

  // Risorse
  resourcesUsed: ResourceUsage

  // Metadata
  tags: string[]
  embedding: number[]  // Per ricerca per similarità

  // Riflessione
  reflection: Reflection | null

  // Attivazione
  activation: number
  accessCount: number
  lastAccess: number
}

interface EpisodeStep {
  id: UUID
  timestamp: number
  state: State
  action: Action
  observation: Observation
  reasoning: ReasoningTrace
  result: StepResult
}
```

### Storage

Episodi memorizzati in un database documentale con:
- Ricerca full-text sulle descrizioni
- Vettori di embedding per ricerca per similarità
- Indicizzazione temporale (query basate sul tempo)
- Indicizzazione dei tag (query basate su categorie)

### Recupero

```typescript
interface EpisodicMemory {
  // Memorizza nuovo episodio
  async store(episode: Episode): Promise<void>

  // Recupera per similarità
  async retrieveSimilar(
    cue: EpisodeCue,
    limit: number
  ): Promise<Episode[]>

  // Recupera per intervallo temporale
  async retrieveByTimeRange(
    start: number,
    end: number
  ): Promise<Episode[]>

  // Recupera per similarità dell'obiettivo
  async retrieveByGoal(
    goal: Goal,
    limit: number
  ): Promise<Episode[]>

  // Recupera per risultato
  async retrieveByOutcome(
    outcomeType: OutcomeType,
    limit: number
  ): Promise<Episode[]>
}

interface EpisodeCue {
  goal?: Goal
  context?: Context
  tags?: string[]
  minActivation?: number
  timeRange?: [number, number]
}
```

### Similarità degli Episodi

Calcola la similarità tra episodi:

```typescript
function computeEpisodeSimilarity(
  episode1: Episode,
  episode2: Episode
): number {
  // Similarità dell'obiettivo
  const goalSim = cosineSimilarity(
    episode1.goal.embedding,
    episode2.goal.embedding
  )

  // Similarità del contesto
  const contextSim = cosineSimilarity(
    episode1.embedding,
    episode2.embedding
  )

  // Similarità del risultato
  const outcomeSim = episode1.outcome.type === episode2.outcome.type ? 1 : 0

  // Combinazione pesata
  return 0.4 * goalSim + 0.4 * contextSim + 0.2 * outcomeSim
}
```

### Consolidamento

Consolida le memorie episodiche in conoscenza semantica:

```typescript
async function consolidateEpisodes(
  episodes: Episode[]
): Promise<SemanticFact[]> {
  // Raggruppa episodi simili
  const clusters = clusterEpisodes(episodes)

  const facts: SemanticFact[] = []

  for (const cluster of clusters) {
    // Estrai pattern comuni
    const pattern = extractPattern(cluster)

    // Crea fatto semantico
    const fact = {
      type: 'pattern',
      description: pattern.description,
      evidence: cluster.map(e => e.id),
      confidence: computeConfidence(cluster),
      generalizability: assessGeneralizability(pattern)
    }

    facts.push(fact)
  }

  return facts
}
```

---

## Memoria Semantica

### Panoramica

La memoria semantica memorizza **fatti e concetti** - conoscenza dichiarativa sul mondo. Risponde a "Cos'è X?" e "Cosa so su Y?"

### Schema Dati

```typescript
interface SemanticFact {
  id: UUID

  // Contenuto
  subject: string
  predicate: string
  object: string

  // O non strutturato
  description: string

  // Metadata
  type: FactType
  confidence: number  // 0-1
  source: Source
  createdAt: number
  updatedAt: number

  // Attivazione
  activation: number
  accessCount: number
  lastAccess: number

  // Embedding per similarità
  embedding: number[]

  // Relazioni
  relatedFacts: UUID[]
}

enum FactType {
  DEFINITION = 'definition',
  PROPERTY = 'property',
  RELATION = 'relation',
  RULE = 'rule',
  CONSTRAINT = 'constraint',
  HEURISTIC = 'heuristic'
}

interface Source {
  type: 'experience' | 'told' | 'inferred' | 'external'
  reference: string
  reliability: number
}
```

### Grafo di Conoscenza

Memoria semantica organizzata come grafo di conoscenza:

```typescript
interface KnowledgeGraph {
  // Nodi (entità/concetti)
  nodes: Map<UUID, Node>

  // Archi (relazioni)
  edges: Map<UUID, Edge>

  // Operazioni
  async addNode(node: Node): Promise<void>
  async addEdge(edge: Edge): Promise<void>
  async query(query: GraphQuery): Promise<GraphResult>
  async getNeighbors(nodeId: UUID, depth: number): Promise<Node[]>
  async findPaths(from: UUID, to: UUID, maxLength: number): Promise<Path[]>
  async spreadActivation(sourceNodes: UUID[]): Promise<Map<UUID, number>>
}

interface Node {
  id: UUID
  type: NodeType
  label: string
  properties: Record<string, any>
  embedding: number[]
  activation: number
}

interface Edge {
  id: UUID
  type: EdgeType
  from: UUID
  to: UUID
  properties: Record<string, any>
  weight: number
}

enum NodeType {
  CONCEPT = 'concept',
  ENTITY = 'entity',
  EVENT = 'event',
  PROPERTY = 'property',
  VALUE = 'value'
}

enum EdgeType {
  IS_A = 'is_a',
  HAS_PROPERTY = 'has_property',
  RELATED_TO = 'related_to',
  CAUSES = 'causes',
  PART_OF = 'part_of',
  INSTANCE_OF = 'instance_of'
}
```

### Query sul Grafo

Interroga il grafo di conoscenza utilizzando pattern di grafi:

```typescript
interface GraphQuery {
  pattern: GraphPattern
  constraints: Constraint[]
  limit: number
  orderBy: OrderBy
}

interface GraphPattern {
  nodes: NodePattern[]
  edges: EdgePattern[]
}

// Esempio: Trova tutti gli strumenti che possono risolvere l'obiettivo X
const query: GraphQuery = {
  pattern: {
    nodes: [
      { var: 'goal', type: NodeType.CONCEPT, label: 'goal_x' },
      { var: 'tool', type: NodeType.ENTITY }
    ],
    edges: [
      { from: 'tool', to: 'goal', type: EdgeType.CAUSES }
    ]
  },
  constraints: [
    { var: 'tool', property: 'available', value: true }
  ],
  limit: 10,
  orderBy: { var: 'tool', property: 'activation', order: 'DESC' }
}
```

### Spreading Activation

Attiva nodi correlati nel grafo di conoscenza:

```typescript
async function spreadActivation(
  sourceNodes: UUID[],
  iterations: number,
  decayFactor: number
): Promise<Map<UUID, number>> {

  // Inizializza attivazione
  const activation = new Map<UUID, number>()
  sourceNodes.forEach(id => activation.set(id, 1.0))

  // Diffonde l'attivazione iterativamente
  for (let i = 0; i < iterations; i++) {
    const newActivation = new Map<UUID, number>()

    for (const [nodeId, act] of activation) {
      // Ottieni vicini
      const neighbors = await graph.getNeighbors(nodeId, 1)

      // Diffonde ai vicini
      for (const neighbor of neighbors) {
        const edge = await graph.getEdge(nodeId, neighbor.id)
        const spread = act * edge.weight * decayFactor

        const current = newActivation.get(neighbor.id) || 0
        newActivation.set(neighbor.id, current + spread)
      }
    }

    // Aggiorna attivazione
    newActivation.forEach((act, id) => {
      const current = activation.get(id) || 0
      activation.set(id, current + act)
    })
  }

  return activation
}
```

---

## Memoria Procedurale

### Panoramica

La memoria procedurale memorizza **abilità e pattern** - come fare le cose. Risponde a "Come faccio a compiere X?"

### Schema Dati

```typescript
interface Procedure {
  id: UUID

  // Identità
  name: string
  description: string

  // Applicabilità
  preconditions: Condition[]
  context: ContextPattern

  // Passi
  steps: ProcedureStep[]

  // Varianti
  variants: Procedure[]

  // Prestazioni
  successRate: number
  avgDuration: number
  avgCost: number

  // Utilizzo
  useCount: number
  lastUsed: number
  activation: number

  // Metadata
  source: Source
  tags: string[]
  embedding: number[]
}

interface ProcedureStep {
  action: Action
  expected: Expectation
  alternatives: ProcedureStep[]
  optional: boolean
}

interface ContextPattern {
  goalType: GoalType
  domainHints: string[]
  featureVector: number[]
}
```

### Storage e Recupero

```typescript
interface ProceduralMemory {
  // Memorizza procedura
  async store(procedure: Procedure): Promise<void>

  // Recupera per obiettivo/contesto
  async retrieveApplicable(
    goal: Goal,
    context: Context,
    limit: number
  ): Promise<Procedure[]>

  // Aggiorna statistiche di prestazione
  async updatePerformance(
    procedureId: UUID,
    success: boolean,
    duration: number,
    cost: number
  ): Promise<void>

  // Ottieni procedure simili
  async findSimilar(
    procedure: Procedure,
    limit: number
  ): Promise<Procedure[]>
}
```

### Selezione della Procedura

Seleziona la procedura migliore per la situazione corrente:

```typescript
async function selectProcedure(
  goal: Goal,
  context: Context,
  procedures: Procedure[]
): Promise<Procedure> {

  // Filtra applicabili
  const applicable = procedures.filter(p =>
    p.preconditions.every(c => c.evaluate(context.state))
  )

  if (applicable.length === 0) {
    throw new Error('No applicable procedures')
  }

  // Assegna un punteggio a ogni procedura
  const scored = applicable.map(p => ({
    procedure: p,
    score: scoreProcedure(p, goal, context)
  }))

  // Ordina per punteggio
  scored.sort((a, b) => b.score - a.score)

  // Ritorna la migliore
  return scored[0].procedure
}

function scoreProcedure(
  procedure: Procedure,
  goal: Goal,
  context: Context
): number {
  // Fattori
  const similarity = cosineSimilarity(
    procedure.embedding,
    goal.embedding
  )

  const successRate = procedure.successRate
  const recency = computeRecency(procedure.lastUsed)
  const activation = procedure.activation

  // Vincoli di risorsa
  const fitsConstraints =
    procedure.avgCost <= context.budget.maxCost &&
    procedure.avgDuration <= context.budget.maxWallTime

  const constraintBonus = fitsConstraints ? 1.0 : 0.5

  // Punteggio pesato
  return (
    0.3 * similarity +
    0.3 * successRate +
    0.2 * recency +
    0.2 * activation
  ) * constraintBonus
}
```

### Apprendimento delle Procedure

Apprende nuove procedure da episodi di successo:

```typescript
async function learnProcedure(
  episode: Episode
): Promise<Procedure | null> {

  // Impara solo da episodi di successo
  if (!episode.outcome.success) {
    return null
  }

  // Estrai procedura dall'episodio
  const procedure: Procedure = {
    id: generateUUID(),
    name: generateName(episode.goal),
    description: episode.goal.description,
    preconditions: inferPreconditions(episode),
    context: extractContextPattern(episode),
    steps: episode.steps.map(s => ({
      action: s.action,
      expected: s.result,
      alternatives: [],
      optional: false
    })),
    variants: [],
    successRate: 1.0,  // Ottimismo iniziale
    avgDuration: episode.duration,
    avgCost: episode.resourcesUsed.cost,
    useCount: 1,
    lastUsed: Date.now(),
    activation: 1.0,
    source: { type: 'experience', reference: episode.id, reliability: 1.0 },
    tags: episode.tags,
    embedding: episode.embedding
  }

  // Controlla procedure esistenti simili
  const similar = await proceduralMemory.findSimilar(procedure, 5)

  if (similar.length > 0) {
    // Unisci o aggiorna esistente
    await mergeOrUpdateProcedure(procedure, similar)
    return null
  }

  // Memorizza nuova procedura
  await proceduralMemory.store(procedure)
  return procedure
}
```

---

## Grafi di Conoscenza

### Schema del Grafo

Conoscenza rappresentata come grafo tipizzato e con attributi:

```
Nodi: Entità, Concetti, Eventi
Archi: Relazioni con proprietà
Proprietà: Attributi chiave-valore

Esempio:
(Tool:browser_action)-[:CAN_SOLVE]->(Goal:open_website)
(Concept:planning)-[:IS_A]->(Concept:reasoning)
(Error:timeout)-[:REQUIRES]->(Action:retry)
```

### Operazioni sul Grafo

```typescript
interface KnowledgeGraphOps {
  // Operazioni CRUD
  async createNode(node: Node): Promise<UUID>
  async createEdge(edge: Edge): Promise<UUID>
  async updateNode(id: UUID, updates: Partial<Node>): Promise<void>
  async deleteNode(id: UUID): Promise<void>

  // Operazioni di query
  async queryPattern(pattern: GraphPattern): Promise<Node[]>
  async shortestPath(from: UUID, to: UUID): Promise<Path>
  async findCommunities(): Promise<Community[]>
  async pageRank(): Promise<Map<UUID, number>>

  // Operazioni di ragionamento
  async inferEdges(rules: InferenceRule[]): Promise<Edge[]>
  async checkConsistency(): Promise<ConsistencyReport>
  async explainPath(path: Path): Promise<Explanation>
}
```

### Regole di Inferenza

Supporta inferenza basata su regole:

```typescript
interface InferenceRule {
  name: string
  pattern: GraphPattern
  conclusion: EdgeTemplate
  confidence: number
}

// Esempio: Transitività della relazione IS_A
const transitivityRule: InferenceRule = {
  name: 'is_a_transitive',
  pattern: {
    nodes: [
      { var: 'a', type: NodeType.CONCEPT },
      { var: 'b', type: NodeType.CONCEPT },
      { var: 'c', type: NodeType.CONCEPT }
    ],
    edges: [
      { from: 'a', to: 'b', type: EdgeType.IS_A },
      { from: 'b', to: 'c', type: EdgeType.IS_A }
    ]
  },
  conclusion: {
    from: 'a',
    to: 'c',
    type: EdgeType.IS_A,
    properties: { inferred: true }
  },
  confidence: 0.95
}
```

---

## Archivi Vettoriali

### Panoramica

Gli archivi vettoriali consentono la ricerca per similarità semantica utilizzando embeddings densi.

### Architettura

```typescript
interface VectorStore {
  // Memorizza vettore
  async store(
    id: UUID,
    vector: number[],
    metadata: Record<string, any>
  ): Promise<void>

  // Ricerca per similarità
  async search(
    query: number[],
    limit: number,
    filter?: Filter
  ): Promise<VectorMatch[]>

  // Operazioni batch
  async storeBatch(items: VectorItem[]): Promise<void>

  // Gestisce collezioni
  async createCollection(name: string, dimension: number): Promise<void>
  async deleteCollection(name: string): Promise<void>
}

interface VectorMatch {
  id: UUID
  score: number  // Punteggio di similarità (0-1)
  metadata: Record<string, any>
}

interface VectorItem {
  id: UUID
  vector: number[]
  metadata: Record<string, any>
}
```

### Strategia di Embedding

Diversi modelli di embedding per diversi contenuti:

```typescript
interface EmbeddingService {
  // Embedding di testo
  async embedText(
    text: string,
    model: EmbeddingModel
  ): Promise<number[]>

  // Embedding di obiettivi
  async embedGoal(goal: Goal): Promise<number[]>

  // Embedding di episodi
  async embedEpisode(episode: Episode): Promise<number[]>

  // Embedding di codice
  async embedCode(code: string, language: string): Promise<number[]>
}

enum EmbeddingModel {
  TEXT_SMALL = 'text-embedding-3-small',      // 1536 dim
  TEXT_LARGE = 'text-embedding-3-large',      // 3072 dim
  CODE = 'code-embedding-ada',                // 1536 dim
  MULTIMODAL = 'multimodal-embedding'         // 1024 dim
}
```

### Ricerca Ibrida

Combina ricerca vettoriale con filtraggio:

```typescript
async function hybridSearch(
  query: string,
  filters: Filter[],
  limit: number
): Promise<SearchResult[]> {

  // 1. Crea embedding della query
  const queryVector = await embeddings.embedText(query, EmbeddingModel.TEXT_LARGE)

  // 2. Ricerca vettoriale (ottieni più di limit)
  const vectorMatches = await vectorStore.search(
    queryVector,
    limit * 3,  // Over-retrieve
    null
  )

  // 3. Applica filtri
  const filtered = vectorMatches.filter(m =>
    filters.every(f => f.evaluate(m.metadata))
  )

  // 4. Riordina usando segnali aggiuntivi
  const reranked = await rerank(filtered, query)

  // 5. Ritorna top-k
  return reranked.slice(0, limit)
}
```

---

## Operazioni sulla Memoria

### Operazioni Core

```typescript
interface MemorySystem {
  // Operazioni di memorizzazione
  async storeEpisode(episode: Episode): Promise<void>
  async storeFact(fact: SemanticFact): Promise<void>
  async storeProcedure(procedure: Procedure): Promise<void>

  // Operazioni di recupero
  async retrieve(cue: MemoryCue): Promise<Memory[]>
  async retrieveEpisodes(cue: EpisodeCue): Promise<Episode[]>
  async retrieveFacts(cue: FactCue): Promise<SemanticFact[]>
  async retrieveProcedures(cue: ProcedureCue): Promise<Procedure[]>

  // Operazioni di aggiornamento
  async updateActivation(memoryId: UUID): Promise<void>
  async updateMemory(memoryId: UUID, updates: Partial<Memory>): Promise<void>

  // Operazioni di gestione
  async consolidate(): Promise<ConsolidationReport>
  async forget(threshold: number): Promise<UUID[]>
  async archive(memoryIds: UUID[]): Promise<void>
}
```

### Memory Cue

Interfaccia unificata per il recupero della memoria:

```typescript
interface MemoryCue {
  // Query testuale
  query?: string

  // Similarità semantica
  embedding?: number[]

  // Filtri
  filters?: Filter[]

  // Soglia di attivazione
  minActivation?: number

  // Intervallo temporale
  timeRange?: [number, number]

  // Limite
  limit: number

  // Contesto per spreading activation
  contextNodes?: UUID[]
}
```

### Algoritmo di Recupero

Recupero unificato tra i tipi di memoria:

```typescript
async function retrieve(cue: MemoryCue): Promise<Memory[]> {
  const results: Memory[] = []

  // 1. Se fornita query testuale, crea embedding
  let queryVector: number[] | null = null
  if (cue.query) {
    queryVector = await embeddings.embedText(cue.query, EmbeddingModel.TEXT_LARGE)
  } else if (cue.embedding) {
    queryVector = cue.embedding
  }

  // 2. Ricerca vettoriale se disponibile embedding
  if (queryVector) {
    const vectorMatches = await vectorStore.search(
      queryVector,
      cue.limit * 2,
      null
    )

    // Recupera memorie complete
    for (const match of vectorMatches) {
      const memory = await fetchMemory(match.id)
      if (memory) {
        results.push({ ...memory, similarity: match.score })
      }
    }
  }

  // 3. Applica filtri
  let filtered = results
  if (cue.filters && cue.filters.length > 0) {
    filtered = results.filter(m =>
      cue.filters!.every(f => f.evaluate(m))
    )
  }

  // 4. Filtra per soglia di attivazione
  if (cue.minActivation !== undefined) {
    filtered = filtered.filter(m => m.activation >= cue.minActivation!)
  }

  // 5. Filtra per intervallo temporale
  if (cue.timeRange) {
    const [start, end] = cue.timeRange
    filtered = filtered.filter(m =>
      m.timestamp >= start && m.timestamp <= end
    )
  }

  // 6. Applica spreading activation se fornito contesto
  if (cue.contextNodes && cue.contextNodes.length > 0) {
    const activation = await knowledgeGraph.spreadActivation(cue.contextNodes)

    // Aumenta memorie correlate a nodi attivati
    filtered = filtered.map(m => ({
      ...m,
      activation: m.activation + (activation.get(m.id) || 0)
    }))
  }

  // 7. Ordina per punteggio combinato
  filtered.sort((a, b) => {
    const scoreA = computeRetrievalScore(a)
    const scoreB = computeRetrievalScore(b)
    return scoreB - scoreA
  })

  // 8. Ritorna top-k
  return filtered.slice(0, cue.limit)
}

function computeRetrievalScore(memory: Memory): number {
  return (
    0.4 * memory.activation +
    0.3 * (memory.similarity || 0) +
    0.3 * computeRecency(memory.lastAccess)
  )
}
```

---

## Gestione della Memoria

### Consolidamento

Trasferisce memorie importanti a semantica/procedurale:

```typescript
async function consolidate(): Promise<ConsolidationReport> {
  const report: ConsolidationReport = {
    episodesProcessed: 0,
    factsCreated: 0,
    proceduresCreated: 0,
    duration: 0
  }

  const startTime = Date.now()

  // 1. Ottieni episodi recenti
  const episodes = await episodicMemory.retrieveByTimeRange(
    Date.now() - CONSOLIDATION_WINDOW,
    Date.now()
  )

  report.episodesProcessed = episodes.length

  // 2. Estrai pattern → fatti semantici
  const facts = await consolidateEpisodes(episodes)
  for (const fact of facts) {
    await semanticMemory.store(fact)
    report.factsCreated++
  }

  // 3. Estrai procedure
  for (const episode of episodes) {
    if (episode.outcome.success) {
      const procedure = await learnProcedure(episode)
      if (procedure) {
        report.proceduresCreated++
      }
    }
  }

  report.duration = Date.now() - startTime

  return report
}
```

### Dimenticanza

Rimuove memorie con attivazione bassa:

```typescript
async function forget(threshold: number): Promise<UUID[]> {
  const forgotten: UUID[] = []

  // Trova memorie sotto la soglia
  const candidates = await findLowActivation(threshold)

  for (const memory of candidates) {
    // Non dimenticare se recentemente acceduto
    if (Date.now() - memory.lastAccess < PROTECTION_PERIOD) {
      continue
    }

    // Non dimenticare se alta importanza
    if (memory.importance > IMPORTANCE_THRESHOLD) {
      continue
    }

    // Archivia o cancella
    if (memory.importance > 0.5) {
      await archive(memory)
    } else {
      await delete(memory.id)
    }

    forgotten.push(memory.id)
  }

  return forgotten
}
```

### Decadimento

Aggiorna le attivazioni in base al tempo:

```typescript
function updateActivation(memory: Memory): number {
  const now = Date.now()
  const timeSince = now - memory.lastAccess

  // Decadimento dell'attivazione base
  const decayedBase = memory.activation * Math.exp(-DECAY_RATE * timeSince)

  // Aggiungi componente di frequenza
  const frequency = Math.log(memory.accessCount + 1)

  // Nuova attivazione
  memory.activation = decayedBase + FREQUENCY_WEIGHT * frequency

  return memory.activation
}

// Decadimento periodico di tutte le memorie
async function decayAll(): Promise<void> {
  const allMemories = await getAllMemories()

  for (const memory of allMemories) {
    updateActivation(memory)
    await updateMemory(memory.id, { activation: memory.activation })
  }
}
```

---

## Specifiche delle Interfacce

### Interfaccia Completa del Sistema di Memoria

```typescript
interface MemorySystem {
  // Memoria di lavoro
  workingMemory: WorkingMemory

  // Memoria a lungo termine
  episodicMemory: EpisodicMemory
  semanticMemory: SemanticMemory
  proceduralMemory: ProceduralMemory

  // Infrastruttura
  vectorStore: VectorStore
  knowledgeGraph: KnowledgeGraph

  // Recupero unificato
  async retrieve(cue: MemoryCue): Promise<Memory[]>

  // Gestione
  async consolidate(): Promise<ConsolidationReport>
  async forget(threshold: number): Promise<UUID[]>
  async archive(memoryIds: UUID[]): Promise<void>
  async decayAll(): Promise<void>

  // Statistiche
  async getStats(): Promise<MemoryStats>
}

interface MemoryStats {
  workingMemory: {
    currentTokens: number
    maxTokens: number
    utilization: number
  }

  episodicMemory: {
    totalEpisodes: number
    avgEpisodeDuration: number
    successRate: number
  }

  semanticMemory: {
    totalFacts: number
    totalNodes: number
    totalEdges: number
    avgDegree: number
  }

  proceduralMemory: {
    totalProcedures: number
    avgSuccessRate: number
    avgUsageCount: number
  }

  vectorStore: {
    totalVectors: number
    dimension: number
    indexSize: number
  }
}
```

---

## Schemi dei Dati

### Schemi JSON

Tutte le strutture di memoria serializzabili in JSON:

```json
{
  "episode": {
    "id": "uuid",
    "startTime": 1699564800000,
    "endTime": 1699565400000,
    "duration": 600000,
    "goal": { ... },
    "plan": { ... },
    "steps": [ ... ],
    "outcome": { ... },
    "resourcesUsed": { ... },
    "tags": ["coding", "python"],
    "embedding": [0.1, 0.2, ...],
    "activation": 0.85,
    "accessCount": 3,
    "lastAccess": 1699570000000
  },

  "semantic_fact": {
    "id": "uuid",
    "subject": "Python",
    "predicate": "is_a",
    "object": "programming_language",
    "confidence": 0.95,
    "source": {
      "type": "experience",
      "reference": "episode_uuid",
      "reliability": 1.0
    },
    "activation": 0.9,
    "embedding": [0.3, 0.1, ...]
  },

  "procedure": {
    "id": "uuid",
    "name": "create_python_script",
    "description": "Create a new Python script file",
    "preconditions": [ ... ],
    "steps": [ ... ],
    "successRate": 0.92,
    "avgDuration": 5000,
    "avgCost": 100,
    "useCount": 15,
    "activation": 0.88
  }
}
```

---

## Caratteristiche di Prestazione

### Analisi della Complessità

| Operazione | Complessità Temporale | Complessità Spaziale |
|------------|----------------------|---------------------|
| Memorizza Episodio | O(1) + O(d) embedding | O(n) dimensione episodio |
| Recupera Episodi | O(log n) + O(k) | O(k) risultati |
| Memorizza Fatto | O(1) + O(d) embedding | O(1) |
| Query su Grafo | O(E × V) | O(V + E) |
| Ricerca Vettoriale | O(log n) approssimato | O(d) dimensione |
| Spread Activation | O(i × E) | O(V) |
| Consolidamento | O(n × m) | O(n + f) |

Dove:
- n = numero di memorie
- d = dimensione embedding
- k = limite risultati
- E = archi, V = vertici
- i = iterazioni
- m = complessità estrazione pattern
- f = fatti creati

### Scalabilità

**Memoria di Lavoro**:
- Capacità: Fissa dalla finestra di contesto (8K-128K token)
- Scalabilità: N/A (limitata)

**Memoria Episodica**:
- Storage: Illimitato (database documentale)
- Recupero: Sub-secondo per milioni di episodi (indicizzato)
- Scalabilità: Orizzontale (sharding per tempo o contenuto)

**Memoria Semantica**:
- Storage: Miliardi di nodi possibili
- Recupero: Millisecondi (database grafo ottimizzato)
- Scalabilità: Orizzontale (grafo distribuito)

**Archivio Vettoriale**:
- Storage: Miliardi di vettori supportati
- Ricerca: Sub-secondo per miliardi (ANN approssimato)
- Scalabilità: Orizzontale (multiple shard)

### Requisiti di Risorse

```typescript
interface ResourceEstimates {
  workingMemory: {
    memory: '100-500 MB',
    latency: '<1 ms'
  }

  episodicMemory: {
    storage: '~1 KB per episodio',
    retrieval: '10-100 ms per query'
  }

  semanticMemory: {
    storage: '~100 bytes per fatto',
    query: '1-50 ms per query'
  }

  vectorStore: {
    storage: '~4-12 KB per embedding',
    search: '10-50 ms per query'
  }
}
```

---

## Conclusione

L'Architettura della Memoria e della Conoscenza fornisce una base comprensiva, efficiente e scalabile per la persistenza dell'agente. Combinando molteplici tipi di memoria con punti di forza complementari:

- **Memoria di Lavoro**: Veloce, limitata, focus corrente
- **Memoria Episodica**: Registri completi delle esperienze
- **Memoria Semantica**: Conoscenza fattuale strutturata
- **Memoria Procedurale**: Pattern di abilità riutilizzabili
- **Archivi Vettoriali**: Ricerca per similarità semantica
- **Grafi di Conoscenza**: Ragionamento relazionale

Il sistema supporta recupero basato su attivazione, consolidamento della memoria, meccanismi di dimenticanza e storage distribuito - tutti essenziali per un agente universale di livello produttivo.

Tutti i componenti hanno interfacce ben definite, schemi dati chiari e caratteristiche di prestazione note, garantendo che il sistema soddisfi i requisiti di prestazione, tracciabilità e testabilità.

---

## Riferimenti

1. Anderson, J. R. (2007). How Can the Human Mind Occur in the Physical Universe? Oxford University Press.
2. Tulving, E. (2002). Episodic memory: From mind to brain. Annual Review of Psychology, 53, 1-25.
3. Collins, A. M., & Loftus, E. F. (1975). A spreading-activation theory of semantic processing. Psychological Review, 82(6), 407-428.
4. Weaviate Documentation (2024). Vector database architecture.
5. Neo4j Graph Data Science (2024). Graph algorithms and applications.
6. Lewis, P., et al. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. NeurIPS 2020.
7. Laird, J. E. (2012). The Soar Cognitive Architecture. MIT Press - Memory systems chapter.
