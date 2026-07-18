# Directory Structure

Package boundaries for the planned TypeScript CLI/core architecture.

## Current Status and Evidence

KPi has no application source tree yet. The layout below is a **PRD-backed design baseline**, not a description of existing production packages. `README.md` identifies the product as a Pi-derived agent harness. `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 6, 7, 10, and 14.1, defines the package-oriented TypeScript architecture and recommended repository shape.

## Baseline Layout

```text
apps/
  cli/                         # kpi command entry points and global flags
packages/
  core/                        # runtime-neutral domain contracts and orchestration primitives
  runtime-pi/                  # the only default Pi integration boundary
  workflow-engine/             # routes, stages, state transitions, execution coordination
  sbtd-workflow-kit/           # versioned prompts, agents, skills, rules, workflows, policies
  kit-registry/                # kit manifests, install/upgrade/doctor
  skill-registry/              # bundled/project/external skill discovery and lazy loading
  rule-engine/                 # dynamic rule evaluation and delivery/tool guards
  policy/                      # command, path, network, dependency, and secret policy
  context/                     # repository, lessons, Trellis, BDD, and skill context
  tools/                       # KPi tool contracts and implementations
  validation/                  # validation runs, evidence, report artifact checks
  memory/                      # lessons and session/project memory
  reporting/                   # human and machine-readable final/session reports
python/
  legacy-onboard/              # temporary v0.1 onboard.py compatibility bridge
native/
  kpi-natives/                 # optional v0.3+ Rust/N-API acceleration only
```

This directory tree is adapted from PRD sections 7 and 14.1. Create only packages required by an approved task; do not materialize the whole roadmap as empty scaffolding.

## Package Ownership

- `apps/cli` parses global flags, dispatches registered commands, lazy-loads command implementations, and renders interactive or machine output. Unknown text may enter `run`; reserved management commands must never silently become model prompts. Evidence: PRD section 7.1.
- `runtime-*` packages implement runtime adapters. Only an adapter imports Pi- or OMP-specific APIs. Evidence: PRD sections 6, 7.2, and 17.
- `workflow-engine` owns stages, routes, state transitions, and coordination; it consumes abstract runtime, tool, policy, and validation contracts. Evidence: PRD sections 6 and 11.
- `policy`, `rule-engine`, and `validation` own enforceable decisions. CLI commands and runtime adapters call them rather than duplicating guards. Evidence: PRD sections 7.4 and 13.
- `context`, `memory`, and registries own loading/indexing their data; they do not render CLI output or call Pi directly. Evidence: PRD sections 7 and 8.2.
- `reporting` converts structured results into final human or JSON output; validators produce typed facts rather than formatted terminal prose. Evidence: PRD sections 7 and 15.1.

## Dependency Direction

Dependencies point inward through contracts:

```text
apps/cli -> workflow-engine -> core contracts
runtime-pi -----------------> core contracts
policy/rule-engine/tools/validation/context/reporting -> core contracts
```

`workflow-engine` may orchestrate packages through interfaces, but core policy or validation logic must not import CLI presentation or Pi internals. Keep provider adapters separate from runtime adapters: a subscription coding-agent CLI is an external runtime/model adapter, not a normal chat-completion implementation. Evidence: PRD sections 7.2, 7.5, and 19.1–19.2.

## File and Symbol Naming

The PRD establishes only these naming rules before production source exists:

- Package directories follow the PRD's kebab-case package names, and published packages use the `@kpi/` scope.
- Public TypeScript contracts use PascalCase names and camelCase members, as shown by `KPiRuntimeAdapter`, `WorkflowState`, `ToolEvidence`, `startSession`, and `toolEvidence` in PRD sections 7.2 and 18.

Do not invent TypeScript filename, barrel-export, or test-placement rules during bootstrap. Record those conventions after the first implementation establishes repeatable source and test examples. The current package vocabulary comes from PRD sections 7, 11, 13, 14, and 18.

## Local Anti-Patterns

- Deep-forking Pi or copying OMP internals for MVP functionality. Use `runtime-pi`; an OMP adapter is optional v0.3 work. Evidence: PRD sections 5.2, 9, 16, and 17.
- Importing Pi APIs directly from workflow, policy, validation, provider, or reporting packages. This defeats the replaceable-runtime boundary in PRD sections 5.3 and 7.2.
- Building browser/React packages for the MVP. No first-party browser frontend is selected; browser tooling is optional and evidence-gated. Evidence: PRD sections 5.2, 11.3, and 17.2.
- Putting unrelated helpers into `core`, `shared`, or `utils` to bypass ownership. A reusable primitive must have a clear contract and at least two legitimate consumers.
- Combining provider credentials, provider protocol logic, model routing, and Pi session control in one module. PRD sections 7.5 and 19 require those concerns to remain separable.
