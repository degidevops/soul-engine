# Context Degradation Research Notes (2025-2026)

## Key Papers

### arXiv:2505.06120 — "LLMs Get Lost In Multi-Turn Conversation" (Microsoft Research, May 2025)
**Authors:** Philippe Laban, Hiroaki Hayashi, Yingbo Zhou, Jennifer Neville (Microsoft Research + Salesforce Research)
**GitHub:** https://github.com/Microsoft/lost_in_conversation
**HuggingFace:** https://huggingface.co/datasets/Microsoft/lost_in_conversation

**Methodology:** Simulated 200,000+ conversations across 15 LLMs (GPT-4o, Claude 3.7, Gemini 2.5, Llama 3.1-8B, o3, Deepseek-R1, etc.) on 6 generation tasks (Code, Database, API Actions, Math, Data-to-text, Summary).

**Key Findings:**
- ALL models show 39% average performance drop from single-turn to multi-turn
- Degradation is mostly due to increased **unreliability** (not loss of aptitude)
- 4 identified root causes:
  1. Premature answer attempts with wrong assumptions
  2. Over-reliance on previous incorrect answers
  3. **Loss-of-middle-turns phenomenon** — model only remembers first and last turns
  4. Overly verbose responses containing more assumptions
- Reasoning models (o3, R1) are NOT immune — degrade similarly despite 33% longer reasoning
- Even 2-turn conversations with underspecification cause degradation
- Recap (restating all info at end) and Snowball (restating at each turn) help ~15-20% but don't fully fix

**Statistics:**
- 15 LLMs tested across 8 model families
- 600 instructions sharded and simulated
- 10 runs per (model, instruction, simulation type)
- ~$5,000 total simulation cost

### arXiv:2502.07365 — "LongReD: Mitigating Short-Text Degradation of Long-Context LLMs" (Feb 2025)
**Finding:** Even models with 1M token context windows experience attention decay at position 0 (where system prompt sits). System prompt length degradation is measurable and significant.

### AgentIF — NeurIPS 2025 Benchmark
**Finding:** Systematic evaluation of instruction following in agentic scenarios. All frontier models lose 15-30% adherence after 50+ turns.

## Practical Implications for Hermes

1. **System prompt position matters** — it's at position 0, which has the highest attention decay
2. **SOUL.md length is a liability** — longer system prompt = more tokens that can degrade. Keep it lean.
3. **Template size target:** <150 lines / <8KB is the current SOUL_TEMPLATE_V7.md size — this is good
4. **Step 6 in template** (Context Degradation Awareness) is critical — it's the prompt-level mitigation
5. **Config-level is most reliable:** `compression.protect_first_n: 5` and `hygiene_hard_message_limit: 300`

## Reddit/Observational Evidence (2025-2026)
- r/ClaudeAI (Aug 2025): "Long conversation reminder causing observable degradation"
- r/ChatGPT (Jul 2025): "ChatGPT becoming insanely stupid lately... Start a fresh convo"
- r/GeminiAI (Jan 2026): "System prompt overhead tokens... performance degradation"
- r/LangChain (Mar 2026): Developers asking how to make agents "stay on task"
