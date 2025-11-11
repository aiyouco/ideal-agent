# Memory System: Knowledge Storage and Retrieval

## Overview

Il Memory System è la componente che permette all'agente di mantenere contesto, ricordare esperienze passate, e accumulare conoscenza nel tempo. È strutturato in tre sottosistemi specializzati, ciascuno con caratteristiche e scopi diversi.

```
┌───────────────────────────────────────────────────────────────┐
│                      MEMORY SYSTEM                            │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  WORKING MEMORY                                         │ │
│  │  • Current task context                                 │ │
│  │  • Intermediate results                                 │ │
│  │  • Active reasoning state                               │ │
│  │  Storage: In-memory, ~20K tokens                        │ │
│  │  Lifetime: Single task execution                        │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  EPISODIC MEMORY                                        │ │
│  │  • Historical task executions                           │ │
│  │  • Complete execution traces                            │ │
│  │  • Outcomes and learnings                               │ │
│  │  Storage: Vector DB, millions of episodes              │ │
│  │  Lifetime: Permanent (with aging)                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  PATTERN CACHE                                          │ │
│  │  • Learned strategies                                   │ │
│  │  • Validated heuristics                                 │ │
│  │  • Reusable templates                                   │ │
│  │  Storage: Structured DB, thousands of patterns         │ │
│  │  Lifetime: Permanent (with validation)                  │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Data Flows:                                                  │
│  • Task Start → Load context into Working Memory              │
│  • During Execution → Query Episodic for similar cases        │
│  • During Planning → Retrieve Patterns from Cache             │
│  • Task End → Store episode in Episodic Memory                │
│  • Reflection → Update Pattern Cache                          │
└───────────────────────────────────────────────────────────────┘
```

## 1. Working Memory

### 1.1 Purpose & Responsibilities

**Core Function**: Mantenere contesto immediato necessario per reasoning e execution della task corrente.

**Characteristics**:
- **Volatile**: Esiste solo per durata di un'esecuzione
- **Size-Limited**: ~20K tokens max (LLM context window constraint)
- **Fast Access**: In-memory, latenza <1ms
- **Structured**: Organizzato per facilitare accesso

**Responsibilities**:
1. **Context Management**: Mantenere informazioni rilevanti per task corrente
2. **State Tracking**: Tracciare stato di esecuzione
3. **Result Caching**: Memorizzare output intermedi
4. **Variable Storage**: Gestire variabili temporanee
5. **Context Pruning**: Gestire limite di size, rimuovere info non più rilevanti

### 1.2 Working Memory Structure

```
┌────────────────────────────────────────────────────────────────┐
│                    WORKING MEMORY LAYOUT                       │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SYSTEM CONTEXT (Static, ~2K tokens)                     │ │
│  │  • Agent capabilities                                    │ │
│  │  • Available tools                                       │ │
│  │  • Configuration                                         │ │
│  │  • Safety bounds                                         │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  TASK CONTEXT (~5K tokens, Hot)                          │ │
│  │  • Original task description                             │ │
│  │  • Parsed goals and constraints                          │ │
│  │  • Success criteria                                      │ │
│  │  • Execution plan                                        │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  EXECUTION STATE (~3K tokens, Dynamic)                   │ │
│  │  • Current phase (analysis, planning, execution)         │ │
│  │  • Completed subtasks                                    │ │
│  │  • In-progress subtasks                                  │ │
│  │  • Failed subtasks with errors                           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  INTERMEDIATE RESULTS (~8K tokens, Warm)                 │ │
│  │  • Recent subtask outputs (last 5-7)                     │ │
│  │  • Key computed values                                   │ │
│  │  • Temporary files/data references                       │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  RETRIEVED CONTEXT (~2K tokens, Cold)                    │ │
│  │  • Relevant episodes (summaries)                         │ │
│  │  • Applicable patterns                                   │ │
│  │  • Domain knowledge snippets                             │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  Total: ~20K tokens (fits in LLM context)                      │
└────────────────────────────────────────────────────────────────┘
```

### 1.3 Context Management Strategies

**Challenge**: LLM context windows sono limitati, ma task possono generare molto più contesto di quanto fittable.

**Solution**: Hierarchical context management con "temperature" layers.

```
┌──────────────────────────────────────────────────────────┐
│            CONTEXT TEMPERATURE LAYERS                    │
│                                                          │
│  HOT (Always in context)                                 │
│  ├─ Current subtask being executed                       │
│  ├─ Immediately previous subtask output                  │
│  ├─ Goal and success criteria                            │
│  └─ Critical constraints                                 │
│      → Total: ~5K tokens                                 │
│                                                          │
│  WARM (Included if space available)                      │
│  ├─ Recent subtask outputs (2-5 back)                    │
│  ├─ Execution plan structure                             │
│  ├─ Key intermediate computations                        │
│  └─ Retrieved patterns/episodes                          │
│      → Total: ~10K tokens                                │
│                                                          │
│  COLD (Summarized or referenced)                         │
│  ├─ Old subtask outputs (>5 back)                        │
│  ├─ Detailed execution traces                            │
│  ├─ Large data structures                                │
│  └─ Full episode histories                               │
│      → Stored externally, retrieved on demand            │
└──────────────────────────────────────────────────────────┘
```

