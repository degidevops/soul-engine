# Context Hygiene Research Basis

## Empirical Foundation

| Study | Finding | Implication |
|-------|---------|-------------|
| arXiv 2505.06120 | LLM metacognition loss is one of the first degradations as context fills | Self-checks unreliable |
| arXiv 2603.26707 | Google models show "Lost in the Middle" — middle-context instructions ignored | Even 1M+ context windows don't fix |
| GitHub gemini-cli #13672 | Empirical validation of middle-context loss | Confirmed in production |
| CodingFleet 2026 | Google-model-specific attention decay | Specific to Gemini/Gemma family |
| arXiv 2510.05381 | Performance degrades 13.9%-85% as context grows | Multiplicative degradation |
| Liu et al. 2023 (TACL) | U-shaped performance curve (primacy + recency bias) | Tail anchors = highest recall position |

## Why Tail Anchors Work

Recency bias (Liu 2023): Model attends strongest to **end of context**. Placing critical constraints there maintains highest recall probability.

Anchor re-injection is the **only reliable mitigation** (arXiv 2603.26707 conclusion): "Attention decay is structural, not behavioral."

## Why Distillation ≠ Summary

| Distillation | Summary |
|--------------|---------|
| Preserves verified facts verbatim | Compresses everything |
| Discards only noise (failed attempts, filler) | Loses detail across the board |
| Maintains provenance traceability | Loses source attribution |
| Suitable for protocol re-injection | Suitable for compact communication |

## Fault Propagation Stats (arXiv 2603.06847)

| Pattern | Token → Auth Lift | Timestamp → Datetime Lift | Memory → State Lift |
|---------|-------------------|---------------------------|---------------------|
| Failure propagation across layers | 181.5 | 121.0 | 89.3 |

These numbers show why isolated fixes fail in agentic systems. Structural reset with provenance trace is the only sustainable path.

## Load Frequency Justification

Why load every 15 messages / 10 tool calls:
- Empirical observation: degraded attention becomes visible around turn 10-15.
- MANDATORY at 20: tail anchor loses effectiveness beyond ~2K tokens after the anchor in some configurations.
- 10 tool calls: tool calls accumulate faster than user messages; faster cycling needed.
