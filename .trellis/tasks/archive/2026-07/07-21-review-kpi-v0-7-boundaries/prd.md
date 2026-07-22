# Review KPi v0.7 product boundaries

## Goal

Before revising `docs/prd/PRD.md` and `docs/prd/ROADMAP.md`, fully execute a `grill-with-docs` interview, stabilize the KPi product boundary and open decisions, run the mandatory independent DDD boundary review, then re-review both documents against the agreed decisions.

## Confirmed Facts

- The repository is greenfield: no production packages, TypeScript source, or tests exist yet (`.trellis/spec/backend/index.md:1-5`).
- PRD v0.7 defines the route `OMP-first → KPi CLI → KPi Core → OMP/Pi dual runtime → controlled fork` (`docs/prd/PRD.md:14-49`).
- The roadmap makes P0 an OMP-hosted SBTD plugin and delays a runtime-neutral Core until P2 (`docs/prd/ROADMAP.md:13-38`, `972-1005`).
- The current repository summary still says KPi is built on a Pi-derived runtime (`README.md:1-2`), while the current backend specs cite a removed v0.2 DOCX and describe a Pi-first `.kpi/` baseline (`.trellis/spec/backend/index.md:1-5`; `.trellis/spec/backend/filesystem-config-persistence.md:5-15`).
- The first review of commit `de27ee9` found 7 high- and 3 medium-priority issues, including incompatible session-mode fields, ambiguous AGENTS import ownership, unsafe unknown-command fallback, runtime/provider phase inversions, missing uninstall semantics, and non-measurable P0 exit criteria.
- No production implementation or source-document revision is authorized by this planning task.

## Requirements

- Ask one product decision at a time, after exhausting repository evidence, and include a recommended answer plus the trade-off.
- Resolve the product identity, runtime ownership, P0 value hypothesis, command/mode semantics, installed-asset ownership, provider delegation boundary, and phase dependencies before revising the source documents.
- Review all 13 open decisions in `docs/prd/PRD.md:2388-2402`; each must be decided, explicitly deferred behind a named gate, or removed as already resolved elsewhere.
- Reconcile decisions already implied by roadmap deliverables with the PRD open-decision list.
- Maintain stable ubiquitous language for Runtime, Adapter, Core, Kit, Onboard, Environment Mode, Runtime Mode, Policy Profile, Gate, Evidence, and Managed Asset.
- After the interview converges, run `book-ddd-distilled-modeling` independently and reach `DDD Boundary Review: confirmed` before producing a final product-boundary recommendation.
- Re-review both v0.7 documents with exact file and line anchors and propose bounded corrections; do not edit those source documents until the user approves the resulting decision set.

## Acceptance Criteria

- [x] The interview resolves every decision that blocks P0/P1 scope, contracts, packaging, safety, or acceptance.
- [x] Each deferred decision has an owner phase, trigger, deadline/gate, and behavior before resolution.
- [x] Product identity and bounded-context ownership are stated without mixing KPi Core, OMP Runtime, Onboard, Kit, and project facts.
- [x] P0 has a measurable experiment and numeric go/no-go criteria rather than directional metrics only.
- [x] Command mode, persistence, update, rollback, and uninstall semantics are non-contradictory.
- [x] Provider promises match the phase that implements the required adapter.
- [x] The independent DDD Boundary Review reaches `confirmed`.
- [x] The final re-review maps each agreed decision to affected PRD/ROADMAP sections and the prior 10 findings.
- [x] `docs/prd/PRD.md` and `docs/prd/ROADMAP.md` remain unchanged during this clarification task unless the user separately authorizes revision.

## Out of Scope

- Production code, tests, package scaffolding, provider integration, or runtime implementation.
- Choosing implementation details that do not affect product boundaries or current open decisions.
- Starting Trellis implementation status or modifying the v0.7 source documents before the review gate.

## Decision Log

### D-001 — Canonical product identity

- **Decision**: KPi is a runtime-neutral SBTD workflow control plane. OMP is the first default Runtime Adapter; `OMP-first` is a validation and delivery strategy, not KPi's domain identity.
- **KPi owns**: workflow classification and state, gates, policy, validation/evidence truth, context orchestration, reports, Provider coordination policy, Kit/Onboard contracts, and runtime capability negotiation.
- **Runtime owns**: the agent loop, model/provider execution, native tools, TUI, approvals, and session primitives.
- **Consequence**: P0 may be OMP-hosted, but OMP-specific events and types must remain behind an explicit adapter boundary; README and stale Pi-first specs require later alignment.

### D-002 — P0 scope

- **Decision**: P0 is a near-complete SBTD MVP rather than a thin experiment or a runtime-only feasibility spike.
- **Included intent**: preserve the broad P0 surface for the OMP plugin, three-layer AGENTS contract, Onboard bridge, deterministic commands/help, recoverable session state, classification/routes, all five Book Gates, Kit/rule/policy loading, validation/report truth, and the defined integration test matrix.
- **Consequence**: P0 must be decomposed into ordered internal milestones and must still pass an objective value gate before P1/P2; “near-complete” cannot mean that every optional future integration blocks the first alpha.