**Context Pruning Algorithm**:
```
Function MANAGE_CONTEXT(working_memory, new_content):

  # Check if adding new content exceeds limit
  IF working_memory.size + new_content.size > LIMIT:

    # STEP 1: Identify what can be pruned
    prunable = []

    # Candidate 1: Old intermediate results
    old_results = working_memory.intermediate_results.older_than(5_subtasks)
    FOR result IN old_results:
      IF NOT result.is_referenced_by_future_tasks:
        prunable.append(result)

    # Candidate 2: Detailed traces (keep summaries)
    detailed_traces = working_memory.execution_traces
    FOR trace IN detailed_traces:
      summary = SUMMARIZE(trace)
      prunable.append({original: trace, replacement: summary})

    # Candidate 3: Retrieved context not yet used
    unused_retrieved = working_memory.retrieved_context.not_accessed
    prunable.append(unused_retrieved)

    # STEP 2: Move to external storage
    FOR item IN prunable:
      IF item.type == "intermediate_result":
        STORE_IN_EPISODIC_MEMORY(item)
        working_memory.add_reference(item.id, "episodic")
      ELSE IF item.type == "trace":
        REPLACE(item.original, item.replacement)

    # STEP 3: Recalculate space
    IF working_memory.size + new_content.size <= LIMIT:
      working_memory.add(new_content)
    ELSE:
      # Escalate: context truly insufficient
      WARN("Context limit reached despite pruning")
      APPLY_AGGRESSIVE_SUMMARIZATION()
```

### 1.4 Variable Storage

**Purpose**: Mantenere variabili e valori computati durante esecuzione.

**Variable Types**:
```
┌──────────────────────────────────────────────────────────┐
│                    VARIABLE TYPES                        │
│                                                          │
│  1. TASK VARIABLES                                       │
│     • Extracted from goal analysis                       │
│     • Example: target_file = "auth.py"                   │
│     • Lifetime: Entire task                              │
│                                                          │
│  2. INTERMEDIATE VALUES                                  │
│     • Results of subtasks                                │
│     • Example: jwt_library = "PyJWT"                     │
│     • Lifetime: Until no longer referenced               │
│                                                          │
│  3. ACCUMULATOR VARIABLES                                │
│     • Aggregated across subtasks                         │
│     • Example: files_modified = ["auth.py", "test.py"]   │
│     • Lifetime: Entire task                              │
│                                                          │
│  4. CONTROL FLOW VARIABLES                               │
│     • Loop counters, retry counts                        │
│     • Example: retry_count = 2                           │
│     • Lifetime: Scope-limited                            │
│                                                          │
│  5. METADATA VARIABLES                                   │
│     • Execution statistics                               │
│     • Example: tokens_used = 15420                       │
│     • Lifetime: Entire task                              │
└──────────────────────────────────────────────────────────┘
```

**Variable Binding and Scope**:
```
Variable Storage Schema:

{
  "global": {
    // Available throughout task
    "task_id": "task_12345",
    "user_id": "user_789",
    "working_directory": "/project/src"
  },

  "task_scope": {
    // From goal analysis
    "primary_goal": "Add JWT authentication",
    "target_modules": ["auth.py", "middleware.py"],
    "constraints": {...}
  },

  "execution_scope": {
    // Current execution state
    "current_phase": "implementation",
    "current_subtask": "create_jwt_encoder",
    "parent_task": "implement_jwt"
  },

  "intermediate_values": {
    // Keyed by subtask_id
    "subtask_001": {
      "output": "PyJWT",
      "type": "library_choice"
    },
    "subtask_002": {
      "output": {...},
      "type": "generated_code"
    }
  },

  "metadata": {
    "start_time": "2024-01-15T10:30:00Z",
    "tokens_consumed": 15420,
    "llm_calls": 8,
    "cost": 0.12
  }
}
```

### 1.5 Working Memory Operations

**Core Operations**:
```
1. INIT()
   • Initialize empty working memory for new task
   • Load system context
   • Allocate initial space

2. LOAD_TASK_CONTEXT(task)
   • Parse task into structured representation
   • Store goals, constraints, context
   • Allocate task variables

3. SET(key, value, scope)
   • Store value in appropriate scope
   • Update size tracking
   • Trigger pruning if needed

4. GET(key, scope)
   • Retrieve value from scope
   • Mark as accessed (for pruning)
   • Return value or null

5. PUSH_STATE(state)
   • Save current execution state
   • Update phase, progress
   • Record timestamps

6. ADD_INTERMEDIATE(subtask_id, output)
   • Store subtask output
   • Update recency tracking
   • May trigger pruning

7. RETRIEVE_FROM_EPISODIC(query)
   • Query episodic memory
   • Load relevant episodes into working memory
   • Summarize if needed to fit

8. PRUNE()
   • Identify cold content
   • Move to external storage
   • Keep references

9. SNAPSHOT()
   • Create checkpoint of current state
   • For debugging, rollback

10. CLEAR()
    • Reset working memory after task completion
    • Keep only metadata for reflection
```

## 2. Episodic Memory

### 2.1 Purpose & Responsibilities

**Core Function**: Storagare permanente di tutti gli episodi di esecuzione per learning e retrieval futuro.

**Characteristics**:
- **Persistent**: Dati sopravvivono tra sessioni
- **Large Scale**: Millions di episodi
- **Searchable**: Semantic e structural search
- **Append-Only**: Episodi non modificati dopo creazione (immutable)
- **Indexed**: Multi-dimensional indexing per fast retrieval

