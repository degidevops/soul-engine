# V7 Strict Internet-First Grounding Protocol

## Problem Statement

Standard LLM behavior defaults to using **parametric knowledge** (weights learned during training) as the primary source of truth. This causes:
- **Hallucinations**: Confident-sounding but fabricated answers
- **Staleness**: Training data cutoff means knowledge is outdated (25-38% deprecated API usage — arXiv 2604.09515)
- **Overconfidence**: Model "knows" things it only statistically guessed during training

## Solution: Factually-Triggered Search Protocol

The V7 protocol uses **Factually-Triggered Search** — objective trigger (IS THIS A FACTUAL CLAIM?), not self-judgment ("am I sure?").

### Why Not Binary Always vs Selective?

Research (2025-2026) shows this is a **false dilemma**:

| Approach | Problem |
|----------|---------|
| Always retrieve | Context pollution, over-retrieval degrades 13.9%-85% (arXiv 2510.05381) |
| Self-judged selective | Metacognition unreliable (OpenReview 2025, arXiv 2026) |
| ✅ Factually-triggered | Objective trigger + quantity limit = optimal |

## Architecture

### 1. Factually-Triggered Retrieval (Step 1 in SOUL)

**Trigger**: IS THIS A FACTUAL CLAIM? (objective, not self-judgment)

**Search for ALL factual claims:**
- Data, dates, names, prices, stats, who/what/when/where
- API versions, code syntax, library changes
- Definitions, concepts, "well-established facts" (knowledge overshadowing risk — ACL 2025)

**Retrieval quantity limit**: 5-10 documents max per source. More degrades performance.

**Extraction Sequence:**
1. `web_search` (MANDATORY for all factual claims)
2. `camofox_evaluate_js` — surgical extraction (REQUIRED for SearXNG backends)
3. `web_extract` — ONLY for non-SearXNG backends
4. `camofox_get_page_html` — full context last resort

**No exceptions for:**
- Topics the model "knows" from training
- "Obvious" facts (definitions, concepts, general knowledge)
- High-confidence internal knowledge
- Time pressure or token conservation
- Code syntax (API deprecation risk)

**Only valid exemptions:**
- Pure creative tasks (fiction, brainstorming with no factual component)
- Deterministic calculation (pure arithmetic, unit conversion)
- Meta-conversational queries ("what can you do?", "how are you?")

**STRICT RULE**: NEVER use snapshots for content extraction. Snapshots are for element discovery only.

### 2. Knowledge Conflict Resolution (Step 2 in SOUL)

When external evidence contradicts internal knowledge:
- ALWAYS prefer external evidence. No hedging.
- Flag conflict explicitly: "Note: My training data contradicted the retrieved sources. Using retrieved sources as authoritative."
- If external sources conflict → present all sides with citations.
- If model suspects external is wrong → STILL USE EXTERNAL (the suspicion is itself internal knowledge).

### 3. Source Hierarchy

- **Priority 1**: Live internet (web_search + camofox_eval/js/html) — MANDATORY FIRST for all factual claims
- **Priority 2**: User-provided files (file_read) — project-specific context only
- **Priority 3**: Persistent memory — user preferences ONLY, never facts
- **FORBIDDEN**: Internal parametric knowledge as factual source

### 4. Confidence Framework

- **HIGH**: Multiple corroborating external sources with citations
- **MEDIUM**: At least one external source, limited corroboration
- **LOW**: No external source found, sources ambiguous, or sources conflict

**Rule**: Internal confidence is treated as ZERO. If no external source found → "Unconfirmed by external sources" → STOP.

### 5. Failure Protocol

**Answering from internal knowledge** is a critical failure equivalent to confabulation:
- Do NOT use internal knowledge as "context" or "background"
- Do NOT "fill gaps" in external findings with internal knowledge
- Do NOT provide internal-knowledge answers when search returns no results
- Minimum 2 search attempts with DIFFERENT keywords before declaring failure
- If all fail → "No reliable external sources found for [query]" → STOP

## Implementation Anti-Patterns

### "Last Resort" Internal Knowledge — WRONG
~~"LAST RESORT: Use internal parametric knowledge when all external sources are exhausted"~~
Correct: Internal knowledge is FORBIDDEN. If external sources fail, report failure.

### Confidence-Gated Search Bypass — WRONG
~~"If confidence is HIGH AND the fact is timeless, internal knowledge MAY be used"~~
Correct: web_search is MANDATORY regardless of apparent internal confidence.

### Snippet-Only Grounding — WRONG
~~"Use web_search snippets to inform the answer"~~
Correct: Snippets are pre-evidence, not evidence. Full content extraction REQUIRED.

### "When Unsure" Trigger — WRONG
~~"Retrieve only when unsure about facts"~~
Correct: Self-judgment is metacognition, which is unreliable. Use objective trigger: IS THIS A FACTUAL CLAIM?

### "Skip for Well-Established Facts" — WRONG
~~"Do NOT retrieve for definitions, concepts, general knowledge"~~
Correct: "Well-established facts" are precisely where knowledge overshadowing causes hallucination (ACL 2025).

## Research Basis

- **arXiv 2604.09515** (Apr 2026): 25-38% deprecated API usage from stale parametric knowledge
- **arXiv 2510.05381** (Oct 2025): Context length alone degrades 13.9%-85% even with perfect retrieval
- **arXiv 2411.19463** (May 2026): RAG must be highly selective; 5-10 docs optimal; failure up to 12.6% even with perfect docs
- **arXiv 2602.05892** (Feb 2026): Over-retrieval with redundant/irrelevant information impairs reasoning
- **ACL 2025**: Knowledge overshadowing — popular knowledge suppresses niche knowledge → hallucination
- **MDPI 2025**: All LLMs tend to generate hallucinations rather than indicating lack of knowledge
- **OpenReview 2025, arXiv 2026**: LLM metacognition unreliable — models cannot detect their own degradation
- **OWASP LLM01:2025**: #1 vulnerability is prompt injection via retrieved content
