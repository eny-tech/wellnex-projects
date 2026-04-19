---
item_id: wnif-01KPJ7F79RT7PWPC5HVAXQV9VJ
item_type: project
title: "Remediation Agent — tone-aware stewardship and consistency sub-agent"
home_repo: wellnex-projects
canonical_stage: project
canonical_path: projects/active/remediation-agent/README.md
created_at: 2026-04-19T06:39:55Z
updated_at: 2026-04-19T06:39:55Z
origin:
  captured_by: user
  captured_from: chat
related:
  github_issues: []
  github_projects: []
  pointer_paths:
    - ideas/2026-04-18.idea-remediation-agent.md
    - issues/2026-04-18.issue-remediation-agent.md
  workflow_paths: []
location_stack:
  - sequence: 1
    repo: wellnex-projects
    stage: idea
    path: ideas/2026-04-18.idea-remediation-agent.md
    entered_at: 2026-04-19T06:39:55Z
    transition_reason: "captured during stewardship-failure post-mortem (session 81ef6acb)"
  - sequence: 2
    repo: wellnex-projects
    stage: project
    path: projects/active/remediation-agent/README.md
    entered_at: 2026-04-19T06:39:55Z
    transition_reason: "fast-path: operator approval, scope clear, idea + issue stages explicitly skipped"
---
<!--
Repository: wellnex-projects
Path: projects/active/remediation-agent/README.md
Purpose: First wellnex-projects project — design and deliver the Remediation Agent (tone-aware stewardship + consistency sub-agent)
Author: GitHub Copilot (Claude Opus 4.7)
Created: 2026-04-18
Last-Modified: 2026-04-18
Version: 0.1.0
-->

# Remediation Agent

> A tone-aware stewardship and consistency sub-agent for the WellNex
> ecosystem. Audits handoffs and drift across sessions, repos, and docs;
> surfaces orphaned work, plan-vs-implementation mismatches, and
> vocabulary inconsistencies; corrects gently when safe and escalates
> with options when not.

| Field | Value |
|-------|-------|
| Item ID | `wnif-01KPJ7F79RT7PWPC5HVAXQV9VJ` |
| Status | Active — design |
| Stage | project (idea + issue + planning all skipped) |
| Captured | 2026-04-18 (session `81ef6acb`) |
| Owner | TBD |
| Lifecycle stubs | [idea](../../../ideas/2026-04-18.idea-remediation-agent.md) · [issue](../../../issues/2026-04-18.issue-remediation-agent.md) |

## Lineage

