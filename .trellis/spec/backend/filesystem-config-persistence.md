# Filesystem and Configuration Persistence

Project-scoped data and configuration rules for the filesystem-first MVP.

## Current Status and Evidence

KPi has no database, ORM, migration system, or production persistence implementation. The MVP persists project-scoped configuration, sessions, reports, lessons, validation history, tool evidence, and profiles under `.kpi/`. This guide is a **PRD-backed design baseline**, not an existing implementation pattern. Evidence: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 10, 12, 13, 14.2, 15, and 19.

## Ownership and Paths

- A filesystem/config module owns path resolution, decoding, schema validation, and writes for `.kpi/`; callers use typed repositories or stores rather than joining paths themselves.
- Resolve project data from an explicit project root. Never rely on ambient `cwd` after command parsing, because commands support `--project-root`. Evidence: PRD section 7.1.
- Treat every file read as untrusted input. Parse JSONC/YAML/JSON/Markdown with the appropriate decoder, then validate structured values with Zod before returning domain types. Evidence: PRD sections 10 and 12.
- Keep transient cache data under `.kpi/cache/`; it must not be treated as durable workflow evidence. Durable reports, validation history, and tool evidence use their named stores. Evidence: PRD sections 14.2 and 15.1.
- Do not introduce a database merely because the bootstrap template mentioned one. Reassess this guide only when an approved design adds a database-backed capability.

## Design-Baseline Example 1: `.kpi/` Layout

The following is a **design-baseline example from PRD section 14.2, not existing production source**:

```text
.kpi/
  config.jsonc
  plugins.lock.json
  sessions/
  reports/
  memory/
    lessons.md
    lessons/
      index.md
      topics/
  validation-history.json
  tool-evidence.json
  project-profile.json
  cache/
  rules/
  skills/
```

Do not add credential values to this tree by default. Provider authentication sources are references to environment variables, keychain entries, approved helpers, official OAuth/token flows, subscription CLIs, or gateways. Evidence: PRD section 19.3.

## Design-Baseline Example 2: Safe Config Shape

This is a **PRD-backed config baseline adapted from sections 12.2–12.3 and 19.3, not existing production configuration**:

```jsonc
{
  "agent": { "runtime": "pi", "defaultMode": "interactive" },
  "workflow": {
    "defaultRoute": "auto",
    "requireValidationAfterCodeChange": true,
    "requireFinalReport": true
  },
  "policy": {
    "profile": "local-guarded",
    "redactSecrets": true
  },
  "credentials": {
    "openai": { "mode": "env", "env": "OPENAI_API_KEY" },
    "codex": { "mode": "subscription-cli", "command": "codex" }
  }
}
```

The config stores a credential locator, never the resolved secret. Secret-bearing fields should be impossible to serialize through normal config/report types.

## Read and Write Rules

1. Resolve the canonical project-root-relative path through the persistence module.
2. Apply path policy before a read or write. `.env*`, `.ssh`, private keys, `.git/objects`, and other denied paths are not ordinary config sources. Evidence: PRD section 13.1.
3. Decode the format and validate the complete structured value at the boundary.
4. Return a typed value or a typed failure; do not return a partially parsed object.
5. Serialize only the owning schema's public fields. Redact or reject unexpected secret material before persistence.
6. Record auditable state changes where the PRD requires history, while keeping cache rebuildable.

The PRD does not yet define file-locking, concurrency, or schema-migration mechanics. Do not invent incompatible formats in isolated call sites; settle those mechanics in the first persistence implementation and update this guide with production evidence.

## Compatibility and Evolution

- Version manifests and lock files explicitly. Kit metadata is a versioned package contract. Evidence: PRD sections 7.3 and 8.4.
- Unknown or newer required fields produce a clear config/manifest failure; they must not be silently discarded.
- Defaults are applied by the validated config layer, not repeated by every command.
- A config migration must preserve a recoverable original and report changed files through normal KPi reporting once implementation exists. The exact backup/transaction mechanism is intentionally deferred because the PRD does not specify one.

## Local Anti-Patterns

- Reading `.env`, browser cookies, private token files, or undocumented client credentials as a convenience. Evidence: PRD sections 5.2 and 19.1, 19.5–19.8.
- Persisting raw API keys or OAuth tokens in `config.jsonc`, session events, reports, tool evidence, or validation history. Evidence: PRD sections 13.1 and 19.3.
- Using `JSON.parse(value) as Config` or trusting YAML/manifest data without schema validation. Evidence: PRD section 10.
- Letting commands or Pi adapters write `.kpi/` files directly.
- Treating cache presence, a stale report, or an old `tool-evidence.json` entry as proof of current availability. Evidence must name the current command/config/index state. Evidence: PRD sections 13.3, 15.2, and 17.
