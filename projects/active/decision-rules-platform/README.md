---
item_id: wnif-01KPK0DRZ4QY8N2VDR0Z4M3W11
item_type: project
title: "Decision Rules Platform — registry, scopes, and evaluator sub-agent"
home_repo: wellnex-projects
canonical_stage: project
canonical_path: projects/active/decision-rules-platform/README.md
created_at: 2026-04-19T07:15:45Z
updated_at: 2026-04-19T07:15:45Z
origin:
  captured_by: user
  captured_from: chat
related:
  github_issues: []
  github_projects: []
  pointer_paths: []
  workflow_paths:
    - https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/decision-rules/README.md
location_stack:
  - sequence: 1
    repo: wellnex-projects
    stage: project
    path: projects/active/decision-rules-platform/README.md
    entered_at: 2026-04-19T07:15:45Z
    transition_reason: "fast-path: operator promoted decision-rules from a single json into a project to evolve the model (per-file rules, two-tier scope, evaluator sub-agent)"
---
<!--
Repository: wellnex-projects
Path: projects/active/decision-rules-platform/README.md
Purpose: Project landing page for evolving the WellNex decision-rules registry, scope model, and Rule Evaluator sub-agent
Author: GitHub Copilot (Claude Opus 4.7) with operator
Created: 2026-04-19
Last-Modified: 2026-04-19
Version: 0.1.0
-->

# Decision Rules Platform

> Evolve the WellNex decision-rules registry from a single JSON file
> into a flat directory of per-rule Markdown files with a two-tier
> scope model and a sub-agent for non-trivial evaluation.

| Field | Value |
|-------|-------|
| Item ID | `wnif-01KPK0DRZ4QY8N2VDR0Z4M3W11` |
| Status | Active — Phase 1 + 1.5 shipped |
| Stage | project (idea + issue + planning all skipped) |
| Captured | 2026-04-19 (session `7fbada70`) |
| Owner | TBD |
| Registry home | [wellnex-docs/planning/workflow-platform/decision-rules/](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/decision-rules/README.md) |
| Pattern | [patterns.md § Decision Rules Registry](https://github.com/eny-tech/wellnex-docs/blob/main/architecture/patterns.md#14-decision-rules-registry-wellnex-custom) |

## Lineage

The original `decision-rules.json` was seeded on 2026-04-18 with a
single rule (stash-rescue when content is hash-verified). That format
worked for one rule but was the wrong shape for many: monolithic
diffs, no operator-scannable index, no place to record evaluation
strategy. Operator (2026-04-19) promoted the registry to a project to
make the model itself improvable.

## Problem Being Solved

As the WellNex agent ecosystem grows, the number of "when may an
agent proceed without asking?" decisions grows with it. Without
structure, those decisions either:

1. Get re-litigated in chat every session, or
2. Get smeared into copilot-instructions where they are invisible to
   tooling and impossible to audit.

The decision-rules registry exists so that operator decisions are:

- Captured **once**, in a stable place.
- **Visually scannable** by humans at the directory level.
- **Machine-readable** for runtime gating.
- **Honest about complexity** — trivial rules evaluate inline; hard
  rules delegate to a named sub-agent.

## Goals

| Goal | Description |
|------|-------------|
| One file per rule | No monolith; diffs and PR review work naturally |
| Operator-scannable filenames | `<scope>.<decision>.<rule-id>.md` shows the most important metadata without opening the file |
| Two-tier scope only | `global` or `<repo-name>` — no third tier, no inheritance |
| Default to ask | Missing rule = ask; silence is never consent |
| Sub-agent escape hatch | Rules that need real reasoning are dispatched to the Rule Evaluator sub-agent instead of growing schema complexity |
| Rules are information items | Each rule carries a `wnif-` ID and the standard lifecycle frontmatter; progresses through Information Disperser phases (`known → trained → consumed → re-originated`) |
| Document the pattern | Captured in `wellnex-docs/architecture/patterns.md` |

## Non-Goals

- Becoming a policy DSL (no Rego / OPA equivalent).
- Inheritance, mixins, or rule composition.
- Subdirectories or rule categories.
- A third scope tier (no team / agent / iteration scopes).
- Replacing copilot-instructions. Rules are gates inside workflows;
  instructions describe the workflows themselves.

If those limits start hurting, the answer is to push more reasoning
into the Rule Evaluator sub-agent — not to grow the registry schema.

## Design Principles

1. **Filename is the index.** Scope and decision are visible without
   opening the file.
2. **Flat directory.** No subfolders. Duplication across repos is
   preferred over abstraction.
3. **Single source of truth.** One file per `rule_id`; no copies, no
   includes.
4. **Audit trail by retirement, not deletion.** Set `superseded_by`;
   never remove a rule file.
5. **Sub-agent owns hard cases.** When a rule cannot be evaluated by
   deterministic checks, it is marked `evaluation: complex` and
   dispatched to the Rule Evaluator sub-agent.
6. **Agent proposes, operator disposes.** When the agent notices a
   recurring or trivial decision that has no covering rule, it
   drafts a *proposal* into a triage folder and surfaces it. The
   agent never writes a live rule on its own — proposals are inert
   until an operator promotes them.

## Phasing

### Phase 1 — Per-file registry (shipped 2026-04-19)

- [x] Create `wellnex-docs/planning/workflow-platform/decision-rules/`.
- [x] Define filename convention `<scope>.<decision>.<rule-id>.md`.
- [x] Write [SCHEMA.md](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/decision-rules/SCHEMA.md).
- [x] Write [\_template.md](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/decision-rules/_template.md).
- [x] Migrate the seed rule out of `decision-rules.json` and remove
      the JSON.
- [x] Document the pattern in
      [patterns.md § Decision Rules Registry](https://github.com/eny-tech/wellnex-docs/blob/main/architecture/patterns.md#14-decision-rules-registry-wellnex-custom).
- [x] Update workflow-platform README to link the new folder.

### Phase 1.5 — Rules as Information Items (shipped 2026-04-19)

- [x] Adopt the standard lifecycle frontmatter (`item_id`, `item_type:
      decision-rule`, `canonical_stage`, `canonical_path`,
      `created_at`, `updated_at`, `related`).
- [x] Add Information Disperser fields (`origin_observation`,
      `dispersion[]`, `re_origination[]`).
- [x] Map disperser phases to rule maturity:
      `known → trained(L1) → trained(L2) → consumed → re-originated`.
- [x] Mint a `wnif-` ULID for the seed rule and backfill its
      `origin_observation` and `dispersion` history.
- [x] Update [SCHEMA.md](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/decision-rules/SCHEMA.md)
      and [patterns.md §14](https://github.com/eny-tech/wellnex-docs/blob/main/architecture/patterns.md#14-decision-rules-registry-wellnex-custom)
      with the disperser mapping.

### Phase 2 — Validation tooling

- [ ] Python validator in `wellnex-agents` that checks:
      - Filename slots match frontmatter.
      - `evidence_required` non-empty.
      - `evaluation` ∈ {`simple`, `complex`}.
      - `item_id` unique across registry, matches `wnif-<ULID>` shape.
      - `origin_observation` fully populated; `dispersion` sequence monotonic.
      - README index row exists for every file.
- [ ] GitHub Action wired to PRs touching `decision-rules/`.

### Phase 3 — Runtime evaluator (`simple`) and applications log

- [ ] CLI command `wellnex-agents decide --rule <rule-id>` that
      runs `evidence_required` checks and returns the decision.
- [ ] Adapter pattern for evidence checks (hash equality, branch
      pushed, file exists, env var set, exit code).
- [ ] Per-repo applications log under `.decision-rules/log/` recording
      every CONSUMED event (rule item_id, evidence outcomes, decision,
      session id, timestamp).
- [ ] When an operator overrides a `proceed` outcome, append the
      override to the parent rule's `dispersion` and open a draft
      successor rule whose `origin_observation` cites the override.

### Phase 4 — Rule Evaluator sub-agent (`complex`) — see [Sub-Agent Contract](#sub-agent-contract-rule-evaluator)

- [ ] Implement the sub-agent against the contract below.
- [ ] Wire `wellnex-agents decide` to delegate when
      `evaluation: complex` or when multiple `simple` rules collide.
- [ ] Log the sub-agent's justification alongside the decision.

### Phase 4.5 — Rule Proposer (agent-suggested rules)

The agent should *notice* when a question has become trivial (similar
to past asks, or already answered for an analogous situation) and
draft a proposal rather than ask again. Proposals are inert until an
operator promotes them.

- [ ] Detection signals (any one is enough to trigger a proposal):
  - **Recurrence.** The same `ask` (by trigger fingerprint) has
    been raised ≥ N times across recent sessions and the operator
    answered the same way each time.
  - **Override.** Operator overrode a `proceed` outcome — the
    Disperser `re_origination` path naturally seeds a successor
    proposal.
  - **Analogy.** The current situation closely matches an existing
    rule's `trigger` but differs on `scope` (e.g., a `wellnex-docs`
    rule that plausibly belongs to `wellnex-server` too).
  - **Sub-agent followup.** The Rule Evaluator sub-agent emitted a
    `followups` entry indicating "this would be `simple` if rule X
    existed."
- [ ] Triage folder: drafts land in
      `wellnex-docs/planning/workflow-platform/decision-rules/_proposals/`
      with filename `<scope>.<decision>.<rule-id>.proposed.md` and
      frontmatter `canonical_stage: known` (Phase 1 of the Disperser).
- [ ] Surface contract: when the agent drafts a proposal it tells the
      operator in one short message — proposed filename, the recurring
      situation, and a one-line "promote / decline / edit" prompt.
- [ ] Promotion path: operator review moves the file from
      `_proposals/` to the registry root and bumps `canonical_stage`
      from `known` to `trained`. The proposal's `dispersion[]` log
      records the promotion.
- [ ] Decline path: operator sets `superseded_by: declined` (or
      deletes the proposal). Either way, a session-memory note is
      written so the agent does not re-propose the same thing in the
      same session.
- [ ] Hard limits:
  - Agent **never** writes into the live registry — only into
    `_proposals/`.
  - Agent **never** raises more than one proposal per situation
    per session.
  - The proposer **must** cite the recurrence / override /
    analogy evidence in the proposal's `origin_observation.source_event`.

### Phase 5 — Adoption

- [ ] Migrate inline "operator decided X" notes from
      copilot-instructions into rule files where they belong.
- [ ] Document a triage flow: when an operator says "next time just
      do it," the agent proposes a draft rule file for review.

## Sub-Agent Contract — Rule Evaluator

The **Rule Evaluator sub-agent** is invoked when:

1. A matched rule has `evaluation: complex`, **or**
2. Two or more `simple` rules match the same situation with
   conflicting decisions, **or**
3. Multiple rules' triggers all plausibly match and the runtime
   cannot pick deterministically.

### Inputs

| Input | Source |
|-------|--------|
| `candidate_rules` | Full text + frontmatter of every rule whose `trigger` plausibly matches |
| `situation` | Structured snapshot: active repo, branch, dirty files, recent commits, session id |
| `operator_history` | Recent overrides logged by the runtime (signal that a rule may be stale) |
| `precedence_hints` | Free-form strings from each candidate rule |
| `evaluator_hints` | Structured hints (e.g. `look_at: ["git-log", "recent-stash"]`) |

### Outputs

| Output | Description |
|--------|-------------|
| `decision` | One of `proceed` \| `ask` \| `refuse` |
| `chosen_rule_id` | Which rule fired (or `null` if none applied and the default `ask` is being returned) |
| `justification` | One paragraph; cited evidence; logged with the action |
| `followups` | Optional list of `ask` items the operator should answer next time so the rule can be promoted to `simple` |

### Behavior Constraints

- The sub-agent **may not** invent rules. It only ranks and selects
  among candidates supplied by the registry.
- The sub-agent **must** prefer `ask` when justification confidence
  is low. There is no concept of "best guess proceed."
- The sub-agent **must** record any rule it would like to see exist
  but doesn't, into `followups`. Operators triage these into draft
  rule files.
- The sub-agent has **no write access** to the registry. New rules
  always require operator review.

### Invocation Surface

```bash
# Direct invocation
wellnex-agents decide \
  --situation-from git-status \
  --rule stash-shared-root-cleanup-when-content-verified

# With escalation enabled (multiple matches → sub-agent)
wellnex-agents decide --auto
```

The `--auto` form is the default in agent-driven workflows; the
single-rule form is for operator-driven triage and tests.

## Sub-Agent Contract — Rule Proposer

The **Rule Proposer sub-agent** runs opportunistically during normal
agent work. Its job is to *notice* when a decision has become trivial
and draft a proposal so operators don't keep re-answering the same
question. It is strictly inert: it only writes into the
`_proposals/` triage folder and never into the live registry.

### Triggers

The proposer is invoked when any of these signals fire:

| Signal | Description |
|--------|-------------|
| Recurrence | The same `ask` (matched by trigger fingerprint) was raised ≥ N times across recent sessions and the operator answered the same way each time |
| Override | Operator overrode a `proceed` outcome — the parent rule's `re_origination` link is the seed |
| Analogy | The current situation closely matches an existing rule's `trigger` but differs only on `scope` (e.g. a `wellnex-docs` rule that plausibly belongs to `wellnex-server`) |
| Followup | Rule Evaluator returned a `followups` entry of the form "this would be `simple` if rule X existed" |

### Inputs

| Input | Source |
|-------|--------|
| `situation` | Same structured snapshot the Evaluator sees |
| `signal` | Which trigger fired, with cited evidence (session ids, log entries, parent rule id) |
| `existing_rules` | Full registry, used to avoid duplicate proposals |
| `recent_proposals` | Contents of `_proposals/` to dedupe within and across sessions |
| `session_memory` | Notes about already-declined proposals in this session |

### Outputs

| Output | Description |
|--------|-------------|
| `proposal_path` | `decision-rules/_proposals/<scope>.<decision>.<rule-id>.proposed.md` |
| `proposal_file` | Full Markdown body, frontmatter populated, `canonical_stage: known`, `origin_observation.source_event` cites the signal evidence |
| `surface_message` | One short sentence the agent says to the operator: filename + recurring situation + promote/decline/edit prompt |

### Behavior Constraints

- The proposer **never** writes outside `_proposals/`.
- The proposer **never** raises more than one proposal per situation
  per session.
- The proposer **must** cite signal evidence in
  `origin_observation.source_event` (session ids, log lines, parent
  rule id) so reviewers can verify the recurrence claim.
- The proposer **must** check `existing_rules` and `recent_proposals`
  to avoid duplicates before drafting.
- If the operator declines a proposal in-session, the proposer
  records that in session memory and does not re-propose the same
  trigger fingerprint until the next session.
- Promotion is an operator action: move the file from `_proposals/`
  to the registry root, drop `.proposed` from the filename, append a
  `dispersion` entry recording the promotion, bump
  `canonical_stage` from `known` to `trained`.

### Invocation Surface

```bash
# Manual: have the agent scan recent ask-history and emit any proposals
wellnex-agents propose-rules --since 7d

# Automatic: enabled inside `decide --auto`; emits a proposal when a
# trigger signal fires alongside the decision
wellnex-agents decide --auto --propose
```

The `--propose` flag is on by default in agent-driven workflows.

## Acceptance Criteria

The project is complete when:

1. Every operator decision currently encoded in copilot-instructions
   that fits the rule shape has been migrated to a rule file.
2. CI validates filename / frontmatter consistency on every PR.
3. The runtime evaluator and Rule Evaluator sub-agent both ship and
   are wired into at least one production agent workflow (e.g. the
   git-cycle stash-rescue path).
4. The Rule Proposer ships and at least one rule in the registry was
   originally drafted by it (visible via `origin_observation.observer`
   and the `_proposals/ → registry` dispersion entry).
5. Operators can read the directory listing of `decision-rules/` and
   understand the active policy without opening any file.

## Open Questions

1. **Where does the Rule Evaluator sub-agent live** — inside
   `wellnex-agents` (Python, sibling to existing dev agents) or as
   an MCP server? Tracks the same question as the Remediation Agent;
   resolving one likely resolves the other.
2. **Rule deprecation cadence.** When `superseded_by` chains get
   long, do we ever archive? Default answer: no, audit value beats
   directory size concerns.

## Related

- Parent platform: [Workflow Platform](https://github.com/eny-tech/wellnex-docs/blob/main/planning/workflow-platform/README.md)
- Sister project: [Remediation Agent](../remediation-agent/README.md) (shares the language/runtime question)
- Pattern: [Decision Rules Registry](https://github.com/eny-tech/wellnex-docs/blob/main/architecture/patterns.md#14-decision-rules-registry-wellnex-custom)
