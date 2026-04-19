<!--
Repository: wellnex-projects
Path: issues/README.md
Purpose: Human-readable issue documents that mirror tracked GitHub Issues across the WellNex ecosystem
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 1.0.0
-->

# Issues

Human-readable issue documents for tracked problems, defects, and backlog
items in the WellNex ecosystem.

This folder is the **doc form** of the canonical `issue` stage in the
[item lifecycle](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md).
The legacy agent-generated machine view continues to live at
[`wellnex-docs/backlog/issues.json`](https://github.com/eny-tech/wellnex-docs/blob/main/backlog/issues.json)
and is treated as a generated index, not a competing source.

## When to file something here

- A defect or risk worth more than a one-line backlog row
- A remediation finding that needs context, options, or links
- A cross-repo concern that does not yet warrant a project space

Anything smaller stays in `backlog/issues.json`. Anything larger graduates
into `planning/` or `projects/`.

## Naming

```
YYYY-MM-DD.issue-<kebab-slug>.md
```

Examples:

- `2026-04-18.issue-localstorage-token-leak.md`
- `2026-04-18.issue-llm-rate-limit-backoff.md`

## Frontmatter

Use [`templates/item-template.md`](../templates/item-template.md) and set
`item_type: issue`, `canonical_stage: issue`. When the issue is mirrored to
GitHub, record the foreign key under `related.github_issues`.

## Relationship to GitHub Issues

| Source | Authority |
|--------|-----------|
| `issues/*.md` (here) | Canonical narrative + design context for the issue |
| GitHub Issue | Live execution backlog, comments, status |
| `wellnex-docs/backlog/issues.json` | Legacy generated rollup view consumed by agents |

The `item_id` ties all three together.

## Promotion

| To stage | Action |
|----------|--------|
| `planning` | Create the plan in `wellnex-docs/planning/`; append `location_stack` entry here; leave pointer stub |
| `projects/active/<slug>/` | Create the project folder; move or pointer-link this issue under it |
| `resolved` | Add a closure note in `wellnex-docs/reports/`; update `location_stack`; leave a pointer stub or move under `projects/resolved/` |
