# Data Flows: Interaction Patterns and Sequences

## Overview

Questo documento descrive come i dati fluiscono attraverso l'architettura durante diverse operazioni, mostrando le interazioni tra componenti e le trasformazioni dei dati.

## 1. Complete Task Execution Flow

### 1.1 End-to-End Data Flow

**Full journey da user request a final result**:

```
┌────────────────────────────────────────────────────────────────┐
│               COMPLETE TASK EXECUTION DATA FLOW                │
│                                                                │
│  USER                                                          │
│    │                                                           │
│    │ Task: "Add JWT authentication to API"                    │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 1. API GATEWAY                                           │ │
│  │    Input: Natural language task                          │ │
│  │    Output: TaskRequest {                                 │ │
│  │      task_id: "T-12345",                                 │ │
│  │      user_id: "U-789",                                   │ │
│  │      input_text: "...",                                  │ │
│  │      timestamp: "2024-01-15T10:00:00Z",                  │ │
│  │      context: {session_id, preferences, ...}             │ │
│  │    }                                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 2. SAFETY VERIFIER (Input Validation)                   │ │
│  │    Input: TaskRequest                                    │ │
│  │    Process:                                              │ │
│  │      • Injection detection                               │ │
│  │      • Schema validation                                 │ │
│  │      • Content filtering                                 │ │
│  │    Output: ValidationResult {                            │ │
│  │      valid: true,                                        │ │
│  │      sanitized_input: "...",                             │ │
│  │      detected_issues: []                                 │ │
│  │    }                                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 3. RESOURCE MANAGER (Budget Check)                      │ │
│  │    Input: TaskRequest + User Budget                      │ │
│  │    Process:                                              │ │
│  │      • Check remaining budget                            │ │
│  │      • Estimate task cost                                │ │
│  │      • Allocate resources                                │ │
│  │    Output: ResourceAllocation {                          │ │
│  │      approved: true,                                     │ │
│  │      allocated_budget: $1.00,                            │ │
│  │      time_limit: 600s,                                   │ │
│  │      priority: "normal"                                  │ │
│  │    }                                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 4. GOAL ANALYSIS MODULE                                  │ │
│  │    Input: Sanitized task + Context                       │ │
│  │    Process:                                              │ │
│  │      • Semantic parsing                                  │ │
│  │      • Goal extraction                                   │ │
│  │      • Constraint identification                         │ │
│  │      • Context retrieval from Episodic Memory            │ │
│  │      • Complexity classification                         │ │
│  │    Output: GoalStructure {                               │ │
│  │      primary_goal: "Implement JWT authentication",       │ │
│  │      sub_goals: [...],                                   │ │
│  │      constraints: ["backward compatible", ...],          │ │
│  │      complexity: "MODERATE",                             │ │
│  │      recommended_strategy: "HTN_PLANNING",               │ │
│  │      retrieved_context: {similar_episodes: [...]}        │ │
│  │    }                                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 5. PLANNING ENGINE                                       │ │
│  │    Input: GoalStructure                                  │ │
│  │    Process:                                              │ │
│  │      • Query Pattern Cache for applicable patterns       │ │
│  │      • Hierarchical task decomposition (HTN)             │ │
│  │      • Dependency analysis                               │ │
│  │      • Resource estimation                               │ │
│  │      • Plan optimization                                 │ │
│  │      • Contingency planning                              │ │
│  │    Output: ExecutionPlan {                               │ │
│  │      tasks: [                                            │ │
│  │        {id: "S1", desc: "Research JWT libs", ...},       │ │
│  │        {id: "S2", desc: "Design flow", ...},             │ │
│  │        {id: "S3", desc: "Implement", ...},               │ │
│  │        ...                                               │ │
│  │      ],                                                  │ │
│  │      dependencies: DAG,                                  │ │
│  │      execution_order: [[S1], [S2], [S3, S4], ...],      │ │
│  │      estimated_duration: 245s,                           │ │
│  │      estimated_cost: $0.18,                              │ │
│  │      contingencies: [...]                                │ │
│  │    }                                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 6. EXECUTION ENGINE (Loop over tasks)                    │ │
│  │    FOR each task IN execution_order:                     │ │
│  │      ↓                                                    │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6a. MODEL ROUTER                                 │  │ │
│  │    │   Input: Task complexity, context size           │  │ │
│  │    │   Output: Selected model (e.g., "claude-sonnet") │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6b. TOOL REGISTRY                                │  │ │
│  │    │   Input: Required capability                     │  │ │
│  │    │   Output: Selected tool(s)                       │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6c. SAFETY VERIFIER (Action Authorization)       │  │ │
│  │    │   Input: Proposed action                         │  │ │
│  │    │   Output: Authorization decision                 │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6d. EXECUTE (LLM + Tools)                        │  │ │
│  │    │   • LLM inference                                │  │ │
│  │    │   • Tool invocation                              │  │ │
│  │    │   • Result capture                               │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6e. SAFETY VERIFIER (Output Validation)          │  │ │
│  │    │   Input: Task output                             │  │ │
│  │    │   Output: Validation result                      │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    ┌──────────────────────────────────────────────────┐  │ │
│  │    │ 6f. WORKING MEMORY (Store result)                │  │ │
│  │    │   Store intermediate result for next task        │  │ │
│  │    └────────────────────┬─────────────────────────────┘  │ │
│  │                         ↓                                  │ │
│  │    END LOOP                                               │ │
│  │                                                            │ │
│  │    Output: ExecutionResult {                              │ │
│  │      status: "SUCCESS",                                   │ │
│  │      outputs: {task_id: output},                          │ │
│  │      trace: [...]                                         │ │
│  │    }                                                      │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 7. REFLECTION MODULE (Async)                             │ │
│  │    Input: ExecutionResult + Full trace                   │ │
│  │    Process:                                              │ │
│  │      • Episode analysis                                  │ │
│  │      • Pattern extraction                                │ │
│  │      • Performance analysis                              │ │
│  │      • Knowledge distillation                            │ │
│  │    Output: ReflectionInsights {                          │ │
│  │      new_patterns: [...],                                │ │
│  │      performance_insights: {...},                        │ │
│  │      recommendations: [...]                              │ │
│  │    }                                                     │ │
│  │    Side Effects:                                         │ │
│  │      • Store episode in Episodic Memory                  │ │
│  │      • Update Pattern Cache                              │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 8. RESOURCE MANAGER (Record actual usage)               │ │
│  │    Update budgets with actual cost                       │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ 9. OBSERVABILITY SYSTEM (Logging & Metrics)              │ │
│  │    • Log complete execution trace                        │ │
│  │    • Update metrics (success rate, latency, cost)        │ │
│  │    • Store distributed trace                             │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  USER                                                          │
│    ← FinalResult {                                            │
│        success: true,                                         │
│        output: "JWT authentication implemented",              │
│        cost: $0.19,                                           │
│        duration: 265s                                         │
│      }                                                        │
└────────────────────────────────────────────────────────────────┘
```

