# MIGRATION GUIDE — soul-management → soul-engine

Migrating from `soul-management` skill cluster (1.0 final, archived 2026-06-23) to `soul-engine` (1.0.0, current).

## Why Migrate

The `soul-engine` refactor was engineered on three empirical drivers:

1. **Template body bloat** — V7/V73 templates were 476 / 843 baris, ~7.500–15.500 token at-rest cost.
2. **Reusability friction** — protocol definitions duplicated across templates.
3. **Lazy-loading infrastructure prep** — Hermes Issue #2045 (March 2026) opened lazy skill loading; this release builds the content layer that will leverage it on merge.

If your profile deployed V7/V73 templates before 2026-06-23 and works well, the migration is **safe but recommended** for forward-compat.

## What Stays the Same

- **Template syntax** — XML-style semantic tags preserved.
- **Placeholder format** — `{{KEY_NAME}}` placeholders identical.
- **Deploy workflow** — read template, replace placeholders, write to profile, validate tag balance.
- **Deployment warning patterns** — `hermes profile list`, etc.
- **Section structure** — A/B/C tripartite preserved.
- **Behavior contracts** — all 12 failure modes, reasoning protocol, governance contract preserved verbatim where not extracted.

## What Changes

### 1. Skill Cluster
- **Before**: 1 master skill `soul-management` (umbrella) + 17 reference documents.
- **After**: 1 master skill `soul-engine` (umbrella) + 5 core skills + 17 reference documents (now under `docs/references/`).

### 2. Template Body
- **Before**: ~7.500 / 15.500 token templates with all detail loaded.
- **After**: ~5.250 / 13.300 token templates with detail extracted to skills.

### 3. Skill Discovery
- **Before**: Single skill `soul-management` at `~/.hermes/skills/devops/soul-management/`.
- **After**: 5 skills under `~/.hermes/skills/devops/soul-engine/core/<name>/`.

## Migration Steps

### Phase A — Pre-Migration Audit

Verify your current deployment is in sync with canonical soul-management:

```bash
# Check for active profile deployments using soul-management
ls ~/.hermes/profiles/*/SOUL.md 2>/dev/null

# Confirm reference docs exist
ls ~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_*.md

# Check git submodule pattern (if any)
git -C ~/.hermes/profiles/dave/plugins/soul-management remote -v
git -C ~/.hermes/profiles/travis/plugins/soul-management remote -v
```

### Phase B — Install soul-engine

```bash
gh repo clone degidevops/soul-engine ~/.local/src/soul-engine
# OR via Hermes:
hermes skills install https://github.com/degidevops/soul-engine
```

Validate install:

```bash
# Confirm 5 core skills installed
ls ~/.hermes/skills/devops/soul-engine/core/
# Expected:
#   context-hygiene
#   failure-mode-recovery-core
#   grounded-research-protocol
#   provenance-tracing
#   task-domain-router

# Confirm templates installed
ls ~/.hermes/skills/devops/soul-engine/templates/
# Expected:
#   SOUL_TEMPLATE_ORCHESTRATOR_V73.md
#   SOUL_TEMPLATE_V7.md
```

### Phase C — Profile Re-Deploy (Optional)

If your profile already had a working SOUL.md with V7 or V73 structure, **no immediate action needed** — the old SOUL.md works as-is. However for forward-compat:

1. Read `templates/SOUL_TEMPLATE_V7.md` (or V73) from `soul-engine`.
2. Compare with your profile's `~/.hermes/profiles/<name>/SOUL.md`.
3. The diffs will reveal which `(steps)` are now `skill://` pointers.
4. Two migration paths:
   - **Soft**: Keep existing profile SOUL.md, just install `soul-engine` skills for future-proof forward-compat.
   - **Hard**: Re-deploy profile from new template with skill pointers active.

### Phase D — Profile Sync Paths (dave, travis)

Both `~/.hermes/profiles/dave/plugins/soul-management/` and `~/.hermes/profiles/travis/plugins/soul-management/` are git submodules pointing to `degidevops/soul-management`.

Re-pointing submodules:

```bash
# For dave profile
cd ~/.hermes/profiles/dave/plugins/soul-management
git remote set-url origin https://github.com/degidevops/soul-engine.git
git pull origin master  # or main, depending on default branch
# Update internal symlinks/pointers if any

# Repeat for travis
cd ~/.hermes/profiles/travis/plugins/soul-management
git remote set-url origin https://github.com/degidevops/soul-engine.git
git pull origin master
```

OR for clean transition, replace submodule entirely:

```bash
# Backup first
mv ~/.hermes/profiles/dave/plugins/soul-management ~/.hermes/profiles/dave/plugins/soul-management.archived-2026-06-23
mv ~/.hermes/profiles/travis/plugins/soul-management ~/.hermes/profiles/travis/plugins/soul-management.archived-2026-06-23

# Re-install soul-engine as sibling plugin
gh repo clone degidevops/soul-engine ~/.hermes/profiles/dave/plugins/soul-engine
gh repo clone degidevops/soul-engine ~/.hermes/profiles/travis/plugins/soul-engine
```

### Phase E — Verify Migration

```bash
# All 5 skills available
hermes skills list | grep -E "grounded-research|context-hygiene|provenance-tracing|failure-mode-recovery|task-domain"
# Expected: 5 hits

# Profile SOUL.md still valid (backward-compat)
hermes profile list
# Expected: dave and travis profiles still load

# Skill pointer integrity (spot-check)
grep "skill://" ~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_*.md 2>/dev/null
# If hit, you still have soul-management installed. Uninstall via:
hermes skills uninstall soul-management  # if Hermes supports; else manual rm
```

### Phase F — Optional Cleanup

After confirming migration success:

```bash
# Remove archived soul-management
rm -rf ~/.hermes/skills/devops/soul-management
# Remove archived submodules
rm -rf ~/.hermes/profiles/dave/plugins/soul-management.archived-2026-06-23
rm -rf ~/.hermes/profiles/travis/plugins/soul-management.archived-2026-06-23
```

The `degidevops/soul-management` GitHub repo is archived (not deleted). Read-only access remains for historical reference.

## Risk Surface

| Risk | Mitigation |
|------|------------|
| Profile SOUL.md becomes invalid | Backward-compat verified; old SOUL.md works as-is until you re-deploy |
| Skill pointer resolution fails | Validate via `hermes skills list`; re-install via Step B if missing |
| Submodule re-point breaks profile | Use Phase D backup workflow before editing |
| Hidden profile-specific behavior drift | Compare with old template before Phase C re-deploy |

## When NOT to Migrate

- If you depend on `soul-management` internal `references/` paths (uncommon; was `/home/degi/...` style).
- If you have non-Hermes tool referencing `soul-management` skill cluster (re-brand instead).

## Support

- Issues: `https://github.com/degidevops/soul-engine/issues`
- Original repo (archived, read-only): `https://github.com/degidevops/soul-management`

---

Last updated: 2026-06-23.
