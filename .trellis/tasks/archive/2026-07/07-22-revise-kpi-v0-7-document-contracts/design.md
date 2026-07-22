# Design — Revise KPi v0.7 document contracts

## DDD Boundary Review

**Status:** confirmed

**Ubiquitous language:**

- **KPi / SBTD Workflow Control Plane** owns task classification, Route, workflow state, Gate/Rule/Policy decisions, validation/evidence truth, context orchestration, reporting, Provider Coordination, and Kit/Onboard contracts.
- **Runtime** owns agent-loop execution, model/provider authentication and execution, credential refresh, streaming transport, native tools, approvals, UI, and host session primitives.
- **Runtime Adapter** translates a Runtime's public events/capabilities to the Control Plane contract; it is neither a Runtime fork nor a provider adapter.
- **Provider Coordination** owns declarative Provider Profiles, secret references, requirements, availability, fallback policy, and selection results only.
- **Environment Management / Onboard** owns baseline observation, provenance inventory, AcceptedSkip lifecycle, Managed Asset transitions, plan/apply/rollback/uninstall semantics, and resulting Environment observations.

**Bounded contexts:**

| Context | Owns | Must not own |
|---|---|---|
| SBTD Workflow Control Plane (core) | state, Route, Gate, Rule, Policy, validation/evidence decisions | Runtime model execution or credential handling |
| Runtime Integration | adapter contract and capability evidence | workflow policy or KPi global state semantics |
| Provider Coordination | declarative selection/fallback inputs and non-sensitive outcomes | model execution, auth, token refresh, transport |
| Environment Management / Onboard | assets, provenance, AcceptedSkip, mutation plans, migration/rollback/uninstall | automatic production-task execution |
| Artifact Supply Chain | Kit, templates, licensing/integrity | local project ownership decisions |

**Invariants and business rules:**

1. Provider Coordination never receives or executes Runtime credentials; a Runtime retains execution/authentication ownership.
2. `runtimeMode` and `policyProfile` remain independent persisted Session dimensions. `effectiveControlState` is derived only and never persisted.
3. `environmentMode` is a deterministic current observation from the selected Profile, Route, effective assets, and valid AcceptedSkip records.
4. A Route-required capability cannot be accepted-skipped or silently degraded.
5. A Managed Asset requires provenance plus an exact digest; installation location alone is never ownership evidence.
6. Every user-visible behavior requires persistent BDD; internal checks alone may be `not-applicable` only where an objective predicate proves no user-visible behavior.

**Core / supporting / generic subdomains:**

- Core: SBTD Workflow Control Plane.
- Supporting: Runtime Integration, Provider Coordination, Environment Management, Artifact Supply Chain.
- Generic: external CLI login, package manager, test runners, and Runtime-provided model execution.

**Corrections to prior planning:** none. The current review findings are documentation drift from already accepted D-003–D-024 decisions, not new domain intent.

**Open conflicts and questions:** none. The archived decision log supplies the corrective source of truth for every scoped finding.

## DDIA Data Design Review

**Status:** confirmed

**Data owner and source of truth:**

| Contract/data | Owner | Source of truth |
|---|---|---|
| `SBTDSessionStateV1` | Workflow / Session | versioned Session record |
| `runtimeMode`, `policyProfile`, Stage/Route, Gate records | designated field owner in canonical state table | Session record |
| `environmentMode` observation | Environment Management / Preflight | current signed/recorded observation plus its inputs/time |
| `AcceptedSkipV1` | Environment Management / Provenance Inventory | versioned per-scope acceptance record |
| Managed-asset inventory and mutation result | Environment Management / Provenance Inventory | versioned asset record with installed digest and backup reference |
| Evidence Envelope | Validation / Evidence | append-only local envelope bound to exact source revision/artifacts |

**Write / read / failure paths:**

1. `on`, Route, or Profile changes calculate their intended state in memory, run read-only Preflight, then atomically write their owned fields plus a fresh Environment observation. `effectiveControlState` is derived after the write.
2. A valid `needs-onboard` or `blocked` result is a successful observation and may be recorded. An indeterminate evaluator failure writes neither the requested mutation nor a fabricated Environment Mode; it returns diagnostics and a repair path.
3. AcceptedSkip creation/revocation and managed-asset update/uninstall are plan-first. Apply requires user confirmation, current provenance/digest checks, a transaction record, backup when replacing a known managed block, and a terminal operation result.
4. Concurrent/manual changes make the current digest invalid. The operation stops at `merge-required` or `blocked`; it never overwrites broad user/project content.

**Consistency model:** local, strongly consistent per Session or Environment Management operation. There is no asynchronous replication, queue, or eventual-consistency contract in P0–P3. Reads must be read-your-writes within the process/session after a successful atomic operation.

