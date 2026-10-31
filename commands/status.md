---
description: "Inspect the current feature and show the next Orbit phase without changing files."
---

# Spec Kit Orbit — Status

This command is strictly read-only.

Inspect the current Spec Kit project and active feature. Do not modify source code, configuration, `spec.md`, `plan.md`, `tasks.md`, checklists, or the constitution.

Report:

| Phase | State | Evidence |
|---|---|---|
| Constitution | ready / missing / template / unknown | concise evidence |
| Specify | complete / incomplete / stale | concise evidence |
| Clarify | complete / incomplete / stale / not-needed | concise evidence |
| Plan | complete / incomplete / stale | concise evidence |
| Checklist | complete / incomplete / disabled | concise evidence |
| Tasks | complete / incomplete / stale | concise evidence |
| Analyze | current / stale / not-run / unknown | concise evidence |
| Implement | complete / incomplete / unknown | completed vs pending tasks |
| Converge | converged / tasks-appended / not-run / stale | latest known outcome |
| Final Review | clean / blocking / not-run / disabled | concise evidence |

Rules:

- Do not use legacy hard-coded `plan.md` heading requirements.
- File existence alone is not sufficient evidence of semantic completion.
- If upstream artifacts are newer or materially changed, downstream phases are stale.
- Do not claim Analyze or Converge ran merely because artifacts look good.
- If no active feature can be identified, say so without creating one.

Finish with exactly one recommended next phase, preferably `__SPECKIT_COMMAND_ORBIT_RUN__` when continuation is appropriate.
