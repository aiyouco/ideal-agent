# Infrastructure Layer: Observability, Resources, and Error Handling

## Overview

L'Infrastructure Layer fornisce i servizi fondazionali che supportano tutti gli altri layer: observability per capire cosa sta accadendo, resource management per controllare consumi, ed error handling per gestire failure in modo robusto.

```
┌───────────────────────────────────────────────────────────────┐
│                   INFRASTRUCTURE LAYER                        │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  OBSERVABILITY SYSTEM                                   │ │
│  │  • Structured logging                                   │ │
│  │  • Metrics collection                                   │ │
│  │  • Distributed tracing                                  │ │
│  │  • Real-time monitoring                                 │ │
│  │  Purpose: Understand system behavior                    │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  RESOURCE MANAGER                                       │ │
│  │  • Budget tracking                                      │ │
│  │  • Quota enforcement                                    │ │
│  │  • Resource allocation                                  │ │
│  │  • Optimization recommendations                         │ │
│  │  Purpose: Control resource consumption                  │ │
│  └─────────────────────────────────────────────────────────┘ │
│                          ↕                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  ERROR HANDLER                                          │ │
│  │  • Error classification                                 │ │
│  │  • Automatic recovery                                   │ │
│  │  • Escalation logic                                     │ │
│  │  • Incident tracking                                    │ │
│  │  Purpose: Handle failures gracefully                    │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

## 1. Observability System

### 1.1 Purpose & Responsibilities

**Core Function**: Rendere il sistema trasparente - capire cosa sta accadendo, perché, e come sta performando.

**Three Pillars of Observability**:
1. **Logs**: Eventi discreti con contesto
2. **Metrics**: Misurazioni quantitative aggregate
3. **Traces**: Flusso di esecuzione end-to-end

**Responsibilities**:
1. **Structured Logging**: Log eventi con metadata strutturato
2. **Metrics Collection**: Raccogliere metriche performance/business
3. **Distributed Tracing**: Tracciare richieste attraverso componenti
4. **Alerting**: Notificare anomalie e problemi
5. **Dashboarding**: Visualizzare stato sistema
6. **Debugging Support**: Fornire info per troubleshooting

### 1.2 Logging System

**Structured Logging Architecture**:
```
┌────────────────────────────────────────────────────────────────┐
│                      LOGGING SYSTEM                            │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  LOG PRODUCERS (Ogni componente)                         │ │
│  │  • Cognitive Layer                                       │ │
│  │  • Memory System                                         │ │
│  │  • Capability Layer                                      │ │
│  │  • Infrastructure                                        │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  LOG AGGREGATOR                                          │ │
│  │  • Collect from all sources                              │ │
│  │  • Add correlation IDs                                   │ │
│  │  • Enrich with context                                   │ │
│  │  • Buffer and batch                                      │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  LOG STORAGE                                             │ │
│  │  • Time-series optimized                                 │ │
│  │  • Indexed by: timestamp, level, component, task_id     │ │
│  │  • Retention: 30 days hot, 1 year cold                  │ │
│  │  Technology: Elasticsearch, Loki, or CloudWatch         │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  LOG QUERY & ANALYSIS                                    │ │
│  │  • Full-text search                                      │ │
│  │  • Filtering and aggregation                             │ │
│  │  • Pattern detection                                     │ │
│  │  • Export for analysis                                   │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

**Log Entry Schema**:
```
LogEntry {
  // Identification
  timestamp: datetime (ISO 8601, UTC),
  log_id: string (UUID),

  // Categorization
  level: "DEBUG" | "INFO" | "WARNING" | "ERROR" | "CRITICAL",
  component: string,  // e.g., "PlanningEngine", "ModelRouter"
  event_type: string,  // e.g., "task_started", "tool_invoked"

  // Correlation
  task_id: string,
  session_id: string,
  user_id: string,
  trace_id: string,  // For distributed tracing
  span_id: string,

  // Content
  message: string,
  structured_data: {
    // Event-specific data
    // Examples:
    // - tool_name, parameters for tool invocations
    // - model_id, tokens_used for LLM calls
    // - error_code, stack_trace for errors
  },

  // Context
  context: {
    execution_phase: string,
    parent_task: string,
    resource_usage: {...}
  },

  // Metadata
  host: string,
  process_id: string,
  thread_id: string,
  version: string
}
```

