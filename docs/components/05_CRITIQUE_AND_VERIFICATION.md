# Component Specification: Critique & Verification System

*Last Updated:* November 2025
*Status:* Pre-Release Research Specification

---

## 1. Overview

The **Critique & Verification System** is responsible for ensuring the quality, correctness, and safety of agent outputs through multi-level verification. It implements **process supervision**, **outcome verification**, and **constitutional checks**, with iterative refinement capabilities.

### 1.1 Primary Responsibilities

- **Process Supervision:** Verify reasoning quality at each step
- **Outcome Verification:** Validate final solutions against requirements
- **Constitutional Checks:** Ensure safety, ethics, and alignment
- **Self-Critique:** Generate feedback for iterative improvement
- **Confidence Estimation:** Quantify uncertainty in outputs

### 1.2 Key Design Goals

1. **Multi-Level Verification:** Catch errors at multiple stages
2. **Early Error Detection:** Identify problems during reasoning, not after
3. **Formal Guarantees:** Use symbolic methods where possible
4. **Iterative Refinement:** Improve outputs through critique-revision cycles
5. **Safety by Design:** Constitutional AI principles throughout

---

## 2. Architectural Design

### 2.1 Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 CRITIQUE & VERIFICATION SYSTEM                  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              INPUT VALIDATOR                              │ │
│  │  • Schema validation                                      │ │
│  │  • Precondition checking                                  │ │
│  │  • Input sanitization                                     │ │
│  └────────────────┬──────────────────────────────────────────┘ │
│                   │                                             │
│                   ▼                                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │          PROCESS SUPERVISOR (Level 1)                     │ │
│  │  ┌──────────────────────────────────────────────────────┐│ │
│  │  │  Process Reward Models (PRMs)                        ││ │
│  │  │  • Token-level scoring                               ││ │
│  │  │  • Step-level validation                             ││ │
│  │  │  • Reasoning coherence check                         ││ │
│  │  └──────────────────────────────────────────────────────┘│ │
│  │  ┌──────────────────────────────────────────────────────┐│ │
│  │  │  Logical Consistency Checker                         ││ │
│  │  │  • Contradiction detection                           ││ │
│  │  │  • Premise-conclusion validity                       ││ │
│  │  └──────────────────────────────────────────────────────┘│ │
│  └────────────────┬──────────────────────────────────────────┘ │
│                   │                                             │
│                   ▼                                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │          OUTCOME VERIFIER (Level 2)                       │ │
│  │  ┌──────────────────────────────────────────────────────┐│ │
│  │  │  Symbolic Verification                               ││ │
│  │  │  • Code: Syntax, type, lint                          ││ │
│  │  │  • Math: Algebraic validity                          ││ │
│  │  │  • Logic: SAT/SMT solvers                            ││ │
│  │  └──────────────────────────────────────────────────────┘│ │
│  │  ┌──────────────────────────────────────────────────────┐│ │
│  │  │  Outcome Reward Models (ORMs)                        ││ │
│  │  │  • Final answer scoring                              ││ │
│  │  │  • Requirement satisfaction                          ││ │
│  │  └──────────────────────────────────────────────────────┘│ │
│  │  ┌──────────────────────────────────────────────────────┐│ │
│  │  │  Test Execution (for code)                           ││ │
│  │  │  • Unit tests                                        ││ │
│  │  │  • Integration tests                                 ││ │
│  │  │  • Property-based tests                              ││ │
│  │  └──────────────────────────────────────────────────────┘│ │
│  └────────────────┬──────────────────────────────────────────┘ │
│                   │                                             │
│                   ▼                                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │       CONSTITUTIONAL CHECKER (Level 3)                    │ │
│  │  • Safety principles                                      │ │
│  │  • Ethical guidelines                                     │ │
│  │  • Bias detection                                         │ │
│  │  • Harmful content filtering                              │ │
│  └────────────────┬──────────────────────────────────────────┘ │
│                   │                                             │
│                   ▼                                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │          CRITIQUE GENERATOR                               │ │
│  │  • Identify errors                                        │ │
│  │  • Explain failure causes                                 │ │
│  │  • Suggest improvements                                   │ │
│  │  • Generate revision prompts                              │ │
│  └────────────────┬──────────────────────────────────────────┘ │
│                   │                                             │
│                   ▼                                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │          ITERATIVE REFINER                                │ │
│  │  • Revision generation                                    │ │
│  │  • Re-verification                                        │ │
│  │  • Convergence detection                                  │ │
│  │  • Max iteration limiting                                 │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Verification Flow

```
Reasoning Trace + Solution
           │
           ▼
    ┌─────────────┐
    │   Process   │
    │ Supervision │
    └──────┬──────┘
           │
           ├─► FAIL ──┐
           │          │
           ▼ PASS     │
    ┌─────────────┐   │
    │  Outcome    │   │
    │Verification │   │
    └──────┬──────┘   │
           │          │
           ├─► FAIL ──┤
           │          │
           ▼ PASS     │
    ┌─────────────┐   │
    │Constitutional│  │
    │   Check     │   │
    └──────┬──────┘   │
           │          │
           ├─► FAIL ──┤
           │          │
           ▼ PASS     │
    ┌─────────────┐   │
    │   ACCEPT    │   │
    └─────────────┘   │
                      │
                      ▼
              ┌──────────────┐
              │   Critique   │
              │  Generator   │
              └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │   Revise &   │
              │  Re-verify   │
              └──────┬───────┘
                     │
                     ├─► Success ──► ACCEPT
                     │
                     ├─► Max Iters ──► REJECT
                     │
                     └─► Continue ──► (Loop)
```