### 1.2 Data Transformations

**Key transformations attraverso il flow**:

```
┌──────────────────────────────────────────────────────────┐
│              DATA TRANSFORMATION STAGES                  │
│                                                          │
│  Stage 1: NATURAL LANGUAGE                               │
│  "Add JWT authentication to API"                         │
│    ↓ (Goal Analysis)                                     │
│                                                          │
│  Stage 2: STRUCTURED GOAL                                │
│  GoalStructure {                                         │
│    primary_goal: {...},                                  │
│    constraints: [...],                                   │
│    complexity: "MODERATE"                                │
│  }                                                       │
│    ↓ (Planning)                                          │
│                                                          │
│  Stage 3: EXECUTION PLAN                                 │
│  ExecutionPlan {                                         │
│    tasks: [T1, T2, ...],                                 │
│    dependencies: DAG,                                    │
│    execution_order: [[T1], [T2, T3], ...]               │
│  }                                                       │
│    ↓ (Execution)                                         │
│                                                          │
│  Stage 4: INTERMEDIATE RESULTS                           │
│  {                                                       │
│    T1_output: "PyJWT chosen",                            │
│    T2_output: "Design doc created",                      │
│    T3_output: "Code implemented",                        │
│    ...                                                   │
│  }                                                       │
│    ↓ (Aggregation)                                       │
│                                                          │
│  Stage 5: FINAL OUTPUT                                   │
│  ExecutionResult {                                       │
│    status: "SUCCESS",                                    │
│    primary_output: "Authentication system with JWT",     │
│    artifacts: [code_files, tests, docs],                 │
│    metadata: {duration, cost, ...}                       │
│  }                                                       │
│    ↓ (Reflection - Async)                                │
│                                                          │
│  Stage 6: LEARNING ARTIFACTS                             │
│  ReflectionInsights {                                    │
│    new_patterns: ["JWT-implementation-pattern"],         │
│    performance_insights: {...},                          │
│    stored_episode: episode_id                            │
│  }                                                       │
└──────────────────────────────────────────────────────────┘
```