**Log Levels and Usage**:
```
┌──────────────────────────────────────────────────────────┐
│                    LOG LEVELS                            │
│                                                          │
│  DEBUG (Development only)                                │
│  • Detailed diagnostic info                             │
│  • Variable values, internal state                      │
│  • Example: "Working memory size: 15234 tokens"         │
│  → Not in production (too verbose)                      │
│                                                          │
│  INFO (Normal operations)                                │
│  • Significant events                                   │
│  • State transitions                                    │
│  • Example: "Task T-123 completed successfully"         │
│  → Production default level                             │
│                                                          │
│  WARNING (Potential issues)                              │
│  • Degraded performance                                 │
│  • Recoverable errors                                   │
│  • Example: "Model Router fallback to tier 2"           │
│  → Investigate if frequent                              │
│                                                          │
│  ERROR (Failures)                                        │
│  • Operation failures                                   │
│  • Unexpected errors                                    │
│  • Example: "Tool invocation failed: timeout"           │
│  → Requires investigation                               │
│                                                          │
│  CRITICAL (System-level failures)                        │
│  • Service unavailable                                  │
│  • Data corruption                                      │
│  • Example: "Memory system unreachable"                 │
│  → Immediate action required                            │
└──────────────────────────────────────────────────────────┘
```

### 1.3 Metrics System

**Metrics Collection Architecture**:
```
┌────────────────────────────────────────────────────────────────┐
│                     METRICS SYSTEM                             │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  METRICS INSTRUMENTATION                                 │ │
│  │  • Counters (increment-only)                             │ │
│  │  • Gauges (current value)                                │ │
│  │  • Histograms (distributions)                            │ │
│  │  • Timers (durations)                                    │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  METRICS AGGREGATOR                                      │ │
│  │  • Collect from all components                           │ │
│  │  • Aggregate at 1min, 5min, 1hr intervals               │ │
│  │  • Compute percentiles (p50, p95, p99)                   │ │
│  │  • Downsampling for long-term storage                    │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  TIME-SERIES DATABASE                                    │ │
│  │  • Store metrics with timestamps                         │ │
│  │  • Indexed by metric name + labels                       │ │
│  │  • Retention: 1 week raw, 1 month aggregated, 1 year     │ │
│  │  Technology: Prometheus, InfluxDB, or CloudWatch         │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  DASHBOARDS & ALERTS                                     │ │
│  │  • Real-time visualization                               │ │
│  │  • Anomaly detection                                     │ │
│  │  • Threshold-based alerts                                │ │
│  │  Technology: Grafana, Datadog, or custom                 │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

**Key Metrics Categories**:
```
┌──────────────────────────────────────────────────────────┐
│                   METRIC CATEGORIES                      │
│                                                          │
│  1. THROUGHPUT METRICS                                   │
│     • tasks_started (counter)                           │
│     • tasks_completed (counter)                         │
│     • tasks_failed (counter)                            │
│     • tasks_per_minute (gauge)                          │
│     Labels: [user_id, task_type, complexity]            │
│                                                          │
│  2. LATENCY METRICS                                      │
│     • task_duration_seconds (histogram)                 │
│     • goal_analysis_duration (histogram)                │
│     • planning_duration (histogram)                     │
│     • execution_duration (histogram)                    │
│     • reflection_duration (histogram)                   │
│     Percentiles: p50, p90, p95, p99                     │
│                                                          │
│  3. COST METRICS                                         │
│     • llm_cost_dollars (counter)                        │
│     • tokens_consumed (counter)                         │
│     • tool_invocation_cost (counter)                    │
│     • cost_per_task (gauge)                             │
│     Labels: [model_id, tool_name]                       │
│                                                          │
│  4. QUALITY METRICS                                      │
│     • success_rate (gauge)                              │
│     • verification_pass_rate (gauge)                    │
│     • human_satisfaction_score (gauge)                  │
│     • retry_rate (gauge)                                │
│                                                          │
│  5. RESOURCE METRICS                                     │
│     • memory_usage_mb (gauge)                           │
│     • cpu_utilization_percent (gauge)                   │
│     • active_tasks (gauge)                              │
│     • queue_depth (gauge)                               │
│                                                          │
│  6. COMPONENT METRICS                                    │
│     • model_router_calls (counter)                      │
│     • tool_registry_lookups (counter)                   │
│     • episodic_memory_queries (counter)                 │
│     • pattern_cache_hits (counter)                      │
│     • safety_violations (counter)                       │
└──────────────────────────────────────────────────────────┘
```

**Metric Naming Convention**:
```
<component>_<metric>_<unit>

Examples:
- planning_engine_duration_seconds
- model_router_cost_dollars
- safety_verifier_rejections_total
- memory_system_cache_hit_ratio

