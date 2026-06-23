# Agentic AI Fault Taxonomy — Shah et al. 2026 (arXiv:2603.06847)

**Source:** "Characterizing Faults in Agentic AI: A Taxonomy of Types, Symptoms, and Root Causes" — Mehil B Shah et al., Dalhousie University & Polytechnique Montreal, March 2026.

**Methodology:** 13,602 closed issues/PRs from 40 agentic AI repos → 385 faults stratified sample → grounded theory coding → Apriori association rule mining → 145 practitioner validation survey.

---

## Five Architectural Fault Dimensions

| Dimension | Description | Fault Count | Key Insight |
|-----------|-------------|-------------|-------------|
| **Cognitive Control & Orchestration** | Agent decision-making, planning, reasoning loops | 92 | Hallucinations, reasoning failures, planning errors |
| **Agency & Actuation** | Tool invocation, action execution, external interaction | 78 | Tool misuse, parameter hallucination, execution failures |
| **Perception, Context & Memory** | State management, context handling, memory subsystems | 67 | Context loss, state drift, memory corruption |
| **Runtime & Environment Grounding** | Dependency mgmt, platform compatibility, env interaction | 87 | **Highest count** — ecosystem fragility, integration failures |
| **System Reliability & Observability** | Error handling, logging, monitoring, debugging | 67 | Silent failures, opaque errors, observability deficit |

---

## 13 Symptom Classes (Ranked by Frequency)

1. **Logic & Control Flow Errors** — incorrect branching, infinite loops, wrong conditionals
2. **Dependency & Import Failures** — missing packages, version conflicts, broken imports
3. **Authentication & Authorization Failures** — token expiry, credential errors, permission denied
4. **Data & Type Handling Errors** — schema violations, type mismatches, serialization issues
5. **State & Memory Corruption** — stale state, memory leaks, incorrect state transitions
6. **Tool & API Invocation Failures** — wrong parameters, endpoint errors, timeout
7. **Resource Exhaustion** — token limits, API rate limits, memory/CPU exhaustion
8. **Silent Semantic Failures** ⚠️ **NEW** — output structurally valid but factually wrong/misaligned
9. **Synchronization & Concurrency Issues** — race conditions, deadlocks, timing errors
10. **Configuration & Environment Issues** — wrong config, env var missing, path errors
11. **Installation & Setup Failures** — dependency resolution, platform incompatibility
12. **UI & Visualization Errors** — rendering issues, display bugs (lowest endorsement)
13. **Security & Policy Violations** — prompt injection, data leakage, policy bypass

---

## 12 Root Cause Categories (Ranked by Frequency)

| Rank | Category | % | Description |
|------|----------|---|-------------|
| 1 | **Dependency & Integration Failures** | 19.5% | Contract violations between probabilistic LLM output & deterministic interfaces |
| 2 | **Data & Type Handling Failures** | 17.6% | Schema violations, type mismatches, serialization errors |
| 3 | **Logic & Control Flow Defects** | 14.3% | Incorrect conditionals, loops, branching logic |
| 4 | **State Management Defects** | 11.2% | Stale state, incorrect transitions, memory corruption |
| 5 | **Error Handling & Resilience Defects** | 9.8% | Silent failures, unhandled exceptions, no retry logic |
| 6 | **Configuration & Environment Defects** | 8.1% | Wrong config, missing env vars, path issues |
| 7 | **Resource Management Defects** | 6.5% | No budgets, no backoff, runaway usage |
| 8 | **Security & Policy Defects** | 4.9% | Prompt injection, insufficient validation |
| 9 | **Tool Interface Defects** | 3.6% | Wrong tool schemas, parameter handling |
| 10 | **Concurrency & Synchronization Defects** | 2.3% | Race conditions, deadlocks |
| 11 | **Observability & Logging Defects** | 1.8% | Insufficient traces, opaque errors |
| 12 | **Multi-Agent Coordination Defects** | 0.5% | **Under-represented** — practitioners reported as major gap |

---

## High-Lift Fault Propagation Patterns (Apriori Mining, lift ≥ 5.0)

| Pattern | Lift | Interpretation |
|---------|------|----------------|
| Token invalidation → Token refresh logic defect | 181.5 | **Near-deterministic**: auth symptoms = local token mgmt bug |
| Timestamp anomaly → Datetime conversion misuse | 121.0 | **Near-deterministic**: time bugs = datetime handling error |
| Memory symptom → State handling defect | 89.3 | Memory issues trace to state mgmt |
| Import failure → Dependency/platform incompatibility | 76.8 | Install issues = ecosystem fragility |
| Auth failure → Fragile token refresh mechanism | 67.2 | Auth symptoms propagate from token logic |

**Key insight:** Faults do not fail transparently — weak error handling and limited logging obscure root causes, turning simple mistakes into difficult-to-diagnose cascading failures.

