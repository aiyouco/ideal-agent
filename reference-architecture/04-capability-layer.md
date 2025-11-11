# Capability Layer: Tools, Models, and Safety

## Overview

Il Capability Layer fornisce le interfacce concrete attraverso cui l'agente interagisce col mondo: strumenti per azioni esterne, modelli per reasoning, e sistemi di sicurezza per bounded emergence. È il layer che traduce intenzioni high-level in azioni concrete sicure.

```
┌───────────────────────────────────────────────────────────────┐
│                     CAPABILITY LAYER                          │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  TOOL REGISTRY                                          │ │
│  │  • Tool discovery and registration                      │ │
│  │  • Capability matching                                  │ │
│  │  • Tool binding and invocation                          │ │
│  │  • Permission management                                │ │
│  │  Purpose: Enable agent to use external capabilities     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  MODEL ROUTER                                           │ │
│  │  • Model selection based on task                        │ │
│  │  • Cost/quality optimization                            │ │
│  │  • Load balancing and fallback                          │ │
│  │  • Context window management                            │ │
│  │  Purpose: Optimize LLM usage for efficiency             │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  SAFETY VERIFIER                                        │ │
│  │  • Input validation and sanitization                    │ │
│  │  • Output verification                                  │ │
│  │  • Action authorization                                 │ │
│  │  • Bounded emergence enforcement                        │ │
│  │  Purpose: Ensure agent operates within safe bounds      │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Interaction Flow:                                            │
│  1. Execution Engine needs to perform action                  │
│  2. Safety Verifier validates request                         │
│  3. Model Router selects appropriate LLM if needed            │
│  4. Tool Registry provides and executes tool                  │
│  5. Safety Verifier validates result before returning         │
└───────────────────────────────────────────────────────────────┘
```

## 1. Tool Registry

### 1.1 Purpose & Responsibilities

**Core Function**: Gestire catalogo di capabilities esterne (tools) disponibili all'agente e mediare loro utilizzo.

**Characteristics**:
- **Extensible**: Nuovi tool facilmente aggiungibili
- **Discoverable**: Agent può trovare tool per capability richiesta
- **Secure**: Permission-based access control
- **Versioned**: Supporto multiple versioni di stesso tool

**Responsibilities**:
1. **Tool Registration**: Registrare nuovi tool con metadata
2. **Discovery**: Trovare tool che soddisfano requirements
3. **Capability Matching**: Match task needs → available tools
4. **Invocation**: Eseguire tool calls in modo sicuro
5. **Result Handling**: Parse e validate tool outputs
6. **Error Handling**: Gestire tool failures gracefully

### 1.2 Tool Specification Schema

**Complete Tool Definition**:
```
Tool {
  // Identification
  tool_id: string (UUID),
  name: string,  // Human-readable name
  version: string,  // Semantic versioning (e.g., "2.1.0")

  // Description
  description: string,
  detailed_description: string,
  category: ToolCategory,  // "file_ops", "web", "compute", etc.

  // Capabilities
  capabilities: [string],  // Semantic tags: ["read_file", "text_file"]

  // Interface
  input_schema: JSONSchema,  // What parameters tool accepts
  output_schema: JSONSchema,  // What tool returns

  // Examples
  examples: [
    {
      description: string,
      input: {...},
      expected_output: {...}
    }
  ],

  // Execution
  invocation: {
    type: "FUNCTION" | "HTTP_API" | "CLI" | "PLUGIN",

    // For FUNCTION type
    function_ref: string | callable,

    // For HTTP_API type
    endpoint: string,
    method: "GET" | "POST" | "PUT" | "DELETE",
    auth: AuthConfig,

    // For CLI type
    command_template: string,

    // For PLUGIN type
    plugin_loader: string
  },

  // Safety & Permissions
  safety: {
    permission_required: Permission,
    dangerous: boolean,  // Requires extra validation
    side_effects: [SideEffect],  // What it changes
    reversible: boolean,  // Can action be undone
    idempotent: boolean,  // Safe to retry
    rate_limit: RateLimit
  },

  // Performance
  performance: {
    typical_latency: float,  // Seconds
    timeout: float,  // Max execution time
    cost: float | null,  // Per invocation, if applicable
    resource_intensive: boolean
  },

  // Dependencies
  dependencies: {
    required_tools: [tool_id],
    required_permissions: [Permission],
    required_environment: {
      os: [string],
      libraries: [string]
    }
  },

  // Metadata
  metadata: {
    author: string,
    documentation_url: string,
    source_code_url: string,
    license: string,
    tags: [string]
  },

  // Lifecycle
  lifecycle: {
    status: "ACTIVE" | "DEPRECATED" | "DISABLED",
    deprecated_by: tool_id | null,
    created_at: datetime,
    updated_at: datetime
  }
}
```

### 1.3 Tool Registry Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                      TOOL REGISTRY                             │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  TOOL CATALOG (Storage)                                  │ │
│  │  • Database of registered tools                          │ │
│  │  • Tool metadata and schemas                             │ │
│  │  • Version management                                    │ │
│  │  Technology: PostgreSQL + Redis cache                    │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  CAPABILITY INDEX (Discovery)                            │ │
│  │  • Inverted index: capability → tools                    │ │
│  │  • Semantic search over tool descriptions                │ │
│  │  • Fast lookup by capability tags                        │ │
│  │  Example: "read_file" → [tool_1, tool_2, tool_3]         │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  TOOL EXECUTOR (Invocation)                              │ │
│  │  • Handles different invocation types                    │ │
│  │  • Input validation against schema                       │ │
│  │  • Execution sandboxing                                  │ │
│  │  • Output parsing and validation                         │ │
│  │  • Timeout and error handling                            │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  PERMISSION MANAGER                                      │ │
│  │  • Permission checking                                   │ │
│  │  • Audit logging                                         │ │
│  │  • Rate limiting enforcement                             │ │
│  │  • Human approval gates for dangerous operations         │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 1.4 Tool Discovery and Matching

