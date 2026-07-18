# Type Safety

Runtime-safe TypeScript contracts for KPi trust boundaries and state.

## Current Status and Evidence

KPi is TypeScript-first on Node.js 22+ LTS, with Zod selected for config, manifests, tool parameters, and workflow-state validation. No production TypeScript exists yet, so every snippet below is a **PRD-backed design-baseline example**, not existing source. Evidence: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 7, 10–13, 18, and 19.

## Boundary Rule

External values enter as `unknown` and become domain values only after parsing. Boundaries include:

- CLI arguments and environment variables;
- JSONC/YAML/Markdown config and kit manifests;
- `.kpi/` persisted state;
- provider/runtime events and external CLI JSON/streams;
- tool parameters/results and report artifacts.

Use Zod schemas as the runtime source of truth and infer TypeScript types from them where practical. Do not duplicate an interface and schema that can drift. Internal constructors and functions accept parsed types, not raw transport objects. Evidence: PRD sections 10, 12, 13, and 19.1–19.4.

## Design-Baseline Example 1: Validated Config Boundary

This is a **PRD-backed baseline adapted from sections 10, 12, and 19.3; it is not production source**:

```ts
import { z } from "zod";

const credentialProfileSchema = z.object({
  provider: z.string().min(1),
  mode: z.enum([
    "env",
    "keychain",
    "apiKeyHelper",
    "oauth-token",
    "subscription-cli",
    "gateway-token",
  ]),
  env: z.string().min(1).optional(),
  helperCommand: z.string().min(1).optional(),
  secret: z.literal(true).optional(),
  policy: z.object({
    redact: z.literal(true),
    allowInCI: z.boolean().optional(),
    allowProjectOverride: z.boolean().optional(),
  }),
});

type CredentialProfile = z.infer<typeof credentialProfileSchema>;

export function parseCredentialProfile(input: unknown): CredentialProfile {
  return credentialProfileSchema.parse(input);
}
```

Cross-field requirements such as `env` being required for `mode: "env"` must be encoded with a discriminated union or refinement when implemented; never defer them to scattered callers.

## Design-Baseline Example 2: Runtime Adapter Contract

This is a **PRD-backed baseline from section 7.2, not production source**:

```ts
export interface KPiRuntimeAdapter {
  readonly id: "pi" | "omp" | (string & {});
  startSession(input: StartSessionInput): Promise<KPiRuntimeSession>;
  injectContext(context: KPiContextInjection): Promise<InjectionResult>;
  registerTools(tools: readonly KPiToolDefinition[]): Promise<void>;
  registerHooks(hooks: readonly KPiHookDefinition[]): Promise<void>;
  sendUserMessage(message: string): Promise<void>;
  events(): AsyncIterable<KPiRuntimeEvent>;
  stop(): Promise<void>;
}
```

Workflow packages depend on this contract. Pi-specific session, hook, and event types remain inside `@kpi/runtime-pi`; do not leak them into core state.

## Design-Baseline Example 3: Workflow and Tool Evidence

This is a **PRD-backed baseline adapted from sections 11, 13.2, and 18.1–18.2, not production source**:

```ts
export type ToolState =
  | "available"
  | "installed"
  | "missing"
  | "configured"
  | "unavailable"
  | "blocked"
  | "skipped"
  | "skipped-by-user"
  | "not-needed";

export interface ToolEvidence {
  tool: string;
  state: ToolState;
  evidence: readonly string[];
  commandsTried: readonly string[];
  risks: readonly string[];
  nextSteps: readonly string[];
}

export interface WorkflowState {
  id: string;
  stage: KPiStage;
  route?: WorkflowRouteId;
  activeSkills: readonly SkillDescriptor[];
  toolEvidence: readonly ToolEvidence[];
  validation: ValidationReport;
  report: FinalReportDraft;
}
```

Represent validation outcomes with the closed PRD vocabulary: `passed`, `failed`, `blocked`, `skipped`, `not-needed`, and `skipped-for-report`. A boolean such as `validationPassed` loses required evidence and skip semantics. Evidence: PRD section 13.2.

## Type Design Rules

- Prefer discriminated unions for stages, validation results, provider authentication modes, policy decisions, and runtime events.
- Make evidence collections `readonly` when consumers should not mutate recorded facts.
- Use branded or opaque IDs when two string identifiers are easy to mix after the domain model stabilizes; do not introduce brands without a real confusion boundary.
- Keep wire/config schemas separate from domain types when normalization is required. Parse first, then transform explicitly.
- Return explicit result/failure types at recoverable boundaries. Reserve thrown exceptions for programmer defects or a single command boundary that immediately normalizes them.
- Exhaustively handle closed unions with `never`; adding a new state must produce compile-time work at every required renderer/transition.
- Keep secret-bearing values in narrow credential-broker types. Public report, log, state, and config types must not expose secret fields. Evidence: PRD sections 13.1 and 19.1–19.3.

## Local Anti-Patterns

- `as Config`, `as WorkflowState`, or `as ToolEvidence` on external data.
- `any` in adapter, provider, tool, config, validation, or persistence contracts.
- Boolean availability/success flags that erase `blocked`, `skipped`, `not-needed`, or evidence details.
- Importing Pi types into workflow/core packages instead of mapping them in the runtime adapter.
- Modeling credentials as plain strings that can flow into generic serialization or logging.
- Silently substituting a provider/model without recording fallback state, cause, and final selection. Evidence: PRD section 19.1.