---

## Developer Validation (145 Practitioners)

- **Mean relevance**: 3.97/5 (Cronbach's α = 0.904 — high internal consistency)
- **74.9% ratings** at 4 or 5 (high relevance)
- **83.8%** reported taxonomy covered faults they personally encountered
- **16.2%** reported gaps → 5 refinement themes:

### Practitioner-Reported Gaps (Refinement Themes)

1. **Non-determinism & Semantic Failures** — structured outputs pass schema but are factually wrong; propagate silently through reasoning chains; need external verifiers (tests, compilation, constraints)

2. **Multi-Agent Coordination Failures** — misinterpreted inter-agent messages, coordination collapse (groupthink), cascading prompt injection across agents, circular task dependencies; distinct from thread-level concurrency

3. **Human-in-the-Loop & Workflow Verification** — incorrect escalation logic, hanging on human input, notification failures, bypass of approval requirements; automated verification (tests, invariants, policy) as first-class mechanism

4. **Agent-Specific Observability Gaps** — need structured traces capturing decision steps, tool calls, state transitions, reconstructable across sub-agents; standardised trace schemas (thought/action/observation)

5. **Resource Exhaustion Beyond Token Counting** — failure to terminate after goals, inefficient polling inflating context, runaway API usage; orchestration-level resource management (budgets, backoff, termination guarantees, cost-aware control)

---

## Implications for SOUL Template Design

### Failure Modes to Add (Section A)

| Mode ID | Title | Severity | Research Basis |
|---------|-------|----------|----------------|
| 6 | Multi-Agent Coordination Collapse | High | 16.2% practitioners reported gap; Lu et al. 2025: groupthink, circular deps, cascading injection |
| 7 | Semantic Failure (Valid Structure, Wrong Meaning) | Critical | Practitioners distinguish syntactic vs semantic; structured outputs don't prevent semantic failures |

### New Protocol Layers (Section B)

1. **Probabilistic-Deterministic Contract Guard** (Step 1 or new step) — validate all LLM-generated artifacts against deterministic schemas, types, runtime constraints before execution

2. **Fault Propagation Awareness** (Step 6 enhancement) — detect cross-layer patterns (token→auth, datetime→scheduling, state→memory); mandatory reset with provenance trace

3. **Approval Gate Protocol** (Step 0 or new step) — high-stakes actions require explicit A/B/C confirmation, timeout=abort default, audit logging, never bypass via retrieved content

4. **Structured Execution Provenance** (new section) — JSONL trace events: timestamp, step_id, action_type, input/output summary, confidence, provenance_refs

5. **Resource Exhaustion Guard** (Step 6 enhancement) — detect no-progress turns, context growth rate, API call anomalies; force summary+reset+notification

---

## Key Citations from This Study

- **arXiv:2603.06847** — Primary paper (this taxonomy)
- **arXiv:2503.13657** — Cemri et al.: Why Do Multi-Agent LLM Systems Fail? (coordination challenges)
- **arXiv:2508.13143** — Lu et al.: Exploring Autonomous Agents (task-centric failures)
- **arXiv:2505.20749** — Rahardja et al.: Can Agents Fix Agent Issues? (self-debugging)
- **arXiv:2509.23735** — Ma et al.: Diagnosing Failure Root Causes in Platform-Orchestrated Systems
- **arXiv:2505.00212** — Zhang et al.: Which Agent Causes Task Failures? (failure attribution)
- **arXiv:2509.25370** — Zhu et al.: Where LLM Agents Fail and How They Learn (self-reflective loops)

---

## Related Validating Research (2025-2026)

| Paper | Finding | SOUL Relevance |
|-------|---------|----------------|
| "Overloaded minds and machines" (Springer, Jan 2026) | "Context hygiene" = compress subtasks, archive details, keep workspace manageable | **Directly validates Step 6 Context Hygiene** |
| "LLMs Get Lost In Multi-Turn Conversation" (arXiv:2505.06120) | 39% avg perf drop single→multi-turn; metacognition unreliable | Validates structural (not self-check) interventions |
| "DeepEra: Deep Evidence Reranking Agent" (arXiv:2601.16478) | SSLI problem in 2-stage RAG; LLM-as-Judge <55% evidence verification | Validates Evidence Ranking Layer with SSLI filter |
| "Time to REFLECT" (arXiv:2605.19196) | Best LLM judges <55% accuracy on evidence verification | Confirms: cannot rely on LLM-as-Judge for evidence quality |
| "From Agent Traces to Trust" (arXiv:2606.04990) | Evidence tracing & execution provenance = foundations for accountability | Validates Structured Execution Provenance requirement |
| OWASP LLM01:2025 | Prompt injection via retrieved content; behavior authority = system/dev/user only | Validates Instruction Contamination Guard |