**Discovery Algorithm**:
```
Function DISCOVER_TOOLS(requirement, context):

  # STEP 1: Parse requirement
  # Requirement might be natural language or structured

  IF requirement.is_structured:
    # Structured: {capability: "read_file", constraints: {...}}
    capabilities_needed = requirement.capabilities
    constraints = requirement.constraints
  ELSE:
    # Natural language: "I need to read a JSON file"
    # Use LLM to extract capabilities
    capabilities_needed = LLM.extract_capabilities(requirement.text)
    constraints = LLM.extract_constraints(requirement.text)

  # STEP 2: Query by capabilities
  candidate_tools = []

  FOR capability IN capabilities_needed:
    # Query capability index
    tools_with_capability = INDEX.query(capability)
    candidate_tools.extend(tools_with_capability)

  # Remove duplicates
  candidate_tools = UNIQUE(candidate_tools)

  # STEP 3: Filter by constraints
  filtered_tools = []

  FOR tool IN candidate_tools:
    # Check if tool meets all constraints
    IF MATCHES_CONSTRAINTS(tool, constraints, context):
      filtered_tools.append(tool)

  # STEP 4: Rank by suitability
  ranked_tools = RANK_TOOLS(filtered_tools, requirement, context)

  # STEP 5: Return top matches
  RETURN ranked_tools[:5]

Function MATCHES_CONSTRAINTS(tool, constraints, context):
  # Permission check
  IF tool.safety.permission_required NOT IN context.available_permissions:
    RETURN False

  # Environment check
  IF NOT environment_compatible(tool, context.environment):
    RETURN False

  # Performance constraint
  IF constraints.max_latency AND tool.performance.typical_latency > constraints.max_latency:
    RETURN False

  # Cost constraint
  IF constraints.max_cost AND tool.performance.cost > constraints.max_cost:
    RETURN False

  # Safety constraint
  IF constraints.safe_only AND tool.safety.dangerous:
    RETURN False

  RETURN True

Function RANK_TOOLS(tools, requirement, context):
  scored = []

  FOR tool IN tools:
    score = 0

    # Capability match quality (primary)
    match_quality = COMPUTE_CAPABILITY_MATCH(
      tool.capabilities,
      requirement.capabilities_needed
    )
    score += 0.4 * match_quality

    # Usage frequency (popular tools preferred)
    usage_factor = LOG(1 + tool.usage_count) / 10
    score += 0.2 * MIN(usage_factor, 1.0)

    # Performance score (faster is better)
    latency_score = 1 / (1 + tool.performance.typical_latency)
    score += 0.15 * latency_score

    # Cost score (cheaper is better)
    IF tool.performance.cost:
      cost_score = 1 / (1 + tool.performance.cost * 10)
      score += 0.15 * cost_score
    ELSE:
      score += 0.15  # Free tools get full cost score

    # Safety score (safer is better)
    safety_score = 1.0 if not tool.safety.dangerous else 0.5
    score += 0.1 * safety_score

    scored.append((tool, score))

  RETURN SORT_BY_SCORE(scored, descending=True)
```

### 1.5 Tool Invocation

**Secure Invocation Flow**:
```
┌────────────────────────────────────────────────────────────────┐
│                   TOOL INVOCATION PIPELINE                     │
│                                                                │
│  User Request: Execute tool X with parameters Y                │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  1. PERMISSION CHECK                                     │ │
│  │  • Verify caller has required permission                 │ │
│  │  • Check rate limits not exceeded                        │ │
│  │  • Log request for audit                                 │ │
│  │  IF FAIL: REJECT with error                              │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓ PASS                               │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  2. INPUT VALIDATION                                     │ │
│  │  • Validate against tool.input_schema                    │ │
│  │  • Type checking                                         │ │
│  │  • Range/constraint verification                         │ │
│  │  • Injection attack detection                            │ │
│  │  IF FAIL: REJECT with validation errors                  │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓ PASS                               │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  3. SAFETY VERIFICATION (if dangerous tool)              │ │
│  │  • Check action against safety bounds                    │ │
│  │  • Simulate/dry-run if possible                          │ │
│  │  • Request human approval if needed                      │ │
│  │  IF FAIL: REJECT with safety violation                   │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓ PASS                               │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  4. EXECUTION                                            │ │
│  │  • Prepare execution environment                         │ │
│  │  • Set timeout                                           │ │
│  │  • Invoke tool based on invocation type:                │ │
│  │    - FUNCTION: Call directly                             │ │
│  │    - HTTP_API: Make HTTP request                         │ │
│  │    - CLI: Execute shell command                          │ │
│  │    - PLUGIN: Load and invoke plugin                      │ │
│  │  • Capture output and errors                             │ │
│  │  IF TIMEOUT: Abort and return timeout error              │ │
│  │  IF ERROR: Capture error details                         │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  5. OUTPUT PROCESSING                                    │ │
│  │  • Parse tool output                                     │ │
│  │  • Validate against tool.output_schema                   │ │
│  │  • Extract structured data                               │ │
│  │  • Handle errors gracefully                              │ │
│  │  IF parse error: Wrap in error response                  │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  6. POST-INVOCATION                                      │ │
│  │  • Log execution (tool, params, result, duration)        │ │
│  │  • Update tool usage statistics                          │ │
│  │  • Update rate limit counters                            │ │
│  │  • Store in working memory if needed                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return ToolResult {                                           │
│    success: boolean,                                           │
│    output: {...} | null,                                       │
│    error: Error | null,                                        │
│    metadata: {duration, cost, ...}                             │
│  }                                                             │
└────────────────────────────────────────────────────────────────┘
```

### 1.6 Built-in Tool Categories

**Essential Tools** (sempre disponibili):
```
┌──────────────────────────────────────────────────────────┐
│                  CORE TOOL CATEGORIES                    │
│                                                          │
│  1. FILE OPERATIONS                                      │
│     • read_file: Read file contents                     │
│     • write_file: Write/overwrite file                  │
│     • append_file: Append to file                       │
│     • delete_file: Delete file                          │
│     • list_directory: List files in directory           │
│     • file_info: Get file metadata                      │
│                                                          │
│  2. CODE OPERATIONS                                      │
│     • parse_code: Parse code into AST                   │
│     • format_code: Auto-format code                     │
│     • lint_code: Run linter                             │
│     • run_tests: Execute test suite                     │
│     • static_analysis: Analyze code quality             │
│                                                          │
│  3. WEB OPERATIONS                                       │
│     • http_get: HTTP GET request                        │
│     • http_post: HTTP POST request                      │
│     • fetch_webpage: Fetch and parse HTML               │
│     • web_search: Search web                            │
│                                                          │
│  4. DATA OPERATIONS                                      │
│     • parse_json: Parse JSON string                     │
│     • parse_yaml: Parse YAML                            │
│     • parse_xml: Parse XML                              │
│     • query_database: SQL query                         │
│     • transform_data: Data transformation               │
│                                                          │
│  5. COMPUTATION                                          │
│     • evaluate_expression: Eval math expression         │
│     • run_script: Execute script                        │
│     • execute_command: Run shell command                │
│                                                          │
│  6. COMMUNICATION                                        │
│     • send_notification: Notify user                    │
│     • request_input: Ask user for input                 │
│     • request_approval: Request human approval          │
│                                                          │
│  7. MEMORY OPERATIONS                                    │
│     • search_episodes: Query episodic memory            │
│     • retrieve_pattern: Get pattern from cache          │
│     • store_note: Save note for later                   │
└──────────────────────────────────────────────────────────┘
```