**Responsibilities**:
1. **Episode Storage**: Persistere execution traces complete
2. **Semantic Search**: Trovare episodi simili a query
3. **Structured Query**: Query su attributi specifici
4. **Aging & Archival**: Gestire lifecycle di episodi vecchi
5. **Retrieval Optimization**: Fast access a episodi rilevanti

### 2.2 Episode Structure

**Complete Episode Schema**:
```
Episode {
  // Identification
  episode_id: string (UUID),
  timestamp: datetime,
  session_id: string,

  // Task Description
  task: {
    original_input: string,
    parsed_goals: GoalStructure,
    constraints: [Constraint],
    context: Context,
    complexity: ComplexityAssessment
  },

  // Execution Trace
  execution: {
    plan: ExecutionPlan,
    trace: [TraceEntry],
    adaptations: [Adaptation],
    duration: float,
    resource_usage: ResourceMetrics
  },

  // Outcome
  outcome: {
    status: "SUCCESS" | "PARTIAL" | "FAILURE",
    primary_output: {...},
    side_effects: [SideEffect],
    quality_metrics: Metrics
  },

  // Learning
  reflection: {
    insights: [Insight],
    patterns_discovered: [Pattern],
    failures_analyzed: [Failure],
    performance_analysis: Analysis
  },

  // Metadata for Retrieval
  metadata: {
    domain: string,
    task_type: string,
    tools_used: [string],
    strategies_applied: [string],
    tags: [string]
  },

  // Embeddings for Semantic Search
  embeddings: {
    task_embedding: vector(768),
    outcome_embedding: vector(768),
    full_episode_embedding: vector(768)
  },

  // Lifecycle
  lifecycle: {
    access_count: int,
    last_accessed: datetime,
    referenced_by: [episode_id],
    archival_status: "active" | "archived" | "deleted"
  }
}
```

### 2.3 Storage Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                   EPISODIC MEMORY STORAGE                      │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  VECTOR DATABASE (Semantic Search)                       │ │
│  │  • Stores episode embeddings                             │ │
│  │  • Fast similarity search (ANN)                          │ │
│  │  • Technology: Pinecone, Weaviate, or Milvus            │ │
│  │  • Index: ~768-dimensional vectors                       │ │
│  │  • Query time: <100ms for top-K                          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  DOCUMENT DATABASE (Structured Storage)                  │ │
│  │  • Stores complete episode documents                     │ │
│  │  • Indexed by episode_id                                 │ │
│  │  • Technology: MongoDB, PostgreSQL with JSONB           │ │
│  │  • Indices: task_type, domain, timestamp, status        │ │
│  │  • Query time: <50ms by ID, <200ms by attributes        │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  OBJECT STORAGE (Large Artifacts)                        │ │
│  │  • Stores large outputs, traces                          │ │
│  │  • Referenced by episode, not embedded                   │ │
│  │  • Technology: S3, MinIO                                 │ │
│  │  • Example: Large generated files, images, logs         │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  CACHE LAYER (Hot Episodes)                              │ │
│  │  • In-memory cache of frequently accessed episodes       │ │
│  │  • Technology: Redis                                     │ │
│  │  • Size: Last 1000 episodes or most accessed            │ │
│  │  • Hit rate target: >70%                                 │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 2.4 Retrieval Strategies

**Multi-Modal Retrieval**: Combine different retrieval approaches per best results.

```
┌──────────────────────────────────────────────────────────┐
│                 RETRIEVAL STRATEGIES                     │
│                                                          │
│  1. SEMANTIC SIMILARITY SEARCH                           │
│     Input: Current task description                      │
│     Method: Vector similarity (cosine, dot product)      │
│     Output: Top-K most similar episodes                  │
│     Use: Find similar past tasks                         │
│                                                          │
│  2. STRUCTURED QUERY                                     │
│     Input: Attribute filters (domain, task_type, etc)    │
│     Method: Database query                               │
│     Output: Episodes matching criteria                   │
│     Use: Find all episodes of specific type              │
│                                                          │
│  3. HYBRID SEARCH                                        │
│     Input: Task description + attribute filters          │
│     Method: Combine semantic + structured                │
│     Output: Relevant episodes meeting criteria           │
│     Use: "Similar to X in domain Y"                      │
│                                                          │
│  4. TEMPORAL SEARCH                                      │
│     Input: Time range                                    │
│     Method: Time-based indexing                          │
│     Output: Recent episodes                              │
│     Use: "What did I do this session?"                   │
│                                                          │
│  5. FAILURE-SPECIFIC SEARCH                              │
│     Input: Current failure signature                     │
│     Method: Match failure patterns                       │
│     Output: Episodes with similar failures               │
│     Use: Learn from past failures                        │
└──────────────────────────────────────────────────────────┘
```

