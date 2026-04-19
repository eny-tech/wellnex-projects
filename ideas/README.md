<!--
Repository: wellnex-projects
Path: ideas/README.md
Purpose: Low-friction capture point for early ideas, seeds, and candidate work before formal planning
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 1.0.0
-->

# Ideas

Earliest stage of the
[WellNex item lifecycle](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md).
Anything that is not yet ready to be an issue, a plan, or a project belongs
here.

## When to drop something here

- A user thought captured during chat
- An agent-authored improvement suggestion
- A rough refactor seed
- A backlog seed not yet ready to become a GitHub Issue

Ideas are first-class items even if they never graduate. They get an
`item_id` like everything else.

## Layout

| Path | Purpose |
|------|---------|
| [`inbox/`](inbox/) | Untriaged drops; move out of inbox once shaped |
| `./` | Triaged ideas with at least a title, summary, and provisional next stage |

## Naming

```
YYYY-MM-DD.idea-<kebab-slug>.md
```

Examples:

- `2026-04-18.idea-confluence-pattern-import.md`
- `2026-04-18.idea-llm-budget-dashboard.md`

## Frontmatter

Use [`templates/item-template.md`](../templates/item-template.md) and set
`item_type: idea`, `canonical_stage: idea`.

## Promotion

When an idea graduates to a later stage:

1. Append a new entry to its `location_stack` with `transition_reason`.
2. Move the file to the new stage's folder (`issues/` here,
   `wellnex-docs/planning/` for plans, or
   `projects/active/<slug>/` for execution spaces).
3. Leave a pointer stub at the old path per
   [03 § Pointer Model](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md#pointer-model).

The `item_id` never changes.