### 1.7 Tool Registry Operations

```
1. REGISTER_TOOL(tool_spec)
   • Validate tool specification
   • Check for name conflicts
   • Assign tool_id
   • Create capability indices
   • Store in catalog
   • Return tool_id

2. UNREGISTER_TOOL(tool_id)
   • Mark tool as disabled
   • Remove from active indices
   • Keep for historical record

3. UPDATE_TOOL(tool_id, updates)
   • Validate updates
   • Increment version
   • Update catalog
   • Update indices
   • Notify dependent tools if breaking change

4. GET_TOOL(tool_id)
   • Retrieve tool spec from catalog
   • Check if active
   • Return tool or error

5. DISCOVER_TOOLS(requirement, context)
   • Parse requirement
   • Query capability index
   • Filter by constraints
   • Rank by suitability
   • Return top matches

6. INVOKE_TOOL(tool_id, parameters, context)
   • Permission check
   • Input validation
   • Safety verification (if dangerous)
   • Execute tool
   • Output processing
   • Logging
   • Return result

7. LIST_TOOLS(filters)
   • Query catalog with filters
   • Return matching tools

8. GET_TOOL_USAGE_STATS(tool_id, time_range)
   • Query usage logs
   • Compute statistics
   • Return metrics

9. CHECK_PERMISSIONS(tool_id, context)
   • Verify required permissions available
   • Return boolean + missing permissions if any

10. REQUEST_APPROVAL(tool_id, parameters, reason)
    • For dangerous operations
    • Present request to human
    • Await approval/denial
    • Return decision
```

## 2. Model Router

### 2.1 Purpose & Responsibilities

**Core Function**: Selezionare optimal LLM per ogni reasoning task, bilanciando quality, cost, e latency.

**Key Insight**: Not all tasks require most capable (expensive) model. Strategic routing can reduce cost 5-10x without quality loss.

**Responsibilities**:
1. **Model Selection**: Choose appropriate model for task
2. **Load Balancing**: Distribute requests across instances
3. **Failover**: Handle model unavailability
4. **Context Management**: Ensure task fits in model's context window
5. **Cost Optimization**: Minimize spend while meeting quality targets
6. **Performance Tracking**: Monitor model performance per task type

### 2.2 Model Tier System

**Three-Tier Architecture**:
```
┌──────────────────────────────────────────────────────────┐
│                    MODEL TIERS                           │
│                                                          │
│  TIER 1: SMALL/FAST MODELS                               │
│  ├─ Examples: GPT-3.5, Claude Haiku, Llama 2 7B         │
│  ├─ Cost: $0.001-0.002 per 1K tokens                    │
│  ├─ Latency: 1-3s typical                               │
│  ├─ Context: 4K-8K tokens                               │
│  ├─ Strengths: Speed, cost, simple tasks                │
│  └─ Use for: Simple queries, formatting, classification  │
│                                                          │
│  TIER 2: MEDIUM MODELS                                   │
│  ├─ Examples: GPT-4o-mini, Claude Sonnet               │
│  ├─ Cost: $0.01-0.03 per 1K tokens                      │
│  ├─ Latency: 3-8s typical                               │
│  ├─ Context: 16K-128K tokens                            │
│  ├─ Strengths: Balance of capability and cost           │
│  └─ Use for: Most coding, moderate complexity tasks     │
│                                                          │
│  TIER 3: LARGE/CAPABLE MODELS                            │
│  ├─ Examples: GPT-4, Claude Opus, GPT-4-turbo          │
│  ├─ Cost: $0.03-0.10 per 1K tokens                      │
│  ├─ Latency: 5-15s typical                              │
│  ├─ Context: 128K-200K tokens                           │
│  ├─ Strengths: Maximum capability, complex reasoning    │
│  └─ Use for: Complex architecture, novel problems       │
└──────────────────────────────────────────────────────────┘
```

### 2.3 Routing Decision Tree

**Decision Algorithm**:
```
Function SELECT_MODEL(task, context):

  # STEP 1: Classify task complexity
  complexity = CLASSIFY_COMPLEXITY(task)
  # Returns: SIMPLE | MODERATE | COMPLEX

  # STEP 2: Check for routing hints
  IF context.model_hint:
    # Explicit hint from planning (e.g., "use_best_model")
    IF context.model_hint == "best":
      RETURN TIER_3_MODEL
    ELSE IF context.model_hint == "fast":
      RETURN TIER_1_MODEL

  # STEP 3: Route by complexity and other factors

  IF complexity == SIMPLE:
    # Simple tasks → Fast models

    IF task.requires_creativity:
      # Creative tasks need better models
      RETURN TIER_2_MODEL
    ELSE IF task.is_classification:
      # Classification fine with small model
      RETURN TIER_1_MODEL
    ELSE IF task.is_formatting:
      # Formatting trivial
      RETURN TIER_1_MODEL
    ELSE:
      # Default simple
      RETURN TIER_1_MODEL

  ELSE IF complexity == MODERATE:
    # Moderate tasks → Medium models mostly

    IF task.is_coding AND task.language IN WELL_KNOWN_LANGUAGES:
      # Common coding task
      RETURN TIER_2_MODEL
    ELSE IF task.requires_extensive_reasoning:
      # Needs thinking → Better model
      RETURN TIER_3_MODEL
    ELSE IF task.context_size > TIER_2_CONTEXT_LIMIT:
      # Too much context for tier 2
      RETURN TIER_3_MODEL
    ELSE:
      # Default moderate
      RETURN TIER_2_MODEL

  ELSE:  # complexity == COMPLEX
    # Complex tasks → Large models

    IF task.is_novel AND NOT task.has_similar_patterns:
      # Novel problem needs best model
      RETURN TIER_3_MODEL
    ELSE IF task.is_safety_critical:
      # Safety critical → Use best
      RETURN TIER_3_MODEL
    ELSE IF budget_constrained(context):
      # Budget tight → Try tier 2 first
      RETURN TIER_2_MODEL_WITH_FALLBACK_TO_TIER_3
    ELSE:
      # Default complex
      RETURN TIER_3_MODEL

Function CLASSIFY_COMPLEXITY(task):
  score = 0

  # Factor 1: Task type
  IF task.type IN ["classification", "formatting", "simple_query"]:
    score += 1
  ELSE IF task.type IN ["coding", "analysis", "planning"]:
    score += 2
  ELSE IF task.type IN ["architecture", "novel_problem", "research"]:
    score += 3

  # Factor 2: Reasoning depth
  IF task.requires_multi_step_reasoning:
    score += 1
  IF task.requires_abstraction:
    score += 1
  IF task.requires_creativity:
    score += 1

  # Factor 3: Context size
  IF task.context_size > 20K:
    score += 1

  # Factor 4: Uncertainty
  IF task.has_high_uncertainty:
    score += 1

  # Classification
  IF score <= 2:
    RETURN SIMPLE
  ELSE IF score <= 5:
    RETURN MODERATE
  ELSE:
    RETURN COMPLEX
```