### D-003 — P0 value and go/no-go gate

- **Decision**: P0 uses a two-layer numeric gate rather than directional metrics.
- **Deterministic contract gate**: zero known silent Hard-Gate bypasses, zero evidence/full-stack misrepresentation, zero credential leakage, and all required compatibility/conformance fixtures passing.
- **Paired value gate**: run at least 20 representative real tasks with the same Runtime and model against an OMP-only baseline; require blinded correctness to be non-inferior, at least 30% fewer severe workflow omissions, 100% recall for mandatory Gate triggers, and no more than 5% unnecessary heavy-workflow activation.
- **Efficiency evidence**: report token and wall-clock deltas separately; do not hide correctness gains or regressions behind a combined score.
- **Consequence**: failure blocks automatic progression to P1/P2; the team may iterate P0 or explicitly stop, but may not replace the threshold with “enough data.”

### D-004 — Runtime mode and policy profile

- **Decision**: persist two orthogonal dimensions: `runtimeMode: enforced | advisory` and `policyProfile: strict | relaxed`.
- **Runtime Mode**: `enforced` activates automatic classification, routing, Gate progression, Skill routing, stage enforcement, and delivery reporting; `advisory` stops those automatic SBTD controls.
- **Policy Profile**: `strict` strengthens optional checks and confirmations; `relaxed` reduces only optional workflow intensity and cannot bypass a Gate required by the current route.
- **Always-on boundary**: Runtime-native permissions plus KPi secret, destructive-operation, installation, and evidence-truth safeguards remain active in both Runtime Modes.
- **Persistence**: `/sbtd off` preserves the selected Policy Profile; a later `/sbtd on` resumes it. Compaction/resume fixtures must cover all four combinations without conflating them with Environment Mode.

### D-005 — Managed asset ownership

- **Decision**: KPi uses a provenance inventory plus exact-digest Managed Blocks; installation location alone never proves ownership.
- **Inventory**: every KPi-managed write records target, scope, source revision, installed digest, prior backup, owner, and operation result.
- **Files**: only the marked, digest-verified block is KPi-managed; block-external content is user/project-owned. The required `.omp/AGENTS.md` import belongs inside the `omp-project-agents` Managed Block.
- **Shared dependencies**: Trellis, GitNexus, RTK, Java, Maestro, Playwright, MCP configuration, and user-level Skills remain externally/shared-owned unless an explicit private KPi scope says otherwise; installation assistance does not grant removal rights.
- **Drift rule**: exact known state may be updated or rolled back; unknown edits, damaged markers, or provenance mismatch require plan-time conflict handling and cannot be overwritten silently.

### D-006 — Upgrade and uninstall lifecycle

- **Decision**: use layered, plan-first uninstall semantics backed by the D-005 provenance inventory.
- **P0 boundary**: removing the OMP plugin removes only OMP-managed plugin package/state; it does not reverse explicit Onboard writes.
- **P1 scopes**: `project`, `user`, `runtime`, and `all-managed` produce a dry-run plan, require confirmation, and remove only matching KPi Managed Assets within the chosen scope.
- **Residuals**: drifted blocks, unknown ownership, shared dependencies, failed removals, and assets outside the selected scope are preserved and reported per project.
- **Backups and purge**: backups remain by default; destructive purge is a separate command/flag with explicit confirmation and must not broaden ownership.
- **Completion**: uninstall reports changed and retained assets, is idempotent, and requires a new Session/complete Reload before claiming KPi context is absent.

### D-007 — Provider and official CLI delegation by phase

- **P0**: use OMP-managed Provider/Auth only; KPi records non-sensitive Provider, Role, capability, availability, and fallback status through Doctor/Report.
- **P1**: keep `kpi provider login`, but it may only launch a user-controlled Runtime/official CLI login flow and observe non-sensitive status. It must not read, copy, persist, proxy, or refresh credentials.
- **P1 exclusion**: Codex/Claude/Kimi CLIs do not execute KPi Coding Sessions through a KPi adapter in this phase.
- **P2**: introduce runtime-neutral Provider Coordination contracts for Provider Profiles, secret references, role/capability requirements, availability, fallback policy, and selection results; the selected Runtime still performs authentication and model execution.
- **P3**: optional external CLI Session Adapters require explicit capability, version, legal/Terms, lifecycle, and compatibility gates.

### D-008 — Runtime contract sequencing

- **P0**: define an internal, versioned `RuntimePort v0` containing only the OMP capabilities required by the near-complete SBTD MVP; keep OMP events/types behind the adapter.
- **P1**: continue through `RuntimePort v0` while productizing the CLI; compatibility changes remain explicit and fixture-backed.
- **P2**: freeze `Runtime Contract v1` before extracting the Runtime-neutral Core, migrate v0 fixtures, and deliver the formal `runtime-omp` Adapter before P2 exit.
- **P3**: add `runtime-pi`, capability negotiation, and cross-Runtime compatibility; do not redefine the OMP boundary from scratch.
- **Consequence**: move Runtime Contract v1 and `runtime-omp` standardization from the current P3 dependency order into P2.