**Retrieval Algorithm**:
```
Function RETRIEVE_RELEVANT_EPISODES(query, context, k=5):

  # STEP 1: Semantic search
  query_embedding = EMBED(query)
  semantic_matches = VECTOR_SEARCH(query_embedding, top_k=20)

  # STEP 2: Apply contextual filters
  filtered = []
  FOR episode IN semantic_matches:
    # Filter by domain if specified
    IF context.domain AND episode.domain != context.domain:
      CONTINUE

    # Filter by recency if preferred
    IF context.prefer_recent AND episode.age > THRESHOLD:
      CONTINUE

    # Filter by success if learning from successful episodes
    IF context.only_successful AND episode.outcome != SUCCESS:
      CONTINUE

    filtered.append(episode)

  # STEP 3: Re-rank by relevance
  scored = []
  FOR episode IN filtered:
    score = COMPUTE_RELEVANCE_SCORE(episode, query, context)
    scored.append((episode, score))

  ranked = SORT_BY_SCORE(scored, descending=True)

  # STEP 4: Return top-K
  RETURN ranked[:k]

Function COMPUTE_RELEVANCE_SCORE(episode, query, context):
  score = 0

  # Semantic similarity (primary)
  score += 0.5 * COSINE_SIMILARITY(query.embedding, episode.embedding)

  # Recency bonus
  age_days = (now - episode.timestamp).days
  recency_factor = EXP(-age_days / 30)  # Decay over ~month
  score += 0.2 * recency_factor

  # Success bonus
  IF episode.outcome == SUCCESS:
    score += 0.15

  # Access frequency bonus (popular episodes likely useful)
  access_factor = LOG(1 + episode.access_count) / 10
  score += 0.1 * MIN(access_factor, 1.0)

  # Domain match bonus
  IF episode.domain == context.domain:
    score += 0.05

  RETURN score
```

### 2.5 Episode Lifecycle Management

**Aging Policy**: Vecchi episodi meno rilevanti nel tempo.

```
┌──────────────────────────────────────────────────────────┐
│                  EPISODE LIFECYCLE                       │
│                                                          │
│  PHASE 1: ACTIVE (0-30 days)                             │
│  • Fully indexed and searchable                          │
│  • High retrieval priority                               │
│  • No degradation                                        │
│                                                          │
│  PHASE 2: MATURE (30-180 days)                           │
│  • Still fully searchable                                │
│  • Lower retrieval priority vs recent                    │
│  • Eligible for consolidation with similar episodes      │
│                                                          │
│  PHASE 3: AGED (180-365 days)                            │
│  • Detailed traces archived to cold storage              │
│  • Summaries remain searchable                           │
│  • Retrieved only if highly relevant                     │
│                                                          │
│  PHASE 4: ARCHIVED (>365 days)                           │
│  • Only metadata and key learnings kept                  │
│  • Full episode in cold storage                          │
│  • Retrieved only on explicit request                    │
│                                                          │
│  PHASE 5: DELETED (Rare)                                 │
│  • Episodes with no value (duplicates, errors)           │
│  • Explicitly flagged for deletion                       │
│  • Soft delete with retention period                     │
└──────────────────────────────────────────────────────────┘
```

**Consolidation Process**: Merge similar episodes per save storage.

```
Consolidation Algorithm:

1. IDENTIFY CLUSTERS
   • Group episodes by high similarity (>0.95 cosine)
   • Same task type, domain, outcome
   • Within short time window

2. ANALYZE CLUSTER
   • Extract common structure
   • Identify variations
   • Compute aggregate statistics

3. CREATE CONSOLIDATED EPISODE
   ConsolidatedEpisode {
     representative_episode_id: string,
     num_instances: int,
     common_structure: {...},
     variations: [{episode_id, diff}],
     aggregate_stats: {
       avg_duration: float,
       success_rate: float,
       common_patterns: [Pattern]
     }
   }

4. ARCHIVE ORIGINAL EPISODES
   • Mark originals as consolidated
   • Keep references for auditability
   • Free up primary storage

5. UPDATE INDICES
   • Consolidated episode gets combined embedding
   • Maintains searchability
```

### 2.6 Episodic Memory Operations

```
1. STORE_EPISODE(episode)
   • Validate episode structure
   • Generate embeddings
   • Insert into vector DB
   • Insert into document DB
   • Update indices
   • Return episode_id

2. RETRIEVE_BY_ID(episode_id)
   • Check cache
   • If miss, query document DB
   • If archived, retrieve from cold storage
   • Update access metadata
   • Return episode

3. SEARCH_SEMANTIC(query, k, filters)
   • Embed query
   • Vector search with filters
   • Retrieve full episodes for top-K
   • Rank by relevance
   • Return ranked results

4. SEARCH_STRUCTURED(criteria)
   • Build database query
   • Execute on document DB
   • Apply limits
   • Return matching episodes

5. GET_RECENT(n, filters)
   • Query by timestamp desc
   • Apply filters
   • Limit to n
   • Return episodes

6. GET_SIMILAR_FAILURES(failure_signature, k)
   • Extract failure features
   • Search for matching patterns
   • Return episodes with similar failures

7. UPDATE_ACCESS_METADATA(episode_id)
   • Increment access_count
   • Update last_accessed
   • Update cache

8. ARCHIVE_OLD_EPISODES(age_threshold)
   • Identify episodes older than threshold
   • Move detailed traces to cold storage
   • Update archival_status
   • Maintain searchable summaries

9. CONSOLIDATE_SIMILAR(similarity_threshold)
   • Run clustering
   • Identify consolidation candidates
   • Create consolidated episodes
   • Archive originals

10. PURGE_DELETED(retention_period)
    • Permanently delete episodes marked for deletion
    • After retention period expires
```

## 3. Pattern Cache

### 3.1 Purpose & Responsibilities

**Core Function**: Storagare strategie, euristiche, e pattern validati che possono essere riusati per migliorare performance e decision-making.

