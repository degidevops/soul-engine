# SOUL V7.3-ORCH vs V7 Base — Complete Diff

> **Purpose:** Document structural differences between worker (V7) and orchestrator (V73) templates.
> **Source:** `SOUL_TEMPLATE_ORCHESTRATOR_V73.md` + `SOUL_TEMPLATE_V7.md`
> **Last updated:** 2026-06-22

---

## When to Use Which Template

| Criteria | V7 (Worker) | V7.3 (Orchestrator) |
|----------|-------------|---------------------|
| **Primary role** | Execute, implement, research | Coordinate, route, decompose |
| **Toolset focus** | `terminal`, `file`, `web` | `kanban_*`, board tools |
| **Task pattern** | Single-agent execution | Fan-out to workers |
| **Persistence** | Session-bound | Survives crash/restart (SQLite) |
| **"Don't do the work yourself"** | N/A | YES — enforced |

**Rule of thumb:** If the profile uses `kanban_list`/`kanban_create` as primary tools → ORCHESTRATOR. If it uses `terminal`/`file`/`web_search` → V7.

---

## Structural Differences

### Steps (Reasoning Protocol)

| Step | V7 (Worker) | V7.3 (Orchestrator) | Purpose |
|------|-------------|---------------------|---------|
| 0 | Intent Validation | Intent Validation + profile verification | Same + verify worker profiles exist |
| 1 | Factually-Triggered Search | Factually-Triggered Search + Semantic Grounding | Same + value-level cross-reference |
| 2 | Conflict Resolution | **Control Plane — VMAO verification loop + HERA adaptive topology** | V7.3: verification-driven replanning + adaptive topology optimization |
| 3 | Tool Use Protocol | **Decomposition — AOrchestra (I,C,T,M) tuple + granularity metrics** | V7.3: tuple concretization + explicit granularity metrics |
| 4 | Reasoning Integrity | Conflict Resolution + board trust | V7.3: board state = operational truth |
| 5 | Anti-Hallucination Gate | Tool Use Protocol — Orchestrator-scoped | V7.3: explicit tool restrictions + delegate_task boundary |
| 6 | Context Hygiene | Reasoning Integrity | Same |
| 7 | Human-in-the-Loop Gate | Anti-Hallucination Gate + decomposition check | V7.3: + verify assignee/DAG/criteria |
| 8 | Structured Provenance | **Context Hygiene + 6 anchors** | V7.3: + kanban anchor + decompose anchor |
| 9 | — | **Human-in-the-Loop Gate** | Extended gate (new scope) |
| 10 | — | **Structured Provenance + Kanban trace** | V7.3: + A2A Protocol mapping |

### Failure Modes (Section A)

| Count | V7 (Worker) | V7.3 (Orchestrator) |
|-------|-------------|---------------------|
| Total | 7 (IDs 1-7) | 12 (IDs 1-12) |
| Mode 8 | — | Orchestrator Executing Worker Tasks (CRITICAL) — cites AOrchestra + Dennis et al. |
| Mode 9 | — | Ignoring Worker Block Signals (HIGH) — cites HERA + Shah et al. + VMAO |
| Mode 10 | — | Context Pollution into Workers (HIGH) — cites Orchestral AI + GitHub #26730 |
| Mode 11 | — | Over/Under Decomposition (MEDIUM) — cites VMAO + AOrchestra |
| Mode 12 | — | Silent Worker Failure (MEDIUM) — cites GitHub #35096 + Adimulam et al. |

### Knowledge Priority (Section B)

| Rank | V7 (Worker) | V7.3 (Orchestrator) |
|------|-------------|---------------------|
| 1 | Live internet | Live internet |
| 2 | User-provided files | **Kanban Board (kanban.db)** — operational truth source |
| 2.5 | — | User-provided files |
| 3 | Persistent memory | Persistent memory |
| 4 | Internal parametric knowledge (disabled) | Internal parametric knowledge (disabled) |

### Anchor Re-Injection (Step 8)

| Anchor | V7 (Worker) | V7.3 (Orchestrator) |
|--------|-------------|---------------------|
| core | ✅ | ✅ |
| safety | ✅ | ✅ |
| semantic | ✅ | ✅ |
| scope | ✅ | ✅ (orchestrator-specific content) |
| kanban | — | ✅ New |
| decompose | — | ✅ New |

### Toolset (Section A → `<tools>`)

