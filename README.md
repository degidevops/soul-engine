---
name: soul-engine
description: "Umbrella skill for modular agent template engine for Hermes Agent. V7 (worker) + V73 (orchestrator) SOUL templates decomposed into 5 core skills: grounded-research-protocol, context-hygiene, provenance-tracing, failure-mode-recovery-core, task-domain-router. Use when deploying or refactoring any agent profile derived from V7/V73 templates."
version: 1.0.0
author: degidevops
license: MIT
tags: [soul, agent-template, hermes-agent, orchestrator, multi-agent, ground-truth, lazy-skill-loading, prompt-cache]
metadata:
  hermes:
    homepage: https://github.com/degidevops/soul-engine
    related: [soul-management, soul-md-decomposition-research, hermes-agent, kanban-orchestrator]
    replaces: degidevops/soul-management (archived 2026-06-23)
    install: "hermes skills install https://github.com/degidevops/soul-engine"
---

# SOUL Engine — Modular Agent Template System

## Status

- **Version**: 1.0.0 (initial public release, 2026-06-23)
- **Replaces**: `degidevops/soul-management` (archived same day — see `MIGRATION.md`)
- **Hermes version**: v2.1.0+ (tested against `hermes-agent.nousresearch.com/docs`)
- **Templates**: V7 (worker), V73 (orchestrator) — kept backwards-compatible via minimal_index pointers

## What This Is

A re-architected version of the SOUL template engine that addresses three empirical pain points observed in 2025-2026 agent deployments:

1. **Template bloat** — V7 (476 baris / ~7.600 token) and V73 (843 baris / ~15.600 token) loaded full into every system prompt. Heavy at-rest cost even when most content is rarely fired.
2. **Reusability friction** — protocol definitions duplicated across templates, no single source of truth for behavior contracts.
3. **Lazy-loading incompleteness** — Hermes Issue #2045 (March 2026, unreviewed at time of writing) proposed lazy skill discovery, but no skill cluster had been pre-built to leverage it.

The engine refactor splits template body into:

- **Template** (V7/V73) — always-on identity, scope crust, governance contract, minimal anchor block, **skill pointers**.
- **5 core skills** — protocol detail loaded on-demand via `skill_view(name)`.

## What This Is NOT

