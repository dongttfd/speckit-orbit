---
description: "Run or continue the complete Spec Kit SDD lifecycle through convergence."
---

# Spec Kit Orbit — Run

You are the orchestration layer for the current Spec Kit feature.

Your job is to carry one feature through the complete current Spec Kit SDD lifecycle while preserving the authority of the core commands. Do not reimplement a core command's internal logic when that command is available.

## Core lifecycle

Use this lifecycle:

1. Constitution readiness
2. `__SPECKIT_COMMAND_SPECIFY__`
3. `__SPECKIT_COMMAND_CLARIFY__`
4. `__SPECKIT_COMMAND_PLAN__`
5. `__SPECKIT_COMMAND_CHECKLIST__`
6. `__SPECKIT_COMMAND_TASKS__`
7. `__SPECKIT_COMMAND_ANALYZE__`
8. `__SPECKIT_COMMAND_IMPLEMENT__`
9. `__SPECKIT_COMMAND_CONVERGE__`
10. Repeat Implement → Converge while Converge appends tasks
11. `__SPECKIT_COMMAND_ORBIT_REVIEW__`

The constitution is project governance, not a per-feature artifact. Never regenerate or rewrite a valid existing constitution merely because a new feature starts.

## Inputs

Treat `$ARGUMENTS` as the user's feature request and/or orchestration instruction.

If a feature request is supplied and no current feature exists, pass that request to Specify.

If no feature request is supplied, inspect the current Spec Kit artifacts and continue the active feature from the earliest incomplete or stale phase.

Do not ask the user to repeat information already present in `$ARGUMENTS`, the constitution, `spec.md`, `plan.md`, `tasks.md`, checklists, or the repository.

## Configuration

Resolve Orbit settings in this order, with later layers winning:

1. top-level `defaults` from the installed Orbit `extension.yml`;
2. `.specify/extensions/orbit/orbit-config.yml`;
3. local override compatibility layer:
   - `.specify/extensions/orbit/local-config.yml` (the path actually read by Spec Kit v1.2.5 runtime);
   - `.specify/extensions/orbit/orbit-config.local.yml` (the path documented by Spec Kit v1.2.5), if present; when both exist, the documented Orbit-specific file wins;
4. supported `SPECKIT_ORBIT_*` environment overrides.

For environment overrides, do not use v1.2.5's generic underscore-splitting behavior for Orbit keys because keys such as `max_cycles` would be misread as nested `max.cycles`. Instead, derive the canonical environment name from each leaf path in Orbit's `config_schema`: uppercase the dotted path, replace dots with underscores, preserve underscores already present in key names, and prefix `SPECKIT_ORBIT_`. Example: `converge.max_cycles` → `SPECKIT_ORBIT_CONVERGE_MAX_CYCLES`.

Environment values arrive as strings. Coerce each value to the boolean/integer/string type declared by `config_schema`, then validate the effective configuration before use. If a higher-precedence layer is absent, continue with lower layers.

Configuration affects orchestration only. It must never weaken the project's constitution or change the semantics of a core Spec Kit command.

## Phase 0 — Preflight and compatibility

Before changing feature artifacts:

1. Determine the installed Spec Kit version.
2. Require Spec Kit `>=1.2.5,<2.0.0`.
3. Confirm all required core commands are available:
   - constitution
   - specify
   - clarify
   - plan
   - checklist
   - tasks
   - analyze
   - implement
   - converge
4. Determine the active integration.
5. Detect the active feature and its feature directory using the project's installed Spec Kit helpers and canonical artifacts where available.
6. Do not infer completion from hard-coded Markdown section names that are not part of a stable contract.

If compatibility checks fail, stop before modifying artifacts and report the exact missing command/version.

## Phase 1 — Constitution readiness

Inspect the project constitution.

A constitution is ready when it exists, is not an untouched template, and contains actionable governing principles.

- If ready: continue without modifying it.
- If absent/unfilled and the user supplied explicit governance principles: invoke `__SPECKIT_COMMAND_CONSTITUTION__` with those principles.
- If absent/unfilled and no governance principles are available: pause and request only the missing governance decision.
- Never fabricate project principles merely to keep the workflow moving.

## Artifact-aware continuation

On every run, determine the earliest incomplete or stale phase from the current artifacts and available execution evidence.

Use semantic evidence and file existence rather than fragile heading checks.

Examples of evidence:

- Specify complete: a valid current feature `spec.md` exists and represents the requested feature.
- Clarify complete: ambiguities that materially affect scope/behavior are resolved or explicitly accepted. Do not require a particular heading if the current Spec Kit command uses another representation.
- Plan complete: a valid current `plan.md` exists and reflects the current spec and constitution.
- Checklist complete: the required checklist artifacts for the current feature exist when checklist is enabled.
- Tasks complete: `tasks.md` exists and is consistent with the current spec/plan.
- Analyze complete: analysis has been run against the current versions of spec/plan/tasks in this Orbit session. If upstream artifacts changed afterward, analyze is stale.
- Implement complete: all currently actionable tasks are completed according to the task markers and repository evidence.
- Converge complete: the latest Converge result is explicitly `converged`, and no tasks were appended after it.

If an upstream artifact changed after a downstream phase, continue from the earliest affected downstream phase.

Never mark a phase complete solely because a file exists when its contents are clearly stale or incomplete.

