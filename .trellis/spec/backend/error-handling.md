# Error Handling

Failure contracts for commands, adapters, policy, providers, and validation.

## Current Status and Evidence

KPi has no production error hierarchy or numeric exit-code registry yet. This guide defines a **PRD-backed design baseline**: non-interactive commands need stable exit codes and machine-readable JSON; validation and tool availability use explicit states; provider failures must be explainable and must not silently change models. Evidence: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 7.1, 11, 13, 15.1, 18, and 19.1.

## Failure Model

Normalize failures at the boundary that can add domain meaning:

- decoders/Zod schemas report invalid external data as config, manifest, tool-input, provider-response, or persisted-state failures;
- policy and rule engines return explicit allow/approval/block decisions rather than throwing for expected denials;
- runtime/provider adapters map dependency-specific exceptions into KPi failures while retaining a safe cause for diagnostics;
- validation returns one of the PRD statuses, with evidence and risk, rather than treating every non-pass as an exception;
- the CLI boundary maps a normalized failure to human output or a JSON envelope and assigns a centrally registered exit code.

Do not catch an error merely to log and rethrow it unchanged. Add domain context once, then log/render once at the command/session boundary.

## Design-Baseline Example: CLI Failure Envelope

This is a **PRD-backed design baseline adapted from sections 7.1, 13.2, and 15.1; it is not production source**. Numeric exit-code assignments remain intentionally centralized because the PRD requires stability but does not define the numbers.

```ts
export type FailureStatus = "failed" | "blocked" | "skipped";

export interface CliFailure {
  code: "CONFIG_INVALID" | "POLICY_BLOCKED" | "PROVIDER_UNAVAILABLE" | "VALIDATION_FAILED";
  status: FailureStatus;
  message: string;
  risks: readonly string[];
  nextSteps: readonly string[];
}

export interface JsonFailureEnvelope {
  ok: false;
  error: CliFailure;
}

export function finishWithFailure(error: CliFailure): JsonFailureEnvelope {
  process.exitCode = exitCodeFor(error.code);
  return { ok: false, error };
}
```

The eventual `exitCodeFor` registry must be versioned and covered by contract tests. Never assign ad hoc numbers in individual command handlers.

## Validation Is Data, Not Exception Control Flow

Use the PRD status vocabulary exactly:

| Status | Meaning |
|---|---|
| `passed` | Command succeeded and required evidence is complete. |
| `failed` | Command or report shows failure. |
| `blocked` | Tool, permission, environment, or prerequisite prevented execution. |
| `skipped` | A required check was skipped; include risk and reason. |
| `not-needed` | The route/scope does not require the check. |
| `skipped-for-report` | A cache/compression layer was unsuitable for report generation. |

Evidence: PRD section 13.2. A `blocked` tool is not a thrown crash, a `skipped` validation is not a pass, and a missing report cannot be normalized to success.

## Provider and Runtime Failures

- Map authentication invalid/expired, rate limit, unavailable region, unsupported capability, missing CLI, and unstable non-interactive support to distinct typed causes when the provider exposes enough evidence.
- Record whether the result is `blocked`, `skipped`, or an explicit fallback, including the original provider/model and final selection. No silent fallback is allowed. Evidence: PRD section 19.1.
- Subscription CLI adapters let the official CLI/SDK own credentials; errors must never include token files, cookies, refresh tokens, raw command environments, or unredacted stderr. Evidence: PRD sections 19.3 and 19.5–19.8.
- Runtime adapters translate Pi/OMP events and exceptions at the adapter boundary. Workflow code must not branch on Pi-specific error classes. Evidence: PRD sections 7.2 and 17.

## User and Machine Output

- Interactive mode renders an actionable message through the CLI/runtime presentation boundary; Pi owns runtime TUI behavior. Evidence: PRD section 6.
- `--json` output emits one valid structured envelope without banners, spinners, or log lines on stdout. Evidence: PRD sections 7.1 and 15.1.
- Diagnostics may include safe cause metadata on stderr or in an audit record, but public messages remain redacted.
- Final reports list changed files, commands, validation, tool evidence, skipped/blocked/not-needed outcomes, risks, and next steps. Evidence: PRD section 15.1.

## Local Anti-Patterns

- Empty `catch` blocks, unconditional `catch { return passed; }`, or replacing an error with a vague `Something went wrong`.
- Using thrown exceptions for expected policy denials or validation states.
- Logging a failure in an adapter and again in workflow, CLI, and reporting layers.
- Returning exit code zero after a non-interactive command has a terminal `failed` or `blocked` outcome.
- Mixing terminal prose with JSON output.
- Reporting mock-backed, contract-backed, or app-mocked validation as full-stack success. Evidence: PRD section 13.3.
- Silent model/provider fallback or raw credential content in errors. Evidence: PRD section 19.1.