**Characteristics**:
- **Curated**: Solo pattern validati con sufficiente evidenza
- **Structured**: Organizzati per facilitare matching e application
- **Evolvable**: Pattern aggiornati quando nuova evidenza emerge
- **Actionable**: Direttamente applicabili in planning/execution

**Difference from Episodic Memory**:
```
EPISODIC MEMORY:
  • Raw experiences (what happened)
  • Comprehensive traces
  • Millions of instances
  • Immutable after creation

PATTERN CACHE:
  • Distilled knowledge (lessons learned)
  • Generalized strategies
  • Thousands of patterns
  • Updated with new evidence
```

### 3.2 Pattern Taxonomy

```
┌────────────────────────────────────────────────────────────────┐
│                     PATTERN CATEGORIES                         │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  1. STRATEGY PATTERNS                                    │ │
│  │  • High-level approaches for problem classes             │ │
│  │  • When to use which planning strategy                   │ │
│  │  • Example: "HTN planning best for code refactoring"     │ │
│  │  • Applicability: Problem classification → Strategy      │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  2. DECOMPOSITION PATTERNS                               │ │
│  │  • How to break down specific types of tasks             │ │
│  │  • Template subtask structures                           │ │
│  │  • Example: "API feature → design, implement, test"      │ │
│  │  • Applicability: Task type → Decomposition template     │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  3. SEQUENCE PATTERNS                                    │ │
│  │  • Ordered steps that work well together                 │ │
│  │  • Dependency relationships                              │ │
│  │  • Example: "Install → Import → Verify → Test"           │ │
│  │  • Applicability: Subtask type → Next steps              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  4. TOOL SELECTION PATTERNS                              │ │
│  │  • Which tools work best for which tasks                 │ │
│  │  • Tool combination strategies                           │ │
│  │  • Example: "Python refactoring → use ast + black"       │ │
│  │  • Applicability: Task features → Tool choices           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  5. FAILURE PATTERNS                                     │ │
│  │  • Common failure modes and causes                       │ │
│  │  • How to recognize and avoid                            │ │
│  │  • Example: "Package install fails without pip upgrade"  │ │
│  │  • Applicability: Failure signal → Diagnosis             │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  6. RECOVERY PATTERNS                                    │ │
│  │  • Effective contingency strategies                      │ │
│  │  • Recovery sequences                                    │ │
│  │  • Example: "API timeout → retry with backoff"           │ │
│  │  • Applicability: Failure type → Recovery action         │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  7. OPTIMIZATION PATTERNS                                │ │
│  │  • Performance improvement opportunities                 │ │
│  │  • Resource usage optimizations                          │ │
│  │  • Example: "Independent tasks → parallelize"            │ │
│  │  • Applicability: Code structure → Optimization          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  8. DOMAIN PATTERNS                                      │ │
│  │  • Domain-specific best practices                        │ │
│  │  • Conventions and idioms                                │ │
│  │  • Example: "FastAPI → middleware for cross-cutting"     │ │
│  │  • Applicability: Domain + goal → Approach               │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 3.3 Pattern Structure

**Complete Pattern Schema**:
```
Pattern {
  // Identification
  pattern_id: string (UUID),
  name: string,
  category: PatternCategory,
  version: int,
  created_at: datetime,
  last_updated: datetime,

  // Description
  description: string,
  summary: string (1-2 sentences),
  detailed_explanation: string,

  // Applicability
  applicability: {
    problem_types: [string],
    task_features: {
      domain: [string],
      complexity: ComplexityRange,
      required_capabilities: [string]
    },
    context_conditions: [Condition],
    anti_conditions: [Condition]  // When NOT to apply
  },

  // Pattern Content
  content: {
    type: "STRATEGY" | "TEMPLATE" | "HEURISTIC" | "PROCEDURE",

    // For Strategy patterns
    strategy: {
      approach: string,
      key_principles: [string],
      trade_offs: string
    },

    // For Template patterns
    template: {
      structure: {...},  // Subtask decomposition template
      parameters: [Parameter],
      instantiation_rules: [Rule]
    },

    // For Heuristic patterns
    heuristic: {
      condition: Condition,
      action: Action,
      rationale: string
    },

    // For Procedure patterns
    procedure: {
      steps: [Step],
      branching_logic: {...},
      termination_conditions: [Condition]
    }
  },

  // Evidence & Validation
  evidence: {
    // Episodes supporting this pattern
    supporting_episodes: [episode_id],
    counter_episodes: [episode_id],

    // Statistics
    num_applications: int,
    success_rate: float,
    avg_time_saving: float,
    avg_cost_saving: float,
    success_rate_improvement: float,

    // Confidence
    confidence_score: float,  // 0-1
    statistical_significance: float,  // p-value
    last_validated: datetime
  },

  // Application Guidance
  application: {
    when_to_apply: string,
    how_to_apply: string,
    expected_benefits: [string],
    potential_risks: [string],
    alternative_patterns: [pattern_id]
  },

  // Lifecycle
  lifecycle: {
    status: "CANDIDATE" | "VALIDATED" | "DEPRECATED",
    usage_count: int,
    last_used: datetime,
    superseded_by: pattern_id | null,
    deprecation_reason: string | null
  },

  // Relationships
  relationships: {
    specializes: pattern_id | null,  // More specific version of
    generalizes: [pattern_id],       // More general versions
    contradicts: [pattern_id],       // Incompatible patterns
    complements: [pattern_id]        // Works well with
  }
}
```

### 3.4 Pattern Storage and Indexing

```
┌────────────────────────────────────────────────────────────────┐
│                   PATTERN CACHE STORAGE                        │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  RELATIONAL DATABASE (Primary Storage)                   │ │
│  │  • Structured pattern storage                            │ │
│  │  • Complex queries on attributes                         │ │
│  │  • Technology: PostgreSQL                                │ │
│  │  • Indices: category, applicability, confidence          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  INVERTED INDEX (Feature-based lookup)                   │ │
│  │  • Map task features → applicable patterns               │ │
│  │  • Fast pattern matching                                 │ │
│  │  • Example: domain="python" → [pattern_ids]              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  DECISION TREE INDEX (Conditional matching)              │ │
│  │  • Tree structure for pattern selection                  │ │
│  │  • Navigate by answering questions                       │ │
│  │  • Example: Is it code task? → Yes → Python? → ...       │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↕                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  IN-MEMORY CACHE (Hot patterns)                          │ │
│  │  • Most frequently used patterns                         │ │
│  │  • Technology: Redis                                     │ │
│  │  • Size: Top 100-200 patterns                            │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 3.5 Pattern Matching and Selection