**Idempotency / ordering / retry / deduplication:**

- Preflight is read-only and repeatable.
- Apply/uninstall keys each asset operation by target, scope, source revision, expected/observed digest, and operation id. Repeating a completed operation reports its terminal result without broadening scope.
- Retry is allowed only after re-reading provenance and digest. A stale/different digest produces a conflict, never implicit replay.
- `AcceptedSkipV1` is keyed by asset-or-capability, scope, Profile, Kit major, and validity interval; duplicate confirmation updates the same logically scoped record only through an explicit versioned operation.

**Schema / migration / backfill / rollback / replay:**

- Keep `SBTDSessionStateV1` versioned. Migrate historical `enabled`/bare `mode` terms to the two canonical fields only where the source is unambiguous; otherwise require repair rather than guess.
- Keep `local-guarded` in a distinct security-baseline configuration contract, not in `policyProfile`.
- Introduce `AcceptedSkipV1` as a new versioned record; no backfill should infer consent from installation state.
- Store prior backup reference and operation result for known managed-block replacement. Rollback restores only the exact proven prior block and rechecks current digest first.
- Uninstall does not replay prior setup or delete shared dependencies; it reports retained residuals and requires Reload/new Session before absence is claimed.

**Observability and repair:** Doctor/Report must expose state owner, record/version, observed time, accepted-skip validity, operation id/result, expected versus observed digest, backup reference, residuals, failure reason, and the specific repair command/plan. Evidence Envelopes bind these facts to the source revision and validation artifacts.

**Required validation:** deterministic document conformance checks for canonical enums, command ownership, phase placement, lifecycle vocabulary, final-validation ordering, BDD prohibition bypass, markdown table/fence integrity, and no stale contradictory phrases in either source document.

## Source-document correction design

### 1. State, command, and Environment contracts

- The canonical Session contract remains `runtimeMode: enforced|advisory` plus `policyProfile: strict|relaxed`. Replace the YAML's erroneous `profile: local-guarded` with a separately named security-baseline setting and explain its independence.
- Add per-Book-Gate reviewer status enum/transition tables; a generic table reference is insufficient because guards must validate non-passing/recovery states.
- Enforce the D-022 parser boundary. Interactive free text is Runtime-owned only after a session begins; it is never the parser fallback for management-like input.
- State `/sbtd on` as a transactional sequence: prepare target mode/profile/Route → execute read-only Preflight → atomically persist target-owned state and current Environment observation if determinable → derive control state. On evaluator failure, retain the pre-command state; on valid `needs-onboard`/`blocked`, persist that result and expose recovery.
- Execute Profile/Route changes through the same atomic re-observation rule; no stale Environment state can authorize a new Route.

### 2. Provider, Runtime, and phase contracts

- Rename all KPi-owned `Provider Gateway` functionality to Provider Coordination. Where a literal Runtime gateway is required, mark it Runtime-owned rather than a KPi component.
- Move Runtime Contract v1 freeze, migration fixture, and formal `runtime-omp` Adapter into P2 before Core exit. P3 consumes that frozen contract for Pi/cross-Runtime work.
- Convert resolved decisions from PRD's open list into closed/phase-gated decisions consistent with D-007 and D-013.

### 3. Environment Management and provenance

`AcceptedSkipV1` must include: `schemaVersion`, `recordId`, asset-or-capability key, `scope`, selected Profile, Kit major, confirmation actor/time, expiry, status (`active|revoked|expired`), reason, evidence reference, and provenance/version metadata. The Environment observer reads only active, unexpired, exact-scope/Profile/Kit-major matches.

`merge-required` resolution provides safe, explicit actions: retain-and-block, user manual merge followed by re-plan, or known-template migration only when the recorded prior digest authorizes it. Apply records the selected action, rechecks digest immediately before mutation, writes/retains a backup as applicable, and returns `exact|merge-required|blocked` with residuals. Unknown content is never broadly overwritten or adopted by location alone.

P1 uninstall defines explicit `project|user|runtime|all-managed` scopes, plan-first confirmation, exact-provenance target selection, idempotent result reporting, default backup retention, separate confirmed purge, residual reporting, and Reload/new Session semantics.

### 4. Validation and Definition of Done

- The canonical sequence ends with planned Final Full Validation and then validates the final report artifacts/freshness/sanitization against that result.
- User-visible behavior requires BDD. `checkRequirement=not-applicable` is limited to internal checks under objective predicates and cannot serve as an alternative completion path for product behavior.
- P0 exit uses the numeric D-003 value gate, not an undefined sufficiency judgment.

## Compatibility and rollback

This is a source-document correction only. No runtime data, user configuration, package, or production behavior is changed. Rollback is one commit reverting `PRD.md` and `ROADMAP.md` together; task artifacts preserve the correction rationale and traceability.
