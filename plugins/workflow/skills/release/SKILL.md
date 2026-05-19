---
name: release
description: Use this skill when one or more ready-to-ship tasks have been merged and the user wants a versioned release. Generates changelog entries, creates tags or release notes, and moves merged tasks to Shipped.
---

# release - Versioning Stage

## Invariants

- Release only merged `Ready To Ship` tasks.
- Keep changelog entries user-focused.
- Do not include unmerged pull requests.
- Do not rewrite existing tags without explicit user approval.
- After moving released items to `Shipped`, re-read `TASKS.md` to confirm each row left `Ready To Ship`. A stale tracker blocks downstream and headless sessions.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `release`.

## Invocation

Supported version inputs:

- `auto` or no argument: derive from task `Version Impact`
- `patch`
- `minor`
- `major`
- explicit version such as `v1.2.3`

## Workflow

1. Read `AGENTS.md`.
2. Read `TASKS.md` and find merged items in `Ready To Ship`.
3. Read each task document for type, version impact, and summary.
4. Determine the next version.
5. Generate or update `docs/changelogs/CHANGELOG.md`.
6. Commit changelog changes if the repository workflow expects release commits.
7. Create a git tag when appropriate.
8. Create release notes using the available repository hosting tool when appropriate.
9. Move released items to `Shipped` in `TASKS.md`, removing each row from `Ready To Ship`.
10. Re-read `TASKS.md` and verify the released task rows are in `Shipped` and absent from `Ready To Ship`. If any row is not in the expected section, the move was missed — perform the edit now before handoff.

## Version Calculation

| Signal | Version Impact |
|--------|----------------|
| Breaking change or required user migration | `major` |
| New feature | `minor` |
| Bug fix, docs, chore, small enhancement | `patch` |

Highest impact wins when multiple tasks are released together.

## Changelog Template

```markdown
## [v{version}] - {Date}

### New Features
- {User-facing feature}

### Fixes
- {Bug fix}

### Improvements
- {Enhancement}

### Documentation
- {Documentation update}

### Breaking Changes
- {Required migration, if any}
```

## Handoff

```text
Release prepared: v{version}
Tasks shipped: {IDs}
Changelog: docs/changelogs/CHANGELOG.md
```
