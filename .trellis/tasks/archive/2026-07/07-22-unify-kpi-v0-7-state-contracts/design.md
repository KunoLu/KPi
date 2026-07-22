# State Contract Design

## Scope

This task changes documentation contracts only. `docs/prd/PRD.md` is the product contract and `docs/prd/ROADMAP.md` is the implementation sequence. Both must use the same names and enums.

## Conflict Inventory

| Conflict | Current evidence | Resolution |
|---|---|---|
| `mode` means Policy Profile in one schema and Runtime Mode elsewhere | PRD `2097-2100` vs PRD `292-305`, ROADMAP `307-310` | Replace with `runtimeMode` and `policyProfile`. |
| `enabled` duplicates Runtime Mode | PRD `2097-2099`, ROADMAP `307-310` | Remove from persisted state; derive activation. |
| Plugin Enforcement duplicates Runtime Mode | PRD `504-515` | Report `runtimeMode` and derived `effectiveControlState`; no separately mutable enforcement flag. |
| Missing Adapter maps to two Environment Modes | PRD `41`, `652`, ROADMAP `212`, `251`, `688` | Deterministic precedence and accepted-skip provenance. |
| `degraded` is both environment readiness and Runtime capability | PRD `793-802`, ROADMAP capability sections | Keep the word only in namespaced `environmentMode` and `capabilityStatus`. |
| `blocked` is reused as a global state | Gate, validation, environment, Evidence, Trellis, GitNexus sections | Preserve domain-local meaning but require the owning field name in schema, status, and reports. |
| Check applicability and execution outcome overlap | PRD Check classification vs validation reporting | Separate `checkRequirement` from `validationStatus`; `not-needed` is a final validation disposition, not Gate state. |
| Gate progress and reviewer result can be conflated | PRD `1103-1145`, ROADMAP `389-392` | Persist `gateState` independently from Gate-specific `reviewerStatus`. |
| Tool evidence is presented as one progression | ROADMAP `594-600`, P1 Doctor `845-851` | Store independent installation/configuration/callability/readiness/freshness facets. |
| `current` and `fresh` are used as synonyms | ROADMAP `599`, GitNexus/report sections | Canonical freshness values are `current|stale|unknown|not-needed`; “fresh artifact collection” remains prose, not an enum. |
| Aggregate Onboard status is confused with per-project detail | PRD `1586-1603` | Keep `onboardProjectStatus`; aggregate is the highest-priority child status and never overwrites child results. |
| Evidence publication `blocked` can be read as validation failure | PRD `1398-1412` | Keep `evidencePublication`, separate from `validationStatus` and `environmentMode`. |

## Canonical State Taxonomy

| Field | Values | Owner | Persisted |
|---|---|---|---|
| `runtimeMode` | `enforced`, `advisory` | Session / user command | yes |
| `policyProfile` | `strict`, `relaxed` | Session/config policy | yes |
| `environmentMode` | `managed`, `needs-onboard`, `degraded`, `blocked` | read-only Preflight | yes as last observation plus evidence timestamp |
| `effectiveControlState` | `active`, `advisory`, `preflight-only`, `blocked` | derived selector | no |
| `gateState` | `planned`, `running`, `passed`, `blocked`, `not-required` | Workflow/Gate engine | yes |
| `stageStatus` | `pending`, `running`, `passed`, `blocked`, `skipped`, `not-needed` | Workflow Stage | yes |
| `reviewerStatus` | Gate-specific enum | Gate reviewer | yes, namespaced by Gate |
| `checkRequirement` | `required`, `optional`, `not-applicable` | Route classification | yes |
| `validationStatus` | `passed`, `failed`, `blocked`, `skipped`, `not-needed` | Validation/Report | yes |
| `capabilityStatus` | `native`, `adapter`, `degraded`, `unsupported` | Runtime Adapter evidence | yes |
| `onboardProjectStatus` | `failed`, `blocked`, `needs-user`, `bootstrap-required`, `success`, `skipped` | Environment Management | yes |
| `managedAssetState` | `absent`, `exact`, `drifted`, `merge-required`, `blocked` | provenance inventory | yes |
| `evidenceSource` | `developer-local`, `ci`, `knowledge-server`, `not-needed` | Evidence Envelope | yes |
| `sourceRevision` | `exact`, `dirty`, `unknown`, `not-needed` | Evidence Envelope | yes |
| `environmentAlignment` | `verified`, `unverified`, `mismatch`, `not-needed` | Evidence Envelope | yes |
| `evidencePublication` | `local-only`, `published`, `blocked`, `not-configured`, `not-needed` | Evidence Envelope | yes |

Domain-specific Trellis and GitNexus fields remain separate and must be printed with their field names. They may map into Tool Evidence but are not replacements for the canonical workflow fields.

## Tool Evidence Facets

```ts
interface ToolEvidenceState {
  installation: "installed" | "missing" | "broken" | "not-needed";
  configuration: "configured" | "not-configured" | "not-needed";
  callability: "callable" | "unavailable" | "blocked" | "not-needed";
  projectReadiness: "ready" | "not-ready" | "blocked" | "not-needed";
  freshness: "current" | "stale" | "unknown" | "not-needed";
  observedAt: string;
  evidence: string[];
  blockedReason?: string;
}
```

No facet implies another. `installed` never proves `configured`, `configured` never proves `callable`, and `callable` never proves project readiness or current evidence.

## Session Contract

