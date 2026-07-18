# KPi CLI and Core TypeScript Guidelines

KPi is currently a greenfield repository: no application packages, production TypeScript, or tests exist yet. These guides therefore record a **PRD-backed design baseline**, not conventions inferred from production code. Future bootstrap work must reconcile them with the first accepted implementation rather than pretending that examples already exist.

Evidence: `README.md` defines KPi as an agent harness derived from Pi; `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 5–7 and 10–14, defines the CLI/core boundary, package architecture, TypeScript stack, workflow states, configuration, policy, validation, and filesystem layout.

## Guides

| Guide | Scope |
|---|---|
| [Directory Structure](./directory-structure.md) | Package ownership, dependency direction, and naming |
| [Filesystem and Configuration Persistence](./filesystem-config-persistence.md) | `.kpi/` data, config parsing, credentials, and persistence boundaries |
| [Type Safety](./type-safety.md) | TypeScript contracts, Zod boundaries, adapters, and state unions |
| [Error Handling](./error-handling.md) | CLI failures, validation states, provider failures, and exit behavior |
| [Logging](./logging-guidelines.md) | Structured audit events, redaction, and human/JSON output separation |
| [Quality](./quality-guidelines.md) | Verification evidence, tests, review rules, and local anti-patterns |

## Pre-Development Checklist

- Confirm the owning package before editing. KPi follows package boundaries such as CLI, runtime adapters, workflow, policy, validation, context, tools, and reporting; do not create a generic shared dumping ground. Evidence: PRD sections 7 and 14.1.
- Keep KPi workflow logic independent of Pi internals. Pi owns the agent loop, model calls, TUI, and tool calling; KPi owns workflow state, evidence, policy, validation, reporting, kit/rule/skill management, and context injection. Evidence: PRD sections 6 and 7.2.
- Use TypeScript on Node.js 22+ LTS and validate external data with Zod. `pnpm` plus `tsup`/`esbuild` is the build baseline. Evidence: PRD section 10.
- Identify every trust boundary: CLI arguments, environment variables, JSONC/YAML/Markdown config, kit manifests, provider responses, runtime events, tool results, and persisted `.kpi/` state. Evidence: PRD sections 10, 12, 14, and 19.
- Decide interactive and non-interactive behavior explicitly. Non-interactive commands require stable exit codes and machine-readable JSON; Pi remains responsible for runtime TUI presentation. Evidence: PRD sections 6 and 7.1.
- Define secret handling before reading provider credentials. Only approved environment, keychain, helper, OAuth, subscription CLI, or gateway sources are valid, and secrets must be redacted. Evidence: PRD sections 13.1 and 19.1–19.3.
- Model tool availability, workflow progress, and validation outcomes as explicit states; do not infer success from missing evidence. Evidence: PRD sections 11, 13.2, and 18.

## Quality Check

- Package imports preserve adapter and engine boundaries; workflow/core code has no direct Pi-internal coupling.
- Boundary inputs are parsed before use, and internal APIs expose domain types rather than raw `unknown`, unvalidated objects, or credentials.
- Human-readable output, TUI events, audit logging, and `--json` output are not mixed together.
- Failures retain a stable machine code, actionable message, explicit validation/tool state, and non-zero exit behavior where applicable.
- Validation evidence names the command, runner, artifact state, and any `blocked`, `skipped`, or `not-needed` reason; mock-backed evidence is never called full-stack.
- Logs, reports, errors, and persisted state contain no secret values or private token material.
- Added examples are labeled as design baselines until corresponding production source and tests exist.

Evidence for this checklist: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 5.2–5.3, 7.1–7.2, 13, 15.1, 17, 18, and 19.1–19.3.

**Language:** project spec documentation is written in English, matching this existing index convention.