**Pattern Matching Algorithm**:
```
Function FIND_APPLICABLE_PATTERNS(task, context):

  # STEP 1: Extract task features
  features = EXTRACT_FEATURES(task, context)
  # Features: {domain, task_type, complexity, required_tools, ...}

  # STEP 2: Query by category (if known)
  IF context.pattern_category_hint:
    candidates = QUERY_PATTERNS_BY_CATEGORY(context.pattern_category_hint)
  ELSE:
    # Get all active patterns
    candidates = QUERY_PATTERNS_BY_STATUS("VALIDATED")

  # STEP 3: Filter by applicability
  applicable = []
  FOR pattern IN candidates:

    # Check problem type match
    IF task.type NOT IN pattern.applicability.problem_types:
      CONTINUE

    # Check feature requirements
    IF NOT MATCHES(features, pattern.applicability.task_features):
      CONTINUE

    # Check context conditions
    IF NOT EVALUATE_CONDITIONS(pattern.applicability.context_conditions, context):
      CONTINUE

    # Check anti-conditions (when NOT to apply)
    IF EVALUATE_CONDITIONS(pattern.applicability.anti_conditions, context):
      CONTINUE

    applicable.append(pattern)

  # STEP 4: Rank by confidence and evidence
  ranked = RANK_PATTERNS(applicable, task, context)

  # STEP 5: Return top-K
  RETURN ranked[:5]

Function RANK_PATTERNS(patterns, task, context):
  scored = []

  FOR pattern IN patterns:
    score = 0

    # Base score: confidence
    score += 0.4 * pattern.evidence.confidence_score

    # Success rate bonus
    score += 0.3 * pattern.evidence.success_rate

    # Usage frequency (popular patterns likely good)
    usage_factor = LOG(1 + pattern.lifecycle.usage_count) / 10
    score += 0.15 * MIN(usage_factor, 1.0)

    # Recency bonus (recently validated patterns preferred)
    days_since_validation = (now - pattern.evidence.last_validated).days
    recency = EXP(-days_since_validation / 90)
    score += 0.1 * recency

    # Feature match quality
    match_quality = COMPUTE_FEATURE_MATCH(task, pattern.applicability)
    score += 0.05 * match_quality

    scored.append((pattern, score))

  RETURN SORT_BY_SCORE(scored, descending=True)
```

### 3.6 Pattern Validation and Evolution

**Pattern Lifecycle**:
```
┌────────────────────────────────────────────────────────────────┐
│                    PATTERN LIFECYCLE                           │
│                                                                │
│  PHASE 1: CANDIDATE                                            │
│  • Newly discovered from reflection                            │
│  • Insufficient evidence (<5 supporting episodes)              │
│  • Not yet used in planning                                    │
│  • Tracked for evidence accumulation                           │
│                                                                │
│  ↓ (When evidence >= 5 episodes, confidence > 0.7)             │
│                                                                │
│  PHASE 2: VALIDATED                                            │
│  • Sufficient evidence gathered                                │
│  • Statistical significance achieved                           │
│  • Used in planning and execution                              │
│  • Continuously monitored for performance                      │
│                                                                │
│  ↓ (If contradictory evidence emerges)                         │
│                                                                │
│  PHASE 3: UNDER REVIEW                                         │
│  • New evidence contradicts pattern                            │
│  • Success rate dropped significantly                          │
│  • Temporarily disabled from use                               │
│  • Re-validation process triggered                             │
│                                                                │
│  ↓ (Branches based on review outcome)                          │
│                                                                │
│  OUTCOME A: REFINED (Back to VALIDATED)                        │
│  • Pattern updated with new insights                           │
│  • Applicability conditions refined                            │
│  • Evidence reset, begins re-accumulation                      │
│                                                                │
│  OUTCOME B: DEPRECATED                                         │
│  • Pattern no longer valid                                     │
│  • Superseded by better pattern                                │
│  • Kept for historical record                                  │
│  • Not used in new executions                                  │
└────────────────────────────────────────────────────────────────┘
```