| Tool | V7 (Worker) | V7.3 (Orchestrator) |
|------|-------------|---------------------|
| `kanban_list` | ❌ | ✅ Orchestrator primary |
| `kanban_create` | ❌ | ✅ + tuple concretization note |
| `kanban_show` | ❌ | ✅ + read-before-intervention note |
| `kanban_complete` | ❌ | ❌ **Explicit exclusion** (Mode 8 warning) |
| `kanban_block` | ❌ | ❌ **Explicit exclusion** |
| `kanban_heartbeat` | ❌ | ❌ **Explicit exclusion** |
| `kanban_unblock` | ❌ | ✅ + diagnose-first protocol |
| `kanban_link` | ❌ | ✅ + VMAO DAG pattern note |
| `kanban_comment` | ❌ | ✅ + board-only coordination note |
| `delegate_task` | ✅ | ✅ Orchestration-level only (explicit boundary) |

### Section C (System Context)

| Element | V7 (Worker) | V7.3 (Orchestrator) |
|---------|-------------|---------------------|
| Position | `{{SYSTEM_POSITION}}` (generic) | `{{SYSTEM_POSITION}}` (functional description, 1 sentence) |
| Constraints | `{{KEY_CONSTRAINTS}}` (generic placeholder) | Pre-filled with boundary-redirect pattern |
| Escalation | `{{ESCALATION_PROTOCOL}}` (generic) | Pre-filled with standard + orchestration-specific triggers |
| Resources | `{{AVAILABLE_RESOURCES}}` (generic) | Pre-filled with kanban tools, worker profiles, workspace, skills |
| Reporting | `{{REPORTING_FORMAT}}` (generic) | Pre-filled with board state, decomposition DAGs, worker liveness |

---

## V7.3 Key Upgrades Over V7.2-orch

1. **Step 2: Control Plane** — VMAO verification-driven replanning loop + HERA adaptive topology optimization
2. **Step 3: Decomposition** — AOrchestra (I,C,T,M) tuple concretization + explicit granularity metrics
3. **Failure Modes 8-12** — Strengthened with arXiv paper citations (not just Hermes SKILL.md)
4. **Kanban Board** — Promoted from Knowledge Priority 2.5 → Level 2 (operational truth source)
5. **Anchors** — 6 anchors (added kanban + decompose)
6. **Tool Exclusions** — Explicit `exclusion="orchestrator_forbidden"` annotations with Mode 8 violation warning
7. **Section C Position** — Metadata → functional description (1 sentence, boundary-redirect)
8. **Section C Constraints** — Positive-first → negative-first (boundary-redirect pairing)
9. **Token Budget** — Added platform_toolsets restriction + ~5K token target
10. **delegate_task Boundary** — Explicit distinction: function call vs durable queue
11. **Triage/Todo Columns** — Added to orchestrator scope and control plane protocol
12. **Workspace Selection** — Added workspace type specification (scratch/dir:/worktree)
13. **Skill Audit** — Added skill evolution + audit recommendation
14. **Worker Liveness** — Added to reporting requirements
15. **Context Compression** — Added to escalation and reporting

---

## Research Sources (V7.3)

| Paper | arXiv | Key Contribution | Maps to |
|-------|-------|------------------|---------|
| VMAO (Zhang et al., 2026) | 2603.11445 | Verification-driven adaptive replanning | Step 2, Mode 9, Mode 11 |
| AOrchestra (Ruan et al., 2026) | 2602.03786 | Agent tuple (I,C,T,M) abstraction | Step 3, Mode 8, Mode 11 |
| HERA (Li & Ramakrishnan, 2026) | 2604.00901 | Adaptive orchestration + role-aware prompts | Step 2, Mode 9 |
| Orchestral AI (Roman, 2026) | 2601.02577 | Workspace sandboxing, modular architecture | Mode 10 |
| Adimulam et al. (2026) | 2601.13671 | MCP + A2A protocols, observability | Step 10, Mode 12 |
| Dennis et al. (2026) | 2604.27891 | In-context vs orchestration (nuanced) | Mode 8 |
| Shah et al. (2026) | 2603.06847 | Agentic AI fault taxonomy | Modes 6-12 |
| TaskWeave (Tan et al., 2026) | 2606.01199 | Hierarchical intent propagation, FPDA cycle | Position format, delegation graph |
| Single-Agent with Skills (Li, 2026) | 2601.04748 | Token budget, skill selection degradation | Token budget section |
| System Prompt Attack Surface (Litvak, 2026) | 2603.25056 | System prompt = security variable | Negative constraints, position as boundary |

---

*Document compiled: 2026-06-22*
*Sources: 10 arXiv papers + Reddit r/hermesagent + r/better_claw + Hermes docs + existing profiles*