### D-009 — P1 OMP Runtime distribution

- **Default**: KPi resolves and manages a pinned-compatible OMP Runtime dependency in KPi's private/package scope so a new user can install and run KPi without a separate global OMP setup.
- **No fork/vendor**: KPi consumes the supported OMP distribution and does not copy or fork OMP source as part of P1.
- **External override**: advanced users may select an external OMP path/version explicitly; KPi must run version/capability Doctor and refuse or degrade unsupported combinations visibly.
- **Mutation boundary**: package installation does not use `postinstall` to modify AGENTS, Skills, tools, MCP, or projects. Any later Runtime install/update operation is plan-first and confirmed.
- **Consequence**: P1 packaging, compatibility CI, upgrade policy, and License/NOTICE must cover both the managed default and external override.

### D-010 — P0 Plugin publication channel

- **Canonical artifact**: publish immutable, versioned `@kpi/omp-sbtd` releases through npm with lockable versions and integrity metadata.
- **Marketplace role**: OMP Marketplace provides discovery and compatibility metadata that resolves to an exact supported npm release; registry/handler/docs tests must detect metadata drift.
- **Git role**: Git install/link is development-only or an explicitly labeled prerelease path, never the normal production update source.
- **Consequence**: the P0 installation contract and CI must test npm installation directly plus Marketplace-to-npm resolution.

### D-011 — Strict/relaxed check semantics

- **Decision**: Policy Profiles are monotonic and operate after Route classification.
- **Route result**: each Check is classified as `required`, `optional`, or `not-applicable` from objective predicates and explicit user acceptance criteria.
- **Relaxed**: performs no optional-to-required promotions; it never downgrades a Route-required Check.
- **Strict**: may promote checks explicitly marked by the Profile, initially independent review, broader affected-scope validation, optional UI/SEO polish, and additional confirmations.
- **Registry rule**: Check metadata, Help, Profile diff, status, and tests share one machine-readable registry; user/project configuration may add promotions but cannot demote Required.

### D-012 — Claude Code Subscription commercial boundary

- **P1**: may launch the official `claude` CLI login flow and display non-sensitive status only; product copy must not promise or imply use of Pro/Max subscription quota.
- **P3 gate**: any Claude CLI Coding Session Adapter remains disabled until a legal/Terms review is recorded for the exact adapter/CLI version, distribution regions, data path, and current commercial terms.
- **Failure behavior**: an absent, stale, or negative Terms decision keeps the adapter disabled; KPi recommends supported API/Bedrock/Vertex/Foundry routes and never falls back to reading local credential files.
- **Revalidation**: material CLI/auth/terms changes invalidate the prior decision and block release until reviewed again.

### D-013 — Python Onboard fallback lifecycle

- **P2 cutover**: TypeScript becomes the default only after read-only and mutating Golden Fixtures, differential runs, rollback tests, and cross-platform acceptance pass.
- **Fallback**: retain Python through the first P3 compatibility release as an explicit `--engine python` choice surfaced by Doctor; never auto-fallback after a TypeScript mutating operation starts.
- **Maintenance**: Python receives security, data-loss, and migration-compatibility fixes only; new Onboard behavior is TypeScript-first and must not expand the legacy engine.
- **Sunset gate**: at the first P3 compatibility release, use unresolved migration blockers and observed explicit-fallback usage to decide removal or conversion into an independent recovery tool.

### D-014 — Upstream workflow source and Kit artifact

- **Maintainer input**: synchronize `640-skills` only from a resolved full commit SHA; record repository, source ref, full SHA, source digest, Section Mapping version, licenses, and generated digests.
- **Canonical user artifact**: publish an immutable, versioned npm Kit package. Its manifest is the single source for generated AGENTS templates, machine rules, bundled Skills, schemas, and compatibility metadata.
- **P0 bootstrap**: the Plugin pins and either depends on or embeds the exact Kit release needed for offline Doctor/Plan/Onboard bootstrap.
- **Prohibited user paths**: no Runtime Floating Main, Git submodule requirement, or unreviewed remote Kit fetch. New upstream content must pass mapping, license, conformance, and release gates first.

### D-015 — Evidence P2 product boundary

- **Decision**: cloud Evidence Store and PR Check are an independent post-v0.3 milestone, not a P3 dual-Runtime exit criterion.
- **P0–P3 contract**: KPi owns local Evidence Envelopes with exact source revision, environment alignment, publication state, checksums, and export interfaces; it does not claim remote publication without an acknowledged target.
- **Future bounded context**: remote Evidence adds tenant identity, authorization, retention, ingestion idempotency, privacy, availability, and PR-provider integration and therefore requires its own PRD and mandatory DDIA review before design stabilizes.
- **Compatibility**: local envelope schemas should permit a future remote publisher, but P0–P3 must not implement a fake/no-op cloud store.

### D-016 — Runtime capability degradation

