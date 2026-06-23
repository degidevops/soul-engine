# Changelog — soul-management

## [Unreleased] — 2026-06-23

### Fixed
- **Placeholder format compliance** — Added mandatory format check in both Worker (V7) and Orchestrator (V73) deployment workflows. Agent must now follow the exact format specified in each placeholder's GUIDANCE/Example. Previously, agents would fill placeholders with free-form text ignoring the specified format (e.g., `{{SYSTEM_POSITION}}` requires `"[Role] for [context] inside [system]; [function] only; [key constraint]."` but agents would write multi-sentence descriptions).
- **Post-deployment validation** — Added mandatory XML tag balance check (`grep -c '<section_'` vs `grep -c '</section_'`), placeholder leak check (no `{{` remaining), and HTML comment leak check. If any check fails, agent must fix immediately and NOT report success.
- **Simplified comment handling** — Removed "Preserve section divider comments" instructions from both Worker and Orchestrator workflows. All HTML comments (`<!-- ... -->`) are now removed on deployment without exception. XML-style tags (`<section_a>`, `<step n="0">`, etc.) already provide structural navigation — comment markers are redundant noise.
- **Pitfall: Follow instructions literally** — Added explicit pitfall: Agent MUST read and follow template instructions and guidance exactly. Do NOT improvise or use free-form text when a specific format is provided. This is the most common failure mode.
- **Pitfall: Use skill_view when directed** — Added explicit pitfall: When user says "use skill X", load it with `skill_view(name='X')` and follow its instructions directly. Do NOT improvise or skip steps.

### Root Cause Analysis
- **Problem 1:** Agent filled `{{SYSTEM_POSITION}}` with free-form text instead of the mandatory format.
- **Cause 1:** No enforcement step in the deployment workflow.
- **Fix 1:** Added explicit "Placeholder compliance check" step to both workflows.

- **Problem 2:** Conflicting instructions about comment handling.
- **Cause 2:** Ambiguous taxonomy created false distinctions.
- **Fix 2:** Simplified to "Remove all HTML comments".

- **Problem 3:** Agent used `sed` for multi-line XML manipulation, breaking section tags.
- **Cause 3:** `sed` is line-oriented and cannot reliably handle XML.
- **Fix 3:** Always use `write_file` to write the entire deployed file from scratch.

- **Problem 4:** Agent did not follow skill instructions when user said "use skill X".
- **Cause 4:** No explicit instruction about what to do when directed to use a specific skill.
- **Fix 4:** Added pitfall: load with `skill_view` and follow directly.

### Changed
- Worker deployment workflow: 8 → 9 steps
- Orchestrator deployment workflow: 8 → 9 steps
- Comment Taxonomy → simplified to "Comment Handling"
- Pitfalls: 5 → 7
