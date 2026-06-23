# Evidence Ranking Layer — Research & Implementation (2026)

## Problem

Retrieved evidence varies wildly in quality. Not all sources are equally reliable. LLM-as-Judge for evidence verification is unreliable. Without explicit ranking, agents treat a random blog post the same as a peer-reviewed paper.

## Key Papers

### Deep Evidence Reranking Agent (DeepEra)
- **arXiv 2601.16478** (Jan 2026)
- Integrates step-by-step reasoning into reranking
- Addresses SSLI: Semantically Similar but Logically Irrelevant passages
- Two-stage RAG: retrieve → rerank with reasoning → generate
- Finding: Surface-level semantic similarity ≠ logical relevance

### LLM-as-Judge is Unreliable for Evidence Verification (REFLECT)
- **arXiv 2605.19196** (May 2026)
- Best LLM judges achieve <55% accuracy on evidence verification
- Especially poor on evidence verification (vs. reasoning or tool-use failures)
- Implication: Do NOT rely on LLM self-assessment for evidence quality

### Hybrid Retrieval + Reranking Framework
- **arXiv 2605.01664** (May 2026)
- Hybrid retrieval + Cohere reranking + conservative prompting + claim-level evaluation
- Result: 100% grounding accuracy when sufficient source evidence available
- Implication: Reranking layer + claim-level verification = reliable grounding

### Evidence Tracing & Execution Provenance
- **arXiv 2606.04990** (Jun 2026)
- Evidence tracing = projection of execution provenance onto evidence-support relations
- Connects retrieval grounding, claim support, tool-use safety, memory lineage
- Implication: Track which evidence supports which claims (provenance)

## Implementation in SOUL V7

The Evidence Ranking Layer is embedded in Step 1 of the reasoning protocol, after extraction and before reasoning.

### Ranking Criteria

| Criterion | What to check | Action |
|-----------|---------------|--------|
| **Source Authority** | .edu, .gov, official docs, peer-reviewed vs. blogs, social media | Rank authoritative higher |
| **Recency** | Publication/update date | Flag stale sources (>1yr for fast-moving topics) |
| **Corroboration** | Multiple independent sources agree? | Single-source = "limited corroboration" |
| **Logical Relevance** | Does passage actually support the claim? (not just semantically similar) | Discard SSLI passages |
| **Conflict Detection** | Sources contradict each other? | Present all sides with citations |

### Confidence Scoring

| Score | Condition |
|-------|-----------|
| **HIGH** | Multiple authoritative + recent + corroborated + logically relevant |
| **MEDIUM** | One authoritative OR multiple weaker sources agree |
| **LOW** | Single source, outdated, logically weak, or sources conflict |
| **REJECT** | Unreliable, SSLI, or unresolvable conflict with stronger evidence |

### Output Routing

- Conflicts detected → Step 2 (Conflict Resolution)
- Ranked evidence → Step 4 (Reasoning)
- Confidence scores → Step 5 (Anti-Hallucination Gate)

## Anti-Patterns

- ❌ "Let the model judge evidence quality" → LLM-as-Judge unreliable (<55%)
- ❌ "Semantically similar = relevant" → SSLI problem
- ❌ "More sources = better" → Over-retrieval pollution
- ❌ "Pick the strongest source silently" → Must present conflicts
- ❌ "All .com sources equal" → Authority matters

## Template Location

Implemented in `~/.hermes/SOUL_TEMPLATE_V7.md` → `<evidence_ranking_layer>` element, and synced to `skills/devops/soul-management/templates/SOUL_TEMPLATE_V7.md`.
