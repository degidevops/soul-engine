# Retrieval Strategy Research 2025-2026

## Key Papers

### Context Degradation from Retrieval
- **arXiv 2510.05381** (Oct 2025): "Context Length Alone Hurts LLM Performance Despite Perfect Retrieval"
  - Finding: Even with PERFECT retrieval, performance degrades 13.9%-85% as input length increases
  - Implication: More retrieved content ≠ better answers. Quantity must be limited.
  - Mitigation: Distill retrieved evidence before reasoning. Prompt model to recite key facts first.

### Over-Retrieval Pollution
- **arXiv 2602.05892** (Feb 2026): "ContextBench: A Benchmark for Context Retrieval in Coding Agents"
  - Finding: "Over-retrieval with redundant or irrelevant information impairs the agent's reasoning"
  - Implication: Retrieval precision > recall. Irrelevant docs actively hurt performance.

### RAG Must Be Selective on Quantity
- **arXiv 2411.19463** (May 2026): "Understanding the Fundamental Design Decisions of RAG"
  - Finding: "RAG deployment must be highly selective" — failure modes affect up to 12.6% even with perfect documents
  - Finding: 5-10 documents optimal for QA tasks
  - Implication: Blindly retrieving "everything relevant" is harmful. Set hard limits.

### Parametric Knowledge Staleness (Even for Code)
- **arXiv 2604.09515** (Apr 2026): 25-38% deprecated API usage from stale parametric knowledge
  - Implication: Even "deterministic" code syntax can be outdated. Search-first applies to code too.

### Knowledge Overshadowing
- **ACL 2025**: Popular/well-established knowledge suppresses niche knowledge → hallucination
  - Implication: "Well-established facts" are NOT safe from parametric staleness

### LLM Cannot Detect Own Knowledge Boundaries
- **MDPI 2025**: "All LLMs tend to generate hallucinations rather than explicitly indicating a lack of knowledge"
  - Implication: Self-judgment ("am I sure?") is unreliable. Use objective triggers instead.

### Metacognition Limits
- **OpenReview 2025, arXiv 2026**: LLM metacognition is among first capabilities lost as context fills
  - Implication: "Retrieve only when unsure" fails because model can't assess its own uncertainty

## Synthesis: Factually-Triggered Search

The optimal strategy is NOT binary (always vs selective) but **factually-triggered**:

1. **Trigger**: IS THIS A FACTUAL CLAIM? (objective, not self-judgment)
2. **Search**: YES → web_search first, then reason
3. **Exemptions**: Pure creative, pure arithmetic, meta-conversation ONLY
4. **Quantity limit**: 5-10 docs max per source
5. **Distillation**: Summarize before reasoning — don't dump raw content
6. **Citation**: Always cite explicitly (lost-in-the-middle phenomenon)

## What NOT to do
- ❌ "Retrieve only when unsure" → metacognition unreliable
- ❌ "Retrieve everything relevant" → over-retrieval pollution
- ❌ "Skip search for well-established facts" → knowledge overshadowing risk
- ❌ "Skip search for code syntax" → API deprecation risk
- ❌ "Self-check if context is degraded" → structural intervention needed instead