### 2.4 Model Router Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                       MODEL ROUTER                             │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  MODEL REGISTRY                                          │ │
│  │  • Available models and their specs                      │ │
│  │  • API endpoints and credentials                         │ │
│  │  • Cost per token, latency profiles                      │ │
│  │  • Context window sizes                                  │ │
│  │  • Capability profiles (what each model is good at)     │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ROUTING ENGINE                                          │ │
│  │  • Task complexity classification                        │ │
│  │  • Model selection logic                                 │ │
│  │  • Constraint satisfaction (budget, latency)             │ │
│  │  • Fallback chain definition                            │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  LOAD BALANCER                                           │ │
│  │  • Multiple instances per model tier                     │ │
│  │  • Request distribution                                  │ │
│  │  • Health checking                                       │ │
│  │  • Circuit breaking for failing instances               │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  INVOCATION LAYER                                        │ │
│  │  • API client for each model provider                    │ │
│  │  • Retry logic with exponential backoff                  │ │
│  │  • Rate limiting (respect API limits)                    │ │
│  │  • Response parsing and normalization                    │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  PERFORMANCE TRACKER                                     │ │
│  │  • Log every model invocation                            │ │
│  │  • Track: latency, cost, quality (when available)        │ │
│  │  • Detect performance degradation                        │ │
│  │  • Generate routing optimization recommendations         │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 2.5 Context Window Management

**Challenge**: Tasks may have context that exceeds model's window.

**Strategies**:
```
┌──────────────────────────────────────────────────────────┐
│            CONTEXT WINDOW MANAGEMENT                     │
│                                                          │
│  STRATEGY 1: MODEL UPGRADE                               │
│  • If context > current_model.context_limit              │
│  • Route to model with larger context window             │
│  • Example: GPT-3.5 (4K) → GPT-4-turbo (128K)           │
│  • Pro: Simple, no information loss                      │
│  • Con: More expensive                                   │
│                                                          │
│  STRATEGY 2: CONTEXT COMPRESSION                         │
│  • Summarize non-essential parts                         │
│  • Keep hot context (recent, relevant) intact            │
│  • Use LLM to generate summary of cold context           │
│  • Pro: Fits in smaller context window                   │
│  • Con: Potential information loss                       │
│                                                          │
│  STRATEGY 3: CONTEXT SPLITTING                           │
│  • Break task into sub-tasks with smaller context        │
│  • Process each independently                            │
│  • Merge results                                         │
│  • Pro: Can use faster models                            │
│  • Con: May lose cross-context reasoning                 │
│                                                          │
│  STRATEGY 4: RETRIEVAL-AUGMENTED                         │
│  • Store full context externally                         │
│  • Retrieve relevant pieces on-demand                    │
│  • Include only retrieved context in prompt              │
│  • Pro: Handles unlimited context size                   │
│  • Con: Retrieval quality critical                       │
└──────────────────────────────────────────────────────────┘
```

**Context Management Algorithm**:
```
Function MANAGE_CONTEXT(task, model):

  context_size = ESTIMATE_TOKENS(task.context)

  IF context_size <= model.context_window * 0.8:
    # Fits comfortably (80% threshold for safety margin)
    RETURN task.context

  ELSE IF context_size <= TIER_3_MODEL.context_window * 0.8:
    # Doesn't fit in current model but fits in larger one
    # Upgrade model
    upgraded_model = SELECT_MODEL_WITH_CONTEXT(context_size)
    RETURN (task.context, upgraded_model)

  ELSE:
    # Doesn't fit even in largest model
    # Must compress or split

    IF task.allows_summarization:
      # Strategy 2: Compress
      compressed_context = COMPRESS_CONTEXT(
        task.context,
        target_size=model.context_window * 0.7
      )
      RETURN compressed_context

    ELSE IF task.is_splittable:
      # Strategy 3: Split
      subtasks = SPLIT_TASK(task, max_context_size=model.context_window * 0.7)
      RETURN subtasks  # Will be processed separately

    ELSE:
      # Strategy 4: Retrieval-augmented
      relevant_context = RETRIEVE_RELEVANT_CONTEXT(
        task.query,
        full_context=task.context,
        max_size=model.context_window * 0.7
      )
      RETURN relevant_context

Function COMPRESS_CONTEXT(context, target_size):
  # Identify hot vs cold content
  hot = IDENTIFY_HOT_CONTENT(context)  # Recent, referenced
  cold = context - hot

  current_size = SIZE(hot)

  IF current_size >= target_size:
    # Even hot content too large, must summarize aggressively
    RETURN AGGRESSIVE_SUMMARIZE(hot, target_size)

  # Summarize cold content to fit
  remaining_budget = target_size - current_size
  cold_summary = SUMMARIZE(cold, max_size=remaining_budget)

  RETURN hot + cold_summary
```

### 2.6 Cost Optimization