---

## 3. Component Specifications

### 3.1 Process Supervisor

**Purpose:** Verify reasoning quality at each step during generation

#### 3.1.1 Process Reward Models (PRMs)

**Conceptual Architecture:**

The Process Reward Model is a specialized verification component trained to evaluate the correctness of individual reasoning steps. Unlike outcome-based verification which only scores final answers, PRMs provide step-by-step quality assessment during the reasoning process itself.

**Core Design Principles:**

| Principle | Description | Rationale |
|-----------|-------------|-----------|
| **Step-Level Granularity** | Score each reasoning step independently given its prefix | Enables early error detection before complete generation |
| **Calibrated Confidence** | Probabilistic scores (0-1) with calibrated uncertainty | Allows risk-aware decision making and threshold tuning |
| **Prefix-Dependent** | Scores consider all previous reasoning steps | Maintains coherence across the reasoning trajectory |
| **Lightweight Models** | Can use smaller models than generators | Reduces verification latency while maintaining accuracy |
| **Explainability** | Optionally generate explanations for scores | Facilitates debugging and trust in verification |

**Functional Model:**

The PRM operates through a scoring function that maps reasoning states to correctness probabilities:

```
PRM: (Prefix, CurrentStep, Context) → (Score, Explanation)

Where:
- Prefix: Sequence of previous reasoning steps
- CurrentStep: Current step being evaluated
- Context: Task description and requirements
- Score: P(step leads to correct solution | prefix, context) ∈ [0,1]
- Explanation: Optional natural language justification
```

**Training Data Generation Framework:**

| Data Source | Labeling Strategy | Label Confidence |
|-------------|-------------------|------------------|
| **Successful Traces** | All steps labeled positive | High (1.0) |
| **Failed Traces (Pre-Error)** | Steps before error labeled positive | Medium (0.8) |
| **Failed Traces (Error Point)** | Error step and after labeled negative | High (0.0) |
| **Synthetic Errors** | Artificially injected mistakes | Variable (0.0-0.5) |
| **Human Annotations** | Expert-labeled step quality | Very High (1.0) |
| **FoVer (May 2025)** | Formally verified training data via Z3/Isabelle | Perfect (1.0) |

**Recent PRM Advances (2025):**

| Research | Key Innovation | Performance Impact |
|----------|---------------|-------------------|
| **ThinkPRM** (Apr 2025) | Generative CoT verification with 1% training data | +8% GPQA Diamond, +7.2pp out-of-distribution |
| **FoVer** (May 2025) | Formally verified step-level labels via Z3/Isabelle | Competitive with SOTA on ProcessBench using synthetic formal verification |
| **PAV** (ICLR 2025) | Process Advantage Verifiers | 8% higher accuracy, 1.5-5x compute efficiency vs outcome reward models, 6x sample efficiency in online RL |
| **R-PRM** (2025) | Reasoning-driven process reward modeling | 94.1% GSM8K, 67.7% MATH-500 |

**Aggregation Strategies:**

Different aggregation methods serve different verification goals:

| Method | Formula | Use Case |
|--------|---------|----------|
| **Geometric Mean** | (∏ scores)^(1/n) | Conservative: one bad step → low overall score |
| **Arithmetic Mean** | (Σ scores) / n | Balanced: average quality assessment |
| **Minimum Score** | min(scores) | Worst-case: identify weakest link |
| **Weighted Sum** | Σ (weight_i × score_i) | Prioritized: weight critical steps higher |
| **Early Stopping** | First score < threshold | Efficient: stop at first major error |

**Design Decision: Geometric vs. Arithmetic Mean**

| Consideration | Geometric Mean | Arithmetic Mean |
|--------------|----------------|-----------------|
| **Sensitivity to Errors** | Very high (multiplicative) | Moderate (additive) |
| **Error Tolerance** | Zero tolerance for bad steps | Averages out individual errors |
| **False Positive Rate** | Higher | Lower |
| **False Negative Rate** | Lower | Higher |
| **Computational Cost** | Same | Same |
| **Recommended For** | Safety-critical tasks | General reasoning |

#### 3.1.2 Logical Consistency Checker

**Purpose:** Detect logical contradictions in reasoning

**Conceptual Framework:**

The Logical Consistency Checker operates on two levels: natural language contradiction detection and formal logical validity checking.

**Contradiction Detection Architecture:**

| Layer | Mechanism | Coverage |
|-------|-----------|----------|
| **Semantic Level** | Natural Language Inference (NLI) models | Paraphrase contradictions |
| **Factual Level** | Knowledge base consistency | World knowledge violations |
| **Temporal Level** | Timeline coherence | Event ordering contradictions |
| **Quantitative Level** | Numerical consistency | Mathematical contradictions |
| **Logical Level** | Formal logic parsers | Logical form contradictions |

**Logical Validity Framework:**

```
Validity Check: (Premise, Conclusion) → ValidityResult

ValidityResult = {
  is_valid: Boolean,
  validity_type: {Deductive, Inductive, Abductive},
  confidence: [0, 1],
  counterexamples: [Example],
  explanation: String
}
```

