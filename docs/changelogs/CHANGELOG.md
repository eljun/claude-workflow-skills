# Changelog

## [v1.6.0] - 2026-05-19

### New Features

- `ship` self-heals when intermediate stages (`simplify`, `test`, `document`) were skipped. It always advances the task row to `Ready To Ship` regardless of starting section, and discloses which stages ran in a new `Skipped Stages` line in the PR body and handoff text.
- `ship` detects skipped stages from durable artifacts (`Implementation Notes` and `Quality Gate Notes` sections in the task doc, `docs/testing/{ID}-*.md`, `docs/learnings/{ID}-*.md`), not from the tracker section — so a stale tracker cannot misreport which stages actually ran.

### Improvements

- `TASKS.md` tracker discipline promoted to a top-level rule in `AGENTS.md`. Every status-moving stage (`task`, `implement`, `test`, `document`, `ship`, `release`) now has the tracker update listed as an invariant and a final "re-read and verify" step before handoff, addressing stale rows that previously blocked downstream headless sessions.
- `simplify` acts as a cross-stage guard: if the task doc has `Implementation Notes` but `TASKS.md` still shows the row in `Planned` or `In Progress`, it reconciles the row to `Testing` rather than propagating the staleness. If `Implementation Notes` is absent, it does not reconcile.
- `document` is now idempotent for tracker moves: it reconciles the row to `Approved` from any earlier section if `test` missed the update, otherwise it is a no-op.

### Documentation

- README explicitly documents that stages between `implement` and `ship` are skippable and that `ship` self-heals — useful for headless sessions running small tasks that do not need the full chain.
