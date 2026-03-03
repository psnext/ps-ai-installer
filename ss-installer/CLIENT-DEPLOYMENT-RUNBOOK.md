# Slingshot Installer - Client Deployment Runbook

> Production-ready guide for client DevOps/platform teams to perform fresh installation and upgrades using release artifacts.

## 1) Overview

This runbook explains how to deploy and upgrade Slingshot using the installer package you receive for a release.

It covers two core paths:
1. Fresh installation
2. Upgrade from one version to another

It is written for client delivery teams and focuses on:
- exact command flow
- required tools and verification
- artifact usage (Terraform zip, Helm zip, deployment-scripts zip)
- prompt behavior and default values
- release-values mapping to installer inputs

---

## 2) Who should use this guide

Use this guide if you are:
- DevOps Engineer
- SRE / Platform Engineer
- Release Engineer

Expected baseline skills:
- Kubernetes (`kubectl`, Helm) basics
- Cloud account and IAM access
- Secrets management basics
- Terminal usage on macOS/Linux/WSL2

---

## 3) Tools Required for Installer Operation

## 3.1 Mandatory tools

| Tool | Why needed | Minimum | Verify command |
|---|---|---:|---|
| `slingshot` | Main installer binary | release-provided | `./slingshot --help` |
| `kubectl` | Cluster access, validation, deploy checks | compatible with cluster | `kubectl version --client` |
| `helm` | Chart deployment/upgrade | v3+ | `helm version` |
| `terraform` | Infra provisioning (if used) | v1.x | `terraform version` |

## 3.2 Cloud/provider tools (as applicable)

| Provider | Tool | Verify command |
|---|---|---|
| Azure | `az` | `az version` |
| AWS | `aws` | `aws --version` |
| GCP | `gcloud` | `gcloud --version` |

## 3.3 Installer command to bootstrap tools

Run:

```bash
./slingshot setup-tools
```

`setup-tools` supports status check, resume/re-run, and complete installation flow.

## 3.4 Connectivity verification

Run before deployment window:

```bash
kubectl cluster-info
kubectl get ns
helm list -A
```

If provisioning with Terraform:

```bash
terraform -help
```

---

## 4) Environment preparation

## 4.1 Client delivery package (per release)

You should receive:
1. Installer binary/package
2. Terraform artifacts zip
3. Helm charts zip/tarball
4. Deployment scripts zip/tarball
5. Installer user guide
6. Release deployment values guide

## 4.2 Pre-flight checklist

- [ ] Cloud account authenticated (provider CLI login complete)
- [ ] Kubernetes target cluster available (or provisioning plan approved)
- [ ] Namespace confirmed
- [ ] Registry access confirmed
- [ ] DB endpoints/credentials available (if required by scripts)
- [ ] Key vault/secrets manager permissions validated
- [ ] Release values guide available for current version
- [ ] Release artifacts accessible from jump host/runner machine

---

## 5) Fresh Installation Runbook

## 5.1 Recommended end-to-end flow

1. `config init`
2. `setup-tools`
3. `provision-infra` (if installer-managed infra is used)
4. `validate-infra`
5. `vault-sync`
6. `package-pull`
7. `install-app`
8. `validate-app`
9. `troubleshoot` (only if needed)

## 5.2 Command execution details

### Step 1 - Initialize configuration

```bash
./slingshot config init
```

Expected result:
- local config created under `.slingshot/config/...`
- cloud/provider and deployment defaults captured

### Step 2 - Setup tools

```bash
./slingshot setup-tools
```

Expected result:
- required CLI tools verified/installed
- missing tools reported clearly

If auto-install fails due to permissions:
- install tool manually
- rerun `setup-tools` status/resume

### Step 3 - Provision infrastructure (optional if infra pre-existing)

```bash
./slingshot provision-infra
```

In Configure mode, select Terraform source:
- tarball/zip (release Terraform artifact)
- git repository

Expected result:
- Terraform workflow configured and executed (or plan-only if chosen)

### Step 4 - Validate infrastructure

```bash
./slingshot validate-infra
```

Expected result:
- infrastructure readiness checks pass
- outputs folder / validation config confirmed

### Step 5 - Vault sync

```bash
./slingshot vault-sync
```

Typical client usage:
- generate secrets template from Helm charts
- validate required secrets exist
- import missing secrets from Excel template if needed

Expected result:
- release-required secrets confirmed available

### Step 6 - Package pull

```bash
./slingshot package-pull
```

Expected result:
- release artifacts resolved/pulled
- chart/image references ready for deployment

### Step 7 - Install app