**Design Patterns for Logic Checking:**

| Pattern | Description | Implementation Approach |
|---------|-------------|-------------------------|
| **Contradiction Pairs** | Find statement pairs that cannot both be true | Pairwise NLI classification |
| **Entailment Chain** | Verify each conclusion follows from premises | Sequential logical inference |
| **Proof Verification** | Validate formal mathematical proofs | Proof assistant integration |
| **Assumption Tracking** | Ensure assumptions don't contradict | Dependency graph analysis |

**Trade-Off Analysis:**

| Aspect | Strict Checking | Lenient Checking |
|--------|-----------------|------------------|
| **False Positives** | Higher | Lower |
| **False Negatives** | Lower | Higher |
| **Usability** | May reject valid reasoning | Accepts more reasoning |
| **Safety** | Higher | Lower |
| **Computational Cost** | Higher (thorough) | Lower (fast) |
| **Recommended For** | Formal domains | Creative domains |

---

### 3.2 Outcome Verifier

**Purpose:** Validate final solutions against requirements

#### 3.2.1 Symbolic Verification

**Design Philosophy:** Use deterministic, rule-based verification wherever possible for formal guarantees.

**Code Verification Framework:**

| Verification Stage | Purpose | Tools/Methods | Guarantee Level |
|-------------------|---------|---------------|-----------------|
| **Syntax Validation** | Parse code without errors | AST parsers | Absolute |
| **Type Checking** | Verify type correctness | Static type checkers | High |
| **Linting** | Enforce style and best practices | Static analyzers | Medium |
| **Security Analysis** | Detect vulnerabilities | Pattern matching, taint analysis | Medium |
| **Requirement Mapping** | Verify specification coverage | Trace matrix | High |

**Code Verification Decision Matrix:**

| Criterion | Syntax | Types | Lint | Security | Tests |
|-----------|--------|-------|------|----------|-------|
| **Automation** | Full | Full | Full | Partial | Full |
| **Precision** | Absolute | High | Medium | Medium | High |
| **Coverage** | 100% | High | Variable | Partial | Test-dependent |
| **False Positives** | Zero | Low | Medium | High | Low |
| **Execution Required** | No | No | No | No | Yes |
| **Latency** | <100ms | <1s | <1s | <5s | Variable |

**Mathematical Verification Framework:**

| Verification Type | Domain | Method | Certainty |
|------------------|--------|--------|-----------|
| **Algebraic Validity** | Equations, expressions | Computer algebra systems (CAS) | Absolute |
| **Numerical Consistency** | Arithmetic operations | Constraint solving | High |
| **Proof Verification** | Mathematical theorems | Proof assistants (Lean, Coq) | Absolute |
| **Constraint Satisfaction** | Inequalities, bounds | SMT solvers | Absolute |
| **Dimensional Analysis** | Physical equations | Unit checking | High |

**Symbolic Verification Architecture:**

```
Symbolic Verifier: (Artifact, Domain, Requirements) → VerificationResult

VerificationResult = {
  stages: {
    stage_name: {
      valid: Boolean,
      errors: [Error],
      warnings: [Warning],
      metrics: {metric_name: value}
    }
  },
  overall_valid: Boolean,
  confidence: [0, 1],
  formal_guarantees: [Guarantee]
}
```

**Design Trade-Offs: Symbolic vs. Neural Verification**

| Aspect | Symbolic Verification | Neural Verification |
|--------|----------------------|---------------------|
| **Precision** | Absolute (within domain) | Probabilistic |
| **Coverage** | Limited to formal domains | Broad applicability |
| **Explainability** | Complete (rule-based) | Limited (black-box) |
| **Robustness** | High (deterministic) | Variable (data-dependent) |
| **Latency** | Fast (algorithmic) | Variable (model-dependent) |
| **Maintenance** | Rule updates required | Retraining required |
| **False Positives** | Very low | Can be high |
| **Recommended For** | Formal artifacts | Natural language, ambiguous domains |

#### 3.2.2 Outcome Reward Models (ORMs)

**Conceptual Design:**

Outcome Reward Models provide probabilistic assessment of solution quality when symbolic verification is unavailable or insufficient. Unlike PRMs which score intermediate steps, ORMs focus on final output evaluation.

**ORM Architecture:**

```
ORM: (Task, Solution) → QualityScore

Where:
- Task: {description, requirements, constraints}
- Solution: Final output
- QualityScore: P(solution correctly solves task) ∈ [0,1]
```

**Evaluation Dimensions:**

| Dimension | Description | Weight (Example) |
|-----------|-------------|------------------|
| **Correctness** | Does solution solve the problem? | 0.40 |
| **Completeness** | Are all requirements addressed? | 0.25 |
| **Efficiency** | Is approach optimal/reasonable? | 0.15 |
| **Clarity** | Is solution understandable? | 0.10 |
| **Robustness** | Does it handle edge cases? | 0.10 |

**Comparative Assessment Framework:**

For preference learning and solution ranking:

```
Comparator: (Task, Solution_A, Solution_B) → Preference

Preference = {
  choice: {'A', 'B', 'tie'},
  confidence: [0, 1],
  reasoning: String,
  dimension_scores: {dimension: comparative_score}
}
```

