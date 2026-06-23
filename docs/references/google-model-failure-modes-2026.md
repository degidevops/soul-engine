# Google Model Failure Modes (Gemini/Gemma) — 2026 Research

## Attention Decay in Large Context Windows

### The "Lost in the Middle" Reality
- **arXiv 2603.26707 (Mar 2026)**: "The Cognitive Divergence" — Quality-adjusted divergence accounting for lost-in-the-middle retrieval degradation. Even frontier models with 1M+ context windows show significant attention decay.
- **GitHub google-gemini/gemini-cli #13672**: Users report model ignores middle-context instructions, defaults to generic low-effort answers despite specific constraints. Confirmed across multiple threads.
- **CodingFleet 2026**: "The Context Window Lie" — Gemini 1.5 Pro showed >99.7% single-item recall at 1M tokens in formal testing, but real-world performance drops significantly due to lost-in-the-middle.

### Key Insight
Attention decay is **structural, not behavioral**. You cannot prompt the model to "pay more attention" — the mechanism itself degrades. Mitigation requires external intervention (re-injection, not self-correction).

## Instruction Following Degradation (Gemma 4 Dense)

### Architecture-Specific Behavior
- **DEV Community (May 2026)**: "Choosing the Right Gemma 4 Model" — Instruction-following failure differs by architecture. Gemma 4 31B Dense with well-engineered system prompt shows substantially better compliance than MoE variants.
- **arXiv 2604.24636 (Apr 2026)**: "Engineering Challenges of On-Device Small Language Models" — Better instruction following requires explicit, structured prompts. Ambiguous or verbose prompts degrade compliance.

### Google-Specific Recommendation
For Google models (Gemini/Gemma), system prompts should be:
1. **Structured** (XML tags, not prose)
2. **Positioned at end** (anchor re-injection, not just top-of-context)
3. **Non-redundant** (each constraint appears exactly once)

## Semantic Failure: Syntactic Success, Factual Failure

### The Problem
- **TMLS NYC (Jun 2026)**: "Structured Outputs and Constrained Decoding in Production" — "A semantic failure [is] output that matches the schema but violates a ground truth."
- **arXiv 2601.06678 (Jan 2026)**: Reflective Reasoning for SQL Generation — Each stage must be implemented as a stateless function that consumes validated JSON input and produces validated JSON output.

### Why Google Models Are Susceptible
Google models (especially Gemini/Gemma) are heavily optimized for format compliance. They produce perfect JSON, valid code syntax, and well-structured outputs — even when the **values** inside are hallucinated. This is "overconfidence in format."

### Mitigation: Value-Level Cross-Reference
Schema validation is necessary but insufficient. Must validate:
1. **Schema** — Is the structure correct? (format check)
2. **Values** — Does each value trace to a specific source passage? (semantic grounding)
3. **Consistency** — Do values conflict with each other or with retrieved evidence?

## Self-Reported Confidence is Unreliable

### Why It Doesn't Work
LLMs do not have self-awareness or metacognitive access to their own uncertainty. "Confidence" from the model is:
- A statistical token probability, not a measurement of truth
- Correlated with format compliance, not factual accuracy
- Subject to overconfidence (especially in Google models)

### Correct Approach: Binary Provenance
Replace "How confident am I?" with "Can this claim be traced to a specific tool output or source passage?"

- **VERIFIED**: Claim has direct citation + value matches source
- **UNVERIFIED**: No source found, or value cannot be traced
- **CONFLICTING**: Multiple sources disagree

## Implications for SOUL Template Design

### What Works
1. **Anchor Re-Injection**: Re-inject critical constraints at end of context window periodically
2. **Distillation (not summarization)**: Preserve verified facts verbatim, discard noise
3. **Binary provenance**: Deterministic traceability, not subjective confidence
4. **Semantic grounding**: Value-level cross-reference for all structured outputs

### What Doesn't Work
1. **Self-checks** ("Am I being less precise?") — metacognition is unreliable
2. **Confidence scores from model** — statistical artifact, not truth measurement
3. **Redundant constraints** — repetition causes compliance dilution
4. **Long static prompts** — attention decay makes middle-content invisible

## Citations
- arXiv 2603.26707: Cognitive divergence / attention decay
- arXiv 2604.24636: Gemma 4 instruction following challenges
- GitHub google-gemini/gemini-cli #13672: Lost-in-the-middle user reports
- TMLS NYC Jun 2026: Semantic failure definition
- arXiv 2601.06678: Stateless validation functions
- arXiv 2505.06120: Metacognition is unreliable (carried from V7.1)
