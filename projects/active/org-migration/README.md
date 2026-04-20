---
item_id: wnif-01KPN2R5MXYZEQWV8NHJTQGKPD
item_type: project
title: "Org migration \u2014 eny-tech \u2192 wellnexsolutionsai"
home_repo: wellnex-projects
canonical_stage: project
canonical_path: projects/active/org-migration/README.md
created_at: 2026-04-19T20:00:00Z
updated_at: 2026-04-19T20:00:00Z
origin:
  captured_by: user
  captured_from: chat
related:
  github_issues: []
  github_projects: []
  pointer_paths: []
  workflow_paths:
    - https://github.com/eny-tech/wellnex-projects/blob/main/projects/active/cloud-governance/decisions/2026-04-19-iac-home.md
location_stack:
  - sequence: 1
    repo: wellnex-projects
    stage: project
    path: projects/active/org-migration/README.md
    entered_at: 2026-04-19T20:00:00Z
    transition_reason: "fast-path: operator created wellnexsolutionsai GitHub org and requested migration plan + repo architecture"
---
<!--
Repository: wellnex-projects
Path: projects/active/org-migration/README.md
Purpose: Plan for migrating WellNex from eny-tech org into the new wellnexsolutionsai org, with a repo architecture that scales with the business
Author: GitHub Copilot (Claude Opus 4.7) with operator
Created: 2026-04-19
Last-Modified: 2026-04-19
Version: 0.1.0
-->

# Org Migration — `eny-tech` → `wellnexsolutionsai`