**Cost Tracking and Optimization**:
```
CostOptimizer {
  // Budget Management
  budget: {
    daily_limit: float,
    per_task_limit: float,
    current_spend_today: float
  },

  // Routing Preferences
  routing_policy: {
    mode: "MINIMIZE_COST" | "BALANCE" | "MAXIMIZE_QUALITY",

    // For BALANCE mode
    cost_weight: float,  // 0-1
    quality_weight: float,  // 0-1
    latency_weight: float  // 0-1
  },

  // Model Cost Profiles
  model_costs: {
    model_id: {
      input_cost_per_1k: float,
      output_cost_per_1k: float,
      minimum_charge: float
    }
  }
}

Function OPTIMIZE_ROUTING(task, budget_remaining):

  IF budget_remaining < MINIMUM_VIABLE_BUDGET:
    # Emergency: Use only cheapest model
    RETURN TIER_1_MODEL

  # Estimate cost for each viable model
  viable_models = []

  FOR tier IN [TIER_1, TIER_2, TIER_3]:
    model = GET_MODEL_FOR_TIER(tier)
    estimated_cost = ESTIMATE_COST(task, model)

    IF estimated_cost <= budget_remaining * 0.1:
      # Use at most 10% of remaining budget per task
      quality_score = ESTIMATE_QUALITY(task, model)
      viable_models.append({
        model: model,
        cost: estimated_cost,
        quality: quality_score
      })

  # Select based on policy
  IF routing_policy.mode == "MINIMIZE_COST":
    RETURN MIN(viable_models, key=lambda x: x.cost).model

  ELSE IF routing_policy.mode == "MAXIMIZE_QUALITY":
    RETURN MAX(viable_models, key=lambda x: x.quality).model

  ELSE:  # BALANCE mode
    # Compute weighted score
    best = None
    best_score = -INF

    FOR candidate IN viable_models:
      # Normalize cost and quality to [0, 1]
      cost_normalized = 1 - (candidate.cost / MAX_COST)
      quality_normalized = candidate.quality

      score = (
        cost_weight * cost_normalized +
        quality_weight * quality_normalized
      )

      IF score > best_score:
        best_score = score
        best = candidate.model

    RETURN best

Function ESTIMATE_COST(task, model):
  input_tokens = ESTIMATE_TOKENS(task.context + task.prompt)
  output_tokens = ESTIMATE_OUTPUT_TOKENS(task)

  input_cost = (input_tokens / 1000) * model.input_cost_per_1k
  output_cost = (output_tokens / 1000) * model.output_cost_per_1k

  RETURN input_cost + output_cost + model.minimum_charge
```

### 2.7 Fallback and Retry Logic

**Handling Model Failures**:
```
Function INVOKE_WITH_FALLBACK(task, primary_model):

  # Define fallback chain
  fallback_chain = BUILD_FALLBACK_CHAIN(primary_model)
  # Example: [GPT-4, Claude-Opus, GPT-4-turbo]

  FOR model IN fallback_chain:
    TRY:
      result = INVOKE_MODEL(model, task, max_retries=3)
      RETURN result

    EXCEPT RateLimitError:
      # Rate limited, try next model
      LOG_WARNING(f"Rate limited on {model}, trying fallback")
      CONTINUE

    EXCEPT TimeoutError:
      # Timeout, try next model
      LOG_WARNING(f"Timeout on {model}, trying fallback")
      CONTINUE

    EXCEPT ModelUnavailable:
      # Model down, try next model
      LOG_WARNING(f"{model} unavailable, trying fallback")
      CONTINUE

    EXCEPT InvalidRequestError:
      # Problem with our request, unlikely to work on other model
      RETURN ERROR(InvalidRequestError)

  # All models in chain failed
  RETURN ERROR("All models failed")

Function INVOKE_MODEL(model, task, max_retries=3):
  FOR attempt IN RANGE(1, max_retries + 1):
    TRY:
      response = model.api_client.complete(
        prompt=task.prompt,
        context=task.context,
        max_tokens=task.max_output_tokens,
        timeout=model.typical_latency * 3
      )
      RETURN response

    EXCEPT (NetworkError, TransientError):
      IF attempt < max_retries:
        # Retry with exponential backoff
        wait_time = 2 ** attempt  # 2s, 4s, 8s
        SLEEP(wait_time)
        CONTINUE
      ELSE:
        RAISE

    EXCEPT NonRetriableError:
      # Don't retry
      RAISE
```

### 2.8 Model Router Operations

```
1. SELECT_MODEL(task, context, constraints)
   • Classify task complexity
   • Evaluate constraints (budget, latency)
   • Apply routing policy
   • Return selected model

2. INVOKE_MODEL(model_id, prompt, context, parameters)
   • Load balance across instances
   • Manage context window
   • Invoke API
   • Handle retries
   • Return response

3. ESTIMATE_COST(task, model_id)
   • Estimate token usage
   • Calculate cost
   • Return estimate

4. GET_MODEL_INFO(model_id)
   • Return model specifications
   • Context window, cost, latency, etc.

5. UPDATE_MODEL_STATS(model_id, invocation_result)
   • Record latency
   • Record cost
   • Update success rate
   • Detect performance issues

6. GET_ROUTING_RECOMMENDATIONS()
   • Analyze routing patterns
   • Identify optimization opportunities
   • Return recommendations

7. SET_ROUTING_POLICY(policy)
   • Update routing preferences
   • Apply new policy to future requests

8. CHECK_BUDGET_STATUS()
   • Return remaining budget
   • Spending rate
   • Projected depletion time
```

## 3. Safety Verifier

### 3.1 Purpose & Responsibilities

**Core Function**: Enforce safety bounds su tutte le operazioni dell'agente, implementando bounded emergence framework.

**Critical Role**: Questo è il layer che previene l'agente da:
- Eseguire azioni non autorizzate
- Generare output dannosi
- Violare constraint di sistema
- Superare limiti di risorse
- Operare fuori dai bounds definiti

**Responsibilities**:
1. **Input Validation**: Verify input safety e sanity
2. **Action Authorization**: Approve/reject action requests
3. **Output Verification**: Validate output meets requirements
4. **Bound Enforcement**: Ensure agent operates within defined bounds
5. **Injection Prevention**: Detect e block injection attacks
6. **Audit Logging**: Record all safety decisions

### 3.2 Safety Bounds Taxonomy

**Five Types of Bounds** (da Bounded Emergence Theory):
```
┌──────────────────────────────────────────────────────────┐
│                    SAFETY BOUNDS                         │
│                                                          │
│  1. INPUT BOUNDS                                         │
│     • What input agent can process                       │
│     • Schema validation                                  │
│     • Size limits                                        │
│     • Content filtering                                  │
│     • Injection detection                                │
│     Example: "Input must be valid JSON < 1MB"            │
│                                                          │
│  2. OUTPUT BOUNDS                                        │
│     • What output agent can generate                     │
│     • Format requirements                                │
│     • Content policy                                     │
│     • Quality thresholds                                 │
│     Example: "Output must be valid Python code"          │
│                                                          │
│  3. ACTION BOUNDS (Most Critical)                        │
│     • What actions agent can perform                     │
│     • Tool whitelist                                     │
│     • Permission system                                  │
│     • Prohibited operations                              │
│     Example: "Can read files but not delete"             │
│                                                          │
│  4. SAFETY BOUNDS                                        │
│     • Hard limits on dangerous operations                │
│     • Human approval gates                               │
│     • Irreversible action protection                     │
│     Example: "Never access production database"          │
│                                                          │
│  5. RESOURCE BOUNDS                                      │
│     • Limits on resource consumption                     │
│     • Time limits                                        │
│     • Token/cost budgets                                 │
│     • Rate limits                                        │
│     Example: "Max 5 minutes, $1 per task"                │
└──────────────────────────────────────────────────────────┘
```

