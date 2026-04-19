<!--
Repository: wellnex-projects
Path: projects/README.md
Purpose: Catalog of active, recent, resolved, and parked WellNex project execution spaces
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 1.0.0
-->

# Project Catalog

Project execution spaces for the WellNex ecosystem. Each discrete project
gets its own folder with a `README.md` landing page and any number of
supporting documents.

This folder is the canonical home for the `project` stage in the
[item lifecycle](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md).

## Lifecycle Navigation

| Catalog | Purpose |
|---------|---------|
| [../ideas/](../ideas/) | Early-stage ideas and seeds |
| [../issues/](../issues/) | Tracked problems and remediation items |
| [`wellnex-docs/planning/`](https://github.com/eny-tech/wellnex-docs/tree/main/planning) | Strategy, migration, and design documents |
| [./](./) | Active execution spaces |

## Status Entry Points

| View | Purpose |
|------|---------|
| [active/](active/) | Currently driving work |
| [recent/](recent/) | Newly created or recently updated |
| [resolved/](resolved/) | Delivered or explicitly closed |
| [parked/](parked/) | Stale, paused, or awaiting reactivation |

## Folder Conventions

- Every discrete project gets its own kebab-case folder.
- Every project folder must have a `README.md` as its landing page.
- A named primary document (`implementation-plan.md`, `architecture-spec.md`,
  `final-summary.md`) is encouraged when the project has one canonical
  artifact; the `README.md` should link to it.
- Parent-folder grouping by domain (e.g. `projects/automation/<slug>/`,
  `projects/platform/<slug>/`) is permitted but not required for the first
  projects — let it emerge as the catalog grows.

## Naming

| Element | Convention |
|---------|------------|
| Project folder | lowercase kebab-case slug, no dates |
| Landing page | `README.md` |
| Supporting docs | descriptive kebab-case (`architecture.md`, `runbook.md`) |
| Completion report | `reports/completion-YYYY-MM-DD-<slug>.md` |

## Item Identity

Every project folder's `README.md` must include the lifecycle frontmatter
contract from
[03 § Core Metadata Contract](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md#core-metadata-contract).
Use [`../templates/item-template.md`](../templates/item-template.md) as the
starting point and set `item_type: project`, `canonical_stage: project`.

## Read First

Until the catalog has its first migrated entry, the in-flight project lives
under [`wellnex-docs/planning/workflow-platform/`](https://github.com/eny-tech/wellnex-docs/tree/main/planning/workflow-platform)
and will be pointer-linked here when it crosses its next stage gate.