## 2. Memory System Interactions

### 2.1 Memory Read Patterns

**Pattern: Retrieve Context for Planning**

```
┌────────────────────────────────────────────────────────────────┐
│            MEMORY RETRIEVAL FOR PLANNING                       │
│                                                                │
│  Planning Engine needs context                                 │
│    │                                                           │
│    ├──→ Query EPISODIC MEMORY                                  │
│    │    Input: Current task description embedding             │
│    │    Process:                                              │
│    │      • Vector similarity search                           │
│    │      • Filter by relevance score > 0.7                    │
│    │      • Rank by recency and success                        │
│    │    Output: Top-5 similar episodes [E1, E2, E3, E4, E5]   │
│    │      Each contains:                                      │
│    │        - Task description                                │
│    │        - Strategy used                                   │
│    │        - Outcome                                         │
│    │        - Key learnings                                   │
│    │    Latency: ~80ms                                        │
│    │                                                           │
│    ├──→ Query PATTERN CACHE                                    │
│    │    Input: Task features {domain: "auth", type: "impl"}   │
│    │    Process:                                              │
│    │      • Feature-based lookup                              │
│    │      • Filter applicable patterns                        │
│    │      • Rank by confidence                                │
│    │    Output: Top-3 patterns [P1, P2, P3]                   │
│    │      Each contains:                                      │
│    │        - Pattern template                                │
│    │        - Application guidance                            │
│    │        - Expected benefits                               │
│    │    Latency: ~20ms (cached)                               │
│    │                                                           │
│    └──→ Load into WORKING MEMORY                               │
│         Combined context: {episodes: [...], patterns: [...]}  │
│         Used by Planning Engine for informed decisions        │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Memory Write Patterns

**Pattern: Store Episode After Completion**

```
┌────────────────────────────────────────────────────────────────┐
│              EPISODE STORAGE FLOW                              │
│                                                                │
│  Task completed → Reflection begins                            │
│    │                                                           │
│    ├──→ Construct Episode Object                               │
│    │    From: ExecutionResult + Trace + Context               │
│    │    Output: Complete Episode structure                    │
│    │                                                           │
│    ├──→ Generate Embeddings                                    │
│    │    • Task description → embedding (768-dim vector)        │
│    │    • Full episode → embedding                            │
│    │    • Outcome → embedding                                 │
│    │    Using: Embedding model (e.g., text-embedding-3)       │
│    │                                                           │
│    ├──→ Store in EPISODIC MEMORY                               │
│    │    Parallel writes:                                      │
│    │      • Vector DB ← embeddings + episode_id               │
│    │      • Document DB ← full episode document               │
│    │      • Object Storage ← large artifacts (if any)         │
│    │    Transaction: Atomic (all or nothing)                  │
│    │    Latency: ~150ms                                       │
│    │                                                           │
│    └──→ Update Indices                                         │
│         • Add to search indices                               │
│         • Update statistics (episode count, avg success, etc) │
│         • Invalidate relevant caches                          │
│                                                                │
│  Episode now available for future retrievals                   │
└────────────────────────────────────────────────────────────────┘
```

### 2.3 Pattern Cache Update

**Pattern: Update Pattern from Reflection**

```
┌────────────────────────────────────────────────────────────────┐
│               PATTERN CACHE UPDATE FLOW                        │
│                                                                │
│  Reflection discovers new pattern                              │
│    │                                                           │
│    ├──→ Check if pattern already exists                        │
│    │    Query Pattern Cache by similarity                     │
│    │    IF exists: Update evidence                            │
│    │    IF not: Create new candidate pattern                  │
│    │                                                           │
│    ├──→ IF NEW PATTERN:                                        │
│    │    │                                                     │
│    │    ├─→ Create Pattern Object                             │
│    │    │   Status: CANDIDATE                                 │
│    │    │   Evidence: [current episode]                       │
│    │    │   Confidence: Low (single episode)                  │
│    │    │                                                     │
│    │    └─→ Store in Pattern Cache                            │
│    │        Monitor for additional evidence                   │
│    │                                                           │
│    └──→ IF EXISTING PATTERN:                                   │
│         │                                                     │
│         ├─→ Add Episode to Evidence                           │
│         │   pattern.evidence.supporting_episodes.append(...)  │
│         │                                                     │
│         ├─→ Recompute Statistics                              │
│         │   • Success rate                                    │
│         │   • Statistical significance                        │
│         │   • Confidence score                                │
│         │                                                     │
│         ├─→ Check for Promotion                               │
│         │   IF confidence > threshold AND significance < 0.05:│
│         │     pattern.status = "VALIDATED"                    │
│         │                                                     │
│         └─→ Update in Cache                                   │
│             Persist changes                                   │
│                                                                │
│  Pattern now has more evidence, possibly validated             │
└────────────────────────────────────────────────────────────────┘
```

## 3. Model Router Interactions

### 3.1 Model Selection Flow

```
┌────────────────────────────────────────────────────────────────┐
│                  MODEL SELECTION FLOW                          │
│                                                                │
│  Execution Engine: Need LLM for subtask                        │
│    │                                                           │
│    ├──→ Prepare Request to Model Router                        │
│    │    ModelRequest {                                        │
│    │      task_description: "...",                            │
│    │      complexity: "MODERATE",                             │
│    │      context_size: 12K tokens,                           │
│    │      requires_creativity: false,                         │
│    │      budget_remaining: $0.50                             │
│    │    }                                                     │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ MODEL ROUTER                                             │ │
│  │                                                          │ │
│  │  1. Complexity Classification                            │ │
│  │     Score task → complexity_level = MODERATE             │ │
│  │                                                          │ │
│  │  2. Context Window Check                                 │ │
│  │     12K tokens → Fits in Tier 2 (up to 128K)            │ │
│  │                                                          │ │
│  │  3. Apply Routing Policy                                 │ │
│  │     IF complexity == MODERATE:                           │ │
│  │       Default → Tier 2 (Medium model)                    │ │
│  │                                                          │ │
│  │  4. Cost Check                                           │ │
│  │     Estimated cost: $0.08                                │ │
│  │     Budget remaining: $0.50                              │ │
│  │     ✓ Approved                                           │ │
│  │                                                          │ │
│  │  5. Model Selection                                      │ │
│  │     Selected: "claude-sonnet-3.5"                        │ │
│  │     Fallback chain: ["gpt-4o-mini", "gpt-4-turbo"]      │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ MODEL INVOCATION LAYER                                   │ │
│  │                                                          │ │
│  │  1. Load Balance                                         │ │
│  │     Select instance: claude-sonnet-instance-3            │ │
│  │                                                          │ │
│  │  2. API Call                                             │ │
│  │     POST /v1/chat/completions                            │ │
│  │     Body: {model, messages, max_tokens, ...}             │ │
│  │                                                          │ │
│  │  3. Response Handling                                    │ │
│  │     Parse response                                       │ │
│  │     Extract: text, tokens_used, finish_reason            │ │
│  │                                                          │ │
│  │  4. Fallback (if error)                                  │ │
│  │     IF error: Try next model in fallback chain           │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return to Execution Engine                                    │
│    ModelResponse {                                            │
│      text: "...",                                             │
│      model_used: "claude-sonnet-3.5",                         │
│      tokens: {input: 12234, output: 1456},                    │
│      cost: $0.082,                                            │
│      latency: 4.2s                                            │
│    }                                                          │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 Model Performance Tracking

