<!--
Repository: wellnex-projects
Path: README.md
Purpose: Landing page and catalog for the WellNex project lifecycle repository (ideas, issues, projects, templates)
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 0.1.0
-->

# WellNex Projects

Lifecycle home for iterative work across the WellNex ecosystem. Every unit
of work — from a rough idea to a delivered project — lives here with a
durable identity that links upstream to GitHub Issues, GitHub Projects, and
the sibling repos (`wellnex`, `wellnex-server`, `wellnex-health-agents`,
`wellnex-agents`, `wellnex-docs`).

The lifecycle model itself is defined in
[`wellnex-docs/planning/workflow-platform/`](https://github.com/eny-tech/wellnex-docs/tree/main/planning/workflow-platform);
this repository operationalizes it.

## Lifecycle Folders

| Directory | Stage | Contents |
|-----------|-------|----------|
| [ideas/](ideas/) | idea | Early ideas, seeds, and untriaged candidates |
| [issues/](issues/) | issue | Human-readable issue docs (mirrors GitHub Issues) |
| [projects/](projects/) | project | Active execution spaces (`active/`, `recent/`, `resolved/`, `parked/`) |
| [templates/](templates/) | — | Reusable item templates with the lifecycle frontmatter contract |

Long-form **plans** continue to live in
[`wellnex-docs/planning/`](https://github.com/eny-tech/wellnex-docs/tree/main/planning).
Plans are referenced from project READMEs, not duplicated here.

## How Items Flow

```
idea  ──►  issue  ──►  plan  ──►  project  ──►  resolved  ──►  archived
(here)    (here)     (docs)      (here)         (docs)        (docs)
```

The `item_id` is assigned at first capture and never changes. Every move
appends a new entry to the item's `location_stack` and may leave a pointer
stub at the previous location. See
[03 — Item Lifecycle Orchestration](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md).

## Naming

| Stage | Filename pattern | Example |
|-------|------------------|---------|
| idea | `YYYY-MM-DD.idea-<slug>.md` | `2026-04-18.idea-confluence-pattern-import.md` |
| issue | `YYYY-MM-DD.issue-<slug>.md` | `2026-04-18.issue-localstorage-token-leak.md` |
| project | folder `<slug>/` with `README.md` landing page | `projects/active/workflow-platform/` |

## Frontmatter Contract

Every item carries the lifecycle frontmatter from
[03 § Core Metadata Contract](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md#core-metadata-contract).
Use [`templates/item-template.md`](templates/item-template.md) as the
starting point.

## Item-Flow Overlay

[`.item-flow/overlay.yaml`](.item-flow/overlay.yaml) maps the canonical
lifecycle stages to physical folders in this repository. The
`wellnex-agents` CLI reads this file when listing, validating, or moving
items.

## Related Repositories

| Repo | Role |
|------|------|
| [wellnex](https://github.com/eny-tech/wellnex) | React frontend |
| [wellnex-server](https://github.com/eny-tech/wellnex-server) | Fastify API |
| [wellnex-health-agents](https://github.com/eny-tech/wellnex-health-agents) | In-app middleware pipeline |
| [wellnex-agents](https://github.com/eny-tech/wellnex-agents) | Dev agent CLI toolchain |
| [wellnex-docs](https://github.com/eny-tech/wellnex-docs) | Documentation hub (plans, architecture, security) |