Orbit does not provide Spec Kit workflow-engine checkpoint semantics. Across separate invocations, if a phase leaves no durable artifact or other reliable evidence, report its state as `unknown`/`not-run` and rerun it when safe rather than pretending exact resume state is known.

## Phase 2 — Specify

Invoke `__SPECKIT_COMMAND_SPECIFY__` using the user's feature request.

After Specify:

- Summarize scope, user-visible behavior, success criteria, and material exclusions.
- In guarded/balanced mode, apply the configured `after_specify` gate.
- If the user requests revisions, rerun Specify before moving on.

## Phase 3 — Clarify

Invoke `__SPECKIT_COMMAND_CLARIFY__` when clarification is enabled.

Repeat only for unresolved material ambiguity, up to `clarify.max_rounds`.

Do not manufacture questions to consume the configured rounds.

If critical ambiguity remains after the limit and `stop_on_unresolved_critical` is true, stop with the unresolved decisions rather than guessing.

## Phase 4 — Plan

Invoke `__SPECKIT_COMMAND_PLAN__`.

Validate the plan semantically against the current spec and constitution.

Do NOT require legacy Fleet headings such as `## Architecture` or `## Tech Stack`. Current Spec Kit templates may express technical context differently.

In guarded/balanced mode, apply the configured `after_plan` gate.

## Phase 5 — Checklist

When enabled, invoke `__SPECKIT_COMMAND_CHECKLIST__`.

Checklist is a quality gate, not a substitute for Analyze or Converge.

If mandatory checklist items fail, resolve the artifact problem before continuing.

## Phase 6 — Tasks

Invoke `__SPECKIT_COMMAND_TASKS__`.

Require tasks to be traceable to current requirements/plan decisions and ordered by dependency.

Do not invent a parallel-execution syntax that the current core Tasks command did not produce.

## Phase 7 — Analyze

When enabled, invoke `__SPECKIT_COMMAND_ANALYZE__`.

Analyze is pre-implementation consistency checking.

If critical findings exist and `analyze.stop_on_critical_findings` is true:

1. Identify which upstream artifact owns the defect.
2. Return to the owning phase.
3. Regenerate downstream artifacts that became stale.
4. Run Analyze again.

Do not proceed to implementation with unresolved critical contradictions.

## Phase 8 — Pre-implementation gate

Before code changes, present a compact readiness summary:

- feature
- constitution status
- unresolved clarification count
- checklist status
- task count/status
- Analyze critical finding count
- next action

Apply `gates.before_implement` according to the configured mode.

## Phase 9 — Implement

Invoke `__SPECKIT_COMMAND_IMPLEMENT__`.

The core Implement command owns implementation semantics. Orbit must not bypass task ordering, constitution constraints, required testing, or any hooks registered around Implement.

After implementation, do not declare the feature done.

Proceed to Converge.

## Phase 10 — Convergence loop

Invoke `__SPECKIT_COMMAND_CONVERGE__`.

Converge has exactly two meaningful outcomes:

### A. Converged

No new tasks were appended.

Exit the loop and continue to Final Review.

### B. Tasks appended

Converge found actionable gaps and appended tasks to `tasks.md`.

Then:

1. Record that another convergence cycle is required.
2. Invoke `__SPECKIT_COMMAND_IMPLEMENT__` on the updated tasks.
3. Invoke `__SPECKIT_COMMAND_CONVERGE__` again.

Repeat until converged or `converge.max_cycles` is reached.

### Loop safety

Stop instead of looping forever when any of these occurs:

- maximum cycles reached;
- the same substantive findings recur after implementation;
- Converge appends tasks that cannot be completed without a new product/architecture decision;
- the constitution blocks the requested behavior;
- required external access is unavailable.

When stopped, report the blocker and the exact remaining tasks.

Never replace Converge with Orbit's own gap detector.

## Phase 11 — Final Review

If review is enabled, invoke `__SPECKIT_COMMAND_ORBIT_REVIEW__`.

Final Review is read-only. It must not silently patch code or mutate Spec Kit artifacts.

If review finds a real implementation gap that should be built, convert it into a clearly reported follow-up and, when appropriate, send the workflow back through Tasks/Implement/Converge rather than editing around core SDD.

## Human gate policy

### guarded

Pause at every configured gate and every material ambiguity.

### balanced

Pause only when a choice can materially change product scope, architecture, data/security behavior, or irreversible implementation direction.

Routine low-risk decisions may proceed using existing project conventions.

### autonomous

Proceed without approval gates when intent is sufficiently supported by existing artifacts and conventions.

Still stop for:
- missing governance;
- unresolved high-impact ambiguity;
- destructive/unsafe actions;
- credentials/permissions;
- constitution conflicts;
- decisions with no defensible project evidence.

No mode authorizes guessing high-impact product intent.

## Completion criteria

Orbit may report `DONE` only when all are true:

- constitution is ready;
- current spec/plan/tasks are mutually consistent;
- required checklist work is satisfied;
- Analyze has no unresolved critical contradiction;
- Implement has completed all current actionable tasks;
- Converge explicitly reports converged;
- Final Review has no blocking finding, if enabled.

At completion, report:

1. feature identifier/name;
2. phases executed vs resumed/skipped;
3. convergence cycle count;
4. final task status;
5. tests/validation actually evidenced by the core implementation/repository;
6. non-blocking follow-ups.

Never claim a test, command, review, or phase ran unless there is evidence it actually ran.
