<!--
Repository: wellnex-projects
Path: projects/active/cloud-governance/plans/2026-04-19-cloud-naming-migration.md
Purpose: Plan to migrate the operator's azure-naming Function App into a clean, vendor-neutral, OSS-ready WellNex repo
Author: GitHub Copilot (Claude Opus 4.7) with operator
Created: 2026-04-19
Last-Modified: 2026-04-19
Version: 1.0.0
-->

# Plan: `cloud-naming` Function App Migration

> Migrate the operator's existing `azure-naming` Function App
> (currently authored under SanMar branding) into a clean,
> vendor-neutral repository that WellNex can adopt today and
> open-source later.

## Table of Contents

1. [Wait-states](#1-wait-states)
2. [Source vs target](#2-source-vs-target)
3. [Vendor-neutral name conventions](#3-vendor-neutral-name-conventions)
4. [File-level rename map](#4-file-level-rename-map)
5. [License and copyright](#5-license-and-copyright)
6. [Migration steps](#6-migration-steps)
7. [Acceptance](#7-acceptance)
8. [Open questions](#8-open-questions)

---

## 1. Wait-states

| Blocker | Owner | Unblocks |
|---------|-------|----------|
| Operator creates the new GitHub repo (proposed name `cloud-naming`) under `eny-tech/` | operator | §6 step 1 (push initial commit) |
| Operator authorizes the source repo to be cloned + scrubbed | operator | §6 step 2 (clone, fork-trim) |
| Operator confirms target package / function names | operator | §3 (naming choices) |

Until §1 row 1 is checked the deliverable of this plan is the **rename
map and migration script**, not commits in the new repo.

## 2. Source vs target

| Field | Source | Target |
|-------|--------|--------|
| Repo | `sanmar/azure-naming` (private, vendor-branded) | `eny-tech/cloud-naming` (private at first, OSS-ready) |
| Author identity in headers | SanMar internal authors | `WellNex contributors` (matches future `CONTRIBUTORS.md`) |
| Default org slug in `base.json` rules | `sm` | none — base config has **no** default org. Each consumer ships an overlay. |
| Generator package name (Python / Node / .NET — TBD) | SanMar-internal | vendor-neutral (`cloud_naming` / `@eny-tech/cloud-naming` / `CloudNaming`) |
| Function App name (live) | sanmar-specific subdomain | `func-wnx-eus2-prd-iac-cnm-01<rand5>` (per WellNex naming standard) |
| Rules layer priorities | base 0, regional 100, vendor 200 | Identical scheme — vendor overlays at 200, base at 0 |
| Storage backend (Table Storage for claim ledger) | yes | yes |
| Auth | unchanged | unchanged (Function App key for now; AAD later) |

## 3. Vendor-neutral name conventions

The migration hinges on stripping vendor-specific identifiers without
losing semantics. Apply these rules wherever the original code names
SanMar:

| Original concept | Renamed concept | Notes |
|------------------|-----------------|-------|
| `sanmar` (org slug in base config) | (no default; each overlay declares its own `org`) | Removes implicit assumption that the base layer is for any one tenant. |
| `SANMAR_*` env vars | `CLOUD_NAMING_*` env vars | Function-app-prefix on its own env; no vendor word. |
| `SanMarPrefix`, `require_sanmar_prefix`, etc. (any field name with vendor in it) | `OrgPrefix`, `require_org_prefix`, etc. | The field already takes an arbitrary string; only the *name* of the field encoded the vendor. |
| Doc copy: "SanMar uses…" | "Consumers use…" or "Each org overlay…" | Generic third-person, not vendor-branded. |
| Logo / README banner | Replaced by neutral mark; project name = `cloud-naming` | OSS-ready. |
| Git history that references internal Jira / Confluence / Slack | Squash-import (no history) for v0.1.0 — clean slate for OSS | See §6 step 2. |
| Internal CHANGELOG entries | Single `CHANGELOG.md` entry: `0.1.0 — initial public cut, derived from prior internal implementation` | No internal history exposed. |

The **only** WellNex-flavored content in the repo is the integration
test fixtures (which exercise `wellnex.json`-shaped overlays). Those
go in `examples/` clearly marked as illustrative.

## 4. File-level rename map

This is a placeholder pending repo access. Once the source is cloned,
this section is replaced with an exhaustive `mv` script. Expected
shape:

```
README.md                 → README.md (re-written, no SanMar refs)
LICENSE                   → LICENSE (re-licensed, see §5)
CHANGELOG.md              → CHANGELOG.md (single 0.1.0 entry)
host.json                 → host.json (unchanged)
local.settings.example.json → local.settings.example.json (env vars renamed)
src/sanmar_naming/        → src/cloud_naming/        (or src/cloudnaming/, choose at §3)
src/sanmar_naming/api/    → src/cloud_naming/api/
src/sanmar_naming/rules/  → src/cloud_naming/rules/
rules/base.json           → rules/base.json (org neutralized)
rules/sanmar.json         → DELETED (vendor overlay; not part of OSS)
rules/<region>.json       → rules/<region>.json (kept; regional overlays are vendor-neutral)
tests/test_sanmar.py      → tests/test_org_overlays.py (rewritten with synthetic org)
tests/fixtures/           → tests/fixtures/ (synthetic only)
examples/                 → examples/wellnex/ (illustrative consumer; not loaded at runtime)
.github/workflows/*       → .github/workflows/* (unchanged behavior; secrets renamed)
```

## 5. License and copyright

For OSS readiness:

- **License**: MIT, unless the operator prefers Apache-2.0. WellNex
  default for OSS components is MIT (one-paragraph license, no
  patent grant overhead). Confirm with operator before push.
- **Copyright header in every source file**:

  ```python
  # SPDX-License-Identifier: MIT
  # Copyright (c) 2026 WellNex contributors
  ```

- **Acknowledgement**: a `NOTICE.md` file credits the original
  internal implementation as the inspiration for the public cut,
  without naming the source company. Phrase: *"This project derives
  patterns from prior production work generously donated by its
  original author."*
- **DCO** (Developer Certificate of Origin): commits are sign-off
  required (`git commit -s`). CI enforces it.

## 6. Migration steps

### Step 1 — wait for `eny-tech/cloud-naming` to exist

Operator creates the empty repo. Confirm visibility (private
initially, OSS-ready license + structure from day one).

### Step 2 — fork-trim the source

```bash
git clone --no-tags git@github.com:sanmar/azure-naming.git /tmp/azure-naming-source
cd /tmp/azure-naming-source
git checkout -b clean-cut

# 1. Strip git history (single squash commit on a fresh root)
git checkout --orphan v0.1.0-prep
git commit -m "feat: initial public cut (squashed from prior internal implementation)"

# 2. Apply rename script (built in this session, lives in
#    wellnex-projects/projects/active/cloud-governance/scripts/
#    once the §4 map is concrete).
bash /path/to/rename-script.sh

# 3. Apply license + headers
bash /path/to/relicense.sh

# 4. Add CONTRIBUTORS.md, NOTICE.md, CODE_OF_CONDUCT.md, SECURITY.md
cp /path/to/oss-templates/* .

# 5. Run tests; fix anything that referenced SanMar-only fixtures
pytest -q
```

### Step 3 — push to `eny-tech/cloud-naming`

```bash
git remote add origin git@github.com:eny-tech/cloud-naming.git
git push -u origin v0.1.0-prep:main
git tag v0.1.0
git push --tags
```

### Step 4 — open consumer PRs

- **`wellnex-docs`** PR replaces every "SanMar `azure-naming`"
  reference with `cloud-naming` (governance README, naming standard,
  overlays README, cloud-governance project README).
- **`wellnex-infra/modules/naming/`** PR sets the function URL to
  the new deployment.
- **`cloud-naming` overlay PR**: drop `wellnex.json` into
  `cloud-naming/rules/wellnex.json` (priority 200) **OR** keep it
  in `wellnex-docs` and have the function fetch it from the raw
  GitHub URL — pick one in §8.

### Step 5 — deploy the function

The function is itself a WellNex-managed Azure resource:

- Lives in `wellnex-infra/modules/cloud-naming-service/` (provisions
  the Function App, plan, storage account for the claim ledger).
- Composed into `envs/prd/` (single instance is enough; not per-env).
- Name: `func-wnx-eus2-prd-iac-cnm-01<rand5>` (system=`iac`,
  subsystem=`cnm`, per the v1.2.0 standard).

### Step 6 — flip `wellnex-infra/modules/naming/` to call the API

The HCL fallback stays for offline / disaster scenarios. CI test
asserts both paths produce identical names for the golden fixture.

## 7. Acceptance

| # | Criterion |
|---|-----------|
| 1 | `eny-tech/cloud-naming` exists, MIT-licensed, no SanMar references in code or docs |
| 2 | CI green on the new repo (lint + unit tests + a functional test that exercises a synthetic overlay) |
| 3 | `cloud-naming` Function App deployed via `wellnex-infra` with a v1.2.0-conformant name |
| 4 | `wellnex-infra/modules/naming/` calls the API and produces names identical to the HCL fallback for the golden fixture |
| 5 | All `wellnex-docs` references to "SanMar `azure-naming`" replaced with `cloud-naming` |

## 8. Open questions

- **Implementation language** of the source — Python / Node / C#?
  The migration script differs slightly per stack (pyproject vs
  package.json vs csproj). Operator to confirm at §6 step 2.
- **Where does `wellnex.json` live for the function to consume?**
  Two viable options:
  a. **In `cloud-naming/rules/`** as a checked-in overlay (function
     reads from disk on cold start). Pro: zero runtime dependency on
     wellnex-docs. Con: every WellNex overlay change requires a
     `cloud-naming` deploy.
  b. **In `wellnex-docs/governance/naming-overlays/wellnex.json`**
     fetched at function cold-start via the GitHub raw URL (pinned to
     a tag). Pro: governance owns the file end-to-end. Con: function
     has a network dependency at cold start.
  Recommendation: **(b)**, pinned to a `governance/<semver>` tag,
  with the last-known-good copy cached in Blob Storage as a fallback.
- **License**: MIT vs Apache-2.0 — operator confirms.
- **Project visibility**: private until v1.0.0, then public? Or
  public from v0.1.0? Recommendation: private through v0.1.x while
  the API surface settles, public from v1.0.0.
