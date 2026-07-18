# Logging Guidelines

Structured audit and output-channel rules for KPi CLI/core packages.

## Current Status and Evidence

No logging library or production event schema has been selected. This guide records the **PRD-backed logging and audit baseline**, not an existing implementation. KPi must retain auditable workflow, tool, command, diff, validation, and report facts while redacting credentials. Evidence: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 6, 12.2, 13, 15, 17, 18, and 19.

## Separate Four Output Channels

1. **Runtime UI:** Pi owns interactive TUI/runtime presentation. KPi emits typed events through the adapter; core packages do not print TUI text. Evidence: PRD section 6.
2. **Human CLI output:** concise command results and final reports intended for a person.
3. **Machine output:** one stable JSON document/stream for non-interactive modes; never contaminate stdout with logs. Evidence: PRD sections 7.1 and 15.1.
4. **Audit/diagnostic events:** structured records for session replay, workflow state, policy decisions, tool evidence, validation, and safe diagnostics.

A logger call is not a substitute for `ToolEvidence`, `ValidationReport`, or the final report. Those are typed product records; logging observes them.

## Required Event Shape

Until a production schema exists, structured events should include only fields relevant to the event:

- `timestamp`, `level`, `event`, and schema/version identifier;
- `sessionId`, `command`, workflow `stage`, and `route` when known;
- tool/provider/runtime identifier and explicit state;
- validation status, command identity, duration, exit status, and report/artifact references;
- policy/rule decision, approval state, and safe reason;
- safe error code and redacted cause metadata.

Never serialize entire config, process environments, provider payloads, runtime events, tool arguments/results, or arbitrary thrown objects into a log record.

## Design-Baseline Example: Credential Redaction

This is a **PRD-backed design baseline adapted from sections 13.1 and 19.1–19.3; it is not production source**:

```ts
const REDACTED = "[REDACTED]" as const;

interface ProviderAuditEvent {
  event: "provider.request" | "provider.failure" | "provider.fallback";
  provider: string;
  model?: string;
  credentialMode: CredentialProfile["mode"];
  credentialValue?: never;
  status: "started" | "failed" | "blocked" | "fallback";
}

export function redactCredential(value: string | undefined): typeof REDACTED | undefined {
  return value === undefined ? undefined : REDACTED;
}
```

The important contract is `credentialValue?: never`: normal audit types must make secret inclusion invalid. A production redactor must also sanitize known secret headers, environment values, query parameters, command output, nested causes, and provider-specific fields before any sink.

## Levels

A concrete logging library and exact level configuration are deferred until implementation. Use these semantic levels consistently:

- `debug`: local diagnostic detail that is safe after redaction and disabled by default.
- `info`: expected lifecycle transitions such as session start/end, route selection, command completion, and validation completion.
- `warn`: recoverable degradation, approval denial, skipped required check, stale/missing evidence, or explicit provider fallback.
- `error`: terminal command/session failure or invariant breach requiring action.

Do not use `error` for normal `not-needed` outcomes. Do not downgrade security blocks, missing required reports, or silent fallback risks to `debug`.

## Audit Requirements

- Record workflow stage/route transitions and their reasons. Evidence: PRD sections 11 and 15.2.
- Record tool evidence with commands attempted, states, risks, and next steps. Evidence: PRD sections 17 and 18.2.
- Record policy/rule decisions without logging denied secret/path contents. Evidence: PRD sections 7.4 and 13.1.
- Record validation command identity, result status, and report evidence; verify current artifact metadata rather than trusting stale output. Evidence: PRD section 13.3.
- Record explicit model/provider fallback and why it occurred. Evidence: PRD section 19.1.
- Final reports include changed files, commands, validation, tool evidence, skipped/blocked/not-needed outcomes, and risks. Evidence: PRD section 15.1.

## Never Log

- API keys, bearer tokens, OAuth/refresh tokens, cookies, private keys, keychain contents, helper output containing secrets, or undocumented provider credential files.
- Full environment dumps, `.env*` contents, credential-bearing request headers, provider raw request bodies, or command strings after secret interpolation.
- Unredacted stdout/stderr from official subscription CLIs.
- Browser session/cookie data used to imitate subscription authentication; that access is prohibited, not merely sensitive. Evidence: PRD sections 5.2 and 19.1.

## Local Anti-Patterns

- `console.log` from core, workflow, policy, validation, provider, or persistence packages.
- Logging and JSON machine output to the same stdout stream.
- Logging raw tool input/output “for debugging.”
- Treating a log line saying a tool is installed as current Tool Evidence.
- Emitting fallback success without the original provider/model, reason, and final selection.
- Reporting a cache hit or old artifact as a current validation pass. Evidence: PRD sections 13.3 and 17.
