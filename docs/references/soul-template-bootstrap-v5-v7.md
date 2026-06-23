# SOUL Template Bootstrap v5 vs v7

## Observed lesson
- A profile can preserve its self-identification better when the bootstrap template includes an instruction guide and a bootstrap wrapper, not just semantic tags.
- For Travis, the compact v5-style bootstrap flow kept the profile identity clearer than the earlier v7 presentation that looked like pure XML/config.

## Practical rule
- Use v7 semantics, but keep a v5-like bootstrap shell:
  - instruction header
  - placeholder instructions
  - root wrapper distinction between template and deployed config
  - explicit finalization step

## Deployment distinction
- Template form: `<soul_template>`
- Deployed profile form: `<soul_config>`
- Keep XML-style semantic tags inside both forms; do not collapse the template into pure XML.

## Why this matters
- The wrapper and instructions are part of the bootstrap behavior.
- Removing them can make the template feel like a final config rather than a template to be instantiated.
- For profiles with identity constraints, the bootstrap framing helps preserve the intended persona/domain during migration.
