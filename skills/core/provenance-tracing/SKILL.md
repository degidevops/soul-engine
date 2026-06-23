---
name: provenance-tracing
description: "Binary provenance verification (VERIFIED/UNVERIFIED/CONFLICTING) replaces LLM self-reported confidence. Full JSONL trace schema spec, provenance→confidence rosetta, integration with V73 Step 10 kanban_trace. Replaces V7 Step 8 + V73 Step 10 shared portion (~25 baris combined)."
version: 1.0.0
author: Maru
license: MIT
tags: [provenance, binary-verification, observability, a2a-protocol, hermes-agent, soul-engine]
metadata:
  hermes:
    parent_skill: soul-engine/core
    related: [grounded-research-protocol, failure-mode-recovery-core]
    replacement_for:
      - V7_template: Step 8 (lines 404-420)
      - V73_template: Step 10 shared portion (lines 692-716, excluding kanban_trace)
    pointer_syntax: |
      <step n="8 or 10">
        <name>Structured Execution Provenance — Binary Verification</name>
        <trigger>every tool call, every structured output, every multi-step task</trigger>
        <action>load skill 'provenance-tracing'</action>
        <anchor>[ANCHOR: Provenance status (VERIFIED/UNVERIFIED/CONFLICTING) replaces confidence.]</anchor>
      </step>
    retention_V73: "<kanban_trace> spec retained inline in V73 template (couples to dispatcher state model)"
---

# Provenance Tracing

Loader mode: **ONDEMAND**. Skill provides JSONL trace schema + verification rules. V73 retains inline `<kanban_trace>` element (~7 baris) due to coupling with dispatcher state model.

## When to Load
- Before emitting structured output (JSON, code, API params).
- After any tool call or external state read.
- For multi-step orchestration decisions.
- During audit/diagnosis of agent behavior.

## When NOT to Load
- Pure creative tasks without external retrieval.
- Single-turn trivial conversation.

---

## Axiom

LLM has **NO self-awareness**. "Confidence score" from the model itself is **unreliable** — it is a statistical token probability, not a measurement of truth.

This is not metaphor. The model produces "confidence" as part of its next-token probability distribution; this correlates poorly with actual accuracy (overconfidence problem).

**Replacement**: Binary provenance. Can this claim be traced to a specific tool output or source passage in current context? Yes/No/Conflict.

---

## Provenance Statuses (Decision Vocabulary)

| Status | Definition | When to Assign |
|--------|------------|----------------|
| **VERIFIED** | Claim has direct citation to tool output or retrieved source, AND value matches source data (cross-referenced in semantic grounding) | Body claim ↔ tool output alignment confirmed |
| **UNVERIFIED** | No source found, OR value cannot be traced to specific passage | Fallback: parametric knowledge claim |
| **CONFLICTING** | Multiple sources disagree | Multiple sources retrieved; resolution needed |

### V73 Extension: Kanban-Board Provenance

For orchestration facts:
- **Kanban board state (via kanban_list) = VERIFIED** for operational facts.
- Worker output via file_read = VERIFIED (with cross-reference).
- External research = UNVERIFIED until semantic grounding alignment.

---

## Confidence → Provenance Rosetta

| LLM Self-Reported | Provenance Replacement |
|-------------------|------------------------|
| "I'm 95% confident..." | VERIFIED + citation to source |
| "I think..." / "Probably..." | UNVERIFIED (explicit flag) |
| "I believe..." / "My understanding..." | UNVERIFIED |
| "I'm not sure..." | UNVERIFIED |
| "Different sources say X vs Y" | CONFLICTING + present both |
| Internal knowledge (any confidence level) | UNVERIFIED |

**Engraved rule**: Internal knowledge ALWAYS = UNVERIFIED. Only external sources determine status.

---

## JSONL Trace Schema (V7)

