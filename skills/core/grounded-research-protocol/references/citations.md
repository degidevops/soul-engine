# Research Basis Citations

## Core Principle
- **arXiv 2505.15962**: Parametric knowledge suffers staleness, hallucination, weak attribution, limited adaptability.
- **MDPI 2025**: "Probabilistic predictions, not verified facts" — model cannot reliably self-detect knowledge boundaries.

## Trigger Logic + Staleness
- **arXiv 2604.09515**: 25-38% deprecated API usage from stale parametric knowledge.
- **arXiv 2510.05381**: 13.9%-85% performance degradation at high retrieval counts.
- **ACL 2025** (knowledge overshadowing): "General knowledge" and "well-established facts" most at risk of staleness.

## Contract Validation
- **arXiv 2603.06847**: 19.5% root cause = dependency/integration failures from probabilistic output violating deterministic interfaces.

## Semantic Grounding
- **arXiv 2603.06847**: Structured outputs reduce syntax errors but not semantic failures.
- **TMLS NYC 2026**: "Semantic failure = output matches schema but violates ground truth".

## Evidence Ranking Layer
- **arXiv 2601.16478** (DeepEra, Jan 2026): step-by-step reasoning into reranking.
- **arXiv 2605.19196** (REFLECT, May 2026): LLM-as-Judge for evidence verification unreliable (<55% accuracy).
- **arXiv 2605.01664** (May 2026): hybrid retrieval + reranking + claim-level grounding.
- **arXiv 2606.04990** (Jun 2026): evidence tracing + execution provenance = process-level accountability foundation.
- **arXiv 2508.08742** (Aug 2025): SSLI (Semantically Similar but Logically Irrelevant) formally defined and benchmarked.

## Security
- **OWASP LLM01:2025**: Instruction contamination vector.
- **Hermes Agent architecture docs**: extraction hierarchy explicitly forbids snapshots.

## Quantified Impact
| Study | Metric | Impact |
|-------|--------|--------|
| 2604.09515 | Deprecated API rate | 25-38% |
| 2510.05381 | Performance degradation at scale | 13.9%-85% |
| 2603.06847 | Dependency/integration failure root cause | 19.5% |
| 2603.06847 | Data/type handling root cause | 17.6% |
| 2605.19196 (REFLECT) | LLM-as-Judge accuracy ceiling | <55% |

These numbers are non-negotiable baseline metrics for protocol design. Any deviation (e.g., using internal knowledge for "general fact") should be flagged as **protocol violation**, not optimization.
