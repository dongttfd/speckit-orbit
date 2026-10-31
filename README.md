# Spec Kit Orbit

**Orbit** is an end-to-end orchestration extension for GitHub Spec Kit **1.2.5+**.

It follows the Fleet-style idea of one agent command that carries a feature through the complete SDD lifecycle, but it is designed around the current Spec Kit core — especially the mandatory **Implement ⇄ Converge** loop.

## Why Orbit

Current Spec Kit separates two primitives:

- **Extensions** add commands, hooks, templates, scripts, configuration, and agent-facing behavior.
- **Workflows** provide CLI-managed multi-step automation with persisted checkpoints, pause/resume, branching, and loops.

Orbit intentionally stays an **extension command** because its primary use case is running inside the coding agent that is already open (for example Codex via `$speckit-orbit-run`). It does **not** claim the exact persisted checkpoint/resume guarantees of `specify workflow resume`.

Orbit's continuation model is artifact-aware: on a later invocation it inspects the current feature artifacts and available execution evidence, then continues from the earliest incomplete or stale phase. If a prior phase leaves no durable evidence, Orbit must report that state as unknown/not-run instead of pretending it can reconstruct workflow-engine state.

## Lifecycle

```text
Constitution readiness
        ↓
     Specify
        ↓
     Clarify
        ↓
       Plan
        ↓
    Checklist
        ↓
      Tasks
        ↓
     Analyze
        ↓
   Implement
        ↓
    Converge ── tasks appended ──┐
        │                        │
        └──── converged          └──→ Implement → Converge
                 ↓
            Final Review
                 ↓
                DONE
```

Clarify, Checklist, and Analyze are quality gates in Spec Kit. Orbit enables them by default because its goal is the complete guarded SDD flow; they can be disabled in Orbit configuration when appropriate.

## Commands

| Command | Purpose |
|---|---|
| `speckit.orbit.run` | Run or continue the full lifecycle |
| `speckit.orbit.status` | Read-only phase/status inspection |
| `speckit.orbit.review` | Read-only post-convergence review |

Invocation syntax depends on the active integration. For Codex, Spec Kit renders these as:

```text
$speckit-orbit-run
$speckit-orbit-status
$speckit-orbit-review
```

Orbit command files use Spec Kit command-reference tokens so core commands render into each integration's native syntax.

## Compatibility

Current extension target:

```text
Spec Kit >=1.2.5,<2.0.0
```

The minimum is intentionally **1.2.5**. Orbit relies on top-level extension `defaults`, and Spec Kit 1.2.5 includes the fix that correctly loads top-level defaults from `extension.yml`.

Required core commands:

```text
constitution
specify
clarify
plan
checklist
tasks
analyze
implement
converge
```

Orbit deliberately avoids legacy fixed-heading checks such as `## Architecture` or `## Tech Stack`.

## Install for local development

Orbit 0.1.1 requires Spec Kit 1.2.5 or newer. Check first:

```bash
specify --version
specify self check
```

From a Spec Kit project:

```bash
specify extension add --dev /absolute/path/to/speckit-orbit
```

Check installation:

```bash
specify extension list
```

For Codex, confirm the rendered skills exist under:

```text
.agents/skills/speckit-orbit-run/SKILL.md
.agents/skills/speckit-orbit-status/SKILL.md
.agents/skills/speckit-orbit-review/SKILL.md
```

## Configuration

Spec Kit installs/scaffolds the Orbit project config under:

```text
.specify/extensions/orbit/orbit-config.yml
```

Configuration follows the Spec Kit extension precedence model, where later layers win:

1. `extension.yml` → `defaults`
2. `.specify/extensions/orbit/orbit-config.yml`
3. local override: `.specify/extensions/orbit/local-config.yml` (v1.2.5 runtime) and, for forward/doc compatibility, `.specify/extensions/orbit/orbit-config.local.yml`; if both exist, the Orbit-specific documented file wins
4. supported `SPECKIT_ORBIT_*` environment overrides

The manifest includes a `config_schema` so invalid configuration can be rejected instead of silently producing ambiguous orchestration behavior. Note: the published v1.2.5 docs show `<ext>-config.local.yml`, while the v1.2.5 runtime `ConfigManager` reads `local-config.yml`; Orbit accepts both paths. Environment overrides use canonical names derived from schema leaf paths (for example `converge.max_cycles` → `SPECKIT_ORBIT_CONVERGE_MAX_CYCLES`) and are type-coerced according to `config_schema`, avoiding v1.2.5's generic underscore-splitting ambiguity.

Modes:

- `guarded` — pauses at major gates and material ambiguities.
- `balanced` — default; pauses only when human judgment materially changes intent.
- `autonomous` — continues unless blocked by governance, safety, permissions, or unsupported high-impact decisions.

## Design principles

1. **Core commands remain authoritative.** Orbit orchestrates; it does not replace Specify, Analyze, Implement, or Converge.
2. **Convergence is mandatory.** Implementation is not done just because all original task boxes are checked.
3. **Artifact-aware continuation.** Orbit infers the next safe phase from current artifacts; it does not claim workflow-engine checkpoint resume.
4. **No invented evidence.** Orbit never claims tests or phases ran unless there is evidence.
5. **Governance wins.** Autonomous mode cannot override the constitution.
6. **Bounded loops.** Convergence has a configurable maximum cycle count and stops on repeated unresolved findings.
7. **Portable command references.** Command files use Spec Kit command-reference tokens so installed integrations render native invocation syntax.
8. **Current extension contract.** Metadata includes process category/effect, configuration defaults, and configuration schema following current extension-development guidance.

## Project status

Version: `0.1.1`

Targeted and reviewed against Spec Kit **v1.2.5**, the latest stable release as of 2026-10-31.
