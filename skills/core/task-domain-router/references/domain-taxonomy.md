# Task-Domain Routing Taxonomy

## Maintenance

This file documents the default domain taxonomy behind `task-domain-router` skill. Update when new skill bundles are added to the engine.

## Domain Matrix

| # | Domain | Primary Keywords (5+) | Skill Bundle | Tools |
|---|--------|------------------------|--------------|-------|
| 1 | System/DevOps | openwrt, hermes-agent, dns, doh, firewall, sysctl | openwrt-management, hermes-agent, system-config-management | terminal, file, web_search |
| 2 | Software Development | python, javascript, react, fastapi, docker, git, debug, test | plan, simplify-code, tdd, python-debugpy, requesting-code-review | terminal, file, web_search |
| 3 | Research / Information Retrieval | research, paper, arxiv, literature, source, citation, evidence | web-research, arxiv, scrapling, qmd, llm-wiki | web_search, web_extract, js, html |
| 4 | Trading Research | xauusd, gold, trading, forex, ohlcv, rsi, macd, mt5, broker | profile-travis, polymarket | web_search, file_read, file (Python analysis) |
| 5 | Multi-Agent Orchestration | orchestrator, kanban, dispatcher, decompose, delegate | V73 profile REQUIRED | (profile switch) |
| 6 | Creative Production | image, video, music, infographic, sketch, excalidraw, tts | creative/* (mapped per artifact type) | image_gen, tts, video_gen |
| 7 | Communication / Messaging | telegram, discord, slack, signal, whatsapp, channel | hermes-agent (gateway), messaging/* | messaging |
| 8 | Network Engineering | mt76x8, ramips, networking, vpn, wireguard, subnet, nat | openwrt-management | terminal, web_search |
| 9 | Persistent Memory / KG | memory, recall, store, kb, obsidian, qmd, knowledge | memory_tool, obsidian, qmd, llm-wiki | memory_tool, file_read, web_search |

## Hard Boundaries (out_of_scope — always respected)

- V7 worker: NEVER execute financial trades (research/analysis only).
- V7 worker: NEVER bypass security protocols or user-defined limits.
- V7 worker: NEVER self-modify core persona without explicit user approval.
- Any profile: NEVER execute illegal, harmful, or ethically compromised tasks.

## Skill Discovery

For profiles with custom domains (e.g., `dave`, `travis`), per-profile override config:

```yaml
# in profile config
soul_engine:
  task_domain_router:
    overrides:
      <custom-domain>:
        primary_skill: <skill-name>
        fallback: <alternate-skill>
```

## Versioning Style

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-06-23 | Initial taxonomy. 9 domains. Covers System/DevOps, Software Dev, Research, Trading, Multi-agent, Creative, Communication, Networking, Memory/KG. |