### 3.3 Safety Verifier Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                      SAFETY VERIFIER                           │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  BOUNDS REGISTRY                                         │ │
│  │  • Store all defined safety bounds                       │ │
│  │  • Hierarchical structure (global → task-specific)       │ │
│  │  • Version controlled                                    │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  INPUT VALIDATOR                                         │ │
│  │  • Schema validation                                     │ │
│  │  • Injection detection (SQL, command, prompt)            │ │
│  │  • Content filtering                                     │ │
│  │  • Size/format checking                                  │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ACTION AUTHORIZER                                       │ │
│  │  • Permission checking                                   │ │
│  │  • Tool whitelist enforcement                            │ │
│  │  • Prohibited action detection                           │ │
│  │  • Human approval routing (for dangerous ops)            │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  OUTPUT VALIDATOR                                        │ │
│  │  • Format verification                                   │ │
│  │  • Content policy enforcement                            │ │
│  │  • Quality checking                                      │ │
│  │  • Constraint satisfaction verification                  │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  RESOURCE MONITOR                                        │ │
│  │  • Track time elapsed                                    │ │
│  │  • Track cost accumulated                                │ │
│  │  • Track token usage                                     │ │
│  │  • Enforce limits, abort if exceeded                     │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ↓                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  AUDIT LOGGER                                            │ │
│  │  • Log every safety decision                             │ │
│  │  • Record violations                                     │ │
│  │  • Traceability for forensics                            │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 3.4 Input Validation

