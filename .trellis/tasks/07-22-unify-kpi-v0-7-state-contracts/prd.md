# Unify KPi v0.7 state contracts

## Goal

Make every state dimension in `docs/prd/PRD.md` and `docs/prd/ROADMAP.md` deterministic, namespaced, non-overlapping, persistable, and testable so implementers cannot confuse user mode, environment readiness, Gate progress, validation outcome, Runtime capability, tool evidence, or publication truth.

## Confirmed Facts

- The parent review recorded the accepted state decisions in `.trellis/tasks/07-21-review-kpi-v0-7-boundaries/prd.md`: D-004, D-011, D-016, D-023, and D-024.
- PRD `2097-2100` persists `enabled` plus `mode: strict|relaxed`, while PRD `292-305`, `466-508` and ROADMAP `307-310` use `enforced|advisory` for Runtime Mode.
- PRD `41`, `652`, `793-802` and ROADMAP `212`, `248-251`, `687-689` allow the same missing Adapter facts to resolve to either `needs-onboard` or `degraded`.
- `blocked` and `degraded` are currently reused without a namespace across Environment Mode, Gate progress, validation/report outcomes, Runtime capabilities, tools, GitNexus, Trellis, and Evidence publication.
- ROADMAP `594-600` lists `installed`, `configured`, `callable`, `project-ready`, and `current/fresh` as if they were one status even though the PRD treats them as independent evidence.
- PRD `125-129` classifies Checks as `required|optional|not-applicable`, while validation/report sections use `passed|failed|blocked|skipped|not-needed`; these are separate dimensions and must not be persisted in one field.
- There is no production implementation yet, so the documents may use a clean canonical state contract without maintaining a deployed legacy schema.

## Requirements

1. Define one canonical state taxonomy shared by both documents:
   - `runtimeMode: enforced|advisory`
   - `policyProfile: strict|relaxed`
   - `environmentMode: managed|needs-onboard|degraded|blocked`
   - derived, non-persisted `effectiveControlState: active|advisory|preflight-only|blocked`
   - `gateState: planned|running|passed|blocked|not-required`
   - `stageStatus: pending|running|passed|blocked|skipped|not-needed`
   - Gate-specific `reviewerStatus`, separate from `gateState`
   - `checkRequirement: required|optional|not-applicable`
   - `validationStatus: passed|failed|blocked|skipped|not-needed`
   - `capabilityStatus: native|adapter|degraded|unsupported`
   - independent Tool Evidence facets for installation, configuration, callability, project readiness, and freshness
   - namespaced Evidence, Trellis, GitNexus, Onboard aggregate, and Managed Asset states.
2. Remove persisted `enabled` and ambiguous bare `mode`; derive automatic-control activation from the canonical dimensions.
3. Define a deterministic Environment Mode decision order and require explicit accepted-skip provenance before `degraded`.
4. Define how `/sbtd on`, `/sbtd off`, `/sbtd strict`, and `/sbtd relaxed` mutate only their owning dimension.
5. Define Compaction/Resume persistence and `stateVersion`; ambiguous or incompatible state must fail closed with a repair path.
6. Keep identical field names, enum values, transition rules, status output, requirements, roadmap milestones, and acceptance tests across both documents.
7. Update state-related open decisions and release blockers so they no longer contradict the canonical contract.
8. Do not modify unrelated Provider, packaging, licensing, Evidence-service scope, Runtime sequencing, or source-distribution decisions in this task.

## Acceptance Criteria

- [x] Both documents contain the same canonical state taxonomy and ownership table.
- [x] No persisted state uses `enabled` or a bare `mode` field.
- [x] Runtime Mode, Policy Profile, and Environment Mode remain orthogonal; their four Runtime/Profile combinations have Resume fixtures.
- [x] The same observed environment facts always map to exactly one Environment Mode.
- [x] `degraded` requires applicable accepted-skip provenance and cannot satisfy a Route missing Required capability.
- [x] Gate progress, Stage Status, Reviewer Status, Check Requirement, Validation Status, Capability Status, and Evidence publication are distinct fields.
- [x] Tool `installed/configured/callable/project-ready/freshness` evidence is represented as independent facets, not a single status progression.
- [x] P0/P1/P2/P3 roadmap work items and exit criteria use the canonical names and include state migration/conformance tests.
- [x] Targeted searches find no `mode: \"strict\" | \"relaxed\"`, `needs-onboard/degraded`, `blocked/degraded`, or `needs-onboard or degraded` ambiguity.
- [x] The source documents remain valid Markdown and their commands, requirements, acceptance criteria, and release blockers agree.

## Out of Scope

- Production code, schemas, fixtures, CLI implementation, package scaffolding, or tests.
- Resolving non-state correction sets from the parent review.
- Reopening D-001 through D-024 unless a direct state-contract contradiction makes a prior decision impossible.
