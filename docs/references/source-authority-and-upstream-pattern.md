# Source Authority & Upstream Recognition Blueprint

## Purpose
Lessons learned 2026-06-23 when an agent treated filesystem copies as canonical and missed the git upstream repo. Distinguish: (a) where is the source of truth, (b) which paths are sync copies, (c) which paths are downstream consumers.

## Source Authority Matrix (soul-management / soul-engine cluster)

| Path | Type | Authority | Editable? |
|---|---|---|---|
| `https://github.com/degidevops/soul-management` | Upstream GH repo | CANONICAL (legacy) | Edit via `write_file` + `git commit` + `gh pr merge`. Pre-replacement. |
| `~/.hermes/skills/devops/soul-management/.git/` | Local git checkout | CANONICAL (filesystem mirror of upstream) | SAME as upstream — never edit files in working tree without committing. |
| `~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_*.md` | Templates inside canonical repo | CANONICAL | Edit via upstream workflow (skill text → PR → merge). |
| `~/.hermes/SOUL_TEMPLATE_*.md` | Sync copy of canonical files | SECONDARY | Sync from upstream. Editing directly = drift. |
| `~/.hermes/SOUL_TEMPLATE_ORCHESTRATOR.md` | Pointer file (V73 redirect) | SECONDARY | Edit to update pointer target. |
| `~/.hermes/profiles/<name>/plugins/soul-management/` | Per-profile clone (often git submodule) | SECONDARY | Sync from upstream. If upgrading repo, repoint submodules. |
| `~/.hermes/profiles/<name>/SOUL.md` | Deployable profile (instance) | PROFILE-LOCAL | Edit freely per deployment workflow. |

## Failure Mode: Silent Drift (2026-06-23 incident)

**Symptom in session:** Agent was asked to refactor templates. It edited `~/.hermes/SOUL_TEMPLATE_V7.md` directly, treating it as canonical. It also created skill files under `~/.hermes/skills/devops/soul-engine/core/*` directly. It did NOT notice that `~/.hermes/skills/devops/soul-management/.git/` existed pointing to `https://github.com/degidevops/soul-management.git`. All template edits were going into the local-sync copy, not the upstream repo. Skill files were created in a path with no git backing at all.

**Why it happened:** The soul-management SKILL.md "Template File Inventory" table stated canonical path = `skills/devops/soul-management/templates/`. But the agent took a shortcut via the `~/.hermes/` sync copy, which the table also labels as a sync path. The synchronization between the two paths was not enforced — there is no hermes-managed inviolable link.

**Why it stayed hidden:** No `git status` was run early in the session. The agent read `~/.hermes/SOUL_TEMPLATE_V7.md`, found it byte-identical to the file in `skills/devops/soul-management/templates/`, and concluded no canonical-vs-sync distinction mattered. This matched the visible truth but missed the upstream-repo truth.

**Cost of the drift:**
1. All template edits + new skill files were at risk of being overwritten on the next sync.
2. The work was invisible to anyone tracking the upstream repo.
3. The agent had to redo the work in `/tmp/soul-engine` clone after user redirected.
4. The session became much longer than necessary.

## Detection Protocol (load this section on every soul-management edit)

**Run BEFORE any read/write of templates, skills, or profile-level deployment files:**

```bash
# 1. Identify upstream git presence
for d in ~/.hermes/skills/devops/soul-management \
         ~/.hermes/skills/devops/soul-engine \
         ~/.hermes/profiles/*/plugins/soul-management; do
  if [ -d "$d/.git" ]; then
    echo "GIT: $d"
    cd "$d" && git remote -v 2>&1 | head -2
  fi
done

# 2. Compare upstream HEAD vs sync copy if both exist
if [ -d ~/.hermes/skills/devops/soul-management/.git ] && [ -f ~/.hermes/SOUL_TEMPLATE_V7.md ]; then
  echo "--- divergence check ---"
  diff -q ~/.hermes/SOUL_TEMPLATE_V7.md \
           ~/.hermes/skills/devops/soul-management/templates/SOUL_TEMPLATE_V7.md
fi
```

If `diff` returns nothing → files are in sync, edit either but commit to upstream. If `diff` shows divergence → STOP and ask user which to resolve.

## Replacement / Archive Workflow (when user authorizes soul-management → soul-engine transition)

```
Step 1  Create new repo:    gh repo create <org>/soul-engine --public --add-readme
Step 2  Clone:              git clone https://github.com/<org>/soul-engine /tmp/<name>
Step 3  Branch:             git checkout -b feat/engine-refactor
Step 4  Populate:           write_file all skill contents + CHANGELOG + MIGRATION + README
Step 5  Commit per phase:   git commit -m "feat(scope): ..."
Step 6  Push branch:        git push origin feat/engine-refactor
Step 7  Create PR:          gh pr create --base main
Step 8  Merge PR:           gh pr merge --merge
Step 9  Document migration: Append migration notice to OLD repo's README pointing at NEW repo
Step 10 Archive OLD:         gh repo archive <org>/soul-management --confirm
Step 11 Submodule repoint:  In each profile's plugins/soul-management submodule:
                              git submodule set-url <new-url>
                              OR add new submodule soul-engine and remove old
```

NEVER do steps out of order. NEVER archive the old repo before the new repo has a published CHANGELOG + MIGRATION.md. Orphan references in user docs/wikis/sessions will cause persistent confusion.

## Why Soul-Management and Soul-Engine Both Exist During Transition

- **soul-management** is the legacy skill — original V7 + V73 template engine. Has a `degidevops/soul-management` upstream repo.
- **soul-engine** is the modular decomposition. Lives at `degidevops/soul-engine` upstream repo + `~/.hermes/skills/devops/soul-engine/core/*` for skill cluster.

Both are loaded this session because the user is mid-transition. Once `degidevops/soul-engine` is published with full migration docs, soul-management will be archived and the canonical skill cluster will sole-source via `soul-engine/core/*`.

A future agent MUST NOT assume soul-management is canonical until they verify the transition state — the table of template inventory above is dynamic.

## Verification

- After loading this skill, check that the user's GitHub account has write access to both repos (they own them).
- Verify the gh CLI is on path and authenticated.
- Confirm both repos exist with `gh repo view`.
- Diff-check sync copies against canonical before any edit.
