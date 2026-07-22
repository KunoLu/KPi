# KPi Product Context

KPi defines the language for a runtime-neutral SBTD workflow product that coordinates engineering work without owning the underlying coding-agent execution environment.

## Language

**KPi**:
The product that provides the runtime-neutral SBTD workflow control plane and its user-facing distribution.
_Avoid_: OMP wrapper, Pi-derived agent, standalone agent runtime

**SBTD Workflow Control Plane**:
The KPi-owned domain that classifies work and governs workflow state, gates, policy, validation evidence, context orchestration, and reporting across compatible runtimes.
_Avoid_: agent loop, runtime, prompt bundle

**Runtime**:
A host coding-agent environment that owns the agent loop, model and provider execution, native tools, approvals, UI, and session primitives consumed by KPi.
_Avoid_: KPi Core, workflow engine

**Runtime Adapter**:
The boundary that translates a Runtime's public capabilities and events into the contracts required by the SBTD Workflow Control Plane.
_Avoid_: Runtime fork, provider adapter

**OMP-first**:
KPi's initial validation and delivery strategy using OMP as the first default Runtime Adapter; it is not KPi's product identity or permanent ownership boundary.
_Avoid_: OMP-only, OMP-native KPi

**Runtime Mode**:
The session-level choice between `enforced`, where KPi automatically governs the SBTD workflow, and `advisory`, where those automatic workflow controls are inactive.
_Avoid_: strict mode, environment status

**Policy Profile**:
The `strict` or `relaxed` setting that changes optional workflow checks without changing Runtime Mode or bypassing route-required Gates.
_Avoid_: on/off mode, enforcement state

**Managed Asset**:
A file block or private artifact that KPi may update because its provenance, ownership, source revision, and installed digest are recorded and still match.
_Avoid_: any file or tool previously touched by Onboard

**Shared Dependency**:
An external tool, Skill, or configuration that KPi may detect or help install but does not own or remove by default.
_Avoid_: KPi-managed asset, bundled component

**Provider Login Delegation**:
A user-controlled handoff that opens a Runtime or official CLI login flow and returns only non-sensitive availability status to KPi.
_Avoid_: credential import, Coding Session delegation

**CLI Session Adapter**:
A Runtime Adapter that delegates Coding Session execution to an official external CLI under explicit capability, version, and Terms gates.
_Avoid_: login helper, token proxy

**SBTD Kit**:
An immutable, versioned KPi artifact generated from a full-SHA upstream workflow source and containing the aligned templates, machine rules, Skills, schemas, and compatibility metadata used by the control plane.
_Avoid_: live upstream checkout, Floating Main, prompt bundle

**Evidence Envelope**:
A local, exportable record that binds validation artifacts to an exact source revision, environment alignment, integrity metadata, and publication state.
_Avoid_: cloud evidence store, test output alone

**Evidence Service**:
A future post-v0.3 bounded context that stores and publishes Evidence Envelopes across tenants and external PR systems.
_Avoid_: local report renderer, P3 exit requirement

**Runtime Capability Status**:
The evidence-backed `native`, `adapter`, `degraded`, or `unsupported` classification of one Runtime Contract capability for a specific Runtime version.
_Avoid_: installed, assumed available, prompt-equivalent

**Project Facts**:
Repository-owned commands, paths, constraints, and local rules expressed through root Project AGENTS and deeper project context, independent of the selected Runtime.
_Avoid_: KPi global policy, Runtime instructions

**Runtime-native Profile**:
A KPi execution profile where a Runtime Adapter injects workflow contracts directly without installing KPi Global AGENTS while still consuming Project Facts.
_Avoid_: no-AGENTS mode, project-facts-free mode

**Environment Mode**:
The deterministic `managed`, `needs-onboard`, `degraded`, or `blocked` assessment derived from effective assets, accepted skips, and the current Route's required capabilities.
_Avoid_: Runtime Mode, ambiguous fallback state

**KPi Core**:
The Runtime-neutral implementation of the SBTD Workflow Control Plane contracts and decisions.
_Avoid_: entire KPi product, CLI shell, Onboard, Kit supply chain, Runtime

**Route**:
The classification result that selects the applicable workflow, required capabilities, Gates, and Checks for a task.
_Avoid_: model prompt, Runtime path

**Gate**:
A stateful workflow decision barrier with an objective predicate, required evidence, allowed transitions, and blocking behavior.
_Avoid_: optional suggestion, test command alone

**Rule**:
A machine-readable predicate or policy decision used by classification, Routing, Gates, or Checks; only Registry entries explicitly marked `configurable` may be toggled.
_Avoid_: arbitrary prompt prose, disableable Hard Gate

**Onboard**:
The Environment Management workflow that plans and applies installation, migration, rollback, and scoped uninstall operations using provenance and Managed Asset contracts.
_Avoid_: SBTD workflow engine, package postinstall

**Artifact Supply Chain**:
The supporting context that synchronizes reviewed upstream sources and produces integrity- and license-verified Plugin, Kit, CLI, SBOM, and notice artifacts.
_Avoid_: live upstream loading, package registry alone

**Provider Coordination**:
The KPi supporting context for Provider Profiles, secret references, role/capability requirements, availability, fallback policy, and selection results passed to a Runtime.
_Avoid_: model execution, credential refresh, streaming transport, Provider Gateway