Labels for dimensionality:
{
  component="PlanningEngine",
  strategy="HTN",
  complexity="moderate",
  user_id="user_123"
}
```

### 1.4 Distributed Tracing

**Tracing Architecture**:
```
┌────────────────────────────────────────────────────────────────┐
│                   DISTRIBUTED TRACING                          │
│                                                                │
│  Request: "User asks: Refactor authentication"                 │
│    ↓                                                           │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  TRACE ROOT (Span 1)                                     │ │
│  │  Operation: handle_task                                  │ │
│  │  trace_id: abc123                                        │ │
│  │  span_id: span-1                                         │ │
│  │  Start: 2024-01-15 10:00:00                              │ │
│  │  Duration: 245s                                          │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SPAN 2: Goal Analysis                                   │ │
│  │  parent_span: span-1                                     │ │
│  │  Duration: 18s                                           │ │
│  │  ├─ SPAN 3: Semantic parsing (5s)                        │ │
│  │  ├─ SPAN 4: Goal extraction (8s)                         │ │
│  │  └─ SPAN 5: Complexity classification (3s)               │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SPAN 6: Planning                                        │ │
│  │  Duration: 42s                                           │ │
│  │  ├─ SPAN 7: Query pattern cache (2s)                     │ │
│  │  ├─ SPAN 8: Task decomposition (30s)                     │ │
│  │  └─ SPAN 9: Dependency analysis (8s)                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SPAN 10: Execution                                      │ │
│  │  Duration: 165s                                          │ │
│  │  ├─ SPAN 11: Subtask 1 execution (35s)                   │ │
│  │  │  ├─ SPAN 12: LLM call (20s)                           │ │
│  │  │  └─ SPAN 13: Tool invocation (10s)                    │ │
│  │  ├─ SPAN 14: Subtask 2 execution (45s)                   │ │
│  │  └─ ... (more subtasks)                                  │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SPAN 20: Reflection                                     │ │
│  │  Duration: 25s (async)                                   │ │
│  │  ├─ SPAN 21: Episode analysis (8s)                       │ │
│  │  ├─ SPAN 22: Pattern extraction (12s)                    │ │
│  │  └─ SPAN 23: Memory update (3s)                          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  Visualization: Waterfall chart showing parent-child           │
│  relationships and timing                                      │
└────────────────────────────────────────────────────────────────┘
```

**Span Schema**:
```
Span {
  // Identification
  trace_id: string,  // Same for entire request
  span_id: string,   // Unique per span
  parent_span_id: string | null,

  // Operation
  operation_name: string,  // e.g., "PlanningEngine.generate_plan"
  component: string,

  // Timing
  start_time: datetime,
  end_time: datetime,
  duration_ms: int,

  // Context
  tags: {
    // Key-value pairs for filtering
    task_type: string,
    model_id: string,
    user_id: string,
    ...
  },

  // Events within span
  events: [
    {
      timestamp: datetime,
      name: string,
      attributes: {...}
    }
  ],

  // Outcome
  status: "OK" | "ERROR",
  error: Error | null
}
```

**Trace Sampling Strategy**:
```
High Traffic → Cannot trace everything → Sampling

SAMPLING STRATEGIES:

1. PROBABILITY-BASED
   • Sample X% of all traces randomly
   • Example: 10% sampling
   • Pro: Statistically representative
   • Con: May miss rare issues

2. RATE-LIMITED
   • Sample max N traces per second
   • Example: 100 traces/sec
   • Pro: Control storage cost
   • Con: May lose detail during spikes

3. TAIL-BASED (Smart Sampling)
   • Keep all errors
   • Keep slow traces (>p95 latency)
   • Sample others at low rate
   • Example: 100% errors, 100% >p95, 1% others
   • Pro: Catch interesting traces
   • Con: More complex logic

RECOMMENDED: Tail-based sampling
```

### 1.5 Monitoring Dashboards

**System Health Dashboard**:
```
┌────────────────────────────────────────────────────────────────┐
│                  SYSTEM HEALTH DASHBOARD                       │
│                                                                │
│  ┌────────────────────────┐  ┌────────────────────────────┐   │
│  │ Tasks/Min: 12.5 ↑      │  │ Success Rate: 89.2% ↓      │   │
│  │ [====Graph====]        │  │ [====Graph====]            │   │
│  └────────────────────────┘  └────────────────────────────┘   │
│                                                                │
│  ┌────────────────────────┐  ┌────────────────────────────┐   │
│  │ P95 Latency: 28.3s ↑   │  │ Cost/Task: $0.18 →         │   │
│  │ [====Graph====]        │  │ [====Graph====]            │   │
│  └────────────────────────┘  └────────────────────────────┘   │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Component Health                                         │ │
│  │ ✅ Cognitive Layer      OK    (avg latency: 85s)        │ │
│  │ ✅ Memory System        OK    (cache hit: 78%)          │ │
│  │ ⚠️  Model Router        DEGRADED (fallback rate: 12%)   │ │
│  │ ✅ Safety Verifier      OK    (violations: 0)           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Active Alerts                                            │ │
│  │ 🔴 CRITICAL: Model Router fallback rate > 10% (12%)     │ │
│  │ 🟡 WARNING: Task success rate < 90% (89.2%)             │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Recent Errors (Last 5)                                   │ │
│  │ • 10:23:15 - Tool timeout: web_search                   │ │
│  │ • 10:18:42 - Model unavailable: gpt-4 (using fallback)  │ │
│  │ • 10:12:33 - Safety violation: path traversal attempt   │ │
│  │ • 10:05:19 - Planning failed: recursion depth exceeded │ │
│  │ • 09:58:07 - Memory query timeout                       │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