**Training Paradigms:**

| Paradigm | Data Source | Label Type | Scalability |
|----------|-------------|------------|-------------|
| **Supervised** | Human-labeled outcomes | Binary (correct/incorrect) | Low (expensive) |
| **Outcome-Based** | Test results, verification | Automatic labels | High |
| **Preference-Based** | Pairwise comparisons | Relative ranking | Medium |
| **Self-Play** | Model-generated solutions | Self-evaluation | Very High |
| **Hybrid** | Combination of above | Mixed | High |

#### 3.2.3 Test Execution Framework (Code Verification)

**Conceptual Model:**

Test execution provides empirical validation through concrete examples, complementing static verification.

**Test Execution Architecture:**

```
Test Executor: (Code, TestSuite, Environment) → TestResults

TestResults = {
  tests_run: Integer,
  tests_passed: Integer,
  tests_failed: Integer,
  pass_rate: Float,
  failures: [{test_id, error, output}],
  coverage: CoverageMetrics,
  execution_time: Float
}
```

**Testing Strategy Matrix:**

| Test Type | Coverage | Execution Cost | Error Detection | Use Case |
|-----------|----------|----------------|-----------------|----------|
| **Unit Tests** | Function-level | Low | High (specific) | Component verification |
| **Integration Tests** | Cross-component | Medium | Medium | Interface verification |
| **Property-Based** | Input space sampling | High | Very High | Edge case discovery |
| **Fuzzing** | Random/guided exploration | Very High | High (crashes) | Robustness testing |
| **Regression Tests** | Known issues | Low | Medium | Prevent regressions |

**Isolation Design Patterns:**

| Pattern | Mechanism | Security | Overhead |
|---------|-----------|----------|----------|
| **Sandboxing** | OS-level containers | High | Medium |
| **Virtual Environments** | Language-specific isolation | Medium | Low |
| **Process Isolation** | Separate processes | Medium | Low |
| **Resource Limiting** | CPU/memory/time caps | High | Low |

**Timeout Strategy:**

| Timeout Type | Value | Rationale |
|-------------|-------|-----------|
| **Per-Test** | 1-10s | Prevent infinite loops |
| **Test Suite** | 30-60s | Overall execution limit |
| **Startup** | 5s | Environment initialization |
| **Adaptive** | Based on complexity | Balance thoroughness vs. speed |

---

### 3.3 Constitutional Checker

**Purpose:** Ensure safety, ethics, and alignment with organizational values

**Theoretical Foundation:**

Based on Constitutional AI (Anthropic, 2022), which uses natural language principles rather than reward hacking to align model behavior.

**Constitutional Framework Architecture:**

```
Constitution = {
  principles: [Principle],
  hierarchy: PriorityStructure,
  context_rules: {context: applicable_principles}
}

Principle = {
  id: String,
  text: Natural Language Statement,
  severity: {critical, high, medium, low},
  domain: {general, domain-specific},
  examples: [positive, negative]
}
```

**Principle Categories:**

| Category | Focus | Example Principles |
|----------|-------|-------------------|
| **Safety** | Preventing harm | "Do not provide information that could harm others" |
| **Privacy** | Data protection | "Respect user privacy and confidentiality" |
| **Truthfulness** | Accuracy | "Provide accurate information and cite sources" |
| **Fairness** | Bias avoidance | "Be objective and avoid demographic bias" |
| **Legality** | Compliance | "Do not provide instructions for illegal activities" |
| **Helpfulness** | User benefit | "Provide useful and relevant information" |

**Constitutional Checking Process:**

```
Constitutional Checker: (Solution, Reasoning, Constitution) → ComplianceResult

ComplianceResult = {
  compliant: Boolean,
  violations: [{
    principle: Principle,
    explanation: String,
    severity: {critical, high, medium, low},
    location: Reference to violation in solution
  }],
  overall_risk: RiskLevel
}
```

**Severity Hierarchy:**

| Severity | Definition | Action |
|----------|------------|--------|
| **Critical** | Immediate safety/legal risk | Automatic rejection |
| **High** | Significant ethical concern | Requires revision |
| **Medium** | Notable but addressable issue | Suggest improvement |
| **Low** | Minor stylistic concern | Optional improvement |

**Harm Classification Framework:**

| Harm Type | Detection Method | Threshold |
|-----------|-----------------|-----------|
| **Violence** | Content classifier | 0.7 |
| **Hate Speech** | Bias detection model | 0.8 |
| **Sexual Content** | Inappropriate content filter | 0.9 |
| **Privacy Violation** | PII detection | 0.6 |
| **Illegal Activity** | Pattern matching + classification | 0.7 |
| **Self-Harm** | Mental health classifier | 0.8 |

**Design Decisions:**

| Decision Point | Option A | Option B | Recommendation |
|---------------|----------|----------|----------------|
| **Evaluation Method** | LLM-based (flexible) | Rule-based (precise) | Hybrid: rules for clear cases, LLM for nuanced |
| **Threshold Setting** | Conservative (high sensitivity) | Permissive (low false positives) | Domain-dependent; conservative for critical |
| **Violation Handling** | Reject immediately | Allow with warning | Severity-based: reject critical, warn medium |
| **Context Sensitivity** | Same principles all contexts | Context-specific principles | Context-specific for nuanced domains |

**Example Constitutional Document Structure:**