> The operator created
> [github.com/wellnexsolutionsai](https://github.com/wellnexsolutionsai)
> as the permanent home for Wellnex Solutions AI. This project plans
> the migration, a modern non-monorepo repo architecture, and the
> org-level scaffolding needed for the business to grow inside it
> without the inevitable "we should have structured this differently"
> rework.

| Field | Value |
|-------|-------|
| Item ID | `wnif-01KPN2R5MXYZEQWV8NHJTQGKPD` |
| Status | Active — Phase 0 (access verified, plan drafted) |
| Stage | project (idea + issue + planning skipped; the operator already decided) |
| Captured | 2026-04-19 (session `cloudgov`) |
| Owner | Operator + Copilot |
| Related | [cloud-governance](../cloud-governance/README.md), [wellnex-infra decision](../cloud-governance/decisions/2026-04-19-iac-home.md) |

---

## Table of Contents

1. [Access verification](#1-access-verification)
2. [Anti-goals](#2-anti-goals)
3. [Architecture: repo families, not monorepo](#3-architecture-repo-families-not-monorepo)
4. [Naming convention for the new org](#4-naming-convention-for-the-new-org)
5. [Full repo inventory (target org)](#5-full-repo-inventory-target-org)
6. [Current → new mapping](#6-current--new-mapping)
7. [Migration order and mechanics](#7-migration-order-and-mechanics)
8. [Org-level config-as-code](#8-org-level-config-as-code)
9. [Wait-states](#9-wait-states)
10. [Acceptance](#10-acceptance)

---

## 1. Access verification

Performed in session `cloudgov` on 2026-04-19.

PAT source: `/home/vscode/workspaces/wellnex/.env` →
`WELLNEX_FULL_PAT` (alias of `GITHUB_FULL_ACCESS_PAT`). Fine-grained
PAT, expires **2026-07-19**.

| API surface | Status | Notes |
|---|---|---|
| `GET /user` | ✅ 200 | Identifies as `gedefili`. |
| `GET /orgs/wellnexsolutionsai` | ✅ 200 | Org `id=277611920`. |
| `GET /orgs/.../repos` | ✅ 200 | Two existing placeholder repos: `demo-repository`, `discussions`. |
| `POST /orgs/.../repos` (create) | ✅ 201 | Test created `_tools-access-probe`. |
| `DELETE /repos/.../_tools-access-probe` | ✅ 204 | Probe removed. |
| `GET /orgs/.../teams` | ❌ 403 | Token lacks `Organization members` scope. |
| `POST /orgs/.../teams` | ❌ 403 | Same. |
| `GET /orgs/.../actions/secrets` | ❌ 403 | Token lacks org `Secrets` scope. |
| `GET /orgs/.../actions/variables` | ❌ 403 | Same. |
| `GET /orgs/.../hooks` | ❌ 403 | Token lacks `Organization administration`. |
| `GET /repos/.../branches/main/protection` | ❌ 403 | Fine-grained PAT needs the specific repo added to its allow-list. |

**Takeaway**: the PAT is sufficient to migrate all content
(§7.1–§7.6). It is **not** sufficient to set up teams, org secrets,
org webhooks, or branch protection via API. Those happen either
through the UI on the day of migration or through a PAT scope bump
(see §9 wait-states).

---

## 2. Anti-goals

- **No** monorepo. The operator has been clear: a single giant repo
  collapses review, release, and ownership into one queue. We won't
  reproduce the pain.
- **No** per-deployable sprawl where every microservice gets its own
  repo the day it's conceived. That ends in 60-repo landscapes where
  half are dormant and cross-cutting changes need eight PRs.
- **No** "temporary" repos that accumulate. Every repo has a family
  (§3) and a stewardship owner or it doesn't exist.
- **No** `wellnex-` prefix on repo names in the new org. The *org*
  is `wellnexsolutionsai`; repeating it in every repo name is
  visual noise.
- **No** history loss during migration. Every repo moves with full
  commit graph, tags, branches, and refs.
- **No** flag day where everything cuts over at once. Migrate in
  dependency-safe waves (§7.6), each one independently revertible.
- **No** dual-home period longer than 48 hours for any single repo.
  Dual-home causes divergent PRs, which is worse than a rushed cut.
- **No** secrets migrated via git. Every `.env`, every key, every
  token lives in Key Vault or repo secrets only.

---

## 3. Architecture: repo families, not monorepo

The compromise between "one repo for everything" and "one repo per
service" is **repo families**: small number of stable *prefixes*,
each a cohesive axis of change, that grow by *adding* repos inside
a family rather than by splitting existing ones prematurely.

This buys four things:

1. **Org page as a product map.** Alphabetical sort on the org
   repos page groups related work (`product-api`, `product-web`,
   `product-health-agents` all cluster). No GitHub Projects gymnastics.
2. **Ownership boundary.** Each family maps to a stewardship role
   (see §8 CODEOWNERS), so reviewers are obvious from the prefix.
3. **CI scope discipline.** A `product-*` change cannot accidentally
   import from a `research-*` repo because they're separate Git
   modules with explicit versioned dependencies.
4. **Growth lane.** When `product-api` outgrows itself, the carve-out
   lands *inside the same family* (`product-api-billing`,
   `product-api-auth`) — no family-boundary crossing, no rename.

### The families

| Prefix | Axis of change | Release cadence | Stewardship |
|---|---|---|---|
| `product-*` | The Wellnex health product users touch | Continuous | Product engineering |
| `platform-*` | Developer + delivery infrastructure | Cautious, behind PR gates | Platform team (Cloud Governance sister project) |
| `governance-*` | Standards, project lifecycle, documentation | By PR as decisions land | Governance project |
| `data-*` | Datasets, training artifacts, analytics code | Snapshot-based | Data stewardship |
| `research-*` | Spikes, prototypes, expected-to-sometimes-fail | Ad-hoc; may be archived | Whoever started it; 90-day review cadence |
| `vendor-*` | Forks / overlays of third-party code | Mirrors upstream + overlay | Platform team |
| `_*` (leading underscore) | Org-level meta (templates, bot config, policies) | By PR | Org admin + governance |

Leading-underscore repos sort to the top of the org page, which is
what you want for meta: a newcomer clicks the org and sees `_org`
and `_templates` first.

### Why not a suffix (e.g. `api-product`)?

GitHub's repo search is *prefix-weighted*. Prefix grouping is also
how humans read lists (they scan the first word). Suffix families
work less well in practice.

---

## 4. Naming convention for the new org

```
<family>-<primary>[-<secondary>][-<variant>]
```

Rules:

- `family` ∈ {`product`, `platform`, `governance`, `data`,
  `research`, `vendor`} or a leading `_` meta prefix.
- `primary` is the single word that uniquely identifies the repo
  within the family. Keep it ≤10 chars where practical.
- `secondary` and `variant` are optional. Use them when one
  `family-primary` inevitably carves up over time
  (`product-api-auth`, `product-api-worker`) — *not* preemptively.
- All lowercase, hyphen-separated, ASCII alphanumeric only.
- No `wellnex-` prefix (the org already carries that word).
- No environment in the name (`-dev`, `-prod`). Environments are
  runtime concerns, governed by branches / tags / IaC.

This pairs naturally with the
[Cloud Resource Naming standard v1.2.1](https://github.com/eny-tech/wellnex-docs/blob/main/governance/cloud-resource-naming.md):
the repo's `family` roughly maps to the cloud-resource `system`
code (`product`↔`web`/`api`/`agents`, `platform`↔`iac`/`core`,
`data`↔`data`, `governance`/`research`↔no cloud presence).

---

## 5. Full repo inventory (target org)

Proposed *initial* repo set on day one of the migrated org. "Initial"
means what we create up-front; families are explicitly designed to
grow.

### `_org` (meta)

| Repo | Purpose |
|---|---|
| `_org` | Org-level config-as-code: CODEOWNERS templates, default labels, default branch protection rules, issue/PR templates, security policy (`SECURITY.md`), community health files, Renovate org config. Consumed by other repos via `.github` template inheritance. |
| `_templates` | GitHub *repository templates* (`Use this template` button). Starter templates for: Node service, Python CLI, Terraform module, static site, MCP server, research spike. New repos are born with headers, CI, license, CONTRIBUTING, and the right `.github/` already in place. |

### `product-*`

| Repo | Source | Purpose |
|---|---|---|
| `product-web` | `eny-tech/wellnex` | React frontend. |
| `product-api` | `eny-tech/wellnex-server` | Fastify API backend. |
| `product-health-agents` | `eny-tech/wellnex-health-agents` | Runtime in-app AI pipeline. |

### `platform-*`

| Repo | Source | Purpose |
|---|---|---|
| `platform-dev-agents` | `eny-tech/wellnex-agents` | Python dev-agent CLI toolchain. |
| `platform-infra` | *(new — see [wellnex-infra decision](../cloud-governance/decisions/2026-04-19-iac-home.md))* | Azure Terraform, bootstrap, env compositions, workflows. |
| `platform-cloud-naming` | *(new — see [cloud-naming migration](../cloud-governance/plans/2026-04-19-cloud-naming-migration.md))* | Vendor-neutral OSS-ready naming Function App. |

### `governance-*`

| Repo | Source | Purpose |
|---|---|---|
| `governance-docs` | `eny-tech/wellnex-docs` | All normative standards, architecture notes, daily reports, reference material. |
| `governance-projects` | `eny-tech/wellnex-projects` | Project lifecycle (active / parked / resolved / recent), decision logs, plans. |

### `data-*`

Empty at creation. Reserved for when the DatasetPreparationAgent's
outputs outgrow their current tucked-in spot inside the agents repo.
Expected first entry: `data-training-snapshots`.

### `research-*`

Empty at creation. Created on demand via the `_templates` research
spike template. Expected early entries: `research-oura-signal-quality`,
`research-agent-benchmark`. A quarterly review sweeps dormant
research repos (§8).

### `vendor-*`

Empty at creation. Reserved for forks/overlays we must maintain
(e.g. a forked SDK pending upstream PR). Keep the upstream name
inside: `vendor-<upstream>-overlay`.

### Initial repo count

- Meta: 2
- Product: 3
- Platform: 3 (two of which are net-new: `platform-infra`, `platform-cloud-naming`)
- Governance: 2
- Data / Research / Vendor: 0

**Total at day-one of new org: 10 repos**, of which 5 are straight
migrations, 2 are new (Cloud Governance phase 1), and 2 are new
meta (`_org`, `_templates`).

---

## 6. Current → new mapping

| Old (`eny-tech/`) | New (`wellnexsolutionsai/`) | History | Cross-links to update |
|---|---|---|---|
| `wellnex` | `product-web` | full mirror-push | package.json, Copilot instructions, README links |
| `wellnex-server` | `product-api` | full mirror-push | package.json, Copilot instructions, README |
| `wellnex-health-agents` | `product-health-agents` | full mirror-push | package.json, README |
| `wellnex-agents` | `platform-dev-agents` | full mirror-push | pyproject.toml, setup.sh, cross-repo `pip install` references |
| `wellnex-docs` | `governance-docs` | full mirror-push | Every `https://github.com/eny-tech/wellnex-docs/...` link across all repos |
| `wellnex-projects` | `governance-projects` | full mirror-push | Every `https://github.com/eny-tech/wellnex-projects/...` link |

Two Copilot-instruction files (`.github/copilot-instructions.md` in
each repo) contain "Related Repositories" tables that must be
rewritten to point at the new names — script this with a single
`sed` pass.

---

## 7. Migration order and mechanics

Migrate in **waves**. Each wave is a commit-ready unit that can be
reverted independently. Later waves depend on earlier ones for
cross-links, but the *code* of each repo is independently safe.

### 7.1 Pre-flight (one-shot, before wave 1)

- Confirm access test results from §1 are still green
  (PAT works, expires 2026-07-19).
- Operator bumps PAT permissions (§9 wait-state #2) if they want
  §8 automation instead of UI clicks.
- Create the `_org` repo (UI or API); populate it with:
  - `CODEOWNERS-default` (family-to-team mapping placeholder — teams
    are set up later but the file is ready).
  - `branch-protection.json` — reusable `main` protection config.
  - `SECURITY.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md` (OSS-ready).
  - `labels.yml` — org label set.
  - `renovate.json5` — base Renovate config.
- Create `_templates` repo and add **repository templates** (one
  branch per template, or one sub-folder per template consumed via
  `gh repo create --template`).

### 7.2 Wave 1 — `governance-*` (the roots)

These move first because every other repo's Copilot instructions
and README link to them.

```bash
for src in wellnex-docs wellnex-projects; do
  case $src in
    wellnex-docs)      dst=governance-docs ;;
    wellnex-projects)  dst=governance-projects ;;
  esac

  # Mirror-push preserves all refs.
  git clone --mirror git@github.com:eny-tech/$src.git /tmp/$src.git
  # Create empty target repo via API.
  curl -sS -X POST -H "Authorization: Bearer $WELLNEX_FULL_PAT" \
    -H "Accept: application/vnd.github+json" \
    https://api.github.com/orgs/wellnexsolutionsai/repos \
    -d "{\"name\":\"$dst\",\"private\":true,\"has_issues\":true,\"has_projects\":false,\"has_wiki\":false,\"auto_init\":false}"
  # Mirror push.
  cd /tmp/$src.git
  git remote set-url --push origin git@github.com:wellnexsolutionsai/$dst.git
  git push --mirror
  cd -
done
```

After the mirror push, **archive** the `eny-tech` source with a
README that points at the new location. Do not delete — the
`eny-tech` GitHub URLs are referenced from session logs,
daily reports, and historical decision records that should keep
working.

### 7.3 Wave 2 — `platform-dev-agents`

Platform tooling consumed by `product-*` and governance alike. Move
it before products so product repo installs can flip their
`pip install` URL in one PR.

Same mirror-push recipe. After archive, update:

- `pyproject.toml` metadata (project URL, homepage).
- `setup.sh` references to any sibling-checkout assumptions.
- CLI docstrings that mention the eny-tech URL.

### 7.4 Wave 3 — `product-*`

Three repos (`web`, `api`, `health-agents`). Can be mirrored in
parallel. Post-push, batch-update:

- Each repo's `.github/copilot-instructions.md` "Related
  Repositories" table (rename all six references).
- `package.json` `repository.url`.
- Cross-references in documentation (badge URLs, contribution
  links).

### 7.5 Wave 4 — `platform-infra` and `platform-cloud-naming`

These are **born** in the new org (never existed in `eny-tech`).
They land per the existing
[wellnex-infra decision](../cloud-governance/decisions/2026-04-19-iac-home.md)
and
[cloud-naming migration plan](../cloud-governance/plans/2026-04-19-cloud-naming-migration.md),
just with the new names:

- `wellnex-infra` → `platform-infra`
- `cloud-naming` → `platform-cloud-naming`

Any reference to these names in prior plan docs should be
find-replaced in the same PR that records the name change.

### 7.6 Wave 5 — link cleanup + eny-tech archive

One PR per repo. The PR:

- Runs the `sed` find-replace over all files:
  ```bash
  git ls-files | xargs sed -i \
    -e 's|eny-tech/wellnex-docs|wellnexsolutionsai/governance-docs|g' \
    -e 's|eny-tech/wellnex-projects|wellnexsolutionsai/governance-projects|g' \
    -e 's|eny-tech/wellnex-server|wellnexsolutionsai/product-api|g' \
    -e 's|eny-tech/wellnex-health-agents|wellnexsolutionsai/product-health-agents|g' \
    -e 's|eny-tech/wellnex-agents|wellnexsolutionsai/platform-dev-agents|g' \
    -e 's|eny-tech/wellnex\b|wellnexsolutionsai/product-web|g'
  ```
- Rewrites the Related Repositories table.
- Renames the session-worktree helper paths (currently
  `/home/vscode/workspaces/wellnex-agents/scripts/git/…`) — leave
  that to a follow-up once the devcontainer mount-points are
  redone.

Finally: archive each `eny-tech/*` repo with a single-commit
`README.md` that points at the new location and says "this repo
has moved; see the new home". Do not delete.

### 7.7 Devcontainer / shared root cleanup (follow-up, not blocking)

The five (six) shared-root workspaces (`/home/vscode/workspaces/wellnex*`)
point at `eny-tech` remotes. Plan a one-evening follow-up to:

- Add new remotes (`origin-new`) to each shared root.
- Verify fetch / pull on the new remote.
- Cut over `origin` → new URL.
- Update `.github/copilot-instructions.md` session-worktree blocks
  in each repo.
- Rename shared workspace directories once everyone's editors are
  closed on them (this is disruptive — coordinate).

---

## 8. Org-level config-as-code

Delivered via the `_org` repo. Not all of these can be set via the
current PAT (see §9); anything flagged 🧑 is UI-only until the PAT
gains the relevant scope.

| Item | How | Blocked? |
|---|---|---|
| **Default branch protection on `main`** for every `product-*` / `platform-*` / `governance-*` repo — require PR, 1 reviewer, no force-push, status checks | `_org/branch-protection.json` applied via `gh api` in a repo bootstrap workflow | 🧑 until PAT gets `Administration` on target repos |
| **CODEOWNERS** per family | Each repo's `.github/CODEOWNERS` generated from `_org/CODEOWNERS-default` at repo create | ✅ (plain file) |
| **Issue / PR templates** | `_org/.github/ISSUE_TEMPLATE/*` + inherited by org default | ✅ |
| **Labels** | `_org/labels.yml` synced via `github-label-sync` action | ✅ |
| **Dependabot** (alerts + security updates) | Org setting | 🧑 |
| **CodeQL default setup** | Org setting, on per repo | 🧑 |
| **Actions permissions** (restrict to selected) | Org setting | 🧑 |
| **Renovate** (org-wide config) | `_org/renovate.json5` + Renovate app installed on org | 🧑 (app install) |
| **Org secrets** (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`) | `gh api orgs/.../actions/secrets/...` | 🧑 (PAT lacks scope) |
| **Teams** (`@platform`, `@product`, `@governance`, `@data`, `@research`) | `gh api orgs/.../teams` | 🧑 (PAT lacks scope) |
| **Team → repo permissions** (prefix-based) | API once teams exist | 🧑 |
| **SSO / SAML** (if / when Entra ID is in place) | Org admin UI | 🧑 |
| **Org profile** (README, pinned repos, profile pic) | `.github` profile repo inside `_org` | ✅ |

### Org-profile README (nice-to-have, 15-minute task)

A public-facing README that shows up on
`github.com/wellnexsolutionsai`:

- Short product pitch.
- Table of public repos (once any are open-sourced, e.g.
  `platform-cloud-naming`).
- Contact / hiring link.

Lives in `_org/profile/README.md` (GitHub looks for a repo literally
named `.github` with a `profile/README.md`; we'll use a folder in
`_org` and symlink, OR just create an additional `.github` repo).

---

## 9. Wait-states

| # | Blocker | Owner | Unblocks |
|---|---|---|---|
| 1 | Operator confirms final family names (see §3) and whether to drop `_org` / `_templates` or rename them | operator | §7.1 pre-flight |
| 2 | Operator bumps PAT permissions to include `Organization members` (teams), `Organization administration` (webhooks + repo admin), `Organization secrets`, `Organization variables`, `Organization custom properties`, and grants the PAT access to all org repos (or "all current and future") | operator | §8 automation (otherwise all 🧑 items stay UI-only) |
| 3 | Operator confirms whether to archive eny-tech repos immediately after wave cutover or run read-only for a short grace period | operator | §7.2–§7.5 archival step |
| 4 | Operator confirms whether to carry over `eny-tech` release tags verbatim or retag in the new repos | operator | Wave execution |

---

## 10. Acceptance

This project is "phase 1 done" when **all** of the following hold:

| # | Criterion | Verifier |
|---|---|---|
| 1 | `_org` and `_templates` exist, populated | `gh repo view wellnexsolutionsai/_org` |
| 2 | All 5 source repos mirrored to their new homes (full history + refs) | `git log --all --oneline \| wc -l` matches old vs new |
| 3 | `platform-infra` and `platform-cloud-naming` created (empty or templated) | `gh repo view` |
| 4 | Every cross-repo `https://github.com/eny-tech/…` link in the new org points at `wellnexsolutionsai/…` | `grep -r eny-tech .` returns zero matches across new repos |
| 5 | `eny-tech` counterparts archived with redirect README | Old repo readme shows "moved to …" |
| 6 | Devcontainer remotes updated | `git remote -v` in each shared root |
| 7 | Branch protection + CODEOWNERS + labels applied to `main` of every non-research repo | `gh api repos/.../branches/main/protection` returns the expected config |
| 8 | Org profile README visible on `github.com/wellnexsolutionsai` | Browser |

Phase 2 (teams, secrets, Renovate, CodeQL, SSO) waits on PAT scope
bump and/or UI time.
