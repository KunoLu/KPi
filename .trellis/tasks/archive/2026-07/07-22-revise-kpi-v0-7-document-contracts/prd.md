# Revise KPi v0.7 document contracts

## Goal

Make `docs/prd/PRD.md` and `docs/prd/ROADMAP.md` implementation-ready by applying the accepted product-boundary decisions and resolving every substantive finding from the current documentation review. The corrected documents must describe one deterministic, runtime-neutral KPi contract without changing product scope or adding production implementation.

## Confirmed Facts

- This repository is currently greenfield; this task changes planning documentation only, not production code, tests, packages, or runtime configuration.
- The canonical product boundary is KPi as the runtime-neutral SBTD Workflow Control Plane; a Runtime owns agent-loop, provider/model execution, native tools, approvals, UI, and session primitives (`docs/CONTEXT.md`).
- The archived product-boundary review resolved the decisions required by this task, including P0 value measurement, Runtime/Policy state, Provider Coordination, Runtime Contract sequencing, managed assets, scoped uninstall, deterministic command dispatch, Environment Mode, and configurable Rules (`.trellis/tasks/archive/2026-07/07-21-review-kpi-v0-7-boundaries/prd.md:D-003` through `D-024`).
- The current review found four blocking, seven high, and five medium document-contract defects. Each finding is in scope; source anchors are preserved below.

## Requirements

### R1 — Restore KPi ownership boundaries

- Replace `Provider Gateway` ownership of model execution/authentication with the established **Provider Coordination** boundary: Profiles, secret references, role/capability requirements, availability, fallback policy, and selection results passed to a Runtime.
- Make P0/P1/P2/P3 Provider promises agree with the boundary: P0 observes non-sensitive Runtime-managed status; P1 only delegates a user-controlled official login; P2 adds runtime-neutral coordination contracts; P3 may add gated external CLI Session Adapters.
- Close PRD decisions already resolved by the roadmap and the archived decision log, including P1 Provider Login and Python fallback lifecycle.

### R2 — Make phase sequencing implementable

- Freeze Runtime Contract v1 and deliver the formal `runtime-omp` Adapter before P2 exit; reserve `runtime-pi` and cross-Runtime compatibility for P3.
- Preserve the P0 numeric gate: deterministic contract gate plus at least 20 paired representative tasks against the same Runtime/model baseline, non-inferior blinded correctness, at least 30% fewer severe workflow omissions, 100% mandatory-Gate recall, and no more than 5% unnecessary heavy-route activation.
- Keep the P1 Rules command surface aligned as `kpi rules list|enable|disable|doctor`, with only explicitly configurable Optional Rules mutable at explicit `session`, `project`, or `user` scope.

### R3 — Define a deterministic state and command contract

- Retain orthogonal `runtimeMode: enforced|advisory` and `policyProfile: strict|relaxed`; split `local-guarded` into a separate security-baseline configuration rather than a Policy Profile value.
- Replace ambiguous unknown-input fallback with the accepted dispatch contract: bare `kpi` starts an interactive session, one-shot natural language uses `kpi run -- <prompt>`, and unmatched KPi or `/sbtd` commands return candidates and Help without creating an Agent Turn.
- Define one atomic `/sbtd on` transition and matching rollback/failure behavior. Route or Profile changes must refresh Environment observation before a new Effective Control State is used.
- Replace obsolete Compaction wording (`SBTD Enabled/Mode`) with canonical state fields.
- Define complete reviewer-status enums and recovery mappings for every Book Gate.

### R4 — Define Environment, managed-asset, and uninstall lifecycles

- Add a versioned `AcceptedSkip` contract with scope, asset/capability identity, Profile, Kit major, confirmation, expiry, revocation, provenance/evidence binding, owner, and storage/read keys.
- Define `merge-required` resolution as plan-first, user-selected, digest-rechecked, backed up, rollback-capable, and idempotent, with explicit terminal state/output.
- Define P1 scoped uninstall over the provenance inventory: `project`, `user`, `runtime`, and `all-managed`; preserve drifted/unknown/shared/out-of-scope assets; keep backups by default; use separate confirmed purge; report residuals and require reload before claiming removal.