| Element | Description |
|---------|-------------|
| **Core Values** | Fundamental principles (helpfulness, harmlessness, honesty) |
| **Domain Principles** | Specific to task type (medical ethics, financial regulations) |
| **Severity Levels** | Prioritization of principles |
| **Exception Cases** | Legitimate edge cases (medical education, security research) |
| **Update Mechanism** | Process for evolving constitution |

---

### 3.4 Critique Generator

**Purpose:** Generate actionable feedback for revision

**Theoretical Foundation:**

Based on CRITIC (Gou et al., 2023) and recent work on self-improvement through critique (2024-2025).

**Critique Generation Architecture:**

```
Critique Generator: (VerificationResults, Solution, Trace) → Critique

Critique = {
  issues: [{
    type: IssueType,
    location: Reference,
    description: String,
    severity: {critical, high, medium, low},
    evidence: Evidence
  }],
  suggestions: [ActionableSuggestion],
  overall_severity: SeverityLevel,
  revision_priority: [IssueID ordered by priority]
}
```

**Issue Taxonomy:**

| Issue Type | Source | Detectability | Fix Complexity |
|-----------|--------|---------------|----------------|
| **Process Error** | PRM failure | High | Medium |
| **Logical Inconsistency** | Contradiction detection | High | High |
| **Syntax Error** | Code parser | Absolute | Low |
| **Type Mismatch** | Type checker | Absolute | Medium |
| **Test Failure** | Test execution | Absolute | Variable |
| **Requirement Gap** | Requirement mapping | Medium | Medium |
| **Constitutional Violation** | Constitutional check | Medium | High |
| **Performance Issue** | Efficiency analysis | Low | High |

**Critique Quality Dimensions:**

| Dimension | Description | Measurement |
|-----------|-------------|-------------|
| **Specificity** | Precise error localization | Line/step number provided |
| **Actionability** | Clear improvement path | Concrete suggestions given |
| **Comprehensiveness** | All errors identified | Issue coverage rate |
| **Prioritization** | Critical issues highlighted | Severity ranking |
| **Constructiveness** | Improvement-focused tone | Positive framing |

**Suggestion Generation Framework:**

```
Suggestion = {
  issue_ref: IssueID,
  action: {fix, revise, reconsider, remove, add},
  specificity: {general, specific, example},
  reasoning: Explanation,
  alternative_approaches: [Approach]
}
```

**Critique Formatting Strategies:**

| Strategy | Format | Use Case |
|----------|--------|----------|
| **Localized Feedback** | Error at specific location | Code, structured reasoning |
| **Step-by-Step Guidance** | Sequential improvement plan | Complex multi-error situations |
| **Comparative Analysis** | Show correct vs. incorrect | Learning-focused revision |
| **Holistic Assessment** | Overall quality evaluation | Final review |
| **Socratic Questioning** | Prompt self-reflection | Metacognitive improvement |

**Critique-to-Revision Transformation:**

```
Revision Prompt Generator: (Critique) → RevisionPrompt

RevisionPrompt = {
  context: Original task + attempt,
  issues: Structured error description,
  suggestions: Improvement guidance,
  constraints: Must maintain correct parts,
  success_criteria: How to know revision succeeded
}
```

**Design Patterns:**

| Pattern | Description | Benefits |
|---------|-------------|----------|
| **Error-First** | Present errors before suggestions | Focus attention on problems |
| **Suggestion-First** | Lead with improvements | Positive framing |
| **Severity-Ordered** | Critical issues first | Efficient prioritization |
| **Location-Grouped** | Group by code section | Contextual coherence |
| **Type-Grouped** | Group by error type | Pattern recognition |

---

### 3.5 Iterative Refiner

**Purpose:** Coordinate revision-verification cycles

**Conceptual Framework:**

The Iterative Refiner orchestrates a feedback loop between verification, critique, and solution generation to progressively improve outputs.

**Refinement Architecture:**

```
Iterative Refiner: (InitialSolution, Task, MaxIterations) → RefinementResult

RefinementResult = {
  success: Boolean,
  final_solution: Solution,
  iterations_used: Integer,
  improvement_trajectory: [QualityScore],
  termination_reason: {verified, max_iterations, convergence, critical_failure},
  refinement_history: [RefinementStep]
}

RefinementStep = {
  iteration: Integer,
  solution: Solution,
  verification: VerificationResult,
  critique: Critique,
  improvements: [Improvement]
}
```

**Refinement Flow State Machine:**

```
START → VERIFY → {
  PASS → SUCCESS,
  FAIL → CRITIQUE → REVISE → VERIFY,
  CRITICAL_FAIL → FAILURE
}

With transitions:
- MAX_ITERATIONS → PARTIAL_SUCCESS
- CONVERGENCE → BEST_EFFORT
```

**Termination Criteria:**

| Criterion | Condition | Outcome |
|-----------|-----------|---------|
| **Verification Success** | All checks pass | Complete success |
| **Max Iterations** | Iteration count exceeded | Best available solution |
| **Convergence** | Solution no longer changing | Local optimum |
| **Critical Failure** | Unfixable fundamental error | Failure |
| **Quality Threshold** | Acceptable quality reached | Partial success |

**Convergence Detection:**