**Alert Rules**:
```
┌──────────────────────────────────────────────────────────┐
│                    ALERT RULES                           │
│                                                          │
│  CRITICAL (Page immediately)                             │
│  • Success rate < 70% for 5 minutes                     │
│  • P95 latency > 5x baseline for 10 minutes             │
│  • Error rate > 50% for 5 minutes                       │
│  • Any component completely unavailable                  │
│  • Safety violations > 10 per minute                     │
│                                                          │
│  WARNING (Investigate within 1 hour)                     │
│  • Success rate < 90% for 15 minutes                    │
│  • P95 latency > 2x baseline for 15 minutes             │
│  • Cost/task > budget by 50%                            │
│  • Memory cache hit rate < 50%                          │
│  • Model router fallback rate > 10%                     │
│                                                          │
│  INFO (Monitor)                                          │
│  • Success rate < 95% for 30 minutes                    │
│  • Any metric trending outside normal range             │
│  • New error types appearing                            │
└──────────────────────────────────────────────────────────┘
```

## 2. Resource Manager

### 2.1 Purpose & Responsibilities

**Core Function**: Controllare e ottimizzare consumption di risorse (tempo, costo, memoria, compute).

**Key Insight**: Senza resource management, agent può:
- Spendere budget intero su singolo task
- Runare indefinitamente (denial of service)
- Exhaustare memoria
- Causare rate limiting su external APIs

**Responsibilities**:
1. **Budget Tracking**: Monitor spending contro limiti
2. **Quota Enforcement**: Enforce per-user, per-task quotas
3. **Resource Allocation**: Distribute resources ottimalmente
4. **Throttling**: Limit rate quando necessario
5. **Optimization**: Suggest improvements per efficiency

### 2.2 Resource Manager Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    RESOURCE MANAGER                            │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  BUDGET CONTROLLER                                       │ │
│  │  • Track spending per user/org                           │ │
│  │  • Enforce limits (daily, monthly, per-task)             │ │
│  │  • Alert approaching limits                              │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  QUOTA MANAGER                                           │ │
│  │  • Define quotas (requests/min, concurrent tasks)        │ │
│  │  • Check quota before operation                          │ │
│  │  • Queue requests if quota exceeded                      │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  RESOURCE ALLOCATOR                                      │ │
│  │  • Prioritize tasks                                      │ │
│  │  • Allocate compute resources                            │ │
│  │  • Load balancing                                        │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  OPTIMIZATION ENGINE                                     │ │
│  │  • Analyze resource usage patterns                       │ │
│  │  • Identify waste                                        │ │
│  │  • Recommend improvements                                │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 2.3 Budget System

**Budget Hierarchy**:
```
┌────────────────────────────────────────────────────────────┐
│                   BUDGET HIERARCHY                         │
│                                                            │
│  Organization Budget (Top Level)                           │
│  └─ $10,000 / month                                        │
│     │                                                       │
│     ├─ User Budget (Per User)                              │
│     │  └─ $500 / month                                     │
│     │     │                                                │
│     │     ├─ Task Budget (Per Task)                        │
│     │     │  └─ $1.00 / task (soft limit)                  │
│     │     │     └─ $5.00 / task (hard limit)               │
│     │     │                                                │
│     │     └─ Daily Budget                                  │
│     │        └─ $20 / day                                  │
│     │                                                       │
│     └─ Service Budget (By Service Type)                    │
│        ├─ LLM: $7,000 / month                              │
│        ├─ Tools: $2,000 / month                            │
│        └─ Infrastructure: $1,000 / month                   │
└────────────────────────────────────────────────────────────┘
```

**Budget Enforcement Logic**:
```
Function CHECK_BUDGET(operation, estimated_cost, context):

  # LEVEL 1: Check organization budget
  org_remaining = org_budget.monthly_limit - org_budget.spent_this_month
  IF estimated_cost > org_remaining:
    IF org_budget.allow_overage:
      LOG_WARNING("Organization budget exceeded, allowing overage")
    ELSE:
      RETURN REJECT("Organization budget exhausted")

  # LEVEL 2: Check user budget
  user_remaining = user_budget.monthly_limit - user_budget.spent_this_month
  IF estimated_cost > user_remaining:
    RETURN REJECT("User monthly budget exhausted")

  # LEVEL 3: Check daily budget
  daily_remaining = user_budget.daily_limit - user_budget.spent_today
  IF estimated_cost > daily_remaining:
    RETURN REJECT("Daily budget exhausted, try tomorrow")

  # LEVEL 4: Check per-task soft limit
  IF estimated_cost > task_budget.soft_limit:
    IF estimated_cost < task_budget.hard_limit:
      # Request approval to exceed soft limit
      approval = REQUEST_APPROVAL("Estimated cost ${estimated_cost} exceeds soft limit ${task_budget.soft_limit}")
      IF NOT approval:
        RETURN REJECT("Soft limit exceeded, approval denied")
    ELSE:
      RETURN REJECT("Hard limit would be exceeded")

  # All checks passed
  RETURN APPROVE()

Function RECORD_ACTUAL_COST(task_id, actual_cost):
  # Update all budget levels
  org_budget.spent_this_month += actual_cost
  user_budget.spent_this_month += actual_cost
  user_budget.spent_today += actual_cost

  # If overspent vs estimate, analyze
  estimate = task_budget.estimates[task_id]
  IF actual_cost > estimate * 1.5:
    ANALYZE_OVERSPEND(task_id, estimate, actual_cost)
```