```bash
./slingshot install-app
```

Expected result:
- deployment groups installed in order
- pre/post deployment scripts executed when configured

### Step 8 - Validate app

```bash
./slingshot validate-app
```

Expected result:
- post-deploy checks pass
- issues surfaced with actionable messages

---

## 6) Upgrade Runbook (version-to-version)

## 6.1 Recommended minimal upgrade flow

1. `config init` (or `config update` for existing config)
2. `setup-tools`
3. `vault-sync` (validate release secrets first)
4. `package-pull`
5. `upgrade-app`

## 6.2 Three-phase upgrade flow

Run:

```bash
./slingshot upgrade-app
```

### Phase 1 - Pre-deployment activities

Typical activities:
- database backup
- secrets backup
- config server updates (optional)
- portal/backlog script execution (optional)

Script-based actions now follow common artifact preparation behavior:
- if scripts already extracted: prompt to re-extract or keep existing
- if archive path missing: re-prompt for path

### Phase 2 - Deployment activities

- uses Helm charts artifacts for upgrade deployment
- if Helm charts already extracted: prompt to re-extract or keep existing
- if archive path missing: re-prompt for tarball/zip path

### Phase 3 - Post-deployment activities

Typical activities:
- prompt migration
- portal scripts
- restart/verification actions

Script-dependent actions re-check scripts readiness using same extraction logic as Phase 1.

---

## 7) Artifact handling (Terraform/Helm/deployment scripts)

## 7.1 What each artifact is used for

| Artifact | Where used | Primary command |
|---|---|---|
| Terraform zip | infra provisioning/infra setup workflows | `provision-infra`, `validate-infra` |
| Helm charts zip/tarball | app deployment and upgrade deployment | `package-pull`, `install-app`, `upgrade-app` |
| Deployment scripts zip/tarball | pre/post deployment scripts, config-server, migrations | `install-app`, `upgrade-app` |

## 7.2 Extraction location behavior

Typical local extraction roots:
- `.slingshot/terraform/...`
- `.slingshot/helm-charts/...`
- `.slingshot/deployment-scripts/...`

## 7.3 Reuse vs re-extract decision pattern

When artifact content is already present, the installer asks whether to:
- **Continue** (extract again, fresh run, overwrite local extracted content)
- **Skip** (preserve local changes and reuse existing extracted files)

This pattern is used consistently in upgrade activities for:
- deployment scripts in Phase 1 and Phase 3
- Helm charts in Phase 2

## 7.4 Missing archive path behavior

If configured tarball/zip path does not exist:
- installer warns path is missing
- prompts user again for a valid path
- proceeds only after valid file is provided

## 7.5 Artifact decision table

| Situation | Recommended action |
|---|---|
| First run of release | Continue (extract fresh) |
| Rerun with local custom edits in extracted scripts/charts | Skip (preserve local changes) |
| Suspected stale or corrupted extracted folder | Continue (force fresh extraction) |
| Archive path changed between releases | Continue and point to new archive |

---

## 8) Prompt Reference (description + defaults + recommendations)

> Note: Exact wording can vary by cloud/provider and release. Use this table as operational guidance.

| Command/Flow | Prompt (example) | Purpose | Accepted input | Default source | Keep/change guidance |
|---|---|---|---|---|---|
| `config init` | Cloud provider | Select deployment provider | menu selection | hard-coded options | choose target cloud |
| `config init` | Region / namespace / cluster | Define environment target | text/menu | config/history/computed | keep if matches target env |
| `setup-tools` | operation (status/resume/install) | choose tool setup action | menu selection | hard-coded options | first run: install; rerun: resume/status |
| `package-pull` | registry auth method | choose auth path | menu selection | hard-coded options | choose org-approved auth method |
| `package-pull` | chart/image version/tag | select release artifact version | text/menu | config/history | use release-approved version only |
| `vault-sync` | operation mode (template/validate/import) | secrets workflow selection | menu selection | hard-coded options | run validate before install/upgrade |
| `install-app` | extract artifacts? continue/skip | fresh extract vs preserve local | menu selection | computed from existing extracted state | first run continue; reruns case-by-case |
| `upgrade-app` Phase 1/3 | extract deployment scripts? continue/skip | force re-extract or reuse scripts | menu selection | computed from local extracted state | continue for clean release run |
| `upgrade-app` Phase 2 | extract Helm charts? continue/skip | force re-extract or reuse charts | menu selection | computed from local extracted state | continue for clean release run |
| `upgrade-app` | target version | set upgrade target | text | existing upgrade config/application version | set to release change target |
| script migrations | proceed with migration? | explicit confirmation for optional migrations | menu selection | hard-coded options | run only if release guide requires |

