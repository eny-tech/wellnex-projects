<!--
Repository: wellnex-projects
Path: projects/active/cloud-governance/plans/2026-04-19-azure-deploy-bootstrap.md
Purpose: Plan to bootstrap Azure subscription + remote Terraform state and stand up wellnex-infra
Author: GitHub Copilot (Claude Opus 4.7) with operator
Created: 2026-04-19
Last-Modified: 2026-04-19
Version: 1.0.0
-->

# Plan: Azure Deploy Bootstrap → `wellnex-infra`

> Companion to the
> [IaC-home decision](../decisions/2026-04-19-iac-home.md). Defines
> the repository skeleton, the bootstrap order, the wait-states, and
> what is acceptance for each milestone.

## Table of Contents

1. [Wait-states](#1-wait-states)
2. [Repository skeleton](#2-repository-skeleton)
3. [Bootstrap order](#3-bootstrap-order)
4. [Naming compliance](#4-naming-compliance)
5. [Cutover from `wellnex-server/infra/azure/`](#5-cutover-from-wellnex-serverinfraazure)
6. [CI / CD](#6-ci--cd)
7. [Acceptance](#7-acceptance)
8. [Open questions](#8-open-questions)

---

## 1. Wait-states

| Blocker | Owner | Unblocks |
|---------|-------|----------|
| `eny-tech/wellnex-infra` repo created | operator | §2 (push skeleton), §5 (cutover PRs) |
| Azure subscription provisioned | operator | §3 (bootstrap apply) |
| `az login` (operator interactive) **or** service-principal creds in devcontainer env | operator | any `terraform apply` |
| GitHub repo secrets/vars set: `AZURE_SUBSCRIPTION_ID`, `AZURE_TENANT_ID`, `AZURE_CLIENT_ID` | operator (after §3 step 4) | §6 (workflows) |
| `cloud-naming` Function App reachable | [cloud-naming migration](2026-04-19-cloud-naming-migration.md) | §4 step 3 (call generator from CI) |

Until the first row is checked, the work product of this plan is the
**skeleton on disk in this session worktree**, not commits in
`wellnex-infra`. Skeleton is staged at:

```
/home/vscode/workspaces/.sessions/cloudgov/_staging/wellnex-infra/
```

(materialized only when the operator gives the go-ahead, to avoid
polluting `wellnex-projects` with an unrelated tree).

## 2. Repository skeleton

```
wellnex-infra/
├── .github/
│   ├── CODEOWNERS                # Cloud Governance project owns everything
│   └── workflows/
│       ├── infra-plan.yml        # PR → terraform plan → comment + artifact
│       └── infra-apply.yml       # main + manual approval → terraform apply
├── .gitignore                    # *.tfstate*, .terraform/, *.tfvars (except *.example)
├── README.md
├── CHANGELOG.md
├── bootstrap/                    # ONE-SHOT, local state, run once per subscription
│   ├── README.md                 # exact steps (see §3)
│   ├── versions.tf
│   ├── providers.tf
│   ├── variables.tf
│   ├── main.tf                   # rg-iac, st-iac (state SA), tfstate container,
│   │                             # GH OIDC app reg + federated cred + RBAC
│   └── outputs.tf                # values to copy into GH repo secrets/vars
├── modules/
│   ├── naming/                   # composes names from the wellnex.json overlay
│   │   ├── README.md             # contract: inputs → name string + tags
│   │   ├── main.tf               # HCL fallback today; calls cloud-naming API later
│   │   ├── variables.tf          # slug, org, region, env, system, subsystem?, instance, purpose?
│   │   ├── outputs.tf            # name (long), name_short, tags
│   │   └── data/
│   │       └── wellnex.json      # SYMLINK or git-subtree of governance overlay
│   ├── platform-core/            # rg, log analytics, identity, base policy assignments
│   ├── api-stack/                # cae, ca(api), psql, kv, acr (carved from current main.tf)
│   ├── web-stack/                # swa
│   └── observability/            # app insights, dashboards (Phase later)
├── envs/
│   └── tst/                      # current early-test slice
│       ├── README.md
│       ├── backend.tf            # azurerm partial backend (init args via workflow)
│       ├── versions.tf
│       ├── providers.tf
│       ├── variables.tf
│       ├── main.tf               # composes platform-core + api-stack + web-stack
│       ├── terraform.tfvars.example
│       └── outputs.tf
├── policies/                     # Phase 2 — Azure Policy definitions + assignments
│   └── README.md
└── docs/
    ├── architecture.md           # diagram + module dependency graph
    └── runbooks/
        ├── first-time-bootstrap.md
        ├── new-environment.md
        └── disaster-recovery-state.md
```

**Anti-goals on the skeleton:**

- No `terragrunt`. Terraform CLI + workflows is enough.
- No `terraform.tfvars` checked in — only `.example`.
- No state file checked in. The `.gitignore` blocks it; the backend
  enforces it.
- No copy of `wellnex.json` checked in independently — it must be a
  symlink or subtree from `wellnex-docs/governance/naming-overlays/`
  to keep the standard authoritative.

## 3. Bootstrap order

Once the operator clears wait-states 1 and 3 of §1:

### Step 1 — register resource providers (idempotent)

```bash
az account set --subscription "$AZURE_SUBSCRIPTION_ID"
for ns in Microsoft.App Microsoft.OperationalInsights Microsoft.DBforPostgreSQL \
          Microsoft.ContainerRegistry Microsoft.KeyVault Microsoft.Storage \
          Microsoft.ManagedIdentity Microsoft.Web Microsoft.Insights; do
  az provider register --namespace "$ns"
done
```

### Step 2 — apply `bootstrap/` with **local state**

```bash
cd wellnex-infra/bootstrap
terraform init                    # local state ON PURPOSE for this run
terraform apply -var "subscription_id=$AZURE_SUBSCRIPTION_ID"
```

Creates (named per the v1.2.0 standard, `system=iac`):

| Resource | Generated name (eus2 prd) |
|----------|----------------------------|
| Resource group | `rg-wnx-eus2-prd-iac-01` |
| Storage account | `stwnxeus2prdiac01<rand5>` |
| Blob container | `tfstate` |
| User-assigned identity (for any local apply) | `id-wnx-eus2-prd-iac-01` |
| Entra app registration (GH OIDC) | `wnx-prd-iac-gh-oidc` |
| Federated credential — main | `subject = repo:eny-tech/wellnex-infra:ref:refs/heads/main` |
| Federated credential — pull-requests | `subject = repo:eny-tech/wellnex-infra:pull_request` |
| Federated credential — environment:production | `subject = repo:eny-tech/wellnex-infra:environment:production` |
| RBAC — Contributor on subscription | scope = `/subscriptions/<sub>` |
| RBAC — User Access Administrator on subscription | scope = `/subscriptions/<sub>` (needed to assign roles in `envs/tst/`) |

Outputs to capture (printed by `terraform output`):

```
state_resource_group_name
state_storage_account_name
state_container_name
gh_oidc_client_id
gh_oidc_tenant_id
azure_subscription_id
```

### Step 3 — commit + push the local state's *outputs* into the bootstrap directory's README

The local `bootstrap/.terraform/terraform.tfstate` is **not** committed
(`.gitignore` blocks it) but is uploaded to the new state container as
`bootstrap/bootstrap.tfstate` for re-runability:

```bash
az storage blob upload \
  --account-name "$(terraform output -raw state_storage_account_name)" \
  --container-name "$(terraform output -raw state_container_name)" \
  --name bootstrap/bootstrap.tfstate \
  --file .terraform/terraform.tfstate \
  --auth-mode login
```

### Step 4 — set GitHub repo secrets/vars

```bash
gh secret set AZURE_SUBSCRIPTION_ID -b "$(terraform output -raw azure_subscription_id)" --repo eny-tech/wellnex-infra
gh secret set AZURE_TENANT_ID       -b "$(terraform output -raw gh_oidc_tenant_id)"     --repo eny-tech/wellnex-infra
gh secret set AZURE_CLIENT_ID       -b "$(terraform output -raw gh_oidc_client_id)"     --repo eny-tech/wellnex-infra
gh variable set TF_STATE_RG         -b "$(terraform output -raw state_resource_group_name)"  --repo eny-tech/wellnex-infra
gh variable set TF_STATE_SA         -b "$(terraform output -raw state_storage_account_name)" --repo eny-tech/wellnex-infra
gh variable set TF_STATE_CONTAINER  -b "$(terraform output -raw state_container_name)"       --repo eny-tech/wellnex-infra
```

### Step 5 — first `envs/tst/` apply via Actions

Open a PR that flips `envs/tst/main.tf` from "empty" to "full
composition". The `infra-plan.yml` workflow runs OIDC-authenticated,
inits with the new backend, plans, and posts the diff. Merge → manual
approval on `production` environment → apply.

### Step 6 — destroy the placeholder API container app revision

The current `wellnex-server/infra/azure/` uses
`mcr.microsoft.com/k8se/quickstart:latest` as a placeholder. The new
`envs/tst/` apply does the same; it is replaced once `wellnex-server`
publishes its first image to ACR.

## 4. Naming compliance

The new repo is born conformant to the [Cloud Resource Naming
v1.2.0 standard](https://github.com/eny-tech/wellnex-docs/blob/main/governance/cloud-resource-naming.md).

Three layers, in priority order:

1. **`modules/naming/`** is a thin Terraform module that composes
   names from the overlay using HCL templates. Used today.
2. **`cloud-naming` Function App** (see [migration plan](2026-04-19-cloud-naming-migration.md))
   is the long-term generator. Once available, `modules/naming/` calls
   it via `data "http"` and falls back to local templating only when
   the function is unreachable. The fallback **must** produce the
   same name for the same input — verified by a CI test that runs both
   paths against a golden fixtures file.
3. The overlay JSON is a **subtree** (`git subtree add --prefix
   modules/naming/data/ git@github.com:eny-tech/wellnex-docs.git
   main --squash`) so the file in `wellnex-infra` is a verbatim copy
   of the authoritative one in `wellnex-docs`. CI fails if they
   diverge.

## 5. Cutover from `wellnex-server/infra/azure/`

The cutover is a single coordinated event:

1. `wellnex-infra` PR adds `envs/tst/` composition that produces
   *exactly* the same set of resources as `wellnex-server/infra/azure/`
   (modulo names — those move to v1.2.0).
2. `wellnex-infra` PR is reviewed and approved but **not** applied.
3. `wellnex-server` PR removes `infra/azure/` and replaces it with a
   short README pointer to `eny-tech/wellnex-infra`.
4. Apply order on the day of cutover:
   a. From `wellnex-server/infra/azure/`: `terraform destroy` (or
      `terraform state rm` followed by manual resource deletion if
      the early test data must be preserved). For the early-test
      window with no real users, **destroy is the simpler call**.
   b. From `wellnex-infra/envs/tst/` Actions: `apply`. Resources
      reappear with v1.2.0 names.
   c. Reseed Key Vault secrets that were generated (`session-secret`)
      — they will be regenerated.
5. Merge both PRs in the same hour; tag both repos `cutover-2026-MM-DD`.

If the early test environment has been used by a real Oura test user
by cutover day, this becomes a **state import** instead of destroy /
recreate. The plan supports either path; pick on the day.

## 6. CI / CD

### `infra-plan.yml`

- Trigger: `pull_request` on `wellnex-infra`.
- Steps:
  1. Checkout.
  2. `azure/login@v2` (OIDC).
  3. `hashicorp/setup-terraform@v3`.
  4. For each changed `envs/<env>/`:
     - `terraform init -backend-config="resource_group_name=${{ vars.TF_STATE_RG }}" -backend-config="storage_account_name=${{ vars.TF_STATE_SA }}" -backend-config="container_name=${{ vars.TF_STATE_CONTAINER }}" -backend-config="key=wellnex/${{ matrix.env }}.tfstate"`
     - `terraform plan -no-color -out tfplan`
     - `terraform show -no-color tfplan` → upload as artifact + comment on PR.
- Concurrency: `cancel-in-progress` per PR + per env.

### `infra-apply.yml`

- Trigger: `push` to `main` *and* manual `workflow_dispatch`.
- `environment: production` (requires reviewer approval).
- Steps mirror `plan`, but `terraform apply tfplan` against the
  artifact from the PR's last successful plan.
- Concurrency: serialized by env (`group: apply-${{ matrix.env }}`,
  `cancel-in-progress: false`).

### Blast-radius gate

Reuse `wellnex-server/infra/azure/scripts/blast-radius.sh` (port it
into `wellnex-infra/scripts/`). Plan output is parsed; if the plan
**replaces** any of `azurerm_postgresql_flexible_server`,
`azurerm_key_vault`, `azurerm_container_registry`, the workflow
fails until a label `infra/blast-ok` is on the PR.

## 7. Acceptance

This plan is "done" when **all** of the following are true:

| # | Acceptance criterion | Verifier |
|---|----------------------|----------|
| 1 | `eny-tech/wellnex-infra` exists with the §2 skeleton | `git ls-tree origin/main` |
| 2 | `bootstrap/` applied; state SA + container + GH OIDC live | `terraform output` printed; `az storage blob list ...` shows `bootstrap/bootstrap.tfstate` |
| 3 | GH repo secrets/vars set | `gh secret list` / `gh variable list` |
| 4 | `envs/tst/` `terraform plan` succeeds via Actions on a PR | green check on PR |
| 5 | `envs/tst/` `terraform apply` succeeds via manual approval | green check on `infra-apply` |
| 6 | Old `wellnex-server/infra/azure/` removed | merged PR |
| 7 | All resources produced by the apply parse the v1.2.0 naming standard | `wellnex-agents` lifecycle validator (or equivalent CI check); zero drift findings |
| 8 | `cloud-governance/README.md` Phase 1 checkboxes for IaC + state moved to `[x]` | git log on README |

## 8. Open questions

- **Region**: current config defaults to `eastus2`. Confirmed for tst,
  open for prd.
- **Custom domain**: deferred per current README ("uses default
  Azure hostnames"). Re-open after Oura-test phase.
- **VNet / private endpoints**: out of scope for tst. In scope for
  prd; modelled in `modules/networking/` (not in §2 yet).
- **State partition**: one storage account, one container, one key
  per env (`wellnex/tst.tfstate`, `wellnex/stg.tfstate`,
  `wellnex/prd.tfstate`) — confirmed standard. No per-stack partition
  inside an env unless modules are split into separate state files,
  which we are not doing in v1.
- **Cross-tenant operator access**: who else gets Contributor on the
  subscription? Captured in `bootstrap/variables.tf` as
  `additional_contributor_object_ids` (default empty); operator fills
  in their own object ID at apply time.
