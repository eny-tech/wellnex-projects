<!--
Repository: wellnex-projects
Path: templates/item-template.md
Purpose: Reusable starting template for any item-flow artifact (idea, issue, plan, project)
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 1.0.0
-->

---
item_id: wnif-REPLACE_WITH_ULID
item_type: idea          # idea | issue | plan | project | workflow | pointer | completion
title: "REPLACE — short human-readable title"
home_repo: wellnex-projects
canonical_stage: idea    # idea | issue | planning | project | workflow | resolved | archived
canonical_path: ideas/YYYY-MM-DD.idea-replace-slug.md
created_at: YYYY-MM-DDTHH:MM:SSZ
updated_at: YYYY-MM-DDTHH:MM:SSZ
origin:
  captured_by: user        # user | agent | automation
  captured_from: chat      # chat | cli | review | scan | other
related:
  github_issues: []
  github_projects: []
  pointer_paths: []
  workflow_paths: []
location_stack:
  - sequence: 1
    repo: wellnex-projects
    stage: idea
    path: ideas/YYYY-MM-DD.idea-replace-slug.md
    entered_at: YYYY-MM-DDTHH:MM:SSZ
    transition_reason: current
---
<!--
Repository: wellnex-projects
Path: <fill-in-on-creation>
Purpose: <one-line purpose>
Author: <name or "GitHub Copilot (model)">
Created: YYYY-MM-DD
Last-Modified: YYYY-MM-DD
Version: 0.1.0
-->

# REPLACE — Title

## Summary

One short paragraph explaining what this item is and why it exists.

## Context

What prompted this? Link to the chat, scan finding, related issue, or
upstream artifact that triggered capture.

## Proposed Next Stage

- [ ] Stay as `idea`
- [ ] Promote to `issue`
- [ ] Promote to `planning`
- [ ] Promote to `project`

## Notes

Free-form thinking. Refine over time. When the item is promoted, update
`canonical_stage`, `canonical_path`, and append a new `location_stack`
entry; leave a pointer stub at the previous path per
[03 § Pointer Model](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md#pointer-model).

## Related

- Parent project (if any):
- Linked issues:
- Linked plans:
