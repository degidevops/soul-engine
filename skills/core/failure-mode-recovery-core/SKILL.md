---
name: failure-mode-recovery-core
description: "Recovery playbooks for the 7 universal agent failure modes (M1-M7) extracted from V7/V73 SOUL templates. Loaded on-demand when a fault signal matches any of M1-M7. Provides diagnostic steps + recovery actions + research-basis citations. Worker/Orchestrator shared."
version: 1.0.0
author: Maru
license: MIT
tags: [failure-modes, recovery, ground-truth, hermes-agent, soul-engine]
metadata:
  hermes:
    parent_skill: soul-engine/core
    related: [soul-management, soul-md-decomposition-research]
    replacement_for:
      - V7_template: <failure_modes> mode id 1-7 (lines 48-104)
      - V73_template: <failure_modes> mode id 1-7 (lines 97-153)
    pointer_syntax: |
      <failure_modes>
        <reference>skill://failure-mode-recovery-core</reference>
        <minimal_index>... See template usage ...</minimal_index>
      </failure_modes>
---

# Failure-Mode-Recovery-Core (M1-M7)

Loader-side mode: **ONDEMAND**. The 7 universal failure modes apply to every profile deployed from V7 or V73 templates. This skill provides the full diagnostic + recovery playbooks so they do not need to live in template body.

## When to Load
- Fault signal detected (any): repeated tool errors of same class, cascading failures, capability mismatch, semantic mismatch.
- Manual audit: when reviewing a profile's behavior drift.
- New profile deployment: bootstrap reference for failure recognition.

## When NOT to Load
- Reading template structure (use template docs instead).
- Configuring tools (template-level concern).

---

## Mode Index (Quick Reference)

| ID | Priority | Title | Trigger | Retry Policy |
|----|----------|-------|---------|--------------|
| M1 | critical | Answering Without Searching | factual claim without web_search | no_retry |
| M2 | critical | Internal Knowledge as Fact Source | parametric knowledge presented as verified | no_retry |
| M3 | critical | Silent Parametric Fallback | search failed → substituted with internal knowledge | no_retry |
| M4 | high | Silent Intent Guessing | executed on ambiguous input without clarification | clarification_loop |
| M5 | high | Snapshot-Based Extraction | used browser snapshot for content extraction | use_camofox_eval_or_html |
| M6 | high | Multi-Agent Coordination Collapse | misinterpreted inter-agent messages, cascading prompt injection | structured_protocol_with_timeouts |
| M7 | critical | Semantic Failure | structured output passes schema but factually wrong | external_verifier_required |

> Template-side `<minimal_index>` block retains 1-line summaries of these 7 entries only. M8-M12 (orchestrator-specific) live in V73 template inline.

---

## Theory of Operation

These 7 modes share a common **diagnose → intervene → verify** lifecycle codified in precedent literature:

- **Shah et al. (arXiv:2603.06847)** — Agentic AI fault taxonomy across 13,602 issues, 40 repos. Established the 19.5% dependency/integration failures, 17.6% data/type handling, 16.2% multi-agent coordination, 11.2% state management, 1.8% observability/logging root causes.
- **OWASP LLM01:2025** — Instruction contamination vector formalized for retrieval-augmented agents.
- **Lu et al. 2025** — Coordination collapse patterns (groupthink, circular dependencies, cascading injection).
- **MDPI 2025** — "Probabilistic predictions, not verified facts" — model cannot self-detect knowledge boundary.

The 7 modes are grounded in this empirical fault surface. Diagnostic steps below are derived from these papers, not heuristic guesses.

---

## Diagnostic + Recovery Playbooks

### M1: Answering Without Searching

**Diagnostic signals:**
- Self-claim of factual content (dates, names, prices, API versions, code syntax) without preceding `web_search` call.
- High confidence-tone language ("X is Y", "Z costs $N") without citation.

**Recovery:**
1. STOP current generation mid-sentence.
2. Issue: `web_search(query=<claim subject>)` immediately.
3. Re-emit the claim WITH citation.
4. Append note: `[Corrected: previously stated without source. Verified via web_search now.]`

**Research basis:** arXiv 2604.09515 — 25-38% deprecated API usage from stale parametric knowledge.

### M2: Internal Knowledge as Fact Source

**Diagnostic signals:**
- Tonal shift from citations to confident assertion.
- Absence of `web_search` tool calls in recent context despite factual claims.

**Recovery:**
1. Flag: `[Internal knowledge — unverified, may be outdated]`.
2. Issue: `web_search` to validate.
3. If validated: re-emit with citation. If contradicted: emit per **M3 fallback protocol**.
4. Higher-stakes: trigger **M7 verification** layer.

**Research basis:** arXiv 2505.15962 + MDPI 2025.

### M3: Silent Parametric Fallback

**Diagnostic signals:**
- `web_search` returned empty/error; response continued without acknowledgment.
- Lack of explicit "no source found" disclosure in user-facing reply.

**Recovery:**
1. Hard STOP — DO NOT emit any factual claim.
2. Explicit user-facing disclosure: `"No reliable external sources found for [query]. Returning UNVERIFIED."`
3. If claim is critical: return to **Step 0 (Intent Validation)** to renegotiate intent — never guess.
4. Never silently substitute internal knowledge.

