# Diagrammi dell'Architettura di Sistema

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Architettura di Sistema ad Alto Livello](#architettura-di-sistema-ad-alto-livello)
3. [Diagramma dei Componenti](#diagramma-dei-componenti)
4. [Flusso del Motore di Ragionamento](#flusso-del-motore-di-ragionamento)
5. [Architettura del Sistema di Memoria](#architettura-del-sistema-di-memoria)
6. [Flusso di Esecuzione degli Strumenti](#flusso-di-esecuzione-degli-strumenti)
7. [Ciclo di Vita delle Richieste](#ciclo-di-vita-delle-richieste)
8. [Diagramma di Flusso dei Dati](#diagramma-di-flusso-dei-dati)
9. [Architettura di Deployment](#architettura-di-deployment)

---

## Panoramica

Questo documento fornisce rappresentazioni visive dell'architettura dell'agente ideale utilizzando diagrammi Mermaid. Questi diagrammi completano le specifiche tecniche presenti negli altri documenti.

---

## Architettura di Sistema ad Alto Livello

### Contesto del Sistema

```mermaid
graph TB
    User[User] --> API[API Gateway]
    API --> Auth[Authentication]
    Auth --> Agent[Reasoning Engine]

    Agent --> Memory[Memory System]
    Agent --> Tools[Tool System]
    Agent --> LLM[Language Model]

    Memory --> VectorDB[(Vector DB)]
    Memory --> GraphDB[(Graph DB)]
    Memory --> DocDB[(Document DB)]

    Tools --> External[External APIs]
    Tools --> FileSystem[File System]
    Tools --> CodeExec[Code Executor]

    Agent --> Observability[Observability]
    Observability --> Traces[(Trace Store)]
    Observability --> Logs[(Log Store)]
    Observability --> Metrics[(Metrics Store)]
```

### Componenti Principali

```mermaid
graph LR
    subgraph Core
        RE[Reasoning Engine]
        GM[Goal Manager]
        PE[Planning Engine]
        EE[Execution Engine]
        MC[Metacognitive Monitor]
        RF[Reflection Engine]
    end

    subgraph Memory
        WM[Working Memory]
        EM[Episodic Memory]
        SM[Semantic Memory]
        PM[Procedural Memory]
    end

    subgraph Tools
        TR[Tool Registry]
        TD[Tool Discovery]
        TE[Tool Executor]
        RI[Result Interpreter]
    end

    subgraph Observability
        Tracer[Tracer]
        Logger[Logger]
        MetricsCollector[Metrics]
    end

    RE --> GM
    RE --> PE
    RE --> EE
    RE --> MC
    RE --> RF

    RE --> Memory
    RE --> Tools
    RE --> Observability
```

---

## Diagramma dei Componenti

### Vista Dettagliata dei Componenti

```mermaid
graph TB
    subgraph Presentation Layer
        UI[User Interface]
        API[REST API]
        WebSocket[WebSocket API]
    end

    subgraph Application Layer
        subgraph Reasoning
            GoalMgr[Goal Manager]
            Planner[Planning Engine]
            Executor[Execution Engine]
            Monitor[Metacognitive Monitor]
            Reflector[Reflection Engine]
        end

        subgraph Orchestration
            ToolRegistry[Tool Registry]
            ToolExecutor[Tool Executor]
            ParamBinder[Parameter Binder]
        end
    end

    subgraph Data Layer
        WorkingMem[Working Memory]

        subgraph Long-term
            EpisodicDB[(Episodic DB)]
            SemanticDB[(Semantic DB)]
            ProceduralDB[(Procedural DB)]
            VectorDB[(Vector DB)]
        end
    end

    subgraph Infrastructure
        LLMProvider[LLM Provider]
        Observability[Observability Stack]
        Security[Security Layer]
    end

    UI --> API
    API --> GoalMgr
    GoalMgr --> Planner
    Planner --> Executor
    Executor --> ToolRegistry
    Executor --> Monitor
    Monitor --> Reflector

    Reasoning --> WorkingMem
    Reasoning --> Long-term
    Reasoning --> LLMProvider

    ToolExecutor --> LLMProvider

    Application Layer --> Security
    Application Layer --> Observability
```

---

## Flusso del Motore di Ragionamento

### Ciclo Decisionale

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> Analyzing: New Goal
    Analyzing --> Planning: Goal Understood
    Planning --> Executing: Plan Ready

    Executing --> Monitoring: Step Complete
    Monitoring --> Executing: Continue
    Monitoring --> Replanning: Progress Poor
    Monitoring --> Evaluating: Goal Achieved
    Monitoring --> Impasse: Stuck

    Replanning --> Executing: New Plan

    Impasse --> Analyzing: Create Subgoal

    Evaluating --> Reflecting: Evaluation Done
    Reflecting --> Idle: Learning Complete
    Reflecting --> [*]
```

### Sequenza del Loop di Ragionamento

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant GoalManager
    participant Planner
    participant Executor
    participant Memory
    participant Tools
    participant LLM

    User->>Agent: Execute Goal
    Agent->>GoalManager: Push Goal
    GoalManager->>Memory: Retrieve Context
    Memory-->>GoalManager: Relevant Memories

    GoalManager->>Planner: Generate Plan
    Planner->>LLM: Reasoning Call
    LLM-->>Planner: Plan Steps
    Planner-->>GoalManager: Plan Ready

    loop For Each Step
        GoalManager->>Executor: Execute Step
        Executor->>Tools: Execute Tool
        Tools-->>Executor: Tool Result
        Executor->>Memory: Store Result
        Executor-->>GoalManager: Step Complete
    end

    GoalManager->>Agent: Goal Achieved
    Agent->>Memory: Store Episode
    Agent->>User: Return Result
```

---

## Architettura del Sistema di Memoria

### Gerarchia della Memoria

```mermaid
graph TB
    subgraph Working Memory
        Context[Context Window<br/>8K-128K tokens]
        Goals[Active Goals]
        Recent[Recent Observations]
    end

    subgraph Retrieval Engine
        Activation[Activation<br/>Computation]
        Similarity[Similarity<br/>Search]
        Spreading[Spreading<br/>Activation]
    end

    subgraph Long-term Storage
        Episodic[(Episodic<br/>Document DB)]
        Semantic[(Semantic<br/>Graph DB)]
        Procedural[(Procedural<br/>Template Store)]
        Vector[(Vector DB<br/>Embeddings)]
    end

    Working Memory --> Retrieval Engine
    Retrieval Engine --> Long-term Storage

    Long-term Storage --> Consolidation[Consolidation<br/>Process]
    Consolidation --> Long-term Storage
```

### Flusso delle Operazioni di Memoria

```mermaid
sequenceDiagram
    participant Agent
    participant WorkingMemory
    participant Retrieval
    participant Vector
    participant Graph
    participant DocDB

    Agent->>WorkingMemory: Need Context
    WorkingMemory->>Retrieval: Query

    par Parallel Retrieval
        Retrieval->>Vector: Similarity Search
        Vector-->>Retrieval: Top Matches
    and
        Retrieval->>Graph: Spread Activation
        Graph-->>Retrieval: Related Nodes
    and
        Retrieval->>DocDB: Episode Query
        DocDB-->>Retrieval: Episodes
    end

    Retrieval->>Retrieval: Compute Activation
    Retrieval->>Retrieval: Rank by Score
    Retrieval-->>WorkingMemory: Top K Memories

    WorkingMemory->>WorkingMemory: Evict if Needed
    WorkingMemory-->>Agent: Context Ready
```

---

## Flusso di Esecuzione degli Strumenti

### Ciclo di Vita degli Strumenti

```mermaid
stateDiagram-v2
    [*] --> Defined
    Defined --> Registered: Register

    Registered --> Validating: Validate
    Validating --> Enabled: Valid
    Validating --> Failed: Invalid

    Enabled --> Discovering: Query
    Discovering --> Enabled: Not Selected
    Discovering --> Binding: Selected

    Binding --> Executing: Parameters Bound
    Binding --> Failed: Validation Error

    Executing --> Completed: Success
    Executing --> Failed: Error
    Executing --> Timeout: Time Exceeded

    Completed --> Enabled: Ready for Reuse
    Failed --> Enabled: Recoverable
    Failed --> Disabled: Unrecoverable
    Timeout --> Failed: Handle Error

    Disabled --> [*]
```

### Sequenza di Esecuzione degli Strumenti

```mermaid
sequenceDiagram
    participant Agent
    participant Registry
    participant Binder
    participant Validator
    participant Sandbox
    participant Tool
    participant Monitor

    Agent->>Registry: Discover Tools
    Registry-->>Agent: Matching Tools

    Agent->>Binder: Bind Parameters
    Binder->>Validator: Validate Input
    Validator-->>Binder: Valid
    Binder-->>Agent: Bound Parameters

    Agent->>Sandbox: Create Sandbox
    Sandbox-->>Agent: Sandbox Ready

    Agent->>Monitor: Start Monitoring
    Agent->>Tool: Execute in Sandbox

    par Parallel Monitoring
        Monitor->>Monitor: Track CPU
        Monitor->>Monitor: Track Memory
        Monitor->>Monitor: Track Time
    end

    Tool-->>Agent: Result
    Monitor-->>Agent: Metrics

    Agent->>Validator: Validate Output
    Validator-->>Agent: Valid

    Agent->>Sandbox: Cleanup
    Sandbox-->>Agent: Destroyed
```

---

## Ciclo di Vita delle Richieste

### Flusso di Richiesta End-to-End

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Auth
    participant Agent
    participant Memory
    participant Tools
    participant LLM
    participant Observability

    Client->>Gateway: HTTP Request
    Gateway->>Auth: Authenticate
    Auth-->>Gateway: Valid Token

    Gateway->>Agent: Execute Task

    rect rgb(200, 220, 250)
    Note over Agent,Observability: Reasoning Loop

    Agent->>Observability: Start Trace

    Agent->>Memory: Retrieve Context
    Memory-->>Agent: Relevant Memories

    Agent->>LLM: Generate Plan
    LLM-->>Agent: Plan Steps

    loop For Each Step
        Agent->>Tools: Execute Tool
        Tools-->>Agent: Tool Result

        Agent->>LLM: Interpret Result
        LLM-->>Agent: Interpretation

        Agent->>Agent: Update State
    end

    Agent->>Memory: Store Episode
    Agent->>Observability: End Trace
    end

    Agent-->>Gateway: Response
    Gateway-->>Client: HTTP Response
```

---

## Diagramma di Flusso dei Dati

### Flusso delle Informazioni

```mermaid
graph LR
    subgraph Input
        UserGoal[User Goal]
        Context[Context Data]
    end

    subgraph Processing
        Parse[Parse & Validate]
        Retrieve[Retrieve Memories]
        Reason[Generate Plan]
        Execute[Execute Steps]
        Evaluate[Evaluate Result]
    end

    subgraph Storage
        Episodes[(Episodes)]
        Knowledge[(Knowledge)]
        Traces[(Traces)]
    end

    subgraph Output
        Result[Task Result]
        Trace[Execution Trace]
    end

    UserGoal --> Parse
    Context --> Parse
    Parse --> Retrieve
    Retrieve --> Reason
    Reason --> Execute
    Execute --> Evaluate

    Execute --> Episodes
    Evaluate --> Knowledge
    Processing --> Traces

    Evaluate --> Result
    Traces --> Trace
```

### Trasformazioni dei Dati

```mermaid
graph TB
    subgraph Input Processing
        Raw[Raw Input] --> Validated[Validated Input]
        Validated --> Structured[Structured Goal]
    end

    subgraph Memory Retrieval
        Structured --> Query[Memory Query]
        Query --> Embeddings[Generate Embeddings]
        Embeddings --> Search[Vector Search]
        Search --> Retrieved[Retrieved Memories]
    end

    subgraph Reasoning
        Retrieved --> Prompt[Construct Prompt]
        Prompt --> LLMCall[LLM Generation]
        LLMCall --> ParsedPlan[Parsed Plan]
    end

    subgraph Execution
        ParsedPlan --> Steps[Plan Steps]
        Steps --> ToolCalls[Tool Executions]
        ToolCalls --> Results[Tool Results]
    end

    subgraph Output Processing
        Results --> Aggregated[Aggregated Result]
        Aggregated --> Filtered[Filtered Output]
        Filtered --> Final[Final Response]
    end
```

---

## Architettura di Deployment

### Deployment di Produzione

```mermaid
graph TB
    subgraph Client Tier
        Browser[Web Browser]
        Mobile[Mobile App]
        CLI[CLI Client]
    end

    subgraph Edge Tier
        LB[Load Balancer]
        CDN[CDN]
    end

    subgraph Application Tier
        API1[API Server 1]
        API2[API Server 2]
        API3[API Server 3]

        Agent1[Agent Instance 1]
        Agent2[Agent Instance 2]
        Agent3[Agent Instance 3]
    end

    subgraph Data Tier
        Redis[(Redis Cache)]
        Postgres[(PostgreSQL)]
        Neo4j[(Neo4j Graph)]
        Pinecone[(Pinecone Vector)]
    end

    subgraph External Services
        OpenAI[OpenAI API]
        Anthropic[Anthropic API]
    end

    subgraph Observability
        Jaeger[Jaeger Tracing]
        Prometheus[Prometheus Metrics]
        Elasticsearch[Elasticsearch Logs]
    end

    Browser --> LB
    Mobile --> LB
    CLI --> LB

    LB --> API1
    LB --> API2
    LB --> API3

    API1 --> Agent1
    API2 --> Agent2
    API3 --> Agent3

    Agent1 --> Redis
    Agent2 --> Redis
    Agent3 --> Redis

    Agent1 --> Postgres
    Agent1 --> Neo4j
    Agent1 --> Pinecone

    Agent1 --> OpenAI
    Agent1 --> Anthropic

    Agent1 --> Jaeger
    Agent1 --> Prometheus
    Agent1 --> Elasticsearch
```

### Architettura dei Container

```mermaid
graph TB
    subgraph Kubernetes Cluster
        subgraph Namespace: Agent
            subgraph Deployment: API
                Pod1[API Pod 1]
                Pod2[API Pod 2]
            end

            subgraph Deployment: Worker
                Worker1[Worker Pod 1]
                Worker2[Worker Pod 2]
            end

            subgraph StatefulSet: Cache
                Redis1[Redis Primary]
                Redis2[Redis Replica]
            end
        end

        subgraph Namespace: Data
            Postgres[PostgreSQL]
            Neo4j[Neo4j]
        end

        subgraph Namespace: Observability
            Jaeger[Jaeger]
            Prometheus[Prometheus]
        end
    end

    Ingress[Ingress Controller] --> Pod1
    Ingress --> Pod2

    Pod1 --> Worker1
    Pod2 --> Worker2

    Worker1 --> Redis1
    Worker2 --> Redis1

    Worker1 --> Postgres
    Worker1 --> Neo4j
```

---

## Flusso del Motore di Ragionamento

### Dettaglio del Loop Principale

```mermaid
flowchart TD
    Start([Start]) --> GetGoal[Get Current Goal]
    GetGoal --> CheckAchieved{Goal<br/>Achieved?}

    CheckAchieved -->|Yes| PopGoal[Pop Goal]
    PopGoal --> CheckMore{More<br/>Goals?}
    CheckMore -->|Yes| GetGoal
    CheckMore -->|No| Reflect[Reflect on Episode]
    Reflect --> End([End])

    CheckAchieved -->|No| AssessProgress[Assess Progress]
    AssessProgress --> DetectImpasse{Impasse<br/>Detected?}

    DetectImpasse -->|Yes| HandleImpasse[Handle Impasse]
    HandleImpasse --> CreateSubgoal[Create Subgoal]
    CreateSubgoal --> GetGoal

    DetectImpasse -->|No| MakeDecision[Make Control Decision]
    MakeDecision --> Decision{Decision}

    Decision -->|Continue| ExecuteStep[Execute Next Step]
    Decision -->|Replan| GeneratePlan[Generate New Plan]
    Decision -->|Abort| PopGoal
    Decision -->|Escalate| CreateSubgoal

    GeneratePlan --> ExecuteStep
    ExecuteStep --> UpdateState[Update State]
    UpdateState --> CheckBudget{Budget<br/>OK?}

    CheckBudget -->|Yes| GetGoal
    CheckBudget -->|No| PopGoal
```

### Processo di Pianificazione

```mermaid
flowchart TD
    Start([Start Planning]) --> Analyze[Analyze Goal]
    Analyze --> Complexity{Complex<br/>Enough?}

    Complexity -->|No| DirectAction[Create Direct Action]
    DirectAction --> Return([Return Plan])

    Complexity -->|Yes| RetrieveSimilar[Retrieve Similar Plans]
    RetrieveSimilar --> GenerateCandidates[Generate Candidate Plans]

    GenerateCandidates --> LLMCall[LLM Plan Generation]
    LLMCall --> ParsePlan[Parse Plan]

    ParsePlan --> ValidateLogic[Validate Logic]
    ValidateLogic --> Valid{Valid?}

    Valid -->|No| FixPlan[Attempt Fix]
    FixPlan --> LLMCall

    Valid -->|Yes| BuildDepGraph[Build Dependency Graph]
    BuildDepGraph --> EstimateCost[Estimate Resources]
    EstimateCost --> Optimize{Optimize?}

    Optimize -->|Yes| OptimizePlan[Optimize Plan]
    OptimizePlan --> Return

    Optimize -->|No| Return
```

---

## Architettura del Sistema di Memoria

### Processo di Recupero

```mermaid
flowchart TD
    Start([Memory Query]) --> HasEmbedding{Has<br/>Embedding?}

    HasEmbedding -->|No| EmbedQuery[Embed Query Text]
    EmbedQuery --> VectorSearch

    HasEmbedding -->|Yes| VectorSearch[Vector Search]

    VectorSearch --> GetMatches[Get Top Matches]
    GetMatches --> FetchFull[Fetch Full Memories]

    FetchFull --> ApplyFilters[Apply Filters]
    ApplyFilters --> ComputeActivation[Compute Activation]

    ComputeActivation --> HasContext{Has<br/>Context?}

    HasContext -->|Yes| SpreadActivation[Spreading Activation]
    SpreadActivation --> BoostScores[Boost Scores]
    BoostScores --> RankByScore

    HasContext -->|No| RankByScore[Rank by Score]

    RankByScore --> SelectTopK[Select Top K]
    SelectTopK --> Return([Return Memories])
```

### Processo di Consolidamento

```mermaid
flowchart LR
    subgraph Episodic Memory
        E1[Episode 1]
        E2[Episode 2]
        E3[Episode 3]
        E4[Episode n]
    end

    subgraph Pattern Extraction
        Cluster[Cluster Similar<br/>Episodes]
        Extract[Extract Common<br/>Patterns]
    end

    subgraph Semantic Memory
        Facts[Semantic Facts]
        Rules[Inferred Rules]
    end

    subgraph Procedural Memory
        Procedures[Procedures]
        Templates[Templates]
    end

    E1 --> Cluster
    E2 --> Cluster
    E3 --> Cluster
    E4 --> Cluster

    Cluster --> Extract
    Extract --> Facts
    Extract --> Rules
    Extract --> Procedures
    Extract --> Templates
```

---

## Flusso di Esecuzione degli Strumenti

### Sequenza di Chiamata degli Strumenti

```mermaid
sequenceDiagram
    participant Agent
    participant Registry
    participant Validator
    participant Sandbox
    participant Tool
    participant Monitor
    participant AuditLog

    Agent->>Registry: Discover Tools
    Registry->>Registry: Capability Match
    Registry-->>Agent: Tool List

    Agent->>Agent: Select Best Tool
    Agent->>Validator: Validate Params
    Validator-->>Agent: Validated

    Agent->>AuditLog: Log Tool Call
    Agent->>Sandbox: Create Sandbox
    Sandbox-->>Agent: Ready

    Agent->>Monitor: Start Monitoring

    Agent->>Tool: Execute in Sandbox
    activate Tool

    par Resource Monitoring
        Monitor->>Monitor: Check CPU
        Monitor->>Monitor: Check Memory
        Monitor->>Monitor: Check Time
    end

    Tool-->>Agent: Result
    deactivate Tool

    Monitor-->>Agent: Metrics

    Agent->>Validator: Validate Output
    Validator-->>Agent: Valid

    Agent->>Sandbox: Destroy
    Agent->>AuditLog: Log Result
```

---

## Ciclo di Vita delle Richieste

### Flusso Completo delle Richieste

```mermaid
stateDiagram-v2
    [*] --> Received

    Received --> Authenticating: Validate Token
    Authenticating --> Authorizing: Auth Success
    Authenticating --> Rejected: Auth Failed

    Authorizing --> Validated: Authz Success
    Authorizing --> Rejected: Authz Failed

    Validated --> Queued: Input Validation
    Validated --> Rejected: Validation Failed

    Queued --> Processing: Resource Available

    Processing --> Executing: Plan Generated
    Executing --> Monitoring: Steps Execute

    Monitoring --> Executing: Continue
    Monitoring --> Completed: Success
    Monitoring --> Failed: Unrecoverable Error

    Completed --> Filtering: Filter Output
    Filtering --> Audited: Log Complete
    Audited --> [*]: Return Result

    Failed --> Audited: Log Failure
    Rejected --> Audited: Log Rejection
```

---

## Diagramma di Flusso dei Dati

### Flusso delle Informazioni attraverso il Sistema

```mermaid
graph LR
    subgraph External
        User[User Input]
        APIs[External APIs]
        Files[File System]
    end

    subgraph Agent Core
        Reasoning[Reasoning<br/>Engine]
        Memory[Memory<br/>System]
        Tools[Tool<br/>System]
    end

    subgraph Storage
        Hot[(Hot<br/>Storage)]
        Warm[(Warm<br/>Storage)]
        Cold[(Cold<br/>Storage)]
    end

    subgraph Outputs
        Response[User Response]
        Traces[Execution Traces]
        Metrics[Performance Metrics]
    end

    User --> Reasoning

    Reasoning --> Memory
    Memory --> Hot
    Hot --> Warm
    Warm --> Cold

    Reasoning --> Tools
    Tools --> APIs
    Tools --> Files

    APIs --> Tools
    Files --> Tools
    Tools --> Reasoning

    Memory --> Reasoning

    Reasoning --> Response
    Reasoning --> Traces
    Reasoning --> Metrics
```

---

## Conclusione

Questi diagrammi forniscono rappresentazioni visive di:

1. **Architettura ad Alto Livello**: Contesto del sistema e componenti
2. **Relazioni tra Componenti**: Come interagiscono i moduli
3. **Flussi di Processo**: Loop di ragionamento, operazioni di memoria, esecuzione strumenti
4. **Macchine a Stati**: Stati del ciclo di vita dei componenti
5. **Diagrammi di Sequenza**: Ordinamento temporale delle operazioni
6. **Flusso dei Dati**: Movimento delle informazioni attraverso il sistema
7. **Deployment**: Topologia dell'infrastruttura di produzione

Tutti i diagrammi utilizzano il formato Mermaid per:
- Controllo di versione (basato su testo)
- Rendering semplice (GitHub, GitLab, molti strumenti)
- Manutenibilità (modifica come testo)
- Integrazione nella documentazione

Queste rappresentazioni visive completano le specifiche tecniche dettagliate presenti negli altri documenti di ricerca.