## 8.1 Default value source legend

- **Derived from existing local config**: prior run values loaded from `.slingshot/config/...`
- **Hard-coded by installer**: static default in command flow
- **Computed from runtime context**: discovered from environment, extracted folders, or cluster state

---

## 9) Release values mapping (mandatory)

Use this as your implementation checklist per release.

| Value from release deployment guide | Entered in installer at | Required | Default behavior | Example format |
|---|---|---|---|---|
| Target release version | `upgrade-app` configuration | Yes (upgrade) | may prefill from prior upgrade config | `R2.2.0` |
| Helm charts archive path | package/upgrade chart prompts | Yes | previous path may appear as default | `/path/release/helm-charts-R2.2.0.zip` |
| Deployment scripts archive path | install/upgrade script prompts | Yes (if scripts enabled) | previous path may appear as default | `/path/release/deployment-scripts-R2.2.0.zip` |
| Terraform archive path | provision infra configure | Conditional | previous terraform source may be retained | `/path/release/terraform-R2.2.0.zip` |
| Registry endpoint/user | config/package-pull auth steps | Yes | may derive from config | `myregistry.azurecr.io` |
| Registry token/password | package-pull auth prompts | Yes | typically no persisted plaintext default | token/password string |
| Namespace | config/install/upgrade context | Yes | from config/environment | `slingshot-ns` |
| Domain URL | script/config prompts | Release-dependent | may derive from config or existing env files | `client.example.com` |
| DB host/user/port | script/config prompts | Release-dependent | may prefill from config/K8s/env files | `dbhost:5432` |
| Secret keys in charts | `vault-sync` validate/import/template flows | Yes for successful deployment | missing values flagged for fill | key names from chart |

---

## 10) Validation checklist (before go-live)

- [ ] `setup-tools` reports all required tools available
- [ ] cluster reachable (`kubectl get nodes`)
- [ ] namespace and context verified
- [ ] required release artifacts present and checksum-verified
- [ ] `vault-sync` validation passes for required chart secrets
- [ ] app install/upgrade completed with no failed groups
- [ ] `validate-app` passed
- [ ] smoke tests completed for critical user journeys
- [ ] rollback instructions reviewed by on-call team

---

## 11) Common issues and troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| Archive not extracted after selecting Continue | stale reuse path or wrong file path | select Continue and provide valid archive path; verify extraction log appears |
| Prompt shows old default path | config still points to prior release artifact | update path in config flow and rerun |
| Tool install fails | permissions or PATH issue | install manually, then rerun `setup-tools` status/resume |
| Secrets validation fails | missing keys in vault | run `vault-sync` template + import + validate |
| Upgrade phase script error | missing/incorrect scripts archive | re-run phase and choose Continue to re-extract scripts |
| Helm upgrade cannot find chart | wrong chart archive/path | update Helm archive path and re-extract in Phase 2 |

If unresolved, run:

```bash
./slingshot troubleshoot
```

---

## 12) Operational best practices

1. Treat each release as immutable: use release-specific artifact folders.
2. Run `vault-sync` validate before deployment windows.
3. Prefer **Continue (extract fresh)** for first run of each release.
4. Use **Skip** only when intentionally preserving local edits.
5. Keep production and non-production configs separate.
6. Keep command outputs/logs for audit and rollback analysis.
7. Validate after each major phase, not only at the end.

---

## 13) Appendix: command quick reference

## 13.1 Core commands

```bash
./slingshot config init
./slingshot setup-tools
./slingshot provision-infra
./slingshot validate-infra
./slingshot vault-sync
./slingshot package-pull
./slingshot install-app
./slingshot upgrade-app
./slingshot validate-app
./slingshot troubleshoot
```

## 13.2 Typical release execution (fresh install)

```bash
./slingshot config init
./slingshot setup-tools
./slingshot provision-infra
./slingshot validate-infra
./slingshot vault-sync
./slingshot package-pull
./slingshot install-app
./slingshot validate-app
```

## 13.3 Typical release execution (upgrade)

```bash
./slingshot config init   # or config update in your operating process
./slingshot setup-tools
./slingshot vault-sync
./slingshot package-pull
./slingshot upgrade-app
./slingshot validate-app
```

---

## 14) Change control recommendation

For each release, publish alongside this guide:
- release artifact list + checksums
- release values guide
- upgrade compatibility notes
- known issues and mitigations

This ensures client teams can execute consistently across environments with minimal ambiguity.
