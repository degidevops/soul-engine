# Confidence → Provenance Rosetta

## Why This Conversion Table Exists

LLMs output natural-language hedges ("I think...", "probably...", "~95% confident..."). These are **NOT** measurements of truth — they are statistical token probabilities that correlate poorly with actual accuracy (overconfidence problem, arXiv 2603.06847).

The rosetta deterministically maps any natural-language confidence expression to one of three binary statuses (VERIFIED, UNVERIFIED, CONFLICTING).

## Rosetta Table

### VERIFIED (high-confidence convert)
| LLM Expression | Action |
|----------------|--------|
| "Confirmed in [source]" | VERIFIED + source_ref |
| "Per [tool output]" | VERIFIED + source_ref |
| "Source X states..." | VERIFIED + source_ref |
| "Retrieved show..." | VERIFIED + source_ref |
| Direct quotation with attribution | VERIFIED + source_ref |
| Kanban board state (V73 via kanban_list) | VERIFIED (operational truth source) |
| Worker output via file_read | VERIFIED + cross-reference |

### UNVERIFIED (medium/low/hedge convert)
| LLM Expression | Action |
|----------------|--------|
| "I think..." | UNVERIFIED + flag |
| "Probably..." | UNVERIFIED + flag |
| "Probably..." | UNVERIFIED + flag |
| "I'm not sure..." | UNVERIFIED + flag |
| "I believe..." | UNVERIFIED + flag |
| "Generally..." | UNVERIFIED + flag |
| "Most likely..." | UNVERIFIED + flag |
| (Any internal knowledge claim) | UNVERIFIED |

### CONFLICTING (multi-source)
| LLM Expression | Action |
|----------------|--------|
| "Source A says X, but source B says Y" | CONFLICTING + present both |
| "There are different views on..." | CONFLICTING + enumerate |
| "Depending on perspective..." | CONFLICTING + cite both |
| Tool output contradicts other tool output | CONFLICTING + reconcile |

## Conflict Resolution Hierarchy

1. **External > Internal**: External sources ALWAYS win over internal knowledge (no hedging, no "both sides have merit" framing).
2. **Most recent > older**: Recency matters for time-sensitive topics.
3. **Multiple corroborating > single**: Higher confidence when multiple agree.
4. **Contradicting sources > silent pick**: Always present sides explicitly.

## Implementation Pattern

When emitting structured output (JSON/code/API), every fact-level value MUST carry provenance_status:

```json
{
  "founded_year": 1998,
  "founder": "Larry Page, Sergey Brin",
  "_provenance_founded_year": "VERIFIED",
  "_source_founded_year": "https://about.google/our-history/",
  "_provenance_founder": "VERIFIED",
  "_source_founder": "https://about.google/our-history/"
}
```

For V73 kanban operations:
```json
{
  "task_id": "task-42",
  "status": "claimed",
  "assignee": "research-profile",
  "_provenance_status": "VERIFIED",
  "_source_status": "kanban_list output at 2026-06-23T15:30:00Z"
}
```

## What NEVER Validates as VERIFIED

- Internal knowledge claim (any confidence level).
- "Well-known fact" / "general knowledge" / "everyone knows".
- Memory tool recall (memory is user-preference source, NOT fact source).
- Cross-conversation memory of prior search results (cache ⊉ truth).

These are **always UNVERIFIED** unless re-verified via fresh `web_search` in current context.
