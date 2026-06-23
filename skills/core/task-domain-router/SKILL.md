---
name: task-domain-router
description: "Routes {{IN_SCOPE_ITEMS}} placeholder content → recommended skill bundles. Replaces verbose in-scope lists in V7 template. Keyword-signal detection + per-profile override. V7-only (V73 uses Kanban dispatch instead)."
version: 1.0.0
author: Maru
license: MIT
tags: [routing, task-domain, scope-management, hermes-agent, soul-engine]
metadata:
  hermes:
    parent_skill: soul-engine/core
    related: []
    replacement_for:
      - V7_template: placeholder {{IN_SCOPE_ITEMS}} content (typically 8-10 verbose items per profile)
    pointer_syntax: |
      <scope>
        <in_scope_summary>Standard worker scopes — see skill 'task-domain-router'</in_scope_summary>
        <in_scope_router>skill://task-domain-router</in_scope_router>
        <out_of_scope>{{OUT_OF_SCOPE_ITEMS}}</out_of_scope>
        <authority>{{AUTHORITY_LEVEL}}</authority>
      </scope>
    note_V73: "V73 orchestrator uses Kanban for routing (delegate_task + kanban_*) — task-domain-router NOT applicable"
---

# Task-Domain Router

Loader mode: **ONDEMAND**. Loaded when user query matches a routed domain keyword.

## When to Load
- User query contains domain keywords (e.g., "OpenWrt", "Hermes config", "trading research").
- Profile deployment: bootstrap default routing.
- User reports "agent doesn't know about X" → check if router missed the trigger.

## When NOT to Load
- Orchestrator profile (V73) — Kanban handles routing.
- Pure creative/meta-conversational queries.
- Tasks already covered by another skill loaded into context.

---

## Purpose

The V7 template previously used `{{IN_SCOPE_ITEMS}}` with verbose lists (8-10 items like "Task execution via tools, Research, Code generation, OpenWrt config, ..."). This caused:

1. **Personality creep** — verbose lists pile up over time.
2. **Misplaced context** — domain knowledge that belongs in dedicated skills.
3. **Maintenance burden** — every new capability required template edit.

**Solution**: Compact template pointer + on-demand router skill that maps query keywords → recommended skill bundles.

---

## Default Domain Taxonomy

### 1. System / DevOps
- **Keywords**: openwrt, hermes, hermes-agent, dns, doh, dmasq, firewall, iptables, ssh, sshd, sysctl, systemctl, journalctl, ssh-keygen, systemd
- **Skill bundle**: `openwrt-management`, `hermes-agent`, `system-config-management`
- **Tools**: terminal, file, web_search (for issue-specific research)
- **Examples**: "configure OpenWrt firewall", "set up DoH on my router"

### 2. Software Development
- **Keywords**: python, typescript, javascript, react, fastapi, langchain, docker, git, code review, code-gen, debug, refactor, test, pytest, lint, types, build, compile
- **Skill bundle**: `hermes-agent`, `plan`, `simplify-code`, `test-driven-development`, `python-debugpy`, `systematic-debugging`, `requesting-code-review`
- **Tools**: terminal, file, web_search
- **Examples**: "refactor this Python module", "write pytest tests"

### 3. Research / Information Retrieval
- **Keywords**: research, paper, arxiv, literature, search, web search, citation, evidence, source, reference, blog, forum, scrape, extract, bibliography
- **Skill bundle**: `web-research`, `arxiv`, `scrapling`, `qmd`, `llm-wiki`
- **Tools**: web_search, web_extract, camofox_evaluate_js, camofox_get_page_html
- **Trigger**: External skill `grounded-research-protocol` for protocol detail.
- **Examples**: "research ARX 2602.12430", "verify this claim via primary source"

### 4. Trading Research
- **Keywords**: xauusd, xau, gold, trading, forex, analysis, backtest, ohlcv, candle, sentiment, rsi, macd, bollinger, signal, market, broker, mt5, mt4, broker api
- **Skill bundle**: `polymarket` (where applicable); profile-level `travis`
- **Tools**: web_search, file_read (OHLCV data), terminal (Python analysis)
- **Examples**: "What does RSI 14 on XAU/USD show in last 4h?"
- **Hard boundary**: NO trading execution (out-of-scope per V7 scope config).

### 5. Multi-Agent Orchestration
- **Keywords**: orchestrator, kanban, worker, dispatcher, decompose, delegate, rtasks, kanban_create, kanban_list, dispatch
- **Skill bundle**: Use **V73 orchestrator profile**, NOT V7 worker.
- **Boundary**: This routing category REQUIRES profile switch at session level.