### R5 — Align validation and BDD completion rules

- Sequence validation as focused validation → affected scope → planned final full validation → formal artifact/freshness/sanitization gate against that final run.
- Require persistent BDD for user-visible behavior. `checkRequirement=not-applicable` remains only for objectively internal/non-user-visible checks and must not bypass the user-visible BDD requirement.

## Review-Finding Traceability

| Requirement | Source finding anchors |
|---|---|
| R1 | `PRD.md:619`; `PRD.md:2472-2474` |
| R2 | `ROADMAP.md:765-766`; `ROADMAP.md:903`; `ROADMAP.md:1331` |
| R3 | `PRD.md:364-369`; `PRD.md:531-544`; `PRD.md:573`; `PRD.md:2120-2124`; `PRD.md:2206`; `ROADMAP.md:211` |
| R4 | `PRD.md:458-459`; `PRD.md:941-948`; `ROADMAP.md:986-988` |
| R5 | `PRD.md:1384-1388`; `ROADMAP.md:1783-1785` |

## Acceptance Criteria

- [ ] Both source documents use **Provider Coordination** consistently and assign model execution/authentication/credential refresh/streaming transport to the Runtime.
- [ ] P2 contains Runtime Contract v1 freeze and formal `runtime-omp` delivery before its exit; P3 begins from that frozen boundary and owns Pi/cross-Runtime scope only.
- [ ] Both documents state the same P0 numeric value gate and the same P1 Rules surface/mutability boundaries.
- [ ] The canonical state table contains no invalid Policy Profile value, obsolete `enabled`/bare `mode`, or undocumented Book Gate reviewer-status domain.
- [ ] Command dispatch, `/sbtd on`, Route/Profile re-observation, Environment derivation, and failure/rollback semantics have one deterministic formulation across both documents.
- [ ] `AcceptedSkip`, `merge-required`, update/rollback, and scoped uninstall have owner, inputs, transitions, terminal outputs, idempotency, and recovery semantics.
- [ ] Validation artifact gating runs only against the final planned validation output; user-visible BDD cannot be bypassed through `checkRequirement=not-applicable`.
- [ ] Targeted searches find no conflicting `Provider Gateway`, P3-only Runtime Contract v1, unresolved P1 Provider Login/Python fallback, stale Rules surface, or obsolete compaction state wording in the affected contracts.
- [ ] Markdown structure is valid; all tables/fences are balanced; no unrelated documentation changes are introduced.

## Book Gate Plan

| Skill | Applicability | Objective trigger | Execution phase | Gate state |
|---|---|---|---|---|
| `book-ddd-distilled-modeling` | required | The correction changes stable Runtime/Provider/Onboard/Workflow ownership language and bounded-context terms. | Planning before artifact stabilization; `DDD Boundary Review=confirmed` | passed |
| `book-ddia-data-design` | required | The correction defines persisted Session state, AcceptedSkip/provenance records, managed-asset transitions, schema evolution, rollback, recovery, and Evidence linkage. | Design before source-document edits; `DDIA Data Design Review=confirmed` | passed |
| `book-legacy-change-safety` | on-demand | No production behavior or existing implementation is being changed. | N/A | not-required |
| `book-refactoring-pass` | on-demand | No existing production code is being edited. | N/A | not-required |
| `book-release-readiness` | on-demand | No production service/API/job/deployment behavior is being changed. | N/A | not-required |

## BDD Decision

BDD is not applicable to this task because it revises planning documentation only; it does not add, remove, or change implemented user-visible behavior. R5 preserves the future BDD product contract but does not itself implement that behavior.

## Out of Scope

- Production code, package scaffolding, Runtime/Provider/Onboard implementation, tests, tool installation, or configuration.
- New product features or additional P0–P3 scope beyond the accepted decisions.
- Changes outside `docs/prd/PRD.md`, `docs/prd/ROADMAP.md`, task artifacts, and any narrowly necessary long-term context correction.

## Planning Notes

`grill-with-docs` is not being fully invoked for this correction pass: the user approved the review checklist, and the archived decision log supplies the accepted product decisions for every finding. This task introduces no new product intent. An independent DDD boundary review remains required because the correction affects stable bounded-context ownership and terminology.
