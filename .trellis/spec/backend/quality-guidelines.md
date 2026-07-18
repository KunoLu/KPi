# Quality Guidelines

Review and verification standards for the first production packages.

## Current Status and Evidence

KPi has no package scripts, linter configuration, test runner, source tests, or production implementation yet. Do not invent command names or claim coverage. This guide is a **PRD-backed quality baseline** for the first TypeScript packages. Evidence: `README.md` and `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 5–19.

## Definition of a Reviewable Change

A change is reviewable when it:

- stays inside the package boundary defined by the architecture;
- validates external data at trust boundaries and exposes typed internal contracts;
- preserves Runtime Adapter isolation from Pi internals;
- handles human, JSON, audit, and TUI output independently;
- records explicit workflow, tool-evidence, policy, validation, and provider states;
- redacts credentials in logs, reports, errors, persisted state, and command output;
- includes focused verification for its observable contract once a test runner exists;
- reports commands, evidence, skipped checks, risks, and remaining uncertainty honestly.

Evidence: PRD sections 5.3, 6–7, 10–13, 15, 18, and 19.

## Implementation Standards

- Use TypeScript on Node.js 22+ LTS. Use `pnpm` and `tsup`/`esbuild` once package scripts are established; Bun remains optional rather than an MVP runtime requirement. Evidence: PRD section 10.
- Keep command handlers thin: parse/validate flags, invoke an owning service/engine, and render one output mode. Lazy-load commands to protect startup. Evidence: PRD section 7.1.
- Depend on runtime/provider/tool interfaces, not Pi internals or external CLI response shapes. Evidence: PRD sections 7.2, 17, and 19.2.
- Centralize policy, rule, validation-status, exit-code, and report-shape registries. Duplicated string literals across handlers create incompatible machine contracts.
- Prefer small explicit modules over framework-style abstractions without two real consumers. Greenfield code does not justify speculative indirection.
- Consider allocations and serialization: do not clone whole workflow states, tool results, provider responses, or file contents merely to select a few report fields.

## Verification Strategy

The first implementation task must establish package-local scripts and tests before this guide can cite production examples. Until then, reviewers must not prescribe nonexistent commands.

Once tests exist, verify behavior at these boundaries:

- CLI command routing, reserved-word protection, `--json` shape, and stable exit mapping;
- Zod rejection and normalization for config, manifests, tool input, runtime/provider events, and persisted state;
- workflow transitions and exhaustive route/stage handling;
- Runtime Adapter contract tests that do not leak Pi-specific types;
- policy blocks/approvals and recursive credential redaction;
- validation status semantics and report-artifact freshness;
- provider failure/fallback reporting and official credential-source constraints;
- filesystem path-policy and `.kpi/` serialization behavior.

Evidence: PRD sections 7.1–7.2, 10–15, 17–19.

For permanent behavior, a test must fail on a plausible regression. Test stable output/state transitions and security boundaries, not source text, internal call counts, or implementation trivia.

## Validation Evidence Rules

- Use the project's native runner when it exists. `rtk` is an output-compression layer, not a test runner. Evidence: PRD section 13.3.
- For report-producing validation, confirm the command, runner, report path, freshness/content, and status. Cache/replay or unchanged artifacts cannot prove the current run. Evidence: PRD sections 13.3 and 17.
- Record `passed`, `failed`, `blocked`, `skipped`, `not-needed`, or `skipped-for-report`; never collapse them into a boolean. Evidence: PRD section 13.2.
- Mock-backed, contract-backed, and app-mocked results cannot be reported as full-stack passed. Evidence: PRD section 13.3.
- Missing tools, permissions, environments, logins, or reports are explicit `blocked`/`skipped` outcomes with risk, not fabricated success.

## Mandatory Review Questions

- Does the change belong to this package, or is it crossing a runtime/policy/validation/provider/persistence boundary?
- Is every untrusted value parsed before domain use?
- Can any secret reach a logger, error, report, JSON output, session record, or `.kpi/` file?
- Does the machine-readable contract remain stable and uncontaminated by prose?
- Are every non-pass and provider fallback visible with evidence and reason?
- Does verification exercise the changed contract and a plausible failure path?
- Does the final report distinguish what was run, skipped, blocked, not needed, or unavailable?
- Is a design-baseline example being mistaken for production evidence?

## Local Anti-Patterns

- Deep-forking Pi in MVP or copying OMP's broad native/tool surface. Evidence: PRD sections 5.2, 9, 16, and 17.
- Direct Pi coupling outside `@kpi/runtime-pi`.
- Raw credential reads, browser cookie/token extraction, private credential-file access, or unredacted logging. Evidence: PRD sections 5.2, 13.1, and 19.1–19.8.
- Silent model/provider fallback. Evidence: PRD section 19.1.
- Reporting mock-backed, contract-backed, or app-mocked validation as full-stack.
- Using unvalidated boundary data or TypeScript assertions as validation.
- Treating `rtk`, cache output, a stale report, or a log line as proof of a current validation run.
- Expanding optional browser, LSP/DAP, native Rust, collaboration, or multi-agent features into MVP without an approved task. Evidence: PRD sections 5.2 and 16.