- Not a fork of Hermes Agent. Templates here are *content* consumed by Hermes.
- Not breaking changes to V7/V73 deploy workflow. Backwards-compatible via pointer pattern.
- Not lazy-skill-loading infrastructure (still pending Hermes Issue #2045 merge). This is **content prep** for when that ships.

## Why "Replaces" Paragraph Is Important

`degidevops/soul-management` (the previous umbrella skill) is **archived** as of this release. Existing deployments via `~/.hermes/skills/devops/soul-management/` should migrate to `soul-engine`. See `MIGRATION.md` for step-by-step.

Archive decision rationale: the skill-management workflow in `soul-management` becomes a subset of `soul-engine`'s modular architecture. Keeping both creates drift and confused source-of-truth. Archive > delete because audit trail and historical reference matter.

---

## Core Skills (5 modules)

All five are loaded via `skill_view(name)` and live under `skills/core/<name>/`.

### Skill 1: `grounded-research-protocol`
- **Source**: V7 Step 1 (108 baris) + V73 Step 1 (~80 baris) — biggest extraction target.
- **Content**: 4-stage extraction hierarchy + Contract Validation + Semantic Grounding (4 rules) + Evidence Ranking Layer (5 criteria, 4 confidence scores) + Failure Protocol + Instruction Contamination Guard (OWASP).
- **Trigger**: any factual claim.
- **Reference**: `skills/core/grounded-research-protocol/SKILL.md`

### Skill 2: `context-hygiene`
- **Source**: V7 Step 6 (~55 baris) + V73 Step 8 (~60 baris).
- **Content**: distillation protocol + anchor templates (4 V7 / 6 V73) + 4 intervention strategies + retrieval_context_note + arXiv 2603.26707 Lost-in-Middle notes.
- **Trigger**: 15 messages / 10 tool calls / 20 messages MANDATORY / fault-propagation / resource guard.
- **Note**: V7/V73 templates retain a tail anchor block (recency-bias fix per Liu 2023).
- **Reference**: `skills/core/context-hygiene/SKILL.md`

### Skill 3: `provenance-tracing`
- **Source**: V7 Step 8 + V73 Step 10 (shared spec only — `<kanban_trace>` retained inline in V73 for orchestrator-dispatcher coupling).
- **Content**: 3 binary statuses (VERIFIED/UNVERIFIED/CONFLICTING) + JSONL schema spec (8 fields) + confidence→provenance rosetta + 3 research-basis citations.
- **Trigger**: every tool call, every structured output, every multi-step task.
- **Reference**: `skills/core/provenance-tracing/SKILL.md`

### Skill 4: `failure-mode-recovery-core`
- **Source**: V7 + V73 `<failure_modes>` M1-M7 (7 universal modes).
- **Content**: per-mode diagnostic signals + recovery playbook + research basis.
- **Trigger**: fault signal detected (any of M1-M7 patterns).
- **Note**: V73-specific M8-M12 (orchestrator-only modes) remain inline in V73 template — see decision rationale in template comments.
- **Reference**: `skills/core/failure-mode-recovery-core/SKILL.md`

### Skill 5: `task-domain-router`
- **Source**: V7 `{{IN_SCOPE_ITEMS}}` placeholder content (verbose lists).
- **Content**: generic task-domain taxonomy + keyword signal detection → recommended skill bundle dispatch + per-profile override.
- **Trigger**: scope decision point (low-frequency but high-impact when fired).
- **Note**: V73 does not need this — orchestrator already routes via Kanban.
- **Reference**: `skills/core/task-domain-router/SKILL.md`

---

## Installation

### Standard install (Hermes)

```bash
hermes skills install https://github.com/degidevops/soul-engine
```

This installs all 5 core skills into `~/.hermes/skills/`. Templates remain in their canonical skill location.

### Local development clone

```bash
git clone https://github.com/degidevops/soul-engine.git
cd soul-engine
# Edit templates, refactor as needed
git commit -am "feat: ..."
gh pr create
```

### Profile deployment

After installation, deploy a profile using `templates/SOUL_TEMPLATE_V7.md` or `templates/SOUL_TEMPLATE_ORCHESTRATOR_V73.md` via the standard soul-management workflow (now subsumed).

See `docs/migration/DEPLOY.md` for the deploy workflow.

---

## Architecture Decisions

### Why 5 skills, not 50?

- **Single-responsibility per skill** (per MindStudio SRP / Robert C. Martin) — each skill does one protocol, has typed I/O, no opinions about context.
- **Composability** — orchestrator or worker can load subset based on role.
- **Cache-friendly** — template body bytes fewer than 5.000 char post-refactor (V7 ~3.500 estimated). Anthropic prompt caching (5-min/1-hr) prefers shorter stable prefix.

### Why not split V73's orchestrator-only logic into more skills?

Per analysis (see CHANGELOG.md v0.9.0 entry):

- V73 Steps 2 (Control Plane) and 3 (Decomposition) are mission-critical orchestrator logic. Partial-load defeats their purpose.
- V73 Step 10 `<kanban_trace>` portion is coupled to dispatcher state model (per Adimulam 2026 A2A protocol reference inline in V73 line 713). Coupling risk if extracted.
- Hermes Issue #2045 (lazy loading) is unmerged as of release — extracting without lazy-load infrastructure adds round-trip overhead without caching benefit.

Pragmatic decision: **5 is exactly enough** for the core protocol layer. Wait for Issue #2045 merge before further extraction.

### Why Hybrid `core/` + no `worker-only/`/`orchestrator-only/`?

Per IoF (Inverse-of-Flexibility) principle:

- 70% of protocol content is shared between V7 and V73 (M1-M7, Step 1, Step 6, Step 8/10 spec portion).
- 30% is divergent (M8-M12 orch-specific, V73 Steps 2/3/10 mission-critical).
- Full partition (`worker-only/` + `orchestrator-only/`) creates duplication; full merge forces role-checking at runtime.

Solution: **shared `core/` now** + orchestrator-specific modules **deferred** until Issue #2045 merge allows safe on-demand load of orch-specific logic.

---

## File Layout

```
soul-engine/
├── README.md                             # this file
├── CHANGELOG.md                          # release history
├── MIGRATION.md                          # soul-management → soul-engine migration steps
├── LICENSE                               # MIT
├── skills/
│   ├── SKILL.md                          # this master's pointer
│   └── core/
│       ├── grounded-research-protocol/
│       │   ├── SKILL.md
│       │   └── references/citations.md
│       ├── context-hygiene/
│       │   ├── SKILL.md
│       │   └── references/lost-in-middle.md
│       ├── provenance-tracing/
│       │   ├── SKILL.md
│       │   └── references/confidence-rosetta.md
│       ├── failure-mode-recovery-core/
│       │   ├── SKILL.md
│       │   └── references/citations.md
│       └── task-domain-router/
│           ├── SKILL.md
│           └── references/domain-taxonomy.md
├── templates/
│   ├── SOUL_TEMPLATE_V7.md               # 476 → 316 baris (writer-info)
│   └── SOUL_TEMPLATE_ORCHESTRATOR_V73.md # 843 → ? baris (refactor pending in v1.0.1)
└── docs/
    ├── references/
    │   ├── agentic-ai-fault-taxonomy-2026.md
    │   ├── retrieval-strategy-research-2025-2026.md
    │   ├── context-degradation-research.md
    │   ├── long-context-degradation-and-prompt-injection-2026.md
    │   ├── google-model-failure-modes-2026.md
    │   ├── evidence-ranking-layer-2026.md
    │   ├── v7-strict-grounding-protocol.md
    │   ├── hermes-kanban-persistent-orchestration.md
    │   ├── karpathy-theoretical-concepts.md
    │   ├── enforcement-mechanism-analysis.md
    │   ├── soul-configuration-protocol.md
    │   ├── soul-template-bootstrap-v5-v7.md
    │   ├── soul-template-orchestrator-vs-v7.md
    │   ├── soul-template-v7-gaps-analysis.md
    │   ├── changelog.md                              # source: soul-management
    │   └── v73-validation-report.md
    ├── migration/
    │   ├── FROM-SOUL-MANAGEMENT.md
    │   └── DEPLOY.md                                 # new deploy workflow
    └── decisions/
        ├── engine-refactor-rationale.md             # why 5 skills, why hybrid
        └── limitations.md                            # known constraints (Issue #2045 dep)
```

---

## Validation Self-Tests (Per Release)

```bash
# 1. Skill files exist + have frontmatter
for skill in grounded-research-protocol context-hygiene provenance-tracing failure-mode-recovery-core task-domain-router; do
  test -f "$(pwd)/skills/core/$skill/SKILL.md" \
    && grep -q '^---' "$(pwd)/skills/core/$skill/SKILL.md" \
    && echo "$skill: OK" || echo "$skill: FAIL"
done

# 2. Template files balanced
for tmpl in templates/SOUL_TEMPLATE_*.md; do
  echo "$tmpl: open=$(grep -c '<section_' $tmpl) close=$(grep -c '</section_' $tmpl)"
done

# 3. Skill pointer count
echo "V7 skill pointers: $(grep -c 'skill://' templates/SOUL_TEMPLATE_V7.md)"
echo "V73 skill pointers: $(grep -c 'skill://' templates/SOUL_TEMPLATE_ORCHESTRATOR_V73.md)"

# 4. Anchor block presence (recency-bias)
grep -A 2 'high_attention_anchors' templates/SOUL_TEMPLATE_*.md | head -20

# 5. Reference docs present (audit trail)
ls docs/references/ | wc -l  # should be ≥ 17
```

All five should pass before any release is tagged.

---

## Contributing

Issues, PRs, and discussion welcome at `https://github.com/degidevops/soul-engine/issues`.

For skill-content proposals: follow `skill_management` YAML frontmatter + numbered steps + pitfalls section pattern. Validate with `hermes skills install <local-clone>` before pushing.

For template-content proposals: write_file full (not patch) per soul-management pitfalls section. Validate XML tag balance before pushing.

---

## License

MIT — see LICENSE.

---

## Acknowledgments

- **Hermes Agent team** (Nous Research) — engine architecture + `skill_view()` API.
- **Anthropic** — prompt caching spec + Claude API contract.
- **arXiv 2603.06847** (Shah et al. fault taxonomy) — failure mode credit.
- **arXiv 2505.15962 + MDPI 2025** — parametric-knowledge empirical basis.
- **arXiv 2307.03172** (Liu et al. Lost in the Middle) — anchoring attention research basis.
- **Robert C. Martin** — single-responsibility principle for skill composition.
- **MindStudio / Zylos Research** — modular skill architecture pattern.
- **Regolo.AI 2026 benchmark** — decomposition efficacy empirical.

Last updated: 2026-06-23.