- **Capability status**: every Runtime Adapter reports each contract capability as `native`, `adapter`, `degraded`, or `unsupported`, with evidence and version.
- **Route requirements**: every Route declares which capabilities are required versus optional before execution.
- **Allowed degradation**: an optional capability may use a weaker, contract-tested path and must surface `degraded` in status/report.
- **Required failure**: missing capability for safety, pre-write/pre-tool blocking, Mandatory Gate enforcement, or evidence truth blocks that Route and recommends a compatible Runtime.
- **No prompt equivalence**: Prompt instructions cannot be reported as equivalent to a missing hard enforcement or trusted event hook.

### D-017 — KPi license

- **Decision**: KPi remains `GPL-3.0-only`, matching the current repository license text.
- **Boundary**: this decision applies to KPi-owned code and documentation; it does not relicense OMP, Pi, `640-skills`, external Skills, npm dependencies, generated third-party content, or user projects.
- **Change gate**: permissive relicensing or commercial dual licensing requires a separate legal review and ADR covering copyright ownership and contributor rights.

### D-018 — Third-party license and NOTICE propagation

- **Artifact manifest**: every bundled component, dependency, generated upstream asset, and External Skill records SPDX expression, source, version/full SHA, copyright/NOTICE material, and `bundled | dependency | external-installer | reference-only` distribution mode.
- **Release outputs**: Plugin, Kit, and CLI releases generate and ship an SBOM plus THIRD_PARTY_NOTICES matching their actual contents.
- **Fail-closed gate**: missing license evidence, prohibited redistribution, incompatible terms, or NOTICE mismatch blocks bundling and publication.
- **External Skills**: installation requires user-visible source/license information; content that KPi cannot redistribute is fetched only from the approved original source and cannot use an unlicensed vendored Stable fallback.
- **Verification**: CI checks artifact contents against the manifest rather than trusting package-manager metadata alone.

### D-019 — Runtime-native operation without Global AGENTS

- **P0/P1**: keep Mode-aware OMP Global AGENTS and project adapters as the default compatibility profile.
- **P2**: add an opt-in Runtime-native profile where the Runtime Adapter injects KPi workflow contracts directly and no KPi Global AGENTS installation is required.
- **Project facts**: Runtime-native mode still reads root Project AGENTS, deeper project rules, and explicit project context; it replaces KPi global/runtime instructions, not repository-owned facts.
- **P3**: promote Runtime-native to a supported profile only after OMP/Pi conformance proves equivalent classification, Gate, safety, state, and report semantics.
- **Migration**: existing Managed environments remain unchanged until an explicit plan/confirmation; rollback restores the prior profile without deleting project facts.

### D-020 — Structured project input and Project Facts

- **P2 feature**: `.kpi/project.yaml` is an optional, versioned, machine-validated input that generates only a dedicated KPi Managed Project Facts Block in root `AGENTS.md`.
- **Effective facts**: root Project AGENTS remains the cross-Agent readable facts layer; deeper project rules and block-external human content remain independent and authoritative at their scope.
- **Direction**: generation is one-way and plan-first. KPi does not infer structured YAML by reverse-parsing arbitrary AGENTS prose and does not regenerate the whole file.
- **Conflict behavior**: unknown schema fields, damaged markers, digest drift, or incompatible manual edits block generation and require an explicit resolution; no last-writer-wins merge.

### D-021 — Monorepo Workspace Adapter generation

- **P0/P1**: discover and report workspace roots, effective native adapter, imports, and shadowing; do not generate per-workspace adapters automatically.
- **P2 command**: provide an explicit, opt-in, plan-first operation for user-selected workspace roots.
- **Generated contract**: each Workspace Adapter has its own provenance and Managed Block and deterministically composes root Project Facts with workspace-local facts without copying either source wholesale.
- **Existing content**: a custom or drifted workspace adapter is `merge-required`; KPi never overwrites it by broad monorepo discovery.
- **Verification**: Doctor and conformance fixtures show the actual effective path and inheritance chain for root-only, nested, and shadowed cases.

### D-022 — Deterministic command versus model dispatch

- **CLI**: `kpi` without a command starts the interactive Session; one-shot natural-language input requires `kpi run -- <prompt>`.
- **Commands**: every unknown top-level or nested KPi command returns a deterministic error, nearest candidates, and Help; it never falls through to the model.
- **Slash namespace**: every unmatched `/sbtd ...` input is a command error and cannot create an Agent Turn.
- **Interactive messages**: ordinary messages entered after a Session starts remain Runtime-owned model input.
- **Registry**: parser, reserved namespaces, nested partials, candidate generation, Help, completions, tests, and telemetry derive from the same Command Registry.

### D-023 — Deterministic Environment Mode