**Data flow for continuous improvement**:

```
┌────────────────────────────────────────────────────────────────┐
│            MODEL PERFORMANCE TRACKING FLOW                     │
│                                                                │
│  Every model invocation →                                      │
│    │                                                           │
│    ├──→ Record Metrics                                         │
│    │    {                                                     │
│    │      model_id: "claude-sonnet-3.5",                      │
│    │      task_type: "code_generation",                       │
│    │      complexity: "MODERATE",                             │
│    │      latency: 4.2s,                                      │
│    │      cost: $0.082,                                       │
│    │      tokens: {in: 12234, out: 1456},                     │
│    │      success: true,                                      │
│    │      quality_score: 0.89  // From verification          │
│    │    }                                                     │
│    │                                                           │
│    ├──→ Aggregate into Time-Series DB                          │
│    │    • 1-minute buckets for real-time monitoring           │
│    │    • 1-hour buckets for trending                         │
│    │    • Daily aggregates for reporting                      │
│    │                                                           │
│    ├──→ Update Model Statistics                                │
│    │    model_stats["claude-sonnet-3.5"] {                    │
│    │      avg_latency: 4.5s (rolling average),               │
│    │      avg_cost: $0.085,                                   │
│    │      success_rate: 94.2%,                                │
│    │      quality_score: 0.87,                                │
│    │      total_invocations: 15,432                           │
│    │    }                                                     │
│    │                                                           │
│    └──→ Trigger Analysis (Weekly)                              │
│         • Compare models performance                          │
│         • Identify optimization opportunities                 │
│         • Adjust routing weights if needed                    │
│         • Generate recommendations                            │
│                                                                │
│  Example Output:                                               │
│    "Model claude-sonnet-3.5 shows 15% higher success rate     │
│     than gpt-4o-mini for code generation tasks. Recommend     │
│     increasing routing weight for this model."                │
└────────────────────────────────────────────────────────────────┘
```

