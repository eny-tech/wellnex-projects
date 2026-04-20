<!--
Repository: wellnex-projects
Path: projects/active/README.md
Purpose: Status view for projects currently driving work
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-20
Version: 1.1.0
-->

# Active Projects

Projects that are currently driving work. Each entry should be a folder
containing at least a `README.md` landing page with the lifecycle
frontmatter contract.

| Project | Status | Owner |
|---------|--------|-------|
| [remediation-agent](remediation-agent/README.md) | Active — design (Phase 0) | TBD |
| [decision-rules-platform](decision-rules-platform/README.md) | Active — Phases 1 + 1.5 shipped | TBD |
| [cloud-governance](cloud-governance/README.md) | Active — Phase 0 (standards bootstrapped) | TBD |

## Adding a project

1. Create `projects/active/<slug>/README.md` from
   [`../../templates/item-template.md`](../../templates/item-template.md).
2. Set `item_type: project`, `canonical_stage: project`,
   `canonical_path: projects/active/<slug>/README.md`.
3. Append a `location_stack` entry for the new location.
4. Add a row to the table above.