This is the **first project** in the `wellnex-projects` repository.
Captured during the 2026-04-18 stewardship post-mortem
([session 81ef6acb report](https://github.com/eny-tech/wellnex-docs/blob/main/reports/2026-04-18-session-81ef6acb-agents-infra-analysis-and-stewardship.md))
where an LLM session walked away from orphan files in a shared workspace
root with a "not mine" disclaimer. That failure made the case for an
agent whose explicit job is to notice and remediate exactly that class
of issue.

## Problem Being Solved

Across multi-session, multi-repo work in the WellNex ecosystem, three
classes of drift accumulate silently:

1. **Stewardship drift** — orphan files in shared roots, uncommitted
   work without a session tag, broken pointer stubs, missing `item_id`
   frontmatter, dangling stashes.
2. **Plan-vs-implementation drift** — a planning doc says one thing, the
   shipped code says another. Vocabulary forks. Schemas diverge.
3. **Cross-repo consistency drift** — naming conventions, header
   formats, copilot-instructions sections, security policies that
   should agree across `wellnex`, `wellnex-server`, `wellnex-agents`,
   `wellnex-health-agents`, `wellnex-docs`, and `wellnex-projects`
   but quietly drift apart.

No existing agent owns this class of work. `SecurityReviewerAgent`,
`CodeReviewerAgent`, `ArchitectAgent`, `UXAuditorAgent`, and the
runtime middleware pipeline each handle a slice — none look across
sessions and repos for *consistency and follow-through*.

## Goals

| Goal | Description |
|------|-------------|
| Detect orphans | Identify uncommitted work in shared roots that is not owned by an active session worktree |
| Detect drift | Compare planning docs to actual implementation; flag terminology forks |
| Surface gently | Default tone is collaborative, not policing; corrections are proposals, not commands |
| Rescue, never discard | Stash with a labeled tag before suggesting deletion; recovery must be one command |
| Be sub-agent friendly | Designed to be invoked by other agents (or Copilot directly) as a final stewardship pass |

## Non-Goals

- Replacing existing agents. The Remediation Agent runs **after**
  domain-specific reviews, not instead of them.
- Auto-applying corrections without operator review for anything
  destructive (deletions, force-pushes, branch resets).
- Becoming a linter. It looks for *consistency and stewardship*, not
  style.

## Design Principles

Drawn from
[2026-04-18 agents-infra adaptation analysis § 5](https://github.com/eny-tech/wellnex-docs/blob/main/analysis/2026-04-18-agents-infra-adaptation-for-wellnex.md#5-ai-patterns-worth-documenting-for-wellnex):

1. **Lifecycle middleware pipeline** — `onPrepare → onExecute → onResolve`
   for any check it performs, so it can be composed with other agents.
2. **Ports & Adapters** — pluggable inspectors (orphan scanner, doc-vs-code
   diff, vocabulary scanner) and pluggable surfacers (chat reply,
   markdown report, GitHub issue, run-controller heartbeat).
3. **Repo-owned stack** — declares its own `.remediation-agent/`
   footprint per repo (config, run ledger, recent findings cache).
4. **Autonomous file-inbox work** — operator can drop a stewardship
   request into `.remediation-agent/issues/inbox/` and run a time-boxed
   sweep.
5. **Three-tier config** — Platform defaults / per-repo overlay /
   per-operator tone & verbosity preferences.
6. **Confirmation gates** — every destructive action requires a token.

## Tone Contract

Per operator preference (recorded in user memory under "Correction Loop"):

- **Direct beats diplomatic.** Name the issue, propose the fix, let the
  operator decide.
- **Surface inconsistencies without softening.** Drift between plan and
  implementation is the *point* of the agent, not a thing to hedge.
- **Treat operator corrections as signal.** When an operator overrides
  a suggestion, log the override; if it recurs, the rule should change.
- **No "not my problem."** If the agent finds a thing, it owns it
  through to either a fix, a labeled stash, an open issue, or an
  explicit "operator deferred."

## Plan / Phasing

The planning stage was absorbed into this section. If phasing grows
beyond what fits here, promote to a real plan in
`wellnex-docs/planning/remediation-agent/`.

### Phase 0 — Scope & contracts (this doc)

- [x] Capture problem, goals, non-goals, principles, tone contract.
- [ ] Operator review of scope.
- [ ] Decide: TypeScript MCP agent (sibling to agents-infra style) **or**
  Python agent in `wellnex-agents` (sibling to existing dev agents).
  See [open question 1](#open-questions).

### Phase 1 — Stewardship inspector (MVP)

- [ ] `OrphanScanner` adapter — finds dirty files in shared workspace
  roots not owned by any active worktree.
- [ ] `StashTagger` adapter — wraps `git stash push -m "rescue:<id> ..."`.
- [ ] CLI surface: `remediation scan --workspace <root>` returning
  markdown report.
- [ ] Confirmation-gate token before any rescue is materialized.

### Phase 2 — Drift inspector

- [ ] `PlanCodeDriftScanner` — diffs planning doc anchors against the
  actual files they reference (paths, function names, schema fields).
- [ ] `VocabularyScanner` — flags two repos using diverging names for
  the same concept (driven by a small synonyms registry).

### Phase 3 — Cross-repo consistency inspector

- [ ] `CopilotInstructionsAuditor` — checks that mandatory sections
  (worktree, security, file headers, related-repos) appear in every
  repo's `.github/copilot-instructions.md`.
- [ ] `HeaderFormatAuditor` — verifies file-header presence and field
  completeness per repo's contract.

### Phase 4 — Autonomous run controller

- [ ] Borrow the
  [confluence-agent run-controller pattern](https://github.com/eny-tech/wellnex-docs/blob/main/analysis/2026-04-18-agents-infra-adaptation-for-wellnex.md#tier-a--direct-lift-high-value).
- [ ] Inbox in `.remediation-agent/issues/inbox/`, heartbeats every N
  seconds, final markdown report, ISO-timestamped.

### Phase 5 — Sub-agent invocation

- [ ] MCP / sub-agent invocation contract so any other agent or Copilot
  session can call `@remediation final-pass` before completing a task.

## Open Questions

1. **Language and runtime.** TS/MCP (consistent with the agents-infra
   patterns we just analyzed) or Python (consistent with `wellnex-agents`
   today)? Trade-off: MCP gives Copilot direct invocation; Python reuses
   existing CLI/registry/event-bus. Recommend MCP, defer to operator.
2. **Where does it live?** A new `wellnex-remediation-agent` repo, or
   inside `wellnex-agents` as a new tier? New repo if MCP, inside
   `wellnex-agents` if Python.
3. **Scope of "tone."** Is tone configurable per operator (preferences
   file), or fixed by the project? Recommend per-operator override of a
   project default.
4. **Authority on conflicts.** When the agent flags drift between two
   repos, who wins by default? Recommend: agent never picks; surfaces
   both with a "decide" prompt unless one is explicitly marked
   canonical in `.remediation-agent/canon.yaml`.

## Risks

- **Becomes the policing agent.** If the tone contract is not enforced,
  the agent will erode trust. Counter: ship the tone contract as a
  test, not a guideline.
- **False positives on drift.** Plan docs are *intentionally* lossy
  compared to code. Counter: every drift finding includes the planning
  source line and the actual code line side-by-side; operator decides.
- **Performance on large monorepos.** Cross-repo scans are I/O-heavy.
  Counter: cache last scan in `.remediation-agent/cache/`, only re-scan
  changed paths.

## Success Criteria

- A future session that finds orphan work in a shared root has
  exactly one obvious tool to invoke (`@remediation rescue` or
  `wellnex-agents remediation scan`).
- Operator-flagged drift between today's plan and tomorrow's code
  becomes a generated finding, not a manual catch.
- The 2026-04-18 stewardship failure pattern cannot recur silently —
  the agent surfaces it.

## References

| Reference | Location |
|-----------|----------|
| Stewardship failure post-mortem (origin) | [reports/2026-04-18-session-81ef6acb-agents-infra-analysis-and-stewardship.md](https://github.com/eny-tech/wellnex-docs/blob/main/reports/2026-04-18-session-81ef6acb-agents-infra-analysis-and-stewardship.md) |
| Agents-infra adaptation analysis | [analysis/2026-04-18-agents-infra-adaptation-for-wellnex.md](https://github.com/eny-tech/wellnex-docs/blob/main/analysis/2026-04-18-agents-infra-adaptation-for-wellnex.md) |
| Item lifecycle model | [planning/workflow-platform/03-item-lifecycle-orchestration.md](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/03-item-lifecycle-orchestration.md) |
| Confluence agent (pattern source) | `/workspaces/agents-infra/packages/confluence-agent/` |
| MCP shared primitives (pattern source) | `/workspaces/agents-infra/packages/mcp-common/` |

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-04-18 | 0.1.0 | Initial project doc; idea + issue + planning stages skipped per operator |