## 4. Safety Verification Flows

### 4.1 Input Validation Flow

```
┌────────────────────────────────────────────────────────────────┐
│                INPUT VALIDATION FLOW                           │
│                                                                │
│  User Input: "Delete all files in /important/data"             │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ SAFETY VERIFIER - INPUT VALIDATOR                        │ │
│  │                                                          │ │
│  │  Layer 1: Schema Validation                              │ │
│  │    ✓ Input is valid string                               │ │
│  │    ✓ Length within limits                                │ │
│  │                                                          │ │
│  │  Layer 2: Injection Detection                            │ │
│  │    Run: CHECK_COMMAND_INJECTION(input)                   │ │
│  │    Detected: Potential dangerous command "delete"        │ │
│  │    Score: 0.85 (high risk)                               │ │
│  │                                                          │ │
│  │  Layer 3: Content Policy                                 │ │
│  │    Check: Request involves destructive operation         │ │
│  │    Path: /important/data (sensitive location)            │ │
│  │    Flag: Requires explicit approval                      │ │
│  │                                                          │ │
│  │  Decision: REQUIRES_HUMAN_APPROVAL                       │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ APPROVAL REQUEST TO USER                                 │ │
│  │                                                          │ │
│  │  "This action will delete files in /important/data.      │ │
│  │   This is a destructive operation.                       │ │
│  │   Do you want to proceed?"                               │ │
│  │                                                          │ │
│  │   [Approve] [Deny] [Modify Request]                      │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  IF APPROVED:                                                  │
│    Proceed with goal analysis                                 │
│    Log approval in audit trail                                │
│                                                                │
│  IF DENIED:                                                    │
│    Return error to user                                       │
│    Log denial                                                 │
│    Suggest safer alternatives                                 │
└────────────────────────────────────────────────────────────────┘
```

### 4.2 Action Authorization Flow