- **Managed**: every baseline asset and capability required by the selected profile and current Route is present, valid, and effective.
- **Needs-onboard**: a required baseline asset such as OMP Global AGENTS, OMP Project Adapter, Required Skill, or normal Onboard baseline is missing without an applicable accepted skip.
- **Degraded**: a versioned provenance record proves that the user explicitly accepted skipping the specific asset/capability for this scope, and the current Route does not require it.
- **Blocked**: the current Route requires a missing, invalid, unsafe, or unsupported capability/contract.
- **Determinism**: skip acceptance is scoped to asset, project/profile, and Kit major; expired or mismatched acceptance returns to `needs-onboard`. Every fact set maps to one state, not an `A or B` acceptance.

### D-024 — Rule enable/disable boundary

- **P1 commands**: keep `kpi rules list|enable|disable|doctor`.
- **Mutability**: only Registry entries marked `configurable` and Optional may be enabled/disabled; Mandatory, Always-on, safety, evidence-truth, and Gate-owning rules are `immutable`.
- **Scope**: every mutation requires explicit `session`, `project`, or `user` scope and follows defined precedence; status shows the effective value and source.
- **Failure**: attempting to disable an immutable or Route-required rule returns a deterministic error naming the owning Gate and cannot be overridden by `relaxed`.
- **Contract**: Rule Registry drives command handlers, Help, config schema, status, migration, and tests so the PRD and roadmap command surfaces cannot drift.

## Open Decision Tree

1. [Resolved by D-001] KPi's canonical product identity and the boundary between workflow control plane and coding-agent runtime.
2. [Resolved by D-002 and D-003] P0 scope, value hypothesis, measurement method, and go/no-go threshold.
3. [Resolved by D-004] `enforced/advisory` Runtime Mode and `strict/relaxed` Policy Profile are independent dimensions.
4. [Resolved by D-005 and D-006] Managed-asset ownership, update/rollback eligibility, scoped uninstall, and residual-state reporting.
5. [Resolved by D-007] P0/P1 use OMP-managed auth; P1 official CLI delegation is limited to login launch and non-sensitive status, not Coding Session execution.
6. [Resolved by D-008] P0 uses internal `RuntimePort v0`; P2 freezes Contract v1 and formalizes `runtime-omp`; P3 adds `runtime-pi`.
7. PRD open decisions:
   1. [Resolved by D-009] KPi defaults to a managed compatible OMP dependency and allows a Doctor-gated external Runtime override.
   2. [Resolved by D-010] npm is canonical; OMP Marketplace is discovery/compatibility metadata; Git is development/prerelease only.
   3. [Resolved by D-011] `relaxed` performs no promotions and never demotes Route-required Checks; `strict` promotes declared Optional Checks.
   4. [Resolved by D-007] P1 `kpi provider login` versus Runtime-only delegation.
   5. [Resolved by D-012] Claude login/status is allowed in P1; Coding Session delegation is P3-only behind a versioned legal/Terms Gate and defaults disabled.
   6. [Resolved by D-013] retain an explicit, time-bounded Python fallback through the first P3 compatibility release; no automatic post-mutation fallback.
   7. [Resolved by D-014] full-SHA maintainer input generates an immutable versioned npm Kit; P0 pins it for offline bootstrap.
   8. [Resolved by D-015] Evidence P2 is an independent post-v0.3 milestone; P0–P3 provide local exportable Evidence Envelopes only.
   9. [Resolved by D-016] capability matrix permits tested degradation for Optional capabilities and blocks Routes missing Required enforcement; Prompt simulation is not equivalent.
   10. [Resolved by D-017 and D-018] KPi is GPL-3.0-only; releases use a fail-closed component manifest, SBOM, and THIRD_PARTY_NOTICES.
   11. [Resolved by D-019] P2 adds opt-in Runtime-native mode without KPi Global AGENTS; P3 supports it after conformance, while Project AGENTS remain authoritative facts.
   12. [Resolved by D-020] P2 may use versioned `.kpi/project.yaml` as one-way input to a dedicated Managed Project Facts Block; root AGENTS remains effective.
   13. [Resolved by D-021] P0/P1 detect only; P2 offers explicit plan-first Workspace Adapter generation with provenance and inheritance checks.

## DDD Boundary Review

- **Status**: `confirmed`
- **Ubiquitous language**: `docs/CONTEXT.md` now distinguishes KPi, SBTD Workflow Control Plane, Runtime, Runtime Adapter, Runtime/Environment Mode, Policy Profile, Managed Asset, Shared Dependency, Provider Login Delegation, Provider Coordination, CLI Session Adapter, SBTD Kit, Evidence Envelope/Service, Runtime Capability Status, Project Facts, and Runtime-native Profile.
- **Bounded contexts**:
  - **SBTD Workflow Control Plane** — classification, Route/workflow state, Gate and Policy decisions, validation/evidence truth, context orchestration, and reports.
  - **Runtime Integration** — Runtime contracts, adapters, capability evidence, and session/event translation.
  - **Environment Management** — Onboard planning/application, provenance, managed assets, migration, rollback, uninstall, and Doctor.
  - **Artifact Supply Chain** — upstream synchronization, Kit/Plugin packaging, integrity, license manifests, SBOM, and publication.
  - **Project Facts** — repository-owned cross-Agent facts consumed by KPi but not owned wholesale by the control plane.
  - **Evidence Service** — separate post-v0.3 context; only local Evidence Envelopes belong to P0–P3.
  - **Provider Coordination** — Provider Profiles, secret references, role/capability requirements, availability, fallback policy, and selection results; Runtime owns authentication and model execution.
