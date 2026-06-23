# CHANGELOG — soul-engine

All notable changes to this project documented here. Format follows [Keep a Changelog](https://keepachangelog.com/) conventions.

## [1.0.0] - 2026-06-23 — Init Public Release

### Status
- **Public release** of `soul-engine`, superseding the archived `soul-management`.
- **Backward-compatible** with all V7/V73 SOUL.md profiles deployed before this date.

### Changed
- **`templates/SOUL_TEMPLATE_V7.md`** — refactored 476 → 316 baris (-33%). Steps 1, 6, 8 etc. extracted to `skills/core/*.md` via skill pointer syntax. `<failure_modes>` collapsed to `<minimal_index>` block referencing M1-M7 in `failure-mode-recovery-core`. `<in_scope>` reduced via `task-domain-router` pointer.
- **`templates/SOUL_TEMPLATE_ORCHESTRATOR_V73.md`** — refactored similarly. `<failure_modes>` now shows M1-M7 minimal index + M8-M12 inline (orchestrator-specific). Step 10 partially extracted: shared spec to `provenance-tracing`, `<kanban_trace>` retained inline (dispatcher coupling).

### Added
- **`skills/core/grounded-research-protocol/SKILL.md`** — extracted from V7/V73 Step 1. Includes 4-stage extraction hierarchy, Contract Validation, Semantic Grounding, Evidence Ranking Layer, Failure Protocol, Instruction Contamination Guard (OWASP LLM01:2025).
- **`skills/core/context-hygiene/SKILL.md`** — extracted from V7/V73 Step 6/8. Includes distillation protocol, anchor templates (4 V7 / 6 V73), 4 intervention strategies, Liu 2023 Lost-in-Middle notes.
- **`skills/core/provenance-tracing/SKILL.md`** — extracted from V7 Step 8 + V73 Step 10 (shared portion). Includes 3 binary statuses, JSONL schema (8 fields), confidence→provenance rosetta, A2A protocol coupling notes.
- **`skills/core/failure-mode-recovery-core/SKILL.md`** — extracted from V7/V73 `<failure_modes>` M1-M7. Includes 7 mode playbooks.
- **`skills/core/task-domain-router/SKILL.md`** — extracted from V7 `{{IN_SCOPE_ITEMS}}`. V7-only; V73 routes via Kanban.

### Migrated (Documentation Trail)
- 17 reference docs from `soul-management` → `docs/references/`. Full audit trail preserved.

### Research Basis Cited
- **arXiv 2603.06847** (Shah et al. fault taxonomy, March 2026) — 19.5% dependency failures, 16.2% coordination collapse.
- **arXiv 2505.15962** (parametric knowledge staleness) — epistemic basis.
- **arXiv 2510.05381** (Context Length Alone Hurts) — quantity limit derivation.
- **arXiv 2505.06120** (metacognition loss) — distillation trigger basis.
- **arXiv 2603.26707** (Google-specific Lost-in-Middle) — anchor re-injection justification.
- **arXiv 2307.03172** (Liu et al. original Lost-in-the-Middle) — U-curve empirical.
- **arXiv 2605.19196** (REFLECT benchmark) — LLM-as-judge invalidation.
- **Hermes Agent Issue #2045** (March 2026, unmerged) — lazy-skill-loading pathway awaited.
- **Hermes Agent Issue #10585** (April 2026, open) — multi-resolution context exploration.

---

## [Unreleased] — Pre-release Cycle (v0.1.0 → v0.9.0)

### v0.9.0 — 2026-06-23-PreRelease
- Decision matrix: 5 skills vs more. Engineered-trade-off analysis (vs all-or-nothing pilot, vs per-category split, vs hybrid).
- Confirmed: hybrid `core/` only. Worker-only/orchestrator-only deferred until Issue #2045 merge.
- Confirmed: V73 Steps 2, 3 (mission-critical orchestrator) DO NOT extract. Step 10 partial-extract only.

### v0.8.0 — 2026-06-23-Analysis
- Identified 5 skill extraction candidates via Capodieci + MindStudio + Zylos methodology.
- Tuning: excluded Step 6 wholesale (would break anchor recency-bias fix); retained tail anchor block.

### v0.7.0 — 2026-06-23-MigrationPlanning
- Eval'd all skill cluster options (per-category, full-merge, hybrid). Settled hybrid.
- Decision: install via `hermes skills install`, do NOT use git submodule (per user preference 2026-06-23).

### v0.5.0 — 2026-06-23-RepositoryBootstrap
- Created `degidevops/soul-engine` via `gh repo create`.
- Verified `gh` v2.95.0 auth as `degidevops` with full repo scope.

### v0.3.0 — 2026-06-23-TemplateAudit
- Audit V7 + V73 at canonical paths under `~/.hermes/skills/devops/soul-management/templates/`.
- Confirmed byte-identical with `~/.hermes/` sync paths.
- Confirmed `degidevops/soul-management` is 1-commit fresh repo; safe to archive.

### v0.1.0 — 2026-06-23-Proposal
- Initial proposal: decompose SOUL.md/SOUL_TEMPLATE_V7.md/SOUL_TEMPLATE_ORCHESTRATOR_V73.md into modular skills.
- Triangulated: forum discussion (r/hermesagent, r/LocalLLaMA), literature (arXiv 2603.06847, 2505.15962, 2307.03172, etc.), official docs (hermes-agent.nousresearch.com/docs/developer-guide/prompt-assembly).
- Empirical backing: Regolo.AI 2026 benchmark (-93.5% system prompt, +32% math accuracy post-decomposition).

---

## Archived History (soul-management)

`soul-management` was archived 2026-06-23 in favor of this `soul-engine` release. Original repo: `https://github.com/degidevops/soul-management`. Full commit history preserved in archive.

Last `soul-management` commit: `9d76c20 feat: initial commit — soul-management skill`.

---

## Future Roadmap

### v1.0.1 — Pending (Small Fixes)
- [ ] V73 template refactor post-port (verify size reduction).
- [ ] Add SKILL.md master pointer validation script.

### v1.1.0 — Pending (Issue #2045 Dependent)
- [ ] Wait for hermes-agent Issue #2045 (lazy skill loading) merge.
- [ ] After merge: extract V73 M8-M12 into `core/failure-mode-recovery-orch/`.
- [ ] After merge: extract V73 Step 2 (Control Plane) into orchestrator-only skill.
- [ ] Add `orchestrator-only/` directory tree.

### v1.2.0 — Pending
- [ ] Add per-skill vector retrieval index (Issue #22620 dependency).
- [ ] Add lazy tool schema integration (Issue #6839 dependency).
- [ ] Auto-generated MIGRATION docs via changelog-to-doc flow.

### v2.0.0 — Hypothetical (Breaking Changes)
- [ ] Schema migration if Hermes Issue #2045 ships incompatible API.
- [ ] Template versioning with `5-arg` semantic version (template-version + skill-version).

---

## Notes

- **No release engineering contract breakage** — versioned semver applied per Hermes convention.
- **Master pointer** at `skills/SKILL.md` on next iteration will surface this CHANGELOG for human auditors via skill-runtime tooling.
- **Auditability** — every released version records which paper-citation backs which decision. See `docs/references/` for full bibliography.
