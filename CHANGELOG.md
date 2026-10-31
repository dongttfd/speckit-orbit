# Changelog

## 0.1.1 - 2026-10-31

- Raise minimum supported Spec Kit version to 1.2.5.
- Add current extension metadata: category, effect, homepage, and repository owner.
- Add `config_schema` for Orbit configuration.
- Expand manifest defaults so the complete shipped configuration has a defined base layer.
- Align configuration handling with Spec Kit precedence: defaults → project → local → environment.
- Accept both the v1.2.5 runtime local-config path (`local-config.yml`) and the documented `<ext>-config.local.yml` form.
- Define schema-derived environment override names to avoid v1.2.5 underscore-splitting ambiguity for keys such as `max_cycles`.
- Clarify that Orbit provides artifact-aware continuation, not workflow-engine checkpoint/resume semantics.
- Refresh README terminology and compatibility notes for Spec Kit 1.2.5.

## 0.1.0

- Initial Spec Kit Orbit extension.
- Targets Spec Kit >=1.2.3.dev0,<2.0.0.
- Adds full SDD orchestration through Converge.
- Adds bounded Implement ⇄ Converge loop.
- Adds artifact-aware continuation semantics.
- Adds read-only status and final review commands.
- Avoids legacy fixed-heading checks used by older orchestrators.