```json
{
  "timestamp": "ISO 8601",
  "step_id": "Step N (0-8 for V7, 0-10 for V73)",
  "action_type": "intent_assessment | search_query | evidence_ranking | reasoning | tool_call | deliver",
  "input_summary": "...",
  "output_summary": "...",
  "provenance_status": "VERIFIED | UNVERIFIED | CONFLICTING",
  "source_refs": ["url1", "url2", "tool_output_id"]
}
```

## JSONL Trace Schema (V73 — adds kanban_task_ids)

```json
{
  "timestamp": "ISO 8601",
  "step_id": "Step N (0-10)",
  "action_type": "intent_assessment | search_query | kanban_op | decomposition | worker_assignment | conflict_resolve | approval_gate | provenance",
  "input_summary": "...",
  "output_summary": "...",
  "provenance_status": "VERIFIED | UNVERIFIED | CONFLICTING",
  "source_refs": ["url1", "url2", "tool_output_id"],
  "kanban_task_ids": ["task-1", "task-2"]
}
```

### V73 Trace Action Types (orchestrator-specific)

- `intent_assessment` — Step 0
- `search_query` — Step 1
- `kanban_op` — Steps 2-5 (board operations)
- `decomposition` — Step 3
- `worker_assignment` — Step 3
- `conflict_resolve` — Step 4
- `approval_gate` — Step 9
- `provenance` — Step 10 (self)

---

## V73 Inline Retention: `<kanban_trace>`

These ~7 baris remain in V73 template (NOT extracted) due to coupling with A2A protocol framework (Adimulam 2601.13671):

```xml
<kanban_trace>
  All kanban_* tool calls are inherently traceable via task history.
  For each orchestration operation, log:
    task_id, action (create/show/link/unblock/comment), resulting state, trigger reason.
  This enables replay and diagnosis across the multi-agent workflow.
</kanban_trace>
```

**Rationale for retention**: Mixing dispatcher state model (ad-hoc) with skill content (modular) defeats the orchestration accountability framework. The 7 baris are coupled to Hermes dispatcher behavior, not optional protocol.

---

## Trace Requirement (V7 + V73)

| Profile | Trigger Events |
|---------|---------------|
| V7 worker | intent assessment (S0), search queries (S1), evidence rankings (S1), reasoning steps, tool calls, provenance_status |
| V73 orchestrator | intent (S0), search (S1), kanban operations (S2-S5), decomposition decisions (S3), worker assignments (S3), conflict resolutions (S4), approval gate decisions (S9), provenance |

---

## Research Basis

- **arXiv 2603.06847**: Standardized trace schemas needed for replay and diagnosis. LLM self-reported confidence correlates poorly with actual accuracy.
- **Adimulam et al. 2026 (arXiv 2601.13671)**: Agent2Agent (A2A) Protocol governs peer coordination. Structured traces = enterprise observability foundation.
- **arXiv 2606.04990**: Evidence tracing + execution provenance = foundation for process-level accountability in trustworthy LLM agents.
- **OWASP LLM09:2025** (overreliance): mitigation requires verification, not trust.

---

## Cross-Skill Dependencies

- `grounded-research-protocol` — feeds VERIFIED with sources; UNVERIFIED triggers re-search.
- `failure-mode-recovery-core` — fires when M7 semantic-failure detected during trace audit.
- `context-hygiene` — runs trace audit pre-distillation to preserve provenance during cleanup.

---

## Validation Self-Test

```bash
# Confirm V73 retains inline kanban_trace (NOT extracted)
grep -E "<kanban_trace>" ~/.hermes/skills/devops/soul-engine/templates/SOUL_TEMPLATE_ORCHESTRATOR_V73.md

# Confirm V7 has compact pointer
grep -E "skill_view\(.provenance-tracing" ~/.hermes/skills/devops/soul-engine/templates/SOUL_TEMPLATE_V7.md

# Confirm skill loaded
skill_view('provenance-tracing')
```

---

## Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-06-23 | Initial extraction. V7 Step 8 fully extracted; V73 Step 10 split (shared provenance model + Extended JSONL extracted; kanban_trace retained inline in template). |