**Multi-Layer Input Validation**:
```
Function VALIDATE_INPUT(input, bounds, context):

  # LAYER 1: Schema Validation
  IF bounds.input_schema:
    validation_result = VALIDATE_SCHEMA(input, bounds.input_schema)
    IF NOT validation_result.valid:
      RETURN REJECT("Schema validation failed", validation_result.errors)

  # LAYER 2: Size Limits
  input_size = COMPUTE_SIZE(input)
  IF input_size > bounds.max_input_size:
    RETURN REJECT(f"Input too large: {input_size} > {bounds.max_input_size}")

  # LAYER 3: Injection Detection
  injection_checks = [
    CHECK_SQL_INJECTION(input),
    CHECK_COMMAND_INJECTION(input),
    CHECK_PROMPT_INJECTION(input),
    CHECK_PATH_TRAVERSAL(input)
  ]

  FOR check IN injection_checks:
    IF check.detected:
      RETURN REJECT(f"Security violation: {check.type}", check.details)

  # LAYER 4: Content Filtering
  IF bounds.content_policy:
    content_check = CHECK_CONTENT_POLICY(input, bounds.content_policy)
    IF NOT content_check.passes:
      RETURN REJECT("Content policy violation", content_check.violations)

  # LAYER 5: Context-Specific Validation
  IF bounds.custom_validators:
    FOR validator IN bounds.custom_validators:
      result = validator(input, context)
      IF NOT result.valid:
        RETURN REJECT(f"Custom validation failed: {validator.name}", result.reason)

  # All checks passed
  RETURN ACCEPT()

# Injection Detection Functions

Function CHECK_SQL_INJECTION(input):
  # Detect common SQL injection patterns
  dangerous_patterns = [
    r"'\s*OR\s+'1'\s*=\s*'1",
    r";\s*DROP\s+TABLE",
    r"UNION\s+SELECT",
    r"--\s*$",  # SQL comment
    r"/\*.*\*/"  # SQL block comment
  ]

  FOR pattern IN dangerous_patterns:
    IF REGEX_MATCH(pattern, input, case_insensitive=True):
      RETURN {detected: True, type: "SQL_INJECTION", pattern: pattern}

  RETURN {detected: False}

Function CHECK_COMMAND_INJECTION(input):
  # Detect shell command injection
  dangerous_chars = [";", "|", "&", "$", "`", "\n", "$(", "${"]
  dangerous_commands = ["rm", "dd", "mkfs", "wget", "curl"]

  FOR char IN dangerous_chars:
    IF char IN input:
      RETURN {detected: True, type: "COMMAND_INJECTION", char: char}

  FOR cmd IN dangerous_commands:
    IF cmd IN input.split():
      RETURN {detected: True, type: "DANGEROUS_COMMAND", command: cmd}

  RETURN {detected: False}

Function CHECK_PROMPT_INJECTION(input):
  # Detect attempts to manipulate agent via prompt injection
  injection_indicators = [
    "ignore previous instructions",
    "disregard all",
    "new instructions:",
    "system:",
    "admin mode",
    "override safety"
  ]

  input_lower = input.lower()

  FOR indicator IN injection_indicators:
    IF indicator IN input_lower:
      RETURN {detected: True, type: "PROMPT_INJECTION", indicator: indicator}

  RETURN {detected: False}
```

### 3.5 Action Authorization

**Permission-Based Authorization**:
```
Permission {
  permission_id: string,
  name: string,  // e.g., "file:read", "file:write", "web:fetch"
  description: string,
  category: "READ" | "WRITE" | "DELETE" | "EXECUTE" | "ADMIN",
  risk_level: "LOW" | "MEDIUM" | "HIGH" | "CRITICAL",
  requires_approval: boolean
}

Function AUTHORIZE_ACTION(action, context):

  # STEP 1: Check if action is explicitly prohibited
  IF action IN context.bounds.prohibited_actions:
    RETURN DENY("Action explicitly prohibited", reason="security_policy")

  # STEP 2: Check required permission
  required_permission = action.tool.safety.permission_required

  IF required_permission NOT IN context.available_permissions:
    RETURN DENY(
      "Insufficient permissions",
      required=required_permission,
      available=context.available_permissions
    )

  # STEP 3: Check if action is dangerous and requires approval
  IF action.tool.safety.dangerous:
    IF action.tool.safety.permission_required.requires_approval:
      IF NOT context.has_human_approval:
        # Request human approval
        approval = REQUEST_HUMAN_APPROVAL(action, context)
        IF approval.denied:
          RETURN DENY("Human approval denied", reason=approval.reason)
        # If approved, continue

  # STEP 4: Check action-specific constraints
  IF action.tool.safety.constraints:
    FOR constraint IN action.tool.safety.constraints:
      IF NOT EVALUATE_CONSTRAINT(constraint, action, context):
        RETURN DENY(f"Constraint violation: {constraint.name}")

  # STEP 5: Check if action would exceed resource bounds
  estimated_resource_usage = ESTIMATE_RESOURCE_USAGE(action)
  IF context.resources_used + estimated_resource_usage > context.resource_limits:
    RETURN DENY("Would exceed resource limits")

  # STEP 6: Log authorization decision
  LOG_AUTHORIZATION(action, context, decision="APPROVED")

  RETURN APPROVE()

Function REQUEST_HUMAN_APPROVAL(action, context):
  # Present action to human for approval
  approval_request = {
    action_description: action.description,
    tool: action.tool.name,
    parameters: action.parameters,
    reason: context.reasoning_for_action,
    risks: action.tool.safety.side_effects,
    reversible: action.tool.safety.reversible
  }

  # Block until human responds
  response = PRESENT_TO_HUMAN(approval_request)

  RETURN response  # {approved: boolean, reason: string}
```

### 3.6 Output Validation

**Multi-Criteria Output Checking**:
```
Function VALIDATE_OUTPUT(output, expected_schema, success_criteria, bounds):

  # LAYER 1: Schema Validation
  IF expected_schema:
    schema_result = VALIDATE_SCHEMA(output, expected_schema)
    IF NOT schema_result.valid:
      RETURN REJECT("Output schema invalid", schema_result.errors)

  # LAYER 2: Success Criteria Verification
  FOR criterion IN success_criteria:
    result = VERIFY_CRITERION(output, criterion)
    IF criterion.required AND NOT result.passed:
      RETURN REJECT(f"Required criterion not met: {criterion.description}")

  # LAYER 3: Content Policy
  IF bounds.output_content_policy:
    policy_check = CHECK_CONTENT_POLICY(output, bounds.output_content_policy)
    IF NOT policy_check.passes:
      RETURN REJECT("Output content policy violation", policy_check.violations)

  # LAYER 4: Safety Checks
  safety_checks = [
    CHECK_NO_SECRETS_LEAKED(output),
    CHECK_NO_MALICIOUS_CODE(output),
    CHECK_NO_PII_EXPOSED(output)
  ]

  FOR check IN safety_checks:
    IF check.failed:
      RETURN REJECT(f"Safety check failed: {check.name}", check.details)

  # LAYER 5: Quality Thresholds (if specified)
  IF bounds.min_quality_score:
    quality = ASSESS_OUTPUT_QUALITY(output)
    IF quality < bounds.min_quality_score:
      RETURN REJECT(f"Output quality too low: {quality} < {bounds.min_quality_score}")

  RETURN ACCEPT()

Function CHECK_NO_SECRETS_LEAKED(output):
  # Detect common secret patterns
  secret_patterns = [
    r"(api[_-]?key|apikey)[\s:=]+['\"]\w+['\"]",  # API keys
    r"(password|passwd|pwd)[\s:=]+['\"]\w+['\"]",  # Passwords
    r"-----BEGIN (RSA |)PRIVATE KEY-----",  # Private keys
    r"sk-[A-Za-z0-9]{32,}",  # OpenAI-style secret keys
    r"ghp_[A-Za-z0-9]{36}",  # GitHub tokens
  ]

  FOR pattern IN secret_patterns:
    matches = REGEX_FINDALL(pattern, output, case_insensitive=True)
    IF matches:
      RETURN {failed: True, name: "SecretDetection", details: "Potential secret found"}

  RETURN {failed: False}

Function CHECK_NO_MALICIOUS_CODE(output):
  # If output is code, scan for dangerous patterns
  IF NOT IS_CODE(output):
    RETURN {failed: False}

  dangerous_code_patterns = [
    r"eval\s*\(",  # Eval is dangerous
    r"exec\s*\(",  # Exec is dangerous
    r"__import__\s*\(",  # Dynamic imports
    r"subprocess\.",  # Shell execution
    r"os\.system",  # Shell execution
    r"rm\s+-rf\s+/",  # Destructive commands
  ]

  FOR pattern IN dangerous_code_patterns:
    IF REGEX_MATCH(pattern, output):
      RETURN {
        failed: True,
        name: "MaliciousCode",
        details: f"Dangerous pattern detected: {pattern}"
      }

  RETURN {failed: False}
```

### 3.7 Resource Monitoring

**Real-Time Resource Tracking**:
```
ResourceMonitor {
  limits: {
    max_time: float,  // seconds
    max_cost: float,  // dollars
    max_tokens: int,
    max_llm_calls: int,
    max_tool_calls: int
  },

  current_usage: {
    time_elapsed: float,
    cost_accumulated: float,
    tokens_used: int,
    llm_calls_made: int,
    tool_calls_made: int
  },

  start_time: datetime
}

Function MONITOR_RESOURCES(monitor, operation, estimated_cost):

  # Check if operation would exceed any limit
  violations = []

  # Time check
  IF monitor.current_usage.time_elapsed >= monitor.limits.max_time:
    violations.append("Time limit exceeded")

  # Cost check
  IF monitor.current_usage.cost_accumulated + estimated_cost.cost > monitor.limits.max_cost:
    violations.append("Cost limit would be exceeded")

  # Token check
  IF monitor.current_usage.tokens_used + estimated_cost.tokens > monitor.limits.max_tokens:
    violations.append("Token limit would be exceeded")

  # Call count checks
  IF operation.type == "LLM_CALL":
    IF monitor.current_usage.llm_calls_made >= monitor.limits.max_llm_calls:
      violations.append("LLM call limit exceeded")

  IF operation.type == "TOOL_CALL":
    IF monitor.current_usage.tool_calls_made >= monitor.limits.max_tool_calls:
      violations.append("Tool call limit exceeded")

  # If any violations, reject operation
  IF violations:
    RETURN REJECT("Resource limit violation", violations)

  # Otherwise, approve and update usage
  monitor.current_usage.time_elapsed = (now - monitor.start_time).seconds
  monitor.current_usage.cost_accumulated += estimated_cost.cost
  monitor.current_usage.tokens_used += estimated_cost.tokens

  IF operation.type == "LLM_CALL":
    monitor.current_usage.llm_calls_made += 1
  IF operation.type == "TOOL_CALL":
    monitor.current_usage.tool_calls_made += 1

  RETURN APPROVE()

Function GET_RESOURCE_STATUS(monitor):
  RETURN {
    time: {
      used: monitor.current_usage.time_elapsed,
      limit: monitor.limits.max_time,
      remaining: monitor.limits.max_time - monitor.current_usage.time_elapsed,
      percentage: (monitor.current_usage.time_elapsed / monitor.limits.max_time) * 100
    },
    cost: {
      used: monitor.current_usage.cost_accumulated,
      limit: monitor.limits.max_cost,
      remaining: monitor.limits.max_cost - monitor.current_usage.cost_accumulated,
      percentage: (monitor.current_usage.cost_accumulated / monitor.limits.max_cost) * 100
    },
    // Similar for tokens, llm_calls, tool_calls
  }
```

### 3.8 Audit Logging

**Comprehensive Audit Trail**:
```
AuditLog Entry {
  timestamp: datetime,
  event_type: "INPUT_VALIDATION" | "ACTION_AUTHORIZATION" | "OUTPUT_VALIDATION" | "RESOURCE_CHECK",
  decision: "APPROVED" | "REJECTED" | "REQUIRES_APPROVAL",

  // Context
  task_id: string,
  execution_phase: string,

  // What was checked
  subject: {...},  // The thing being validated
  bounds_applied: [BoundReference],

  // Decision details
  reason: string,
  violations: [Violation] | null,
  warnings: [Warning] | null,

  // If human approval involved
  approval_request_id: string | null,
  human_decision: "APPROVED" | "DENIED" | null,
  human_reason: string | null,

  // Metadata
  verifier_version: string,
  rule_version: string
}

Function LOG_SAFETY_DECISION(event_type, decision, details):
  entry = AuditLogEntry {
    timestamp: NOW(),
    event_type: event_type,
    decision: decision,
    ...details
  }

  # Write to audit log (append-only, immutable)
  AUDIT_LOG.append(entry)

  # If rejection or violation, also alert
  IF decision == "REJECTED":
    ALERT_SECURITY_TEAM(entry)

  # If human approval required, track
  IF decision == "REQUIRES_APPROVAL":
    TRACK_APPROVAL_REQUEST(entry)
```

### 3.9 Safety Verifier Operations

```
1. VALIDATE_INPUT(input, bounds, context)
   • Multi-layer validation
   • Schema, size, injection, content
   • Return validation result

2. AUTHORIZE_ACTION(action, context)
   • Permission check
   • Danger assessment
   • Human approval if needed
   • Return authorization decision

3. VALIDATE_OUTPUT(output, schema, criteria, bounds)
   • Schema validation
   • Success criteria check
   • Safety checks (secrets, malicious code)
   • Return validation result

4. MONITOR_RESOURCES(monitor, operation, estimated_cost)
   • Check resource limits
   • Update usage tracking
   • Abort if limit exceeded
   • Return approval/rejection

5. REQUEST_HUMAN_APPROVAL(action, context)
   • Present to human
   • Wait for decision
   • Log decision
   • Return approval result

6. GET_RESOURCE_STATUS(monitor)
   • Return current resource usage
   • Remaining budgets
   • Percentage utilized

7. LOG_SAFETY_DECISION(event_type, decision, details)
   • Create audit log entry
   • Alert if violation
   • Track for forensics

8. UPDATE_BOUNDS(bound_updates)
   • Update safety bounds
   • Version increment
   • Validate new bounds
   • Propagate changes

9. GET_VIOLATIONS(time_range, filters)
   • Query audit log
   • Return violations matching criteria
   • For security analysis

10. GENERATE_SAFETY_REPORT(time_range)
    • Analyze audit logs
    • Compute statistics
    • Identify patterns
    • Return safety health report
```

## 4. Capability Layer Integration

### 4.1 Typical Interaction Flow

```
┌────────────────────────────────────────────────────────────────┐
│         CAPABILITY LAYER: COMPLETE INTERACTION FLOW            │
│                                                                │
│  Execution Engine: "Need to read file 'data.json'"             │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 1: Safety Verification (Pre-check)                 │ │
│  │  • Validate request is safe                              │ │
│  │  • Check file path for traversal attacks                 │ │
│  │  • Verify file read permission available                 │ │
│  │  Decision: APPROVED                                      │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 2: Tool Discovery                                  │ │
│  │  • Query Tool Registry for "read file" capability        │ │
│  │  • Get matching tools: [read_file, read_text_file, ...]  │ │
│  │  • Select best match: read_file                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 3: Tool Invocation                                 │ │
│  │  • Validate parameters against tool schema               │ │
│  │  • Execute: read_file("data.json")                       │ │
│  │  • Capture output                                        │ │
│  │  Result: File contents returned                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 4: Output Validation                               │ │
│  │  • Check output is valid JSON                            │ │
│  │  • Scan for sensitive data                               │ │
│  │  • Verify meets success criteria                         │ │
│  │  Decision: APPROVED                                      │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return to Execution Engine: File contents (validated)         │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  Next: "Parse JSON and extract field 'users'"            │ │
│  │  • LLM reasoning needed → Route to Model Router          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 5: Model Selection                                 │ │
│  │  • Task: Parse JSON (simple)                             │ │
│  │  • Complexity: LOW                                       │ │
│  │  • Route to: TIER_1_MODEL (fast/cheap)                   │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 6: LLM Invocation                                  │ │
│  │  • Context: File contents                                │ │
│  │  • Prompt: "Extract 'users' field"                       │ │
│  │  • Invoke model with retry/fallback                      │ │
│  │  Result: Extracted users list                            │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  STEP 7: Resource Monitoring                             │ │
│  │  • Update time elapsed                                   │ │
│  │  • Update cost ($0.001 for LLM call)                     │ │
│  │  • Update token count                                    │ │
│  │  • Check all within limits ✓                             │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  Return to Execution Engine: Users list (validated, within budget) │
└────────────────────────────────────────────────────────────────┘
```

### 4.2 Performance Characteristics

**Capability Layer Metrics**:
```
TOOL REGISTRY
  • Tool lookup: <10ms (cached)
  • Tool invocation overhead: <50ms
  • Permission check: <5ms
  • Total tools supported: 50-200 (typical)

MODEL ROUTER
  • Model selection: <20ms
  • Model invocation: 1-15s (depends on model tier)
  • Cost optimization: 5-10x reduction vs always-best-model
  • Fallback handling: <2s additional latency

SAFETY VERIFIER
  • Input validation: <50ms
  • Action authorization: <30ms
  • Output validation: <100ms
  • Audit logging: <10ms (async)
  • Human approval wait: 30s - 5min (human dependent)
```

---

**Next**: [05-infrastructure.md](05-infrastructure.md) → Observability, Resource Management, Error Handling specifications