```ts
interface SBTDSessionStateV1 {
  stateVersion: 1;
  runtimeMode: "enforced" | "advisory";
  policyProfile: "strict" | "relaxed";
  environmentMode: "managed" | "needs-onboard" | "degraded" | "blocked";
  environmentObservedAt: string;
  route: WorkflowRouteId | "auto";
  stage: string;
  classification?: SBTDClassification;
  activeSkills: string[];
  bookGates: Record<string, { gateState: GateState; reviewerStatus?: string }>;
  toolEvidence: Record<string, ToolEvidenceState>;
  validation: ValidationState;
  provider: ProviderSessionState;
  lessons: LessonsState;
  decisions: DecisionRecord[];
}
```

`enabled` and bare `mode` are prohibited persisted fields.

## Derived Effective Control State

| Runtime Mode | Environment Mode | Effective Control State | Behavior |
|---|---|---|---|
| `advisory` | any | `advisory` | Automatic classification, routing, Gate advancement, Skill routing, and delivery enforcement remain off; Always-on safety/evidence boundaries remain. |
| `enforced` | `managed` | `active` | Full automatic SBTD controls may run. |
| `enforced` | `degraded` | `active` | Controls run only because all current Route requirements are satisfied; missing accepted Optional capabilities remain visible. |
| `enforced` | `needs-onboard` | `preflight-only` | Help/Doctor/Plan and Always-on safeguards only; do not claim SBTD activation. |
| `enforced` | `blocked` | `blocked` | Block the affected Route/stage and report recovery. |

## Environment Mode Decision Order

Evaluate on every `/sbtd on`, Doctor, Route change, relevant asset/config change, and Resume:

1. `blocked`: the current Route needs a missing, invalid, unsafe, or unsupported capability/contract.
2. `needs-onboard`: a selected profile baseline is missing or invalid and no applicable accepted-skip record exists.
3. `degraded`: every missing item is Optional for the current Route and has an unexpired accepted-skip record matching asset/capability, scope, profile, and Kit major.
4. `managed`: all selected-profile baseline assets and current Route requirements are present, valid, effective, and current.

This order makes one fact set map to one result. `degraded` cannot be a fallback for unknown state.

## Command Ownership

- `/sbtd on` mutates only `runtimeMode=enforced`; Preflight recomputes `environmentMode` and `effectiveControlState`.
- `/sbtd off` mutates only `runtimeMode=advisory` and preserves `policyProfile`.
- `/sbtd strict` mutates only `policyProfile=strict`.
- `/sbtd relaxed` mutates only `policyProfile=relaxed`; it cannot alter Route-required Checks.
- Doctor/Preflight observes `environmentMode`; users do not set it directly.

## Persistence and Compatibility

- Persist `stateVersion`; Compaction/Resume restores all three canonical persisted dimensions and recomputes `effectiveControlState` from current evidence.
- Because no production implementation exists, v1 is the clean baseline. Draft fixtures using `enabled` or bare `mode` are migrated only in compatibility tests:
  - `enabled=true|false` maps to `runtimeMode=enforced|advisory`.
  - bare `mode=enforced|advisory` maps to `runtimeMode`.
  - bare `mode=strict|relaxed` maps to `policyProfile`.
  - conflicting duplicate values fail closed and require repair; no last-writer-wins.
- `environmentMode` is re-observed on Resume rather than trusted indefinitely.

## Documentation Migration

- Add the taxonomy once as the PRD source of truth and reference it from later sections.
- ROADMAP repeats the exact contract in P0 Session State and uses the canonical names in all milestones, tests, exits, and release blockers.
- Replace slash-separated ambiguous outcomes such as `needs-onboard/degraded` and `blocked/degraded` with decision predicates or separate test cases.
- Preserve domain-specific status vocabularies only when the field name is explicit.

## Rollback

Documentation-only rollback restores the two Markdown files. The parent task decision log and `docs/CONTEXT.md` remain the authoritative rationale if the source edit must be reapplied.

## DDIA Data Design Review

- **Status**: `confirmed`
- **Data owner and source of truth**: SBTD Workflow Control Plane owns the logical state schema and transitions. Runtime Adapter owns serialization mechanics only. Environment Management owns observations and accepted-skip provenance. Validation/Reporting owns validation and publication truth.
- **Consistency model**: Session choices are read-your-writes within one Session. `effectiveControlState` is derived, never independently persisted. `environmentMode` is an observed value with a timestamp and is recomputed after Resume or relevant environment change.
- **Write/read/failure path**: User commands mutate one owned field idempotently; Preflight observes environment facts; the selector derives effective control. Unknown versions, duplicate conflicting fields, stale observations, or invalid enum values fail closed and return a repair path.
- **Schema/migration/rollback**: `stateVersion: 1` is the clean baseline. Compatibility fixtures may map unambiguous draft fields; collisions cannot use last-writer-wins. Documentation rollback must restore PRD and ROADMAP together.
- **Recovery and observability**: Status/Doctor/Report print every namespaced field, observation time, evidence, and blocked reason. They never collapse independent Tool Evidence facets or child project results into one opaque status.
- **Required conformance cases**: all Runtime Mode × Policy Profile combinations; all Runtime Mode × Environment Mode effective states; Route change invalidating `degraded`; accepted-skip expiry; Resume with environment drift; unknown version; duplicate conflicting draft fields; stale Tool Evidence; multi-project aggregate precedence.