- **Invariants and business rules**:
  - KPi does not own the Agent Loop, model execution, Runtime-native tools/UI, authentication refresh, streaming transport, or Runtime credential stores.
  - Required Route checks cannot be downgraded by Policy Profile or mutable Rule configuration.
  - Managed Asset mutation requires matching provenance and digest; Shared Dependency installation does not grant removal ownership.
  - Missing hard enforcement or trusted events cannot be represented as Prompt-equivalent capability.
  - Project Facts remain readable outside KPi-specific Runtime injection.
  - Remote Evidence publication cannot be claimed without an acknowledged external target.
- **Core / supporting / generic subdomains**:
  - **Core**: SBTD Workflow Control Plane.
  - **Supporting**: Runtime Integration, Provider Coordination, Environment Management, Artifact Supply Chain, local Evidence Envelope production, and CLI product shell.
  - **Generic/external**: OMP/Pi Runtime execution, Provider authentication/model transport, package registries, official Provider CLIs, secret stores, Git/CI, and Shared Dependencies.
  - **Future separate context**: Evidence Service.
- **Corrections applied to the grill-with-docs result**:
  - `KPi Core` means the Runtime-neutral implementation of the SBTD Workflow Control Plane, not the entire CLI/Onboard/Kit/Provider product.
  - Onboard and Kit are KPi supporting contexts with explicit contracts; they are not workflow-engine internals.
  - D-007's P2 `Provider Gateway` was narrowed to `Provider Coordination`; Runtime retains authentication and model execution.
- **Open conflicts and questions**: none.

## Final Document Re-review

- **Decision**: `request changes`
- **Source scope**: `docs/prd/PRD.md` and `docs/prd/ROADMAP.md`
- **Source mutation**: none
- **Finding counts carried from the commit review**: 7 high, 3 medium, 0 low, 0 blocking
- **Re-review conclusion**: the route is coherent, but the two source documents are not implementation-ready until the agreed product boundaries and contracts below are applied consistently.

### Bounded correction matrix

