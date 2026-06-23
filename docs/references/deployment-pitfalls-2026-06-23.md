# SOUL Deployment Pitfalls — 2026-06-23

## Pitfall 1: Agent bypassed skill workflow entirely

**Symptom:** Agent read template directly via `read_file` instead of loading the soul-management skill via `skill_view` first. Agent then improvised the deployment workflow, ignoring the standardized steps.

**Root cause:** Agent treated the task as "edit a file" rather than "follow a skill workflow." The skill was available and matched the task, but the agent didn't load it.

**Lesson:** When a skill matches the task, ALWAYS load it via `skill_view` first and follow its workflow step-by-step. Do not improvise.

## Pitfall 2: Placeholder format not followed

**Symptom:** Agent filled `{{SYSTEM_POSITION}}` with free-form text `"Maru — default profile. Host: WSL (Windows Subsystem for Linux). User: Degi."` instead of the mandatory format `"[Role] for [context] inside [system]; [function] only; [key constraint]."`

**Root cause:** Agent read the template guidance and example but didn't treat the format as mandatory. The SKILL.md already said "follow the format specified" but didn't replicate the format string inline for the Worker workflow (only the Orchestrator workflow had it).

**Lesson:** Format strings from template guidance MUST be treated as mandatory constraints, not suggestions. The skill now replicates key format strings inline in the Worker workflow to prevent this.

## Pitfall 3: Comment handling confusion

**Symptom:** Agent initially preserved "section divider" comments (e.g., `<!-- SECTION A: IDENTITY -->`) based on an older version of the skill, then later removed all comments after user clarification.

**Root cause:** The skill previously had a "Comment Taxonomy" that distinguished three comment types. This was overly complex and led to inconsistent behavior. The skill has since been simplified to "Remove all HTML comments."

**Lesson:** Simpler rules are better. "Remove all HTML comments" is unambiguous. When updating the skill, ensure the template instructions and skill instructions are consistent.

## Pitfall 4: sed/awk for multi-line XML

**Symptom:** Agent used `sed` to replace placeholders in multi-line XML content, producing malformed output with empty tags and missing sections.

**Root cause:** `sed` is line-oriented and cannot reliably handle XML tags spanning multiple lines or containing special characters.

**Lesson:** ALWAYS use `write_file` to write the entire deployed SOUL.md from scratch. Never use `sed`/`awk`/`patch` for multi-line XML content.