| Method | Metric | Threshold |
|--------|--------|-----------|
| **Edit Distance** | Character/token differences | >95% similarity |
| **Semantic Similarity** | Embedding cosine distance | >0.98 |
| **Error Persistence** | Same errors multiple iterations | 2-3 iterations |
| **Quality Plateau** | No score improvement | <0.01 change |

**Iteration Budget Allocation:**

| Strategy | Distribution | Use Case |
|----------|--------------|----------|
| **Fixed Budget** | All iterations equal | Unknown difficulty |
| **Progressive** | More time per iteration | Complex improvements |
| **Adaptive** | Based on improvement rate | Efficient resource use |
| **Early Stopping** | Stop when threshold met | Performance optimization |

**Design Decision: Iteration Limits**

| Consideration | Few Iterations (1-2) | Medium (3-5) | Many (5+) |
|--------------|---------------------|--------------|-----------|
| **Latency** | Low | Medium | High |
| **Quality** | Lower | Good | Diminishing returns |
| **Cost** | Low | Medium | High |
| **Success Rate** | Lower | Higher | Marginal improvement |
| **Recommended For** | Simple tasks | General use | Critical tasks |

**Improvement Tracking:**

```
Improvement Metric: (Iteration_t, Iteration_t+1) → ImprovementScore

Dimensions:
- Error reduction: Δ(error_count)
- Score improvement: Δ(verification_score)
- Requirement coverage: Δ(requirements_met)
- Quality dimensions: Δ(quality_metrics)
```

**Revision Generation Strategies:**

| Strategy | Approach | Trade-Off |
|----------|----------|-----------|
| **Focused Revision** | Only modify error locations | Conservative, may miss root causes |
| **Holistic Rewrite** | Regenerate entire solution | Thorough but may lose correct parts |
| **Incremental Repair** | Small targeted fixes | Efficient but may accumulate patches |
| **Alternative Approach** | Try completely different method | Novel but expensive |

---

## 4. Integration with Other Components

### 4.1 Reasoning Core Integration

**Real-Time Integration Pattern:**

```
During Generation:
  For each reasoning step:
    PRM → score_step()
    If score < threshold:
      Trigger: {backtrack, alternative_sampling, beam_search_pruning}
```

**Post-Generation Integration Pattern:**

```
After Complete Generation:
  Full Verification Pipeline
  If not passed:
    Iterative Refinement Loop
  Return: Best verified solution
```

**Integration Points:**

| Phase | Reasoning Core Action | Verification Action |
|-------|---------------------|---------------------|
| **Planning** | Generate approach | Verify approach validity |
| **Step Generation** | Produce reasoning step | PRM scoring |
| **Solution Formation** | Extract final answer | Symbolic + ORM verification |
| **Refinement** | Generate revision | Re-verify with same pipeline |

### 4.2 Memory System Integration

**Verification Data Storage Schema:**

```
Episodic Memory Entry:
{
  task: TaskDescription,
  solution: FinalSolution,
  verification_results: {
    process: ProcessResults,
    outcome: OutcomeResults,
    constitutional: ConstitutionalResults
  },
  revision_history: [RevisionStep],
  final_quality_score: Float,
  success: Boolean
}
```

**Learning from Verification:**

| Memory Type | Stored Information | Learning Application |
|-------------|-------------------|---------------------|
| **Episodic** | Verification outcomes | Similar task retrieval |
| **Procedural** | Error patterns | Strategy adjustment |
| **Semantic** | Constitutional violations | Principle refinement |
| **Working** | Current verification state | Multi-step verification |

**Failure Pattern Recognition:**

```
Procedural Learning:
  Pattern = {task_type, error_type, frequency, typical_fixes}

  On new task:
    Retrieve similar failure patterns
    Proactively avoid known pitfalls
    Adjust verification sensitivity
```

### 4.3 Meta-Cognitive System Integration

**Strategy Selection Based on Verification:**

| Verification Result | Metacognitive Action |
|-------------------|---------------------|
| **Repeated PRM Failures** | Switch to more structured reasoning |
| **Outcome Verification Fails** | Increase solution generation samples |
| **Constitutional Violations** | Adjust generation constraints |
| **Test Failures** | Switch to test-driven generation |

---

## 5. Research Foundations

### 5.1 Process Supervision

**Key Research Papers:**

| Paper | Authors/Year | Key Contribution |
|-------|-------------|------------------|
| "Let's Verify Step by Step" | Lightman et al., OpenAI (2023) | Demonstrated process supervision superiority |
| "Scaling Automated Process Verifiers" | (2024) | Automated PRM training from outcomes |
| "Process Reward Models That Think" | (2025) | Self-explanatory PRMs |

**Key Finding:**
Process supervision (verifying intermediate steps) significantly outperforms outcome supervision (verifying only final answers) in mathematical reasoning tasks, reducing error rates by 35-50%.

**Theoretical Framework:**

```
Error Detection Timing:
  Early (Process): Lower cost to fix, higher detection coverage
  Late (Outcome): Higher cost to fix, may miss reasoning flaws

Process Supervision Advantage = Early Detection Benefit - Annotation Cost
```

**Research Evolution:**

| Period | Focus | Achievement |
|--------|-------|-------------|
| **2021-2022** | Outcome reward models | Baseline verification |
| **2023** | Process reward models | Step-level verification |
| **2024** | Automated PRM training | Scalable data generation |
| **2025** | Integrated verification | Multi-level systems |