**Validation Process**:
```
Function VALIDATE_PATTERN(pattern):

  # Collect recent applications
  recent_episodes = GET_EPISODES_USING_PATTERN(
    pattern.pattern_id,
    since=now - 90_days
  )

  IF len(recent_episodes) < MIN_EVIDENCE_THRESHOLD:
    RETURN "INSUFFICIENT_EVIDENCE"

  # Compute success rate
  successes = COUNT(episode for episode in recent_episodes
                     if episode.outcome == SUCCESS)
  success_rate = successes / len(recent_episodes)

  # Statistical significance test
  p_value = BINOMIAL_TEST(
    successes,
    len(recent_episodes),
    expected_rate=0.5  # Null hypothesis: no better than random
  )

  # Update pattern evidence
  pattern.evidence.success_rate = success_rate
  pattern.evidence.statistical_significance = p_value
  pattern.evidence.num_applications = len(recent_episodes)
  pattern.evidence.last_validated = now

  # Decision
  IF success_rate >= SUCCESS_THRESHOLD AND p_value < 0.05:
    # Compute confidence based on evidence strength
    confidence = COMPUTE_CONFIDENCE(success_rate, len(recent_episodes), p_value)
    pattern.evidence.confidence_score = confidence

    IF confidence >= CONFIDENCE_THRESHOLD:
      pattern.lifecycle.status = "VALIDATED"
      RETURN "VALIDATED"
    ELSE:
      pattern.lifecycle.status = "CANDIDATE"
      RETURN "CANDIDATE"
  ELSE:
    pattern.lifecycle.status = "UNDER_REVIEW"
    RETURN "NEEDS_REVIEW"

Function COMPUTE_CONFIDENCE(success_rate, sample_size, p_value):
  # Confidence increases with:
  # 1. High success rate
  # 2. Large sample size
  # 3. Strong statistical significance

  success_component = success_rate

  # Sample size confidence (asymptotic to 1)
  sample_component = 1 - EXP(-sample_size / 20)

  # Statistical significance component
  significance_component = 1 - p_value

  # Weighted combination
  confidence = (
    0.5 * success_component +
    0.3 * sample_component +
    0.2 * significance_component
  )

  RETURN CLIP(confidence, 0, 1)
```

### 3.7 Pattern Cache Operations

```
1. ADD_PATTERN(pattern)
   • Validate structure
   • Set initial status to CANDIDATE
   • Insert into database
   • Create indices
   • Return pattern_id

2. GET_PATTERN(pattern_id)
   • Check cache
   • If miss, query database
   • Return pattern

3. FIND_APPLICABLE(task, context)
   • Extract features
   • Query indexed patterns
   • Filter by applicability
   • Rank by relevance
   • Return top-K

4. UPDATE_PATTERN(pattern_id, updates)
   • Retrieve current pattern
   • Apply updates
   • Increment version
   • Update timestamp
   • Persist changes

5. RECORD_APPLICATION(pattern_id, episode_id, outcome)
   • Link pattern to episode
   • Update usage statistics
   • Add to evidence
   • Update cache

6. VALIDATE_PATTERNS()
   • For each VALIDATED pattern:
     - Collect recent evidence
     - Recompute statistics
     - Update confidence
     - Adjust status if needed

7. DEPRECATE_PATTERN(pattern_id, reason, superseded_by)
   • Update status to DEPRECATED
   • Record reason
   • Link to superseding pattern if exists
   • Remove from active indices

8. GET_SIMILAR_PATTERNS(pattern)
   • Compare structure
   • Compare applicability
   • Return similar patterns
   • Use for deduplication

9. MERGE_PATTERNS(pattern_ids)
   • Combine evidence
   • Generalize applicability
   • Create merged pattern
   • Deprecate originals

10. PRUNE_LOW_VALUE_PATTERNS()
    • Identify patterns with:
      - Low confidence
      - Low usage
      - Contradictory evidence
    • Mark for review or deletion
```

## 4. Memory System Integration

### 4.1 Cross-Memory Operations

**Memory systems cooperate per provide comprehensive support**:

```
┌────────────────────────────────────────────────────────────────┐
│              MEMORY SYSTEM COLLABORATION                       │
│                                                                │
│  SCENARIO 1: Task Initiation                                   │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  1. Goal Analysis extracts task features                 │ │
│  │     ↓                                                     │ │
│  │  2. Query PATTERN CACHE for applicable strategies        │ │
│  │     → Load top-3 patterns into WORKING MEMORY            │ │
│  │     ↓                                                     │ │
│  │  3. Query EPISODIC MEMORY for similar past tasks         │ │
│  │     → Load top-5 episodes (summaries) into WORKING MEM   │ │
│  │     ↓                                                     │ │
│  │  4. Planning Engine uses both patterns and episodes      │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  SCENARIO 2: During Execution (Failure)                        │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  1. Task fails with error signature                      │ │
│  │     ↓                                                     │ │
│  │  2. Query PATTERN CACHE for recovery patterns            │ │
│  │     → Load into WORKING MEMORY                           │ │
│  │     ↓                                                     │ │
│  │  3. Query EPISODIC MEMORY for similar failures           │ │
│  │     → Learn how past failures were resolved              │ │
│  │     ↓                                                     │ │
│  │  4. Adaptation Engine applies learned recovery           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  SCENARIO 3: Post-Execution (Reflection)                       │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  1. Task completes, execution trace in WORKING MEMORY    │ │
│  │     ↓                                                     │ │
│  │  2. Store complete episode in EPISODIC MEMORY            │ │
│  │     ↓                                                     │ │
│  │  3. Reflection analyzes episode                          │ │
│  │     ↓                                                     │ │
│  │  4. Extract patterns → Update PATTERN CACHE              │ │
│  │     • New patterns added as CANDIDATES                   │ │
│  │     • Existing patterns updated with new evidence        │ │
│  │     ↓                                                     │ │
│  │  5. WORKING MEMORY cleared                               │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 4.2 Memory Consolidation Pipeline

**Periodic background process per maintain memory quality**:

```
┌────────────────────────────────────────────────────────────────┐
│            MEMORY CONSOLIDATION PIPELINE                       │
│            (Runs nightly or weekly)                            │
│                                                                │
│  STAGE 1: EPISODIC MEMORY CONSOLIDATION                        │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • Identify highly similar episodes (clustering)         │ │
│  │  • Merge similar episodes into consolidated records      │ │
│  │  • Archive old detailed traces to cold storage           │ │
│  │  • Update episode lifecycle statuses                     │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  STAGE 2: PATTERN EXTRACTION                                   │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • Analyze recent episodes for new patterns              │ │
│  │  • Run pattern mining algorithms                         │ │
│  │  • Create CANDIDATE patterns                             │ │
│  │  • Add to Pattern Cache for validation                   │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  STAGE 3: PATTERN VALIDATION                                   │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • For each VALIDATED pattern:                           │ │
│  │    - Gather recent application evidence                  │ │
│  │    - Recompute statistics                                │ │
│  │    - Update confidence scores                            │ │
│  │  • For each CANDIDATE pattern:                           │ │
│  │    - Check if sufficient evidence accumulated            │ │
│  │    - Promote to VALIDATED if criteria met                │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  STAGE 4: PATTERN REFINEMENT                                   │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • Identify patterns with declining performance          │ │
│  │  • Analyze failure cases                                 │ │
│  │  • Refine applicability conditions                       │ │
│  │  • Merge similar patterns                                │ │
│  │  • Deprecate obsolete patterns                           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  STAGE 5: INDEX OPTIMIZATION                                   │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • Rebuild vector indices for episodic memory            │ │
│  │  • Update inverted indices for pattern cache             │ │
│  │  • Optimize database query plans                         │ │
│  │  • Refresh cache with hot items                          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  STAGE 6: REPORTING                                            │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  • Generate memory system health report                  │ │
│  │  • Statistics on storage usage                           │ │
│  │  • Pattern validation summary                            │ │
│  │  • Consolidation results                                 │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Performance Characteristics

**Memory System Metrics**:
```
WORKING MEMORY
  • Capacity: ~20K tokens (~80KB text)
  • Access time: <1ms (in-memory)
  • Operations: 1000s per task
  • Lifetime: Single task execution

EPISODIC MEMORY
  • Capacity: Millions of episodes
  • Storage size: ~10KB per episode (avg)
  • Semantic search: <100ms for top-K=10
  • Structured query: <50ms by ID, <200ms by attributes
  • Cache hit rate: >70% target
  • Episodes stored per day: 100-1000 (depends on usage)

PATTERN CACHE
  • Capacity: Thousands of patterns
  • Storage size: ~5KB per pattern (avg)
  • Pattern matching: <50ms
  • Pattern retrieval: <20ms (cached)
  • Validation cycle: Weekly
  • Pattern discovery rate: 1-5 new patterns per day
```

**Storage Scaling**:
```
Year 1 (10 users, active development):
  Episodes: ~100K episodes
  Storage: ~1GB episodic + 50MB patterns
  Cost: ~$10/month (cloud storage)

Year 2 (100 users, production):
  Episodes: ~1M episodes
  Storage: ~10GB episodic + 500MB patterns
  Cost: ~$50/month

Year 5 (1000 users, mature):
  Episodes: ~10M episodes (with consolidation)
  Storage: ~50GB episodic + 2GB patterns
  Cost: ~$200/month
  Note: Older episodes archived to cheaper cold storage
```

### 4.4 Memory System APIs

**High-Level API for Other Components**:
```
// Working Memory API
WorkingMemory.init(task)
WorkingMemory.set(key, value, scope)
WorkingMemory.get(key, scope)
WorkingMemory.push_state(state)
WorkingMemory.add_intermediate(subtask_id, output)
WorkingMemory.prune()
WorkingMemory.clear()

// Episodic Memory API
EpisodicMemory.store_episode(episode)
EpisodicMemory.retrieve_by_id(episode_id)
EpisodicMemory.search_semantic(query, k, filters)
EpisodicMemory.search_structured(criteria)
EpisodicMemory.get_recent(n, filters)
EpisodicMemory.get_similar_failures(failure_signature, k)

// Pattern Cache API
PatternCache.add_pattern(pattern)
PatternCache.get_pattern(pattern_id)
PatternCache.find_applicable(task, context)
PatternCache.record_application(pattern_id, episode_id, outcome)
PatternCache.validate_patterns()
PatternCache.deprecate_pattern(pattern_id, reason)

// Integrated Queries
Memory.get_context_for_task(task)
  → Returns: {
      applicable_patterns: [Pattern],
      similar_episodes: [Episode],
      domain_knowledge: [...]
    }

Memory.learn_from_episode(episode, reflection_insights)
  → Stores episode, updates patterns, returns learning_summary
```

---

**Next**: [04-capability-layer.md](04-capability-layer.md) → Tool Registry, Model Router, Safety Verifier specifications
