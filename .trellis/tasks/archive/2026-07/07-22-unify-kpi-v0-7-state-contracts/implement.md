# Implementation Plan

## Book Gate Plan

| Skill | Classification | Trigger | Phase | Gate state / reviewer status |
|---|---|---|---|---|
| `book-ddd-distilled-modeling` | on-demand | Parent task already confirmed the vocabulary and bounded contexts; this task does not change those boundaries | Planning review | `not-required` |
| `book-ddia-data-design` | required | Persisted Session State, state versioning, Environment observation, tool evidence, migration, Resume, and recovery semantics | Before task activation | `passed` / `confirmed` |
| `book-legacy-change-safety` | on-demand | No production behavior or weakly tested legacy code changes | N/A | `not-required` |
| `book-refactoring-pass` | on-demand | No existing production code changes | N/A | `not-required` |
| `book-release-readiness` | on-demand | No service/API/job/integration/deployment behavior changes | N/A | `not-required` |

## Ordered Changes

1. Add a canonical State Contract section to `docs/prd/PRD.md` near the command/session model.
   - Define every field, owner, enum, persistence rule, and derived state.
   - Add the Environment Mode decision order and effective-control matrix.
2. Repair PRD command and marker semantics.
   - `/sbtd on|off` mutate only `runtimeMode`.
   - `/sbtd strict|relaxed` mutate only `policyProfile`.
   - Runtime marker names the dimensions explicitly.
3. Repair PRD environment, Gate, validation, capability, Tool Evidence, Trellis, GitNexus, Evidence, and aggregate status sections.
   - Remove slash-separated ambiguous outcomes.
   - Keep reused words such as `blocked` only behind explicit field names.
4. Replace the PRD `SBTDSessionState` example with `SBTDSessionStateV1`.
   - Remove `enabled` and bare `mode`.
   - Add `stateVersion`, the three canonical persisted dimensions, observation time, and namespaced Gate records.
5. Update PRD requirements, acceptance criteria, success metrics, and state-related open decisions to point to the canonical contract.
6. Apply the identical taxonomy and transition behavior to `docs/prd/ROADMAP.md`.
   - P0 command acceptance and Session State.
   - Gate/Rule/Validation/Tool Evidence work items.
   - P1 Doctor/config/status output.
   - P2 migration and state conformance.
   - P3 capability matrix.
   - Test matrix, release blockers, exit criteria, and Definition of Done.
7. Run a cross-document conformance pass and tighten wording without changing unrelated product decisions.

## Validation

1. Targeted forbidden-pattern searches must return no contract ambiguity:
   - persisted `enabled`
   - bare `mode: "strict" | "relaxed"`
   - `needs-onboard/degraded`
   - `blocked/degraded`
   - prose saying missing Adapter “needs-onboard or degraded”.
2. Verify both documents contain exact canonical enums for:
   - Runtime Mode
   - Policy Profile
   - Environment Mode
   - Effective Control State
   - Gate State
   - Check Requirement
   - Validation Status
   - Runtime Capability Status
   - Tool Evidence facets.
3. Read every changed section in context and verify heading/table/code-fence integrity.
4. Compare PRD and ROADMAP enum token sets with an in-memory script; any mismatch fails the task.
5. Confirm no non-state section was intentionally changed.

## Rollback Points

- After the PRD edit, keep the ROADMAP untouched until the PRD contract is internally consistent.
- After the ROADMAP edit, validation either accepts both documents or both source-file edits are reverted together.
- Planning artifacts and parent decisions are not rolled back with source documents.

## Completion Gate

- Run `trellis-before-dev` before source edits.
- Run `trellis-check` after edits.
- This docs-only task does not add BDD scenarios or run project code tests because it changes design contracts, not implemented user-visible behavior; targeted state-contract conformance is the proof.

## Check Summary

- **Trellis check**: passed for this documentation-only task.
- **Canonical contract**: 15 state fields, value sets, owners, and persistence rules are identical in PRD and ROADMAP.
- **Transition conformance**: Session v1, command ownership, deterministic Environment precedence, five Effective Control rows, four Runtime/Profile combinations, and eight Runtime/Environment combinations passed targeted assertions.
- **Forbidden patterns**: no persisted `enabled`, bare policy `mode`, `needs-onboard/degraded`, `blocked/degraded`, combined tool lifecycle chain, or `current/fresh` enum remains.
- **Markdown structure**: all code fences are balanced; changed and existing tables have consistent column counts.
- **Project checks**: lint, type-check, unit, integration, API, Web, and Mobile tests are not applicable because no executable code or user-visible implemented behavior changed.
- **BDD**: skipped; the edited artifacts specify future implementation contracts rather than changing currently executable behavior.
- **Spec sync**: no `.trellis/spec` update. The canonical product state contract belongs in the task artifacts and v0.7 PRD/ROADMAP until implementation creates an executable backend contract; duplicating it into current Pi-derived code specs would create a second source of truth.
- **Remaining scope**: non-state corrections recorded by the parent review remain intentionally unchanged.