| ID | Decisions | Exact affected source sections | Required bounded correction |
|---|---|---|---|
| C-01 | D-001 | PRD `14-37`, `529-563`; ROADMAP `13-38`, `972-1003` | Replace “independent CLI Coding Agent” with Runtime-neutral SBTD workflow product/control plane. Define `KPi Core` as only the Runtime-neutral control-plane implementation; keep CLI shell, Onboard, Kit supply chain, Runtime execution, and Provider execution outside Core. |
| C-02 | D-002, D-003 | PRD `144-172`, `2335-2349`, `2370-2382`; ROADMAP `40-48`, `84-151`, `670-725` | Preserve the near-complete P0 scope but decompose it into ordered internal milestones. Replace “enough data” and directional metrics with the deterministic zero-bypass/zero-misrepresentation/zero-leak contract gate plus the paired 20-task numeric value gate. |
| C-03 | D-004, D-011 | PRD `287-305`, `464-517`, `2094-2142`; ROADMAP `184-218`, `287-336`, `508-531` | Persist `runtimeMode: enforced|advisory`, `policyProfile: strict|relaxed`, and `environmentMode` as separate dimensions. Add resume/compaction fixtures for all four Runtime Mode/Profile combinations; neither `relaxed` nor advisory state may falsify Environment Mode or demote Route-required checks. |
| C-04 | D-022, D-024 | PRD `317-371`; ROADMAP `761-782`, `855-862` | Define `kpi` as interactive-session launch and `kpi run -- <prompt>` as the only one-shot natural-language path. Unknown CLI or `/sbtd` commands must fail deterministically and never reach the model. Restore `rules enable|disable` in the roadmap only for explicitly `configurable` Optional Rules with explicit scope and immutable-rule rejection. |
| C-05 | D-005, D-006, D-023 | PRD `443-462`, `626-652`, `793-802`; ROADMAP `84-117`, `220-253`, `940-949` | Add a provenance inventory and exact-digest Managed Asset contract. Move the required `@../AGENTS.md` import inside the `omp-project-agents` Managed Block or a separately managed preamble. Define plan-first scoped uninstall, residual reporting, backups/purge, reload semantics, and deterministic Environment Mode derived from accepted-skip provenance and current Route capabilities. |
| C-06 | D-007, D-012 and DDD correction | PRD `168-170`, `1901-2022`, `2311-2321`; ROADMAP `641-668`, `872-892`, `1183-1213`, `1377-1393`, `1561-1577` | Replace P2 `Provider Gateway`/native model adapters with `Provider Coordination`: profiles, secret references, role/capability requirements, availability, fallback policy, and selection result. Runtime retains authentication, credential refresh, streaming, and model execution. P1 login remains launch/status only; external CLI Coding Session delegation is P3-only behind versioned Terms/capability gates. |
| C-07 | D-008, D-016 | PRD `529-563`, `2207-2213`, `2311-2330`; ROADMAP `1007-1028`, `1242-1251`, `1283-1350` | Introduce internal `RuntimePort v0` in P0, freeze Runtime Contract v1 and formalize `runtime-omp` before P2 exit, then add `runtime-pi` in P3. Require evidence-backed `native|adapter|degraded|unsupported` capability status; missing Route-required hard enforcement blocks rather than using prompt simulation. |
| C-08 | D-009, D-010, D-014 | PRD `39`, `804-840`, `1620-1626`, `2388-2397`; ROADMAP `50-68`, `119-151`, `792-809`, `940-949`, `1623-1630` | Make immutable npm releases canonical for Plugin and Kit; Marketplace resolves to exact npm metadata and Git is development/prerelease only. KPi P1 manages a pinned-compatible OMP dependency in private/package scope with a Doctor-gated external override. Maintainer sync accepts resolved full SHA only; remove Floating Main and user-side live upstream choices. |
| C-09 | D-015 | PRD `1430-1439`, `2388-2398`; ROADMAP `961-968`, `1420-1430` | Keep local Evidence Envelopes with exact revision, environment, integrity, and publication state in P0-P3. Move cloud Evidence Store/PR Check into an independent post-v0.3 bounded context with its own PRD and DDIA review; remove it from the P3 exit path and do not implement a no-op store. |
| C-10 | D-017, D-018 | PRD `1620-1626`, `2388-2400`; ROADMAP `940-949`, `1117-1125`, `1539-1557`, `1623-1630`, `1719-1732` | State `GPL-3.0-only` for KPi-owned assets. Add a fail-closed component/artifact manifest recording SPDX/source/version/full SHA/NOTICE/distribution mode, and require release-content-verified SBOM plus `THIRD_PARTY_NOTICES` for Plugin, Kit, and CLI. |
| C-11 | D-013 | PRD `32-34`, `2388-2397`; ROADMAP `1154-1181`, `1581-1609` | Make TypeScript default only after Golden Fixture/differential/rollback/cross-platform acceptance. Retain Python as an explicit, time-bounded `--engine python` fallback through the first P3 compatibility release; never auto-fallback after a mutating TS operation starts. |
| C-12 | D-019, D-020, D-021 | PRD `626-652`, `2388-2402`; ROADMAP `1127-1152`, `1319-1334`, `1581-1610` | Add P2 opt-in Runtime-native profile, one-way versioned `.kpi/project.yaml` input to a dedicated Managed Project Facts Block, and explicit plan-first Workspace Adapter generation. Keep root/deeper Project Facts authoritative; P0/P1 only detect workspace shadowing. P3 promotes Runtime-native only after OMP/Pi conformance. |
| C-13 | D-001–D-024 | PRD `2388-2402`; ROADMAP phase/dependency and exit sections | Replace the 13-item open-decision list with a resolved decision register or links to the task/ADR records. Update phase dependencies, acceptance criteria, release blockers, Doctor output, and compatibility fixtures so no resolved boundary remains documented as open. |
| C-14 | DDIA review of D-004–D-006, D-013, D-015, D-020, D-023, D-024 | PRD `793-802`, `1538-1626`, `2094-2142`; ROADMAP `287-336`, `894-922`, `1154-1181`, `1420-1430`, `1581-1610` | Separate versioned Session State, durable provenance inventory/operation journal, user/project configuration, derived Managed Blocks, and immutable Evidence Envelopes by owner and lifecycle. Require per-project transaction boundaries, single-writer locks, idempotent plan application, crash recovery, explicit schema migration/rollback, and repair evidence. |

### Prior 10 findings trace

| Prior finding | Resolution | Correction |
|---|---|---|
| Session mode conflates enforcement and strictness | D-004 | C-03 |
| Required root import has no managed owner | D-005 | C-05 |
| Unknown management command may reach the model | D-022 | C-04 |
| Floating Main conflicts with immutable Kit | D-014 | C-08 |
| Subscription CLI delegation precedes executable adapters | D-007, D-012 | C-06 |
| Runtime Contract v1 follows P2 adapter extraction | D-008 | C-07 |
| Uninstall lacks an ownership contract | D-005, D-006 | C-05 |
| P0 has no measurable go/no-go threshold | D-003 | C-02 |
| `kpi rules enable|disable` is absent from the roadmap | D-024 | C-04 |
| Missing Adapter state maps nondeterministically | D-023 | C-05 |

### Recommended source revision order

1. Update PRD identity, glossary, bounded contexts, and the resolved decision register.
2. Correct state, command, ownership, Provider, Runtime, and Evidence contracts in the PRD.
3. Re-sequence ROADMAP P0-P3 deliverables and exits to match those contracts.
4. Apply packaging, licensing, migration, Runtime-native, project-config, and monorepo decisions.
5. Run a final cross-document conformance pass over commands, schemas, states, phases, requirements, acceptance criteria, and release blockers.

