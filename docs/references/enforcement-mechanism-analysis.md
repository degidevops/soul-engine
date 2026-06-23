# Enforcement Mechanism Analysis — Hermes Agent

**Last updated:** 2026-06-17

## The Core Problem

SOUL.md (system prompt) is a **nudge**, not a **constraint**. The model can still:
- Ignore "MANDATORY web_search" and answer from internal knowledge
- Self-assess confidence unreliably
- Search once, claim "no results," fallback to internal
- Favor parametric knowledge due to similarity bias

**Source:** arxiv 2503.10996, 2506.06485, 2601.03746, 2606.00251

## What `tool_use_enforcement` Actually Does

**Config:** `agent.tool_use_enforcement` in `~/.hermes/config.yaml`
**Source code:** `agent/system_prompt.py`, `agent/prompt_builder.py`

**Behavior:**
- `true` -> inject `TOOL_USE_ENFORCEMENT_GUIDANCE` for ALL models
- `false` -> never inject
- `"auto"` (default) -> inject only for models in `TOOL_USE_ENFORCEMENT_MODELS`
- `list` -> custom model-name substrings

**Verdict:** This is GENERIC tool-use enforcement. It prevents "describing plans instead of executing." It does NOT mandate `web_search` as first tool call.

**Current user config:** `tool_use_enforcement: true` — already active, but does NOT solve internet-first.

## SOUL.md Loading Pipeline

1. `_r.load_soul_md()` reads `~/.hermes/SOUL.md`
2. Content -> `stable_parts[0]` — slot #1 in system prompt
3. SOUL.md has highest prompt priority — but still just prompt

## AGENTS.md / CLAUDE.md Injection

Hermes auto-loads `AGENTS.md` and `CLAUDE.md` from working directory. Still just prompt — model can ignore.

## Enforcement Options (Ranked)

- **Option A: Modify source code guidance** (Strength: Strong | Effort: High)
- **Option B: Wrapper script (intercept -> search -> inject)** (Strength: Strong | Effort: Medium)
- **Option C: AGENTS.md with search-first protocol** (Strength: Medium | Effort: Low)
- **Option D: Optimize SOUL.md only** (Strength: Weak | Effort: Low)

## Key Insight

**Prompt + Enforcement = Real compliance.**

When user asks for "internet-first" behavior:
1. Update SOUL.md (prompt layer) — necessary but not sufficient
2. Ask: "Do you also need framework-level enforcement?"
3. If yes, implement Option B or A
