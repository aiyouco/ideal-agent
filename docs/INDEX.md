# Documentation Index
## Access Charter for the Ideal Agent Dossier

Use this index to navigate the research corpus. Files are grouped by role: the architecture spine, component monographs, and annexes. References to supporting research are explicit so every reader can trace requirements back to published work.

---

## Architecture Spine
- `architecture/00_MAIN_ARCHITECTURE.md` — Complete blueprint grounded in CoALA++, MAP, Plan-and-Act, and the 2025 world-model necessity proof. Contains the layered stack definition, formal notation, interface matrices, sequence diagrams, and comprehensive open research questions (§12).

## Component Monographs
Each entry gives the owning document, scope, and primary open-source anchors. Full specifications (pseudo code, diagrams, research links) sit inside the referenced file.

| Document | Scope | Research Anchors |
|----------|-------|------------------|
| `components/01_META_COGNITIVE_SYSTEM.md` | Governance, compute budgeting, strategy arbitration | Kimi K2 Thinking control loops, Reflexion-2025, ThinkGuard constitutional monitors |
| `components/02_PLANNING_ENGINE.md` | Strategic/tactical decomposition, replanning | Plan-and-Act 2025, LangGraph 0.3, Brain-inspired MAP |
| `components/03_REASONING_CORE.md` | Test-time scaling, multi-path search, neurosymbolic traces | DeepSeek-R1, Llama 4 Maverick deliberate heads, SymCode/Scallop chains |
| `components/04_MEMORY_SYSTEM.md` | Four-memory design, retrieval and consolidation | Zep Graphiti, MemoTime, AriGraph 2 |
| `components/05_CRITIQUE_AND_VERIFICATION.md` | Process evaluation, proof obligations, auditor routing | ThinkPRM, RepV latent verifiers, Imandra Universe |
| `components/06_ACTION_EXECUTION.md` | Tool orchestration, grounding, telemetry | OpenHands 3.0, MCP, A2A v0.2 adapters |
| `components/07_WORLD_MODELS.md` | Counterfactual simulation, predictive coding | MindJourney, Llama 4 Scout predictive heads, WorldTree ensembles |
| `components/08_SAFETY_AND_ALIGNMENT.md` | Constitutional policy, adversarial defense, risk scoring | Constitutional Classifiers, RepV, OpenSafe instrumentation |
| `components/09_NEUROSYMBOLIC_INTEGRATION.md` | Differentiable logic, verified reasoning | Scallop 2.1, SymCode compilers, Imandra connectors |
| `components/10_MULTI_AGENT_COORDINATION.md` | Protocol adherence, ensemble arbitration, consensus | A2A v0.2, AgentMaster, LangGraph multi-actor topologies |
| `components/11_HUMAN_IN_THE_LOOP.md` | Oversight, preference learning, transparency tooling | RLHF 2025 corpora, DPO-Next, TransparentUI consoles |
| `components/12_LEARNING_AND_ADAPTATION.md` | Continual learning, PEFT stacks, exploration | Meta-optimizer pipelines, REMAX replay buffers, Continual LoRA |

## Annexes
- `annexes/Research_Papers_Summary.md` — Curated citation notebook covering cognitive architectures, reasoning stacks, planning, memory, neurosymbolic systems, safety, coordination, and evaluation protocols. Includes research methodology (selection criteria, search strategy, institutional analysis), evolutionary timeline (2020-2025 research phases), alternative perspectives & active debates (§14), and abandoned approaches & lessons learned (§15).
- `annexes/Research_Synthesis_2025.md` — Comprehensive 2025 research breakthroughs including DeepSeek-R1, Kimi K2, Llama 4, and other November 2025 developments. Deep-dive chronological analysis covering the test-time compute era with 90+ papers.
- `annexes/Component_Diagrams.md` — Central diagram catalog. All ASCII schematics referenced in the architecture and component files originate here.
- `annexes/Design_Decisions_Matrix.md` — Quantitative trade-off ledger mapping each architectural decision to alternatives, criteria, and rationale.

---

## Reading Orders
- **Orientation Path (≈30 min):** `README.md` → `architecture/00_MAIN_ARCHITECTURE.md` Sections 1–2 → `annexes/Component_Diagrams.md` overview plates.
- **Systems Deep Dive (3–4 h):** Full read of `00_MAIN_ARCHITECTURE.md`, followed by `components/03_REASONING_CORE.md`, `components/02_PLANNING_ENGINE.md`, `components/04_MEMORY_SYSTEM.md`, and annex citations as needed.
- **Safety and Governance Focus (2 h):** `components/01_META_COGNITIVE_SYSTEM.md` → `components/05_CRITIQUE_AND_VERIFICATION.md` → `components/08_SAFETY_AND_ALIGNMENT.md` → `annexes/Design_Decisions_Matrix.md` safety entries.
- **Research Context Path (2–3 h):** `annexes/Research_Papers_Summary.md` Research Methodology → Research Timeline (2020-2025) → Alternative Perspectives & Active Debates (§14) → Abandoned Approaches & Lessons Learned (§15) → `architecture/00_MAIN_ARCHITECTURE.md` Open Research Questions (§12). Provides complete scholarly context for architectural decisions.

## Topic Cross-References
- **Reasoning/Test-Time Compute:** `components/03_REASONING_CORE.md`, `architecture/00_MAIN_ARCHITECTURE.md` §4.3, `annexes/Research_Papers_Summary.md` §2, §14.3 (test-time compute debate).
- **Memory & Retrieval:** `components/04_MEMORY_SYSTEM.md`, `architecture/00_MAIN_ARCHITECTURE.md` §4.4, `annexes/Research_Papers_Summary.md` §3, §14.5 (memory architecture debate), §15.5 (flat memory lessons).
- **Planning & Control:** `components/02_PLANNING_ENGINE.md`, `architecture/00_MAIN_ARCHITECTURE.md` §4.2, `annexes/Research_Papers_Summary.md` §4, §15.1 (symbolic planning lessons).
- **Safety & Alignment:** `components/05` + `components/08`, `architecture/00_MAIN_ARCHITECTURE.md` §4.5 + §12.6, `annexes/Design_Decisions_Matrix.md` safety table, `annexes/Research_Papers_Summary.md` §10, §14.4 (safety debate).
- **Multi-Agent:** `components/10`, `architecture/00_MAIN_ARCHITECTURE.md` §4.7, `annexes/Research_Papers_Summary.md` §5, §14.6 (ensemble vs. specialized), `annexes/Component_Diagrams.md` §7.
- **Research Methodology & Evolution:** `annexes/Research_Papers_Summary.md` Research Methodology, Research Timeline (2020-2025), Research Landscape Analysis.
- **Architectural Alternatives:** `annexes/Research_Papers_Summary.md` §14 (active debates), `architecture/00_MAIN_ARCHITECTURE.md` §12.4 (tradeoffs), §12.10 (epistemic humility).
- **Open Research Questions:** `architecture/00_MAIN_ARCHITECTURE.md` §12 (comprehensive gaps), `annexes/Research_Papers_Summary.md` §14 (contrarian positions), §15 (failed approaches).

All documents use consistent terminology. When updating research references or diagrams, reflect the change both in the owning file and in this index to ensure discoverability.
