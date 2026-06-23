# Failure-Mode Research Basis Citations (M1-M7)

Authoritative sources backing each mode's diagnostic + recovery playbook.

## M1: Answering Without Searching
- **arXiv 2604.09515** — 25-38% deprecated API usage attributed to stale parametric knowledge.
- **MDPI 2025** — "Probabilistic predictions, not verified facts" — model cannot reliably self-detect knowledge boundaries.

## M2: Internal Knowledge as Fact Source
- **arXiv 2505.15962** — Parametric knowledge suffers staleness, hallucination, weak attribution, limited adaptability.
- **MDPI 2025** — same as M1.

## M3: Silent Parametric Fallback
- **arXiv 2603.06847** — Shah et al. fault taxonomy. 19.5% dependency/integration failures + 17.6% data/type handling as top root causes.

## M4: Silent Intent Guessing
- Repository-specific — pattern observed across practice. No single citation; treated as community consensus.

## M5: Snapshot-Based Extraction
- **Hermes Agent architecture docs** — `agent/system_prompt.py` extraction hierarchy explicitly forbids snapshots for content.
- **camofox_evaluate_js / camofox_get_page_html** — tool layer handling.

## M6: Multi-Agent Coordination Collapse
- **arXiv 2603.06847** — Shah et al. 16.2% practitioners reported missing multi-agent coordination failures.
- **Lu et al. 2025** — Coordination collapse, groupthink, circular dependencies.

## M7: Semantic Failure
- **arXiv 2603.06847** — Distinguishes syntactic vs. semantic failures; structured outputs reduce syntax errors but not semantic failures.
- **TMLS NYC 2026** — "Semantic failure = output matches schema but violates ground truth".

---

## Background Reading

The 7 modes together cover roughly **80%** of practitioner-reported agentic system fault surface (per Shah et al. empirical study). They are **not** exhaustive — emergent failure patterns will require mode extensions over time.

When in doubt: the **diagnose → intervene → verify** lifecycle is universal. The specific playbook is variant.
