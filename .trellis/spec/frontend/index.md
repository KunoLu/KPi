# Frontend Boundary

KPi currently has **no first-party browser or React frontend and no frontend application source**. This directory intentionally contains only this boundary note; component, hook, state-management, frontend type-safety, frontend quality, and frontend directory templates do not apply.

`README.md` describes KPi as an agent harness derived from Pi. `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 5.1–5.2 and 6, defines an independent CLI whose runtime UI/TUI, model loop, and tool calling are provided by Pi. KPi owns workflow, policy, validation, context, evidence, and reporting around that runtime. The PRD does not select React or any browser application framework for the MVP.

Browser-related items in PRD sections 5.2, 11.3, 13.3, and 17.2 are optional tools, diagnostics, or validation/report gates—not a KPi frontend. Do not create components, hooks, client state stores, browser routing, or frontend build configuration to implement CLI/core work.

## Where UI-Adjacent Work Belongs Today

- Pi runtime/TUI integration belongs behind the backend Runtime Adapter boundary.
- Human CLI rendering and non-interactive JSON belong to `@kpi/cli` and reporting contracts.
- Playwright/Chrome capabilities, if approved, belong to evidence-gated tool or validation packages rather than a frontend package.

Evidence: `docs/prd/KPi_Coding_Agent_技术方案_PRD_v0.2-provider-update.docx`, sections 6, 7.1–7.2, 11.3, and 13.3.

## Trigger for Future Frontend Specs

Add frontend spec files only after an approved product design introduces first-party browser/native UI source and establishes its actual framework, package boundary, state/data contracts, accessibility requirements, testing approach, and production examples. At that point, derive the file set from accepted code and tests; do not restore generic React templates preemptively.

Until that trigger occurs, this index is the complete frontend specification.