### 2.4 Quota System

**Quota Types**:
```
┌──────────────────────────────────────────────────────────┐
│                    QUOTA TYPES                           │
│                                                          │
│  RATE QUOTAS (Requests per time period)                  │
│  • tasks_per_minute: 10                                 │
│  • llm_calls_per_minute: 100                            │
│  • tool_invocations_per_minute: 50                      │
│  Purpose: Prevent API rate limiting, DoS                │
│                                                          │
│  CONCURRENCY QUOTAS (Parallel operations)                │
│  • max_concurrent_tasks: 5                              │
│  • max_concurrent_llm_calls: 10                         │
│  Purpose: Prevent resource exhaustion                   │
│                                                          │
│  VOLUME QUOTAS (Total amount)                            │
│  • max_tasks_per_day: 1000                              │
│  • max_tokens_per_month: 10M                            │
│  Purpose: Prevent abuse, control costs                  │
│                                                          │
│  SIZE QUOTAS (Per-item limits)                           │
│  • max_task_duration: 600s (10 min)                     │
│  • max_context_size: 200K tokens                        │
│  • max_output_size: 100KB                               │
│  Purpose: Prevent runaway operations                    │
└──────────────────────────────────────────────────────────┘
```

**Quota Enforcement with Queuing**:
```
Function ENFORCE_QUOTA(operation_type, user_id):

  quota = GET_QUOTA(operation_type, user_id)
  current_usage = GET_CURRENT_USAGE(operation_type, user_id)

  IF current_usage < quota.limit:
    # Under quota, allow immediately
    INCREMENT_USAGE(operation_type, user_id)
    RETURN ALLOW()

  ELSE:
    # Quota exceeded
    IF quota.allow_queueing:
      # Add to queue, will be processed when quota available
      queue_position = ENQUEUE(operation, user_id)
      RETURN QUEUED(position=queue_position, estimated_wait=...)
    ELSE:
      # Reject immediately
      RETURN REJECT("Quota exceeded", retry_after=...)
```

### 2.5 Resource Optimization

**Optimization Analyzer**:
```
Function ANALYZE_RESOURCE_USAGE(time_period):

  tasks = GET_TASKS_IN_PERIOD(time_period)

  # ANALYSIS 1: Cost efficiency
  cost_analysis = {
    total_cost: SUM(task.cost for task in tasks),
    avg_cost_per_task: MEAN(task.cost for task in tasks),
    cost_by_component: GROUP_BY(tasks, 'component', SUM('cost')),

    # Identify expensive outliers
    expensive_tasks: tasks WHERE cost > PERCENTILE(tasks.cost, 95),

    # Model routing efficiency
    model_routing_savings: ESTIMATE_SAVINGS_FROM_ROUTING(tasks)
  }

  # ANALYSIS 2: Time efficiency
  time_analysis = {
    total_time: SUM(task.duration for task in tasks),
    avg_time_per_task: MEAN(task.duration for task in tasks),

    # Bottlenecks
    bottlenecks: IDENTIFY_BOTTLENECKS(tasks),

    # Parallelization opportunities missed
    parallelization_potential: FIND_PARALLELIZATION_OPPORTUNITIES(tasks)
  }

  # ANALYSIS 3: Resource utilization
  utilization_analysis = {
    cpu_avg: MEAN(sample.cpu for sample in metrics),
    memory_avg: MEAN(sample.memory for sample in metrics),

    # Under/over provisioned
    cpu_utilization_rate: cpu_avg / cpu_allocated,
    memory_utilization_rate: memory_avg / memory_allocated
  }

  # RECOMMENDATIONS
  recommendations = GENERATE_RECOMMENDATIONS(
    cost_analysis,
    time_analysis,
    utilization_analysis
  )

  RETURN OptimizationReport {
    analysis: {...},
    recommendations: recommendations,
    potential_savings: ESTIMATE_POTENTIAL_SAVINGS(recommendations)
  }

Function GENERATE_RECOMMENDATIONS(cost_analysis, time_analysis, utilization_analysis):

  recommendations = []

  # Cost optimization
  IF model_routing_savings.potential > 0.2:  # 20%+ savings possible
    recommendations.append({
      type: "COST_OPTIMIZATION",
      title: "Improve model routing",
      description: f"Current routing could save {model_routing_savings.potential:.0%} by using smaller models for simple tasks",
      potential_savings: "$X/month"
    })

  # Time optimization
  FOR bottleneck IN time_analysis.bottlenecks:
    recommendations.append({
      type: "TIME_OPTIMIZATION",
      title: f"Optimize {bottleneck.component}",
      description: f"{bottleneck.component} takes {bottleneck.avg_time}s on average, {bottleneck.pct:.0%} of total time",
      actions: bottleneck.suggested_actions
    })

  # Resource utilization
  IF utilization_analysis.cpu_utilization_rate < 0.3:
    recommendations.append({
      type: "RESOURCE_OPTIMIZATION",
      title: "Reduce CPU allocation",
      description: "CPU utilization is only 30%, can reduce allocation to save costs"
    })

  RETURN recommendations
```