### 5.2 Self-Critique

**Key Research Papers:**

| Paper | Authors/Year | Key Contribution |
|-------|-------------|------------------|
| "CRITIC: LLMs Can Self-Correct with Tool-Interactive Critiquing" | Gou et al. (2023) | Tool-augmented self-critique |
| "Training LMs to Critique via RL" | CTRL (2025) | RL-based critique learning |
| "Self-Reflection in LLM Agents" | (2024) | Metacognitive critique mechanisms |

**Key Finding:**
Self-critique with external verification tools improves task success rates by 20-40% across diverse domains, with greatest impact on complex multi-step reasoning.

**Critique Quality Factors:**

| Factor | Impact on Improvement |
|--------|----------------------|
| **Specificity** | High: Precise critiques → targeted fixes |
| **Actionability** | Very High: Concrete suggestions crucial |
| **Accuracy** | Critical: False critiques degrade performance |
| **Coverage** | Medium: Diminishing returns beyond major issues |

**Theoretical Model:**

```
Self-Improvement = f(Critique Quality, Revision Capability, Verification Accuracy)

Where:
- Critique Quality: Error identification accuracy
- Revision Capability: Agent's ability to improve
- Verification Accuracy: Correctness of validation
```

### 5.3 Constitutional AI

**Key Research Papers:**

| Paper | Authors/Year | Key Contribution |
|-------|-------------|------------------|
| "Constitutional AI: Harmlessness from AI Feedback" | Anthropic (2022) | Principle-based alignment |
| "Contextual Constitutional AI" | (2024) | Context-dependent principles |

**Key Finding:**
Training with natural language principles and AI feedback (Constitutional AI) produces models that are helpful, harmless, and honest without reward hacking, improving safety while maintaining capability.

**Constitutional Design Principles:**

| Principle | Description | Benefit |
|-----------|-------------|---------|
| **Natural Language** | Express values in human language | Interpretable, flexible |
| **Self-Improvement** | AI evaluates AI outputs | Scalable alignment |
| **Principle Hierarchy** | Prioritized values | Conflict resolution |
| **Transparency** | Explicit value statements | Auditability |

**Comparison to Other Alignment Methods:**

| Method | Scalability | Transparency | Flexibility | Robustness |
|--------|-------------|--------------|-------------|------------|
| **Constitutional AI** | High | High | High | Medium |
| **RLHF** | Medium | Low | Medium | High |
| **Rule-Based** | High | Very High | Low | Medium |
| **Supervised Finetuning** | Low | Medium | Low | Low |

### 5.4 Verification in Formal Domains

**Mathematical Reasoning:**

| System | Type | Strengths |
|--------|------|-----------|
| **Lean** | Proof assistant | Formal guarantees |
| **Coq** | Proof assistant | Mature ecosystem |
| **Isabelle** | Proof assistant | Automation |
| **Z3** | SMT solver | Constraint solving |
| **SymPy** | CAS | Symbolic computation |

**Code Verification:**

| Approach | Coverage | Precision | Automation |
|----------|----------|-----------|------------|
| **Static Analysis** | High | Medium | Full |
| **Type Systems** | Type errors only | Very High | Full |
| **Formal Verification** | Targeted | Absolute | Low |
| **Testing** | Test coverage | High | Full |
| **Fuzzing** | Edge cases | High | Full |

---

## 6. Performance Characteristics

### 6.1 Verification Latency

| Level | Latency | Thoroughness | Bottleneck |
|-------|---------|--------------|------------|
| **Process (online)** | 10-50ms/step | Medium | Model inference |
| **Outcome (symbolic)** | 100-500ms | High | Parser complexity |
| **Outcome (model)** | 1-5s | Medium | Model size |
| **Constitutional** | 1-3s | High | Principle evaluation |
| **Full Pipeline** | 5-10s | Very High | Sequential stages |
| **With Refinement (3 iters)** | 15-30s | Highest | Iteration count |

### 6.2 Accuracy Improvement

| Verification Level | Error Reduction | False Positive Rate | Computational Cost |
|-------------------|-----------------|--------------------|--------------------|
| **None (Baseline)** | 0% | N/A | 1x |
| **Outcome Only** | 30% | 5-10% | 1.5x |
| **Process Only** | 50% | 10-15% | 2x |
| **Multi-Level** | 70% | 5-8% | 3x |
| **+ Iterative Refinement** | 85% | 3-5% | 5-8x |

### 6.3 Scalability Analysis

| Dimension | Small Scale | Medium Scale | Large Scale |
|-----------|-------------|--------------|-------------|
| **Task Complexity** | Simple queries | Multi-step problems | Research-level tasks |
| **Solution Length** | <100 tokens | 100-1000 tokens | 1000+ tokens |
| **Verification Time** | <1s | 1-10s | 10-60s |
| **Iteration Budget** | 1-2 | 2-3 | 3-5 |
| **Success Rate** | >95% | >85% | >70% |

### 6.4 Trade-Off Analysis

**Speed vs. Quality:**

| Configuration | Verification Depth | Latency | Quality | Use Case |
|--------------|-------------------|---------|---------|----------|
| **Fast** | Outcome only | 1s | Good | High-volume, simple |
| **Balanced** | Outcome + Process | 5s | Very Good | General purpose |
| **Thorough** | Full multi-level | 10s | Excellent | Critical tasks |
| **Exhaustive** | + Refinement | 30s | Optimal | Research, safety-critical |

