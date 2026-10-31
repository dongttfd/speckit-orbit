---
description: "Perform a read-only final review after convergence."
---

# Spec Kit Orbit — Final Review

Perform a read-only review of the current feature after the latest `__SPECKIT_COMMAND_CONVERGE__` result.

## Preconditions

Require:

- current `spec.md`;
- current `plan.md`;
- current `tasks.md`;
- a ready constitution when the project uses one;
- evidence that the latest Converge outcome is `converged`.

If the feature has not converged, stop and recommend `__SPECKIT_COMMAND_ORBIT_RUN__` instead of pretending final review is meaningful.

## Review dimensions

Check only what can be supported by repository/artifact evidence:

1. Requirement coverage
   - user stories and acceptance scenarios;
   - functional requirements;
   - buildable success criteria.

2. Plan adherence
   - architecture/technical decisions actually represented in the implementation;
   - no material implementation divergence hidden from the plan.

3. Constitution compliance
   - MUST-level principles;
   - required quality/testing/security constraints.

4. Task integrity
   - completed task markers agree with repository evidence;
   - no obviously unfinished task represented as complete.

5. Test and validation evidence
   - report tests that actually ran or are evidenced;
   - never infer passing tests from the existence of test files.

6. Unrequested scope
   - identify material behavior implemented without support in spec/plan/tasks.

## Severity

Use:

- BLOCKING — feature should not be considered done.
- IMPORTANT — should be corrected soon but does not invalidate the converged implementation.
- NOTE — non-blocking observation.

## Output

Return:

- verdict: CLEAN or BLOCKING;
- concise findings table with evidence;
- requirement/task traceability summary;
- verified test evidence;
- recommended next action.

This command MUST NOT:

- modify application code;
- modify `spec.md`, `plan.md`, `tasks.md`, or constitution;
- append remediation tasks directly;
- claim cross-model review unless another model actually performed the review.

If BLOCKING findings require implementation work, hand control back to `__SPECKIT_COMMAND_ORBIT_RUN__` so the change re-enters the governed Tasks → Implement → Converge path.