## Book Gate Plan Outcome

| Skill | Trigger evidence | Stage | Gate state | Reviewer status |
|---|---|---|---|---|
| `book-ddd-distilled-modeling` | Mandatory after completed `grill-with-docs`; product identity and bounded contexts changed | After interview, before final re-review | `passed` | `confirmed` |
| `book-ddia-data-design` | Session persistence, provenance inventory, config/Managed Block ownership, migration/rollback, and Evidence Envelope schemas are in scope | Before stabilizing the review contract | `passed` | `confirmed` |
| `book-legacy-change-safety` | No existing behavior or production code changes | N/A | `not-required` | N/A |
| `book-refactoring-pass` | No existing production code changes | N/A | `not-required` | N/A |
| `book-release-readiness` | No production service/API/job/integration/deployment behavior changes | N/A | `not-required` | N/A |

## DDIA Data Design Review

- **Status**: `confirmed`
- **Data owner and source of truth**:
  - SBTD Workflow Control Plane owns the logical, versioned Session State; the Runtime Adapter may provide persistence mechanics but cannot redefine fields or transitions.
  - Environment Management owns the durable provenance inventory and operation journal. A target file is mixed ownership: only matching Managed Blocks are KPi-owned; block-external content remains user/project-owned.
  - User/project configuration remains user/project-owned input. `.kpi/project.yaml` is a source for one derived Managed Project Facts Block, never a reverse-generated mirror of arbitrary AGENTS prose.
  - Validation/Reporting owns immutable Evidence Envelope metadata; referenced runner artifacts remain tool/project artifacts bound by checksum.
  - Runtime or approved secret stores own credentials. KPi persists references and non-sensitive status only.
- **Write / read / async / failure paths**:
  - Mutating Onboard/upgrade/uninstall commands produce a deterministic plan and plan digest, acquire a scope lock, journal each step, write atomically per target, and update inventory only after the target result is known.
  - Multi-project operations are atomic per project, not globally atomic. One project failure does not roll back successful independent projects; aggregate output records each result and residual.
  - Session events are serialized by the Runtime Adapter with monotonic sequence/version data; Compaction/Resume loads the latest compatible state and rejects ambiguous downgrade.
  - P0–P3 Evidence export is local and synchronous to acknowledged artifacts. No remote queue/store/publication path exists in these phases.
  - Crash, cancellation, digest drift, lock conflict, unknown schema, or partial external-tool installation enters explicit recovery/merge-required/residual state; it never becomes silent success.
- **Consistency model**:
  - Read-your-writes and single-writer consistency within one Session or one project mutation scope.
  - Strong digest/provenance checks for Managed Assets and Evidence Envelopes.
  - Per-project results may differ after a multi-project run; aggregate status is explicit rather than pretending global atomicity.
  - Derived Managed Blocks are refreshed only from a validated source version and matching prior digest; human edits cause conflict, not last-writer-wins.
- **Idempotency / ordering / retry / deduplication**:
  - Operation identity includes scope, canonical target set, source/schema version, and plan digest. Reapplying the same completed plan is a no-op with the prior result.
  - Journal sequence prevents duplicate or reordered target mutations. Destructive steps are not automatically retried after an unknown result.
  - Session state uses monotonic event/state versions; stale writes are rejected.
  - Evidence Envelope identity binds source revision, environment, artifact checksums, and run identity; envelopes are append-only and never overwritten to change publication truth.
- **Schema / migration / backfill / rollback / replay**:
  - Session State, provenance inventory, operation journal, config, Rule Registry overrides, Runtime capability evidence, and Evidence Envelope each require an explicit `schemaVersion`.
  - Readers either migrate from a declared compatible version or fail with a repair plan; unknown future versions fail closed.
  - Asset migration preserves backups and the prior inventory entry. Rollback restores only digest-matching KPi-owned state and records residuals.
  - Python-to-TypeScript Onboard migration uses read-only and mutating Golden Fixtures plus differential, rollback, and crash-recovery tests; no automatic engine fallback after mutation begins.
  - No remote Evidence backfill/replay is designed before the separate Evidence Service PRD.
- **Observability and repair**:
  - Doctor/report expose effective schema versions, owner, configured/effective paths, digests, lock/journal state, migration status, residuals, and repair commands without secrets.
  - Corrupt journals, orphan backups, inventory/target drift, stale Session State, and checksum mismatch are distinguishable failure classes.
  - Repair is plan-first and cannot broaden ownership.
- **Required tests**:
  - Compaction/Resume across all Runtime Mode/Policy Profile combinations and schema migration boundaries.
  - Duplicate plan/apply, concurrent writer, cancellation, crash at every target step, corrupted journal, stale plan, digest drift, and rollback failure.
  - Multi-project partial failure with preserved successful project results.
  - `.kpi/project.yaml` to Managed Project Facts generation, unknown field/version, drift, and downgrade rejection.
  - Python/TypeScript differential and recovery fixtures.
  - Evidence Envelope checksum tamper, stale revision/environment, duplicate run, and publication-state immutability.