**Cost vs. Benefit:**

| Metric | 1 Iteration | 2 Iterations | 3 Iterations | 5 Iterations |
|--------|------------|--------------|--------------|--------------|
| **Cost Multiplier** | 1x | 2x | 3x | 5x |
| **Error Reduction** | 50% | 70% | 85% | 90% |
| **Marginal Benefit** | - | 20% | 15% | 5% |
| **Recommended** | Simple | Standard | Critical | Safety-critical only |

---

## 7. Design Patterns and Best Practices

### 7.1 Verification Pipeline Design Patterns

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Sequential Cascade** | Each level feeds into next | Standard comprehensive verification |
| **Parallel Verification** | All levels run concurrently | Time-sensitive applications |
| **Early Termination** | Stop at first critical failure | Efficiency-focused systems |
| **Adaptive Depth** | Adjust thoroughness by task | Variable task complexity |
| **Layered Defense** | Multiple redundant checks | Safety-critical domains |

### 7.2 Critique Quality Principles

| Principle | Implementation | Impact |
|-----------|----------------|--------|
| **Specificity** | Provide exact locations and examples | Enables targeted fixes |
| **Actionability** | Suggest concrete improvements | Facilitates effective revision |
| **Constructiveness** | Frame as opportunities to improve | Maintains solution quality |
| **Comprehensiveness** | Identify all significant issues | Prevents iteration waste |
| **Prioritization** | Rank issues by importance | Efficient improvement path |

### 7.3 Refinement Strategy Selection

| Task Characteristic | Recommended Strategy | Rationale |
|--------------------|---------------------|-----------|
| **Well-defined with tests** | Test-driven refinement | Clear success criteria |
| **Formal domain** | Symbolic verification focus | Absolute guarantees available |
| **Creative/open-ended** | Constitutional + ORM | Subjective quality assessment |
| **Safety-critical** | Conservative thresholds + many iterations | Risk mitigation |
| **Time-sensitive** | Single refinement pass | Latency constraints |

---

## 8. Future Research Directions

### 8.1 Open Problems

| Problem | Description | Potential Approaches |
|---------|-------------|---------------------|
| **Scalable Process Supervision** | PRM training requires extensive data | Automated annotation, transfer learning |
| **Critique Accuracy** | Self-critique can be overconfident | Calibration, external validation |
| **Verification-Generation Gap** | Verifiers may reject creative solutions | Uncertainty-aware verification |
| **Computational Cost** | Multi-level verification expensive | Selective verification, caching |
| **Convergence Guarantees** | No guarantee refinement converges | Theoretical analysis, better termination |

### 8.2 Emerging Techniques

| Technique | Status | Promise |
|-----------|--------|---------|
| **Learned Verifiers** | Research | Adaptive verification strategies |
| **Neural-Symbolic Fusion** | Early | Best of both paradigms |
| **Meta-Verification** | Conceptual | Verify the verifiers |
| **Continuous Verification** | Emerging | Real-time quality monitoring |
| **Collaborative Verification** | Research | Multi-agent consensus |

### 8.3 Integration Opportunities

| Integration | Benefit | Challenge |
|-------------|---------|-----------|
| **Tool Use** | External verification tools | Tool selection and integration |
| **Human-in-Loop** | Expert validation | Latency and cost |
| **Ensemble Verification** | Multiple verification methods | Consensus mechanisms |
| **Active Learning** | Targeted verification data collection | Efficient exploration |

---

## 9. Conclusion

The Critique & Verification System represents a comprehensive quality assurance framework for AI agent outputs, grounded in recent research on process supervision, self-critique, and constitutional AI.

### 9.1 Key Design Principles

1. **Multi-Level Verification:** Layered defense through Process, Outcome, and Constitutional checking
2. **Early Error Detection:** Process supervision identifies problems during generation
3. **Formal Guarantees:** Symbolic verification provides absolute certainty where applicable
4. **Iterative Refinement:** Critique-revision cycles progressively improve solutions
5. **Safety by Design:** Constitutional principles embedded throughout

### 9.2 Core Innovations

| Innovation | Impact |
|------------|--------|
| **Process Reward Models** | 50% error reduction over outcome-only |
| **Symbolic-Neural Hybrid** | Precision where possible, flexibility where needed |
| **Constitutional Framework** | Explicit, interpretable value alignment |
| **Iterative Refinement** | 85% total error reduction |

### 9.3 Success Criteria

The system is successful when it:
- Reduces error rates by >70% compared to unverified generation
- Maintains false positive rate below 10%
- Completes verification within acceptable latency (5-30s depending on task)
- Provides actionable critique for >90% of failures
- Achieves convergence within 3 iterations for >80% of tasks

This system is critical for achieving high reliability in agent outputs, especially for complex and safety-critical tasks.

---

**References:**
- Lightman et al. (2023). "Let's Verify Step by Step." OpenAI.
- Gou et al. (2023). "CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing."
- Anthropic (2022). "Constitutional AI: Harmlessness from AI Feedback."
- See Main Architecture for complete bibliography
- See Reasoning Core for PRM integration details
- See Meta-Cognitive System for strategy selection based on verification results