## 3. Error Handler

### 3.1 Purpose & Responsibilities

**Core Function**: Gestire failures in modo robusto - detect, classify, recover quando possibile, escalate quando necessario.

**Philosophy**: Errors are inevitable. Goal is graceful degradation, not perfect reliability.

**Responsibilities**:
1. **Error Detection**: Catch errors from all components
2. **Error Classification**: Categorize by type and severity
3. **Automatic Recovery**: Apply recovery strategies
4. **Escalation**: Route to human when can't auto-recover
5. **Learning**: Track error patterns per improve over time

### 3.2 Error Handler Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                      ERROR HANDLER                             │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ERROR DETECTION                                         │ │
│  │  • Exception catching                                    │ │
│  │  • Health checks                                         │ │
│  │  • Anomaly detection                                     │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ERROR CLASSIFIER                                        │ │
│  │  • Categorize error type                                 │ │
│  │  • Assess severity                                       │ │
│  │  • Determine recoverability                              │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  RECOVERY ENGINE                                         │ │
│  │  • Select recovery strategy                              │ │
│  │  • Execute recovery                                      │ │
│  │  • Verify recovery success                               │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ESCALATION MANAGER                                      │ │
│  │  • Determine if escalation needed                        │ │
│  │  • Route to appropriate handler                          │ │
│  │  • Track until resolution                                │ │
│  └────────────────────────┬─────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  ERROR ANALYTICS                                         │ │
│  │  • Track error frequencies                               │ │
│  │  • Identify patterns                                     │ │
│  │  • Suggest preventions                                   │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 3.3 Error Taxonomy

**Error Categories**:
```
┌──────────────────────────────────────────────────────────────┐
│                    ERROR TAXONOMY                            │
│                                                              │
│  1. TRANSIENT ERRORS (Temporary, retry may work)             │
│     • Network timeout                                       │
│     • Rate limit hit                                        │
│     • Service temporarily unavailable                       │
│     • Database connection timeout                           │
│     Recovery: Retry with exponential backoff                │
│                                                              │
│  2. RESOURCE ERRORS (Insufficient resources)                 │
│     • Budget exhausted                                      │
│     • Memory limit exceeded                                 │
│     • Timeout (task too long)                               │
│     • Context window exceeded                               │
│     Recovery: Abort, notify user, suggest alternatives      │
│                                                              │
│  3. LOGIC ERRORS (Internal bugs or assumptions)              │
│     • Assertion failure                                     │
│     • Null pointer / undefined variable                     │
│     • Index out of bounds                                   │
│     • Type mismatch                                         │
│     Recovery: Fallback to safe default, escalate            │
│                                                              │
│  4. INPUT ERRORS (Bad user input)                            │
│     • Invalid syntax                                        │
│     • Schema validation failure                             │
│     • Unsupported operation                                 │
│     • Conflicting constraints                               │
│     Recovery: Ask user for clarification                    │
│                                                              │
│  5. EXTERNAL ERRORS (Third-party failures)                   │
│     • API unavailable                                       │
│     • API breaking change                                   │
│     • Tool execution failure                                │
│     • Model unavailable                                     │
│     Recovery: Use fallback, alternative approach            │
│                                                              │
│  6. SAFETY ERRORS (Security violations)                      │
│     • Permission denied                                     │
│     • Safety bound violation                                │
│     • Injection attempt detected                            │
│     • Prohibited action requested                           │
│     Recovery: Reject, log, alert security team              │
└──────────────────────────────────────────────────────────────┘
```