```
┌────────────────────────────────────────────────────────────────┐
│              ACTION AUTHORIZATION FLOW                         │
│                                                                │
│  Execution Engine wants to: write_file("/app/config.json", data)│
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ SAFETY VERIFIER - ACTION AUTHORIZER                      │ │
│  │                                                          │ │
│  │  Step 1: Permission Check                                │ │
│  │    Required: FILE_WRITE permission                       │ │
│  │    User has: [FILE_READ, FILE_WRITE, WEB_FETCH]         │ │
│  │    ✓ Permission granted                                  │ │
│  │                                                          │ │
│  │  Step 2: Path Validation                                 │ │
│  │    Requested path: "/app/config.json"                    │ │
│  │    Check whitelist: /app/** (allowed)                    │ │
│  │    Check blacklist: /etc/**, /sys/** (not matched)       │ │
│  │    ✓ Path allowed                                        │ │
│  │                                                          │ │
│  │  Step 3: Tool Safety Check                               │ │
│  │    Tool: write_file                                      │ │
│  │    Dangerous: false                                      │ │
│  │    Reversible: true (can restore from backup)            │ │
│  │    Side effects: [FILE_MODIFICATION]                     │ │
│  │    ✓ Tool safe for use                                   │ │
│  │                                                          │ │
│  │  Step 4: Rate Limit Check                                │ │
│  │    Current rate: 5 file writes in last minute            │ │
│  │    Limit: 20 writes/minute                               │ │
│  │    ✓ Under limit                                         │ │
│  │                                                          │ │
│  │  Step 5: Resource Budget Check                           │ │
│  │    Estimated cost: $0.001                                │ │
│  │    Budget remaining: $0.25                               │ │
│  │    ✓ Budget OK                                           │ │
│  │                                                          │ │
│  │  Decision: APPROVED                                      │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ TOOL REGISTRY - EXECUTE                                  │ │
│  │   Execute: write_file("/app/config.json", data)          │ │
│  │   Result: Success                                        │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ AUDIT LOGGER                                             │ │
│  │   Log {                                                  │ │
│  │     action: "FILE_WRITE",                                │ │
│  │     path: "/app/config.json",                            │ │
│  │     authorized: true,                                    │ │
│  │     executed: true,                                      │ │
│  │     result: "success"                                    │ │
│  │   }                                                      │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Output Validation Flow

```
┌────────────────────────────────────────────────────────────────┐
│               OUTPUT VALIDATION FLOW                           │
│                                                                │
│  Task completed, output generated:                             │
│    Code: (Python function for JWT encoding)                    │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ SAFETY VERIFIER - OUTPUT VALIDATOR                       │ │
│  │                                                          │ │
│  │  Check 1: Schema Validation                              │ │
│  │    Expected: Python code (string)                        │ │
│  │    Actual: String containing Python code                 │ │
│  │    ✓ Schema valid                                        │ │
│  │                                                          │ │
│  │  Check 2: Syntax Validation                              │ │
│  │    Parse with Python AST parser                          │ │
│  │    Result: Valid Python syntax                           │ │
│  │    ✓ Syntax valid                                        │ │
│  │                                                          │ │
│  │  Check 3: Security Scan                                  │ │
│  │    Scan for dangerous patterns:                          │ │
│  │      • eval() → Not found ✓                              │ │
│  │      • exec() → Not found ✓                              │ │
│  │      • __import__() → Not found ✓                        │ │
│  │      • os.system() → Not found ✓                         │ │
│  │    ✓ No security issues                                  │ │
│  │                                                          │ │
│  │  Check 4: Secret Detection                               │ │
│  │    Scan for secret patterns:                             │ │
│  │      • API keys → Not found ✓                            │ │
│  │      • Passwords → Not found ✓                           │ │
│  │      • Private keys → Not found ✓                        │ │
│  │    ✓ No secrets leaked                                   │ │
│  │                                                          │ │
│  │  Check 5: Quality Thresholds                             │ │
│  │    Assess code quality:                                  │ │
│  │      • Has docstring: ✗ (warning)                        │ │
│  │      • Has type hints: ✓                                 │ │
│  │      • Error handling: ✓                                 │ │
│  │    Overall quality: 0.85 (above threshold 0.7)          │ │
│  │    ✓ Quality acceptable                                  │ │
│  │                                                          │ │
│  │  Decision: APPROVED (with 1 warning)                     │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return to Execution Engine:                                   │
│    ValidationResult {                                         │
│      approved: true,                                          │
│      warnings: ["Missing docstring"],                         │
│      validated_output: (original output)                      │
│    }                                                          │
└────────────────────────────────────────────────────────────────┘
```

## 5. Error Handling Flows

### 5.1 Error Detection and Recovery

```
┌────────────────────────────────────────────────────────────────┐
│          ERROR DETECTION AND RECOVERY FLOW                     │
│                                                                │
│  Execution: tool_invocation("web_search", query="...")         │
│    │                                                           │
│    ├──→ Tool execution fails                                   │
│    │    Error: TimeoutError("Request timed out after 30s")    │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ ERROR HANDLER - CLASSIFIER                               │ │
│  │                                                          │ │
│  │  Analyze Error:                                          │ │
│  │    Type: TimeoutError                                    │ │
│  │    Component: Tool (web_search)                          │ │
│  │    Category: TRANSIENT (network issue)                   │ │
│  │    Severity: MEDIUM                                      │ │
│  │    Recoverable: YES (retry likely to work)               │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ ERROR HANDLER - RECOVERY ENGINE                          │ │
│  │                                                          │ │
│  │  Select Strategy: RETRY with exponential backoff         │ │
│  │                                                          │ │
│  │  Attempt 1: Immediate retry                              │ │
│  │    → FAIL (timeout again)                                │ │
│  │    Wait: 2s                                              │ │
│  │                                                          │ │
│  │  Attempt 2: Retry after 2s                               │ │
│  │    → FAIL (timeout again)                                │ │
│  │    Wait: 4s                                              │ │
│  │                                                          │ │
│  │  Attempt 3: Retry after 4s                               │ │
│  │    → SUCCESS!                                            │ │
│  │    Result: Search results returned                       │ │
│  │                                                          │ │
│  │  Recovery: SUCCESSFUL                                    │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ OBSERVABILITY SYSTEM - LOGGING                           │ │
│  │                                                          │ │
│  │  Log Entry {                                             │ │
│  │    level: "WARNING",                                     │ │
│  │    event: "tool_timeout_recovered",                      │ │
│  │    tool: "web_search",                                   │ │
│  │    attempts: 3,                                          │ │
│  │    total_delay: 6s,                                      │ │
│  │    outcome: "success"                                    │ │
│  │  }                                                       │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return to Execution Engine:                                   │
│    RecoveryResult {                                           │
│      success: true,                                           │
│      output: (search results),                                │
│      recovery_applied: "retry_with_backoff",                  │
│      attempts_needed: 3,                                      │
│      delay_incurred: 6s                                       │
│    }                                                          │
│                                                                │
│  Task continues with successful result                         │
└────────────────────────────────────────────────────────────────┘
```

### 5.2 Error Escalation Flow

```
┌────────────────────────────────────────────────────────────────┐
│                ERROR ESCALATION FLOW                           │
│                                                                │
│  Multiple tool failures → Recovery not working                 │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ ERROR HANDLER - ESCALATION MANAGER                       │ │
│  │                                                          │ │
│  │  Assess Situation:                                       │ │
│  │    • 3 consecutive tool failures                         │ │
│  │    • All recovery attempts exhausted                     │ │
│  │    • Task cannot proceed                                 │ │
│  │    • Severity: HIGH                                      │ │
│  │                                                          │ │
│  │  Decision: ESCALATE to user                              │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ CREATE INCIDENT                                          │ │
│  │                                                          │ │
│  │  Incident {                                              │ │
│  │    id: "INC-123",                                        │ │
│  │    type: "TASK_FAILURE",                                 │ │
│  │    severity: "HIGH",                                     │ │
│  │    task_id: "T-12345",                                   │ │
│  │    error_summary: "Multiple tool failures",              │ │
│  │    recovery_attempts: [...]                              │ │
│  │  }                                                       │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ NOTIFY USER                                              │ │
│  │                                                          │ │
│  │  Message to User:                                        │ │
│  │  "Task 'Add JWT authentication' encountered an error     │ │
│  │   and could not be completed automatically.              │ │
│  │                                                          │ │
│  │   Error: Multiple attempts to search documentation       │ │
│  │   failed due to network timeouts.                        │ │
│  │                                                          │ │
│  │   Options:                                               │ │
│  │   1. Retry task (may fail again)                         │ │
│  │   2. Skip documentation search step                      │ │
│  │   3. Provide manual input for this step                  │ │
│  │   4. Cancel task                                         │ │
│  │                                                          │ │
│  │   Incident ID: INC-123"                                  │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ ALERT ENGINEERING (if critical)                          │ │
│  │                                                          │ │
│  │  IF severity == CRITICAL:                                │ │
│  │    Send alert to on-call engineer                        │ │
│  │    Include: incident details, system state, logs         │ │
│  │                                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Wait for user decision or engineering intervention            │
└────────────────────────────────────────────────────────────────┘
```

## 6. Observability Data Flows

### 6.1 Logging Flow

```
┌────────────────────────────────────────────────────────────────┐
│                    LOGGING DATA FLOW                           │
│                                                                │
│  Every component generates log entries →                       │
│    │                                                           │
│    ├──→ LOG_INFO("Task started", {task_id: "T-123", ...})     │
│    ├──→ LOG_DEBUG("Context retrieved", {...})                 │
│    ├──→ LOG_WARNING("Fallback used", {...})                   │
│    ├──→ LOG_ERROR("Operation failed", {...})                  │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ LOG AGGREGATOR                                           │ │
│  │  • Collect from all sources                              │ │
│  │  • Add correlation_id (trace_id, task_id)                │ │
│  │  • Enrich with context (host, component, timestamp)      │ │
│  │  • Buffer (100 entries or 5s, whichever first)           │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ BATCH WRITE TO STORAGE                                   │ │
│  │  • Elasticsearch (if self-hosted)                        │ │
│  │  • CloudWatch Logs (if AWS)                              │ │
│  │  • Retention: 30 days hot, 1 year archive                │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ INDEX FOR SEARCH                                         │ │
│  │  • Full-text index on message                            │ │
│  │  • Field indices on: level, component, user_id, task_id │ │
│  │  • Time-based partitioning for efficiency               │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Available for:                                                │
│    • Real-time search (Kibana, CloudWatch Insights)           │
│    • Debugging (trace all events for specific task)           │
│    • Analytics (error rate trends, performance analysis)      │
│    • Compliance (audit trail)                                 │
└────────────────────────────────────────────────────────────────┘
```

### 6.2 Metrics Flow

```
┌────────────────────────────────────────────────────────────────┐
│                   METRICS DATA FLOW                            │
│                                                                │
│  Component operations generate metrics →                       │
│    │                                                           │
│    ├──→ COUNTER("tasks_started", labels={user_id, type})      │
│    ├──→ HISTOGRAM("task_duration_seconds", value=245.3)       │
│    ├──→ GAUGE("active_tasks", value=12)                       │
│    ├──→ COUNTER("llm_cost_dollars", value=0.18)               │
│    │                                                           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ METRICS COLLECTOR                                        │ │
│  │  • In-memory aggregation (1-minute window)               │ │
│  │  • Compute statistics: count, sum, min, max, percentiles │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ WRITE TO TIME-SERIES DB                                  │ │
│  │  • Prometheus (self-hosted)                              │ │
│  │  • CloudWatch Metrics (AWS)                              │ │
│  │  • InfluxDB (alternative)                                │ │
│  │  • Every 1 minute for detailed metrics                   │ │
│  │  • Downsampled to 5min, 1hr, 1day for long-term         │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ VISUALIZATION & ALERTING                                 │ │
│  │  • Grafana dashboards (real-time)                        │ │
│  │  • Alert rules evaluation (every 1min)                   │ │
│  │  • Anomaly detection (ML-based, optional)                │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ ALERT ROUTING (if threshold breached)                    │ │
│  │  • PagerDuty (critical alerts)                           │ │
│  │  • Slack (warnings)                                      │ │
│  │  • Email (info)                                          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  Used for:                                                     │
│    • Real-time monitoring (system health)                     │
│    • Performance optimization (identify bottlenecks)           │
│    • Capacity planning (predict resource needs)               │
│    • SLA tracking (success rate, latency targets)             │
└────────────────────────────────────────────────────────────────┘
```

### 6.3 Tracing Flow

```
┌────────────────────────────────────────────────────────────────┐
│                  DISTRIBUTED TRACING FLOW                      │
│                                                                │
│  Request arrives → trace_id generated                          │
│    │                                                           │
│    ├──→ Root Span created                                      │
│    │    trace_id: "abc-123"                                   │
│    │    span_id: "span-1"                                     │
│    │    operation: "handle_task"                              │
│    │                                                           │
│    └──→ As request flows through system:                       │
│         │                                                     │
│         ├─→ Goal Analysis creates child span                  │
│         │   span_id: "span-2", parent: "span-1"              │
│         │                                                     │
│         ├─→ Planning creates child span                       │
│         │   span_id: "span-3", parent: "span-1"              │
│         │   │                                                │
│         │   ├─→ Pattern Cache query creates span             │
│         │   │   span_id: "span-4", parent: "span-3"          │
│         │   │                                                │
│         │   └─→ Memory query creates span                    │
│         │       span_id: "span-5", parent: "span-3"          │
│         │                                                     │
│         └─→ Execution creates multiple spans                  │
│             (one per subtask)                                │
│             span_id: "span-6..N", parent: various           │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ SPAN COLLECTION                                          │ │
│  │  • Each span sent to collector when complete             │ │
│  │  • Includes: timing, tags, events, status                │ │
│  │  • Batched for efficiency                                │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ TRACE STORAGE                                            │ │
│  │  • Jaeger (open source)                                  │ │
│  │  • AWS X-Ray (managed)                                   │ │
│  │  • Store: span data + relationships                      │ │
│  │  • Retention: 7 days full detail, 30 days sampled        │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ TRACE VISUALIZATION                                      │ │
│  │  • Waterfall view of spans                               │ │
│  │  • Gantt chart showing parallelism                       │ │
│  │  • Critical path highlighting                            │ │
│  │  • Flamegraph for CPU profiling                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Available for:                                                │
│    • Debugging slow requests (where is time spent?)           │
│    • Understanding dependencies (what calls what?)            │
│    • Finding bottlenecks (which operations are slow?)         │
│    • Optimizing parallelism (what could run in parallel?)     │
└────────────────────────────────────────────────────────────────┘
```

---

**Next**: [08-deployment.md](08-deployment.md) → Deployment models, scaling strategies, operational considerations