### 6. Creative Production
- **Keywords**: image, video, music, infographic, ascii, manim, sketch, excalidraw, tts, voice
- **Skill bundle**: `creative/*` (depends on artifact type)
- **Tools**: terminal, file, image_gen (where enabled), TTS
- **Examples**: "Generate an architecture SVG", "Sketch a flow diagram in ASCII"

### 7. Communication / Messaging
- **Keywords**: telegram, discord, slack, signal, whatsapp, sendmessage, gateway, channel
- **Skill bundle**: `hermes-agent` (gateway setup), `telegram`, `discord_admin`
- **Tools**: messaging platform tools
- **Examples**: "send this report to Telegram channel"

### 8. Network Engineering
- **Keywords**: openwrt, mt76x8, ramips, networking, routing, vpn, wireguard, subnet, nat, dhcp, dns
- **Skill bundle**: `openwrt-management`
- **Tools**: terminal, web_search

### 9. Persistent Memory / Knowledge Management
- **Keywords**: memory, recall, store, knowledge, obsidian, qmd, kb, knowledge base, memory purge
- **Skill bundle**: `memory_tool` (built-in), `obsidian`, `qmd`, `llm-wiki`, `knowledge-graph-construction`
- **Tools**: memory_tool, file_read, web_search
- **Examples**: "remember OpenWrt parser compiles", "recall last Kanban cycle"

---

## Routing Algorithm

```
user_query → keyword_extraction → domain_match (1+ domains) →
  for each matched domain:
    load skill bundle + tools + verify capability match
  primary skill:
    if single domain: bundle recommendation
    if multi-domain: orchestrator-class (delegate or flag for profile switch)
  confidence:
    HIGH (>0.8): direct routing
    MEDIUM (0.5-0.8): confirm with user before proceeding
    LOW (<0.5): explicit clarification loop
```

### Confidence Scoring
- HIGH: 3+ unique keywords match single domain.
- MEDIUM: 1-2 keywords match, OR keywords span 2 domains with clear primary.
- LOW: keywords are ambiguous (e.g., "research coding agents" — research + software dev).

---

## Per-Profile Override

Each V7 profile can override default via:

```yaml
# in profile config
soul_engine:
  task_domain_router:
    overrides:
      research:
        primary_skill: web-research  # instead of generic arxiv
      trading:
        forbidden: true  # disable category entirely
```

Override applies at session startup. Per-session override via `{{IN_SCOPE_ITEMS}}` mini-list (max 3 items) takes precedence.

---

## Template Placeholder Companion

| Placeholder | Content |
|-------------|---------|
| `{{IN_SCOPE_ITEMS}}` | Optional mini-list, max 3 items. Used ONLY for profile-specific most-common tasks. |
| `{{OUT_OF_SCOPE_ITEMS}}` | Required. Profile-specific forbidden categories. |
| `{{AUTHORITY_LEVEL}}` | Required. Permissions summary. |

**Reduced verbosity**: Template retains ONE line instead of 8-10 verbose items:

```xml
<scope>
  <in_scope_summary>Standard worker scopes — see skill 'task-domain-router'</in_scope_summary>
  <in_scope_router>skill://task-domain-router</in_scope_router>
  <out_of_scope>{{OUT_OF_SCOPE_ITEMS}}</out_of_scope>
  <authority>{{AUTHORITY_LEVEL}}</authority>
</scope>
```

---

## Cross-Skill Dependencies

- **All skill bundles in section above** — this is a *meta-skill* that dispatches to others.
- `grounded-research-protocol` — fires when routed to research category.

---

## Pitfalls

1. **Don't auto-route to a category based on single keyword**: "Python" in "Python the snake" vs "Python the language" — context matters.
2. **Multi-domain queries need explicit user confirmation** before orchestrator-class routing.
3. **Forbidden categories are HARD boundaries**, not suggestions. If user query touches trading research when overridden forbidden, escalate to user.
4. **Routing is a recommendation, not a constraint** — user can override at any time.

---

## Validation Self-Test

```bash
# Confirm V7 template has compact pointer
grep -E "task-domain-router" ~/.hermes/skills/devops/soul-engine/templates/SOUL_TEMPLATE_V7.md

# Confirm V73 does NOT have this router (uses Kanban instead)
! grep -E "task-domain-router" ~/.hermes/skills/devops/soul-engine/templates/SOUL_TEMPLATE_ORCHESTRATOR_V73.md

# Confirm skill loaded
skill_view('task-domain-router')
```

---

## Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-06-23 | Initial extraction. Replaces verbose {{IN_SCOPE_ITEMS}} in V7 profiles. 9 default domain taxonomies. V73-not-applicable. |
