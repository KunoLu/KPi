# Workflow Lessons

## LESSON-20260721-book-gate-trigger-recheck: Re-evaluate Book Gates after scope decisions

- Date: 2026-07-21
- Tags: workflow, book-gate, ddia, data-ownership, planning
- Applicable scenarios: Requirements or design review starts as docs-only but introduces persisted state, schemas, data ownership, migration, rollback, replay, or recovery contracts.
- Severity: high
- Source: `.trellis/tasks/07-21-review-kpi-v0-7-boundaries/prd.md`
- Problem: The initial Book Gate Plan treated `book-ddia-data-design` as on-demand/not-required because the task was a document review, although the clarified decisions covered Session State persistence, provenance inventory, configuration-derived Managed Blocks, schema migration, rollback, and Evidence Envelopes.
- Root cause: Gate selection followed the artifact type (“docs review”) instead of the semantic change triggers defined by the gate matrix.
- Fix: Promote DDIA to required as soon as those triggers appear, run the independent DDIA review before stabilizing the design, and map its required changes into the source-document correction plan.
- Prevention: Re-evaluate objective Book Gate triggers after every scope-changing product decision and once more before a PRD/design review is declared stable. Persisted/shared data, ownership, migration, and recovery semantics trigger DDIA even when no production code is edited.
