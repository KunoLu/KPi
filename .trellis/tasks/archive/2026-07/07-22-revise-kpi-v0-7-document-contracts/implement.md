# Implementation Plan — Revise KPi v0.7 document contracts

## Preconditions

- User approved creating this Trellis task and applying the current review checklist.
- `DDD Boundary Review: confirmed` and `DDIA Data Design Review: confirmed` are recorded in `design.md`.
- The correction source of truth is the archived D-003–D-024 decision log, `docs/CONTEXT.md`, and the reviewed current documents.
- BDD is skipped because this task changes documentation only and introduces no implemented user-visible behavior.

## Ordered changes

1. **Canonical terminology and phase placement**
   - Update PRD Provider table/architecture, Provider sections, requirements, and open-decision wording to use Provider Coordination and Runtime-owned execution/authentication.
   - Update ROADMAP P2/P3 deliverables, dependencies, milestones, and exits to freeze Runtime Contract v1 and deliver formal `runtime-omp` in P2; keep Pi/cross-Runtime work in P3.
   - Close or phase-gate P1 Provider Login and Python fallback decisions to match D-007/D-013.

2. **Canonical state and command semantics**
   - Align PRD/ROADMAP `/sbtd on` to one atomic Preflight/commit/derived-control-state sequence and explicit evaluator-failure rollback.
   - Make unknown command dispatch deterministic; retain Runtime-owned interactive messaging only after session start.
   - Keep `runtimeMode`/`policyProfile` orthogonal; replace `local-guarded` misuse with a separate security-baseline configuration.
   - Add exhaustive Book Gate reviewer-status tables and recovery mappings.
   - Require Route/Profile changes to re-observe Environment atomically; replace stale Compaction `SBTD Enabled/Mode` wording with canonical fields.

3. **Environment Management lifecycle**
   - Add `AcceptedSkipV1` schema and lifecycle, including input match rules, confirmation/revocation/expiry, owner, storage/provenance, and Doctor/Report/Evidence visibility.
   - Add `merge-required` resolution workflow with safe action choices, digest recheck, backup/rollback, operation result, and terminal state.
   - Add P1 scoped uninstall/update lifecycle with provenance, plan/apply confirmation, residuals, idempotency, purge, and Reload semantics.

4. **Validation, BDD, and exit criteria**
   - Move final full validation before final artifact verification, or verify the final run's artifacts after it completes.
   - Eliminate user-visible BDD bypass through `checkRequirement=not-applicable`.
   - Replace P0 “enough data” with the D-003 numeric value gate.
   - Restore P1 configurable Rules commands and their mutation scope/immutability contract.

5. **Conformance and task verification**
   - Search every changed term before/after editing; inspect all owning sections rather than patching isolated sentences.
   - Run deterministic Markdown structure and cross-document contract checks.
   - Run `$trellis-check`; no code build/lint/unit/API/E2E command applies because no runtime code changes.

## Files in scope

| File | Intended change |
|---|---|
| `docs/prd/PRD.md` | Canonical terminology, state/command semantics, Environment/Managed Asset lifecycle, Provider boundary, validation/BDD rules, decision log, compaction terms |
| `docs/prd/ROADMAP.md` | P0 value exit, P1 Rules/uninstall scope, P2 Runtime Contract/OMP Adapter, P3 scope, `/sbtd on`, Definition of Done |
| `.trellis/tasks/07-22-revise-kpi-v0-7-document-contracts/{prd,design,implement}.md` | Traceability, gate outcomes, implementation/check evidence |
| `docs/CONTEXT.md` | Only if a stable term must be corrected; expected unchanged because it is already the accepted boundary source |

## Validation plan

1. Parse both Markdown files for balanced tables/fences.
2. Assert canonical state fields/enums, phase ownership, command surface, Provider boundary, Environment lifecycle, validation ordering, and BDD wording agree across the two documents.
3. Search for stale contradictory phrases, including `Provider Gateway`, P3-only Runtime Contract v1, unresolved Provider Login/Python fallback, `SBTD Enabled/Mode`, and the old P1 Rules surface.
4. Execute `$trellis-check` and compare the final diff against `prd.md` and `design.md`.

## Risks and rollback

- **Risk:** changing one repeated contract statement leaves another source of truth stale. **Control:** use a deterministic cross-document conformance script plus targeted searches.
- **Risk:** adding lifecycle precision accidentally changes accepted product scope. **Control:** use only D-003–D-024 and `docs/CONTEXT.md`; do not introduce new product features.
- **Rollback:** revert the document-correction commit as one unit; no data migration or external cleanup is needed.

## Completion evidence

- Scope remained documentation-only: `docs/prd/PRD.md`, `docs/prd/ROADMAP.md`, and this task evidence. No production code, package, test, configuration, or runtime behavior changed.
- Deterministic document conformance passed after loading both sources: PRD has 130 balanced fenced blocks and 26 valid tables; ROADMAP has 74 balanced fenced blocks and 7 valid tables. Reused PRD subheadings occur under distinct parent sections and are not contract keys.
- Cross-document contract checks passed for Provider Coordination ownership, P2 Runtime Contract v1 plus formal `runtime-omp` delivery, P3 frozen-contract consumption, P0 numeric value gate, P1 Rule mutability, atomic `/sbtd on`, Route/Profile re-observation, AcceptedSkip/`merge-required`, scoped uninstall, final-validation order, user-visible BDD, Book Gate reviewer states, and resolved Provider Login/Python fallback decisions.
- Targeted stale-term search returned no occurrences of `Provider Gateway`, `provider-gateway`, `ProviderAdapter`, `CredentialSource`, `SBTD Enabled/Mode`, the legacy unknown-input dispatch rule, or P3-only Runtime Contract milestones.
- Trellis quality check: passed for this docs-only scope. No project lint/type-check/test command applies because no configured documentation package or production code changed.

- Final Phase 2.2 recheck found one remaining validation-order contradiction in `PRD.md` §15.1: its Artifact Gate preceded `Final Full Rerun`. The corrected canonical order is Focused Validation → Affected Scope → Planned Final Full Validation → Formal Report Artifact Gate, matching the ROADMAP and R5.
- Post-correction full-scope conformance passed: PRD/ROADMAP fenced blocks and table separators are valid, and 12 targeted ownership/state/phase/lifecycle/BDD/validation assertions passed.

- Phase 3.3 code-spec review: no `.trellis/spec/` update is warranted. This task created no implementation convention, signature, API, schema, or reusable coding pattern; the precise product contracts belong to `docs/prd/PRD.md` and `docs/prd/ROADMAP.md`, which are their intentional source of truth.