**Error Severity Levels**:
```
┌──────────────────────────────────────────────────────────┐
│                  ERROR SEVERITY                          │
│                                                          │
│  LOW (Degraded but operational)                          │
│  • Non-critical feature unavailable                     │
│  • Performance degraded but acceptable                  │
│  • Example: Pattern cache miss (still works, just slower)│
│  Action: Log, continue operation                        │
│                                                          │
│  MEDIUM (Partial failure)                                │
│  • Subtask failed but task can continue                 │
│  • Non-preferred but acceptable alternative used        │
│  • Example: Tool timeout, using alternative tool        │
│  Action: Log, apply recovery, notify if frequent        │
│                                                          │
│  HIGH (Major failure)                                    │
│  • Task cannot complete successfully                    │
│  • User action blocked                                  │
│  • Example: All model APIs unavailable                  │
│  Action: Abort task, notify user, escalate              │
│                                                          │
│  CRITICAL (System-level failure)                         │
│  • Multiple tasks affected                              │
│  • Core service down                                    │
│  • Data integrity risk                                  │
│  • Example: Memory system unreachable                   │
│  Action: Emergency escalation, may pause new tasks      │
└──────────────────────────────────────────────────────────┘
```

### 3.4 Recovery Strategies

**Recovery Strategy Selection**:
```
Function HANDLE_ERROR(error, context):

  # STEP 1: Classify error
  classification = CLASSIFY_ERROR(error)
  # Returns: {category, severity, recoverability}

  # STEP 2: Select recovery strategy
  strategy = SELECT_RECOVERY_STRATEGY(classification, context)

  # STEP 3: Execute recovery
  TRY:
    recovery_result = EXECUTE_RECOVERY(strategy, error, context)

    IF recovery_result.success:
      LOG_INFO(f"Error recovered using {strategy}")
      RETURN CONTINUE(recovery_result.output)
    ELSE:
      # Recovery failed, try escalation
      RETURN ESCALATE(error, strategy, "Recovery failed")

  EXCEPT RecoveryError as recovery_error:
    # Recovery itself failed
    RETURN ESCALATE(error, strategy, f"Recovery error: {recovery_error}")

Function SELECT_RECOVERY_STRATEGY(classification, context):

  category = classification.category
  severity = classification.severity

  # TRANSIENT ERRORS → Retry
  IF category == "TRANSIENT":
    IF context.retry_count < MAX_RETRIES:
      RETURN RetryStrategy(
        max_attempts=MAX_RETRIES - context.retry_count,
        backoff=EXPONENTIAL
      )
    ELSE:
      RETURN EscalateStrategy("Max retries exceeded")

  # RESOURCE ERRORS → Abort or optimize
  ELSE IF category == "RESOURCE":
    IF error.type == "BUDGET_EXHAUSTED":
      RETURN AbortStrategy("Budget exhausted, cannot continue")
    ELSE IF error.type == "CONTEXT_TOO_LARGE":
      RETURN CompressContextStrategy()
    ELSE IF error.type == "TIMEOUT":
      RETURN AbortStrategy("Task taking too long")

  # EXTERNAL ERRORS → Fallback
  ELSE IF category == "EXTERNAL":
    IF FALLBACK_AVAILABLE(error.component):
      RETURN FallbackStrategy(
        fallback=GET_FALLBACK(error.component)
      )
    ELSE:
      RETURN EscalateStrategy("No fallback available")

  # INPUT ERRORS → Ask user
  ELSE IF category == "INPUT":
    RETURN AskUserStrategy(
      question=GENERATE_CLARIFICATION_QUESTION(error)
    )

  # LOGIC ERRORS → Safe default or escalate
  ELSE IF category == "LOGIC":
    IF HAS_SAFE_DEFAULT(error.operation):
      RETURN SafeDefaultStrategy()
    ELSE:
      RETURN EscalateStrategy("Internal error, no safe recovery")

  # SAFETY ERRORS → Reject and escalate
  ELSE IF category == "SAFETY":
    RETURN RejectAndEscalateStrategy(
      reason="Security violation",
      alert_security=True
    )

  # Unknown category
  ELSE:
    RETURN EscalateStrategy("Unknown error type")
```

**Retry Strategy**:
```
RetryStrategy {
  max_attempts: int,
  backoff_type: "EXPONENTIAL" | "LINEAR" | "CONSTANT",
  base_delay: float,  // seconds
  max_delay: float,
  jitter: boolean  // Add randomness to prevent thundering herd
}

Function EXECUTE_RETRY(strategy, operation, context):

  FOR attempt IN RANGE(1, strategy.max_attempts + 1):

    TRY:
      result = EXECUTE(operation, context)
      RETURN SUCCESS(result)

    EXCEPT Error as error:
      IF attempt == strategy.max_attempts:
        # Last attempt failed
        RETURN FAILURE("All retry attempts exhausted")

      # Calculate wait time
      IF strategy.backoff_type == "EXPONENTIAL":
        wait = MIN(strategy.base_delay * (2 ** attempt), strategy.max_delay)
      ELSE IF strategy.backoff_type == "LINEAR":
        wait = MIN(strategy.base_delay * attempt, strategy.max_delay)
      ELSE:
        wait = strategy.base_delay

      # Add jitter
      IF strategy.jitter:
        wait = wait * (0.5 + RANDOM() * 0.5)

      LOG_INFO(f"Retry attempt {attempt}, waiting {wait}s")
      SLEEP(wait)
      # Loop continues to next attempt
```