**Research basis:** arXiv 2603.06847 (compound failure modes) + Shah et al. fault taxonomy.

### M4: Silent Intent Guessing

**Diagnostic signals:**
- User query had ambiguity (multiple plausible interpretations).
- Agent proceeded without clarification.
- Lower-than-expected output quality vs. simple restate of intent.

**Recovery:**
1. Halt execution.
2. Use `clarify` tool with A/B/C options.
3. Identify missing parameters: audience, stack, goal, deadline.
4. Re-issue intent contract: `"I will orchestrate X, using Y, to achieve Z. Confirmed?"`

**Severity note:** This is HIGH priority but also an opportunity for prevention better than cure — Intent validation (Step 0) preceding Execution (Step 1+) is built into the template.

### M5: Snapshot-Based Extraction

**Diagnostic signals:**
- Snapshots used for content extraction from web pages.
- Content visibly truncated or missing details.

**Recovery:**
1. Rethrow with hierarchy:
   - `camofox_evaluate_js` (SearXNG backend) OR
   - `web_extract` (non-SearXNG backend) OR
   - `camofox_get_page_html` (full context, last resort).
2. Re-extract using chosen method.
3. Validate content matches expected scope before continuing.

**Engraved rule:** Snapshots truncate content; they are for snapshot-of-state only, never for content extraction. The extraction hierarchy is fixed (see `grounded-research-protocol` skill).

### M6: Multi-Agent Coordination Collapse

**Diagnostic signals:**
- Inter-agent messages misinterpreted, even with explicit context.
- Groupthink: multiple agents converging on wrong consensus.
- Circular dependencies in task graph.
- Cascading prompt injection across agents.

**Recovery:**
1. Apply **structured_protocol_with_timeouts**:
   - Establish explicit message contracts (typed I/O).
   - Add per-message timeout (no infinite delegation loops).
   - Insert verification checkpoints between agent hops.
2. For Kanban orchestrator: re-read `kanban_list`, validate task graph is DAG.
3. Reset coordination: clear accumulated miscommunication context.
4. Add provenance trace per Step 10 protocol.

**Research basis:** arXiv 2603.06847 (16.2% coordination failure rate) + Lu et al. 2025.

### M7: Semantic Failure (Valid Structure, Wrong Meaning)

**Diagnostic signals:**
- Output passes schema/syntax check.
- Output is factually incorrect or misaligned with ground truth.
- High syntactic compliance + low semantic accuracy (Google models/Codex-prone).

**Recovery:**
1. **External verifier required** — never self-assess semantic correctness.
2. Cross-reference EVERY value in structured output against raw tool output or retrieved content passage.
3. If value cannot be traced: FLAG as "unverified", do NOT silently include.
4. Format correctness ≠ factual correctness. A valid JSON with wrong data is FAILURE.
5. Google-specific: dense models (Gemma 4 31B) require explicit value-level verification.

**Engraved rule:** Semantic grounding is required AFTER every structured output generation. See `grounded-research-protocol` for the spec.

**Research basis:** arXiv 2603.06847 (semantic vs. syntactic distinction) + TMLS NYC 2026.

---

## Cross-Skill Dependencies

- `grounded-research-protocol` — handles M1, M2, M3 retrieval semantics.
- `context-hygiene` — handles fault-propagation detection (signals M3, M6, M7).
- `provenance-tracing` — provides the JSONL spec required by M6 recovery (line 562-577 area in M6 playbook).

When fault signal detected, this skill is entry point. After playbook selection, it may delegate to one of the three cross-skill deps.

---

## Pitfalls When Using This Skill

1. **Don't auto-trigger M4 correction during fast multi-turn** — first turn might be exploratory. Only apply after observed pattern.
2. **M3 hard-stop often feels premature** — but emitting UNVERIFIED is cheaper than delivered-wrong answer. Trust the protocol.
3. **M6 coordination collapse can masquerade as M4 intent ambiguity** — diagnostic test: if other agents are involved, switch to M6 first.
4. **M7 semantic failure often invisible to self-check** — DO NOT skip external verification because "looks right to me". Always run cross-reference.

---

## Validation Self-Test (Run Per Profile Deploy)

```bash
# Confirm skill is loaded
skill_view('failure-mode-recovery-core')

# Confirm anchor IDs are consistent with template
grep -E "M[1-7]" ~/.hermes/skills/devops/soul-engine/core/failure-mode-recovery-core/SKILL.md

# Confirm template references resolve
grep -E "skill_view\(.failure-mode-recovery-core" ~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_*.md

# Verify minimal_index block present in both templates
grep -E "minimal_index" ~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_*.md
```

Expected: skills + templates mutually reference M1-M7 identifiers.

---

## Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-06-23 | Initial extraction from V7 line 48-104 + V73 line 97-153 (M1-M7 only). M8-M12 remain in V73 template inline. |

Next planned: extend M8-M12 into orchestrator-only skill IF Issue #2045 lazy-loading proves stably resolved.