### 3.5 Escalation Management

**Escalation Decision Tree**:
```
Error Occurred
    ↓
Can recover automatically?
├─ YES → Apply Recovery
│        ↓
│        Success?
│        ├─ YES → Continue (Log for learning)
│        └─ NO → Escalate
│
└─ NO → Assess Severity
          ↓
          Severity?
          ├─ LOW → Log, Continue with degraded functionality
          ├─ MEDIUM → Log, Notify user of issue
          ├─ HIGH → Abort task, Notify user, Log incident
          └─ CRITICAL → Emergency escalation, Alert team

Escalation Paths:
• User notification (for HIGH severity affecting their task)
• Engineering team alert (for CRITICAL system issues)
• Security team alert (for safety violations)
• Incident creation (for repeated failures)
```

**Escalation Actions**:
```
Function ESCALATE(error, context, reason):

  severity = error.classification.severity

  # Create incident record
  incident = Incident {
    id: GENERATE_ID(),
    timestamp: NOW(),
    error: error,
    context: context,
    reason: reason,
    status: "OPEN"
  }

  STORE_INCIDENT(incident)

  # Escalation based on severity
  IF severity == "CRITICAL":
    # Immediate action required
    SEND_ALERT(
      channel="pager",
      recipients=ON_CALL_ENGINEERS,
      message=f"CRITICAL: {error.summary}",
      incident_id=incident.id
    )

    # May need to stop accepting new tasks
    IF error.affects_core_system:
      SET_SYSTEM_STATUS("DEGRADED")

  ELSE IF severity == "HIGH":
    # Task failed, notify user
    NOTIFY_USER(
      user_id=context.user_id,
      message=f"Task failed: {error.user_friendly_message}",
      incident_id=incident.id,
      retry_possible=error.retry_possible
    )

    # Alert engineering team (non-urgent)
    SEND_ALERT(
      channel="slack",
      recipients=ENGINEERING_TEAM,
      message=f"HIGH severity error: {error.summary}",
      incident_id=incident.id
    )

  ELSE IF severity == "MEDIUM":
    # Log and notify user if appropriate
    NOTIFY_USER(
      user_id=context.user_id,
      message=f"Task completed with issues: {error.user_friendly_message}",
      level="WARNING"
    )

  RETURN incident.id
```

### 3.6 Error Analytics

**Error Pattern Detection**:
```
Function ANALYZE_ERROR_PATTERNS(time_window):

  errors = GET_ERRORS_IN_WINDOW(time_window)

  # PATTERN 1: Frequency spikes
  error_rate = len(errors) / time_window.duration
  baseline_rate = GET_BASELINE_ERROR_RATE()

  IF error_rate > baseline_rate * 2:
    ALERT("Error rate spike detected", {
      current: error_rate,
      baseline: baseline_rate
    })

  # PATTERN 2: New error types
  error_types = SET(error.type for error in errors)
  known_types = GET_KNOWN_ERROR_TYPES()
  new_types = error_types - known_types

  IF new_types:
    ALERT("New error types detected", {
      new_types: list(new_types),
      frequency: {type: COUNT(errors where error.type == type) for type in new_types}
    })

  # PATTERN 3: Correlated errors
  # Find errors that tend to occur together
  correlations = FIND_ERROR_CORRELATIONS(errors)

  FOR correlation IN correlations WHERE correlation.significance > THRESHOLD:
    ALERT("Error correlation detected", {
      error_A: correlation.type_A,
      error_B: correlation.type_B,
      correlation: correlation.coefficient,
      # May indicate common root cause
    })

  # PATTERN 4: Cascading failures
  # Errors that trigger other errors
  cascades = FIND_ERROR_CASCADES(errors)

  FOR cascade IN cascades:
    ALERT("Cascading failure detected", {
      trigger: cascade.initial_error,
      caused: cascade.subsequent_errors,
      # Need to fix root cause
    })

  # PATTERN 5: User-specific patterns
  user_error_distribution = GROUP_BY(errors, 'user_id')

  FOR user_id, user_errors IN user_error_distribution:
    IF len(user_errors) > PERCENTILE(error_counts, 95):
      INVESTIGATE("User experiencing high error rate", {
        user_id: user_id,
        error_count: len(user_errors),
        common_error_types: MOST_COMMON(user_errors, key='type')
        # May be user-specific issue or abuse
      })
```

---

**Next**: [06-data-flows.md](06-data-flows.md) → Detailed interaction patterns and data transformations
