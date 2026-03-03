# Slingshot Installer — Client User Guide

> **Version:** 2.2.0 · **Audience:** Client DevOps / SRE / Platform Engineers · **Classification:** External

---

## Table of Contents

1. [Overview](#1-overview)
2. [Who Should Use This Guide](#2-who-should-use-this-guide)
3. [Tools Required](#3-tools-required)
4. [Environment Preparation](#4-environment-preparation)
5. [Fresh Installation Runbook](#5-fresh-installation-runbook)
6. [Upgrade Runbook](#6-upgrade-runbook)
7. [Artifact Handling](#7-artifact-handling)
8. [Prompt Reference](#8-prompt-reference)
9. [Release Values Mapping](#9-release-values-mapping)
10. [Validation Checklist](#10-validation-checklist)
11. [Common Issues and Troubleshooting](#11-common-issues-and-troubleshooting)
12. [Operational Best Practices](#12-operational-best-practices)
13. [Appendix: Command Quick Reference](#13-appendix-command-quick-reference)

---

## 1. Overview

The **Slingshot Installer** is an interactive command-line tool used to deploy and upgrade the Slingshot platform on Kubernetes clusters across Azure, AWS, GCP, and OpenShift environments.

The installer guides you through every step — from configuring the environment and provisioning infrastructure, to pulling container images, syncing secrets, deploying applications, and validating the result. It is designed so that a delivery engineer can run the complete process end-to-end using this guide alone, together with the release-specific deployment values guide provided by the Slingshot delivery team.

### What the installer does

| Flow | Purpose |
|---|---|
| **Fresh Installation** | Deploy Slingshot for the first time onto a new Kubernetes cluster |
| **Upgrade** | Upgrade an existing Slingshot deployment from one release version to another |

### Command flow at a glance

**Fresh Installation:**
```
config init → setup-tools → provision-infra → validate-infra → vault-sync → package-pull → install-app → validate-app
```

**Upgrade:**
```
config init → setup-tools → vault-sync → package-pull → upgrade-app → validate-app
```

---

## 2. Who Should Use This Guide

This guide is written for:

- **DevOps / Platform engineers** responsible for deploying or upgrading Slingshot on behalf of a client.
- **SRE / Deployment engineers** managing Kubernetes-based workloads in Azure, AWS, or GCP.
- Engineers working from the artifacts and values provided by the Slingshot release team.

**Assumed knowledge:**
- Basic Kubernetes operations (`kubectl`, namespaces, contexts)
- Basic Helm usage
- Cloud CLI familiarity (`az`, `aws`, or `gcloud` depending on target cloud)
- General understanding of Terraform for infrastructure provisioning steps

**Not required:** Knowledge of Slingshot internals or Go development.

---

## 3. Tools Required

### 3.1 Tool categories

The installer distinguishes between tools it can install automatically via `setup-tools` and tools that must be present beforehand.

#### Tools installed by `setup-tools`

| Tool | Purpose | Verify |
|---|---|---|
| `kubectl` | Kubernetes cluster management | `kubectl version --client` |
| `helm` (≥ 3.12) | Helm chart deployment | `helm version` |
| `terraform` (≥ 1.5) | Infrastructure provisioning | `terraform version` |
| `az` | Azure cloud operations (Azure only) | `az version` |
| `aws` | AWS cloud operations (AWS only) | `aws --version` |
| `gcloud` | GCP cloud operations (GCP only) | `gcloud --version` |
| `oc` | OpenShift CLI (OpenShift only) | `oc version` |
| `crane` | Preferred image copy tool for `package-pull` | `crane version` |
| `docker` | Image copy fallback (if crane unavailable) | `docker --version` |
| `podman` | Rootless image copy alternative | `podman --version` |

#### Tools that must be pre-installed

| Tool | Purpose | Verify |
|---|---|---|
| `make` | Build/automation dependencies | `make --version` |
| `git` | Git-based artifact source workflows | `git --version` |
| `curl` | Downloads and connectivity checks | `curl --version` |
| `bash` | Shell script execution | `bash --version` |
| `python` + `pip` | Script runtime dependencies | `python --version` |
| `npm` + `node` | Script runtime dependencies | `npm --version` |
| `mongosh` | MongoDB operations in deployment scripts | `mongosh --version` |
| `mongodump` | MongoDB backups | `mongodump --version` |
| `pg_dump` | PostgreSQL backups | `pg_dump --version` |

#### Optional security tools

| Tool | Purpose | Verify |
|---|---|---|
| `skopeo` | Alternative image copy tool | `skopeo --version` |
| `cosign` | Image signing and verification | `cosign version` |
| `trivy` | Container vulnerability scanning | `trivy --version` |

### 3.2 Minimum version requirements

| Component | Minimum Version |
|---|---|
| Kubernetes cluster | 1.28+ |
| Helm | 3.12+ |
| Terraform | 1.5+ |
| kubectl | matching cluster version (1.28+) |

### 3.3 `crane` behavior — important

`package-pull` uses `crane` as its preferred image copy tool because it is the fastest and most resource-efficient option. The installer selects the copy tool in this priority order:

1. `crane` (recommended)
2. `skopeo`
3. `docker`
4. `podman`

If `crane` is not installed when you run `package-pull`, the installer will prompt you to install it via `setup-tools`. If you decline, the workflow falls back to `docker pull/tag/push`, which is slower but fully supported.

### 3.4 Installation path

`setup-tools` installs tools to `~/.ss-tools/bin` by default — no `sudo` required. If you prefer system-wide installation, the installer will prompt for it. After installation, the installer provides PATH configuration guidance if needed.

---

## 4. Environment Preparation

### 4.1 Release delivery package

For each release, the Slingshot delivery team provides these artifacts. Have them downloaded and locally accessible before starting:

| Artifact | File example | Used in |
|---|---|---|
| Installer binary | `slingshot` (Linux/macOS) | All commands |
| Terraform artifacts zip | `terraform-R2.2.0.zip` | `provision-infra`, `validate-infra` |
| Helm charts zip/tarball | `helm-charts-R2.2.0.zip` | `install-app`, `upgrade-app` Phase 2 |
| Deployment scripts zip/tarball | `deployment-scripts-R2.2.0.zip` | `install-app`, `upgrade-app` Phase 1 & 3 |
| Release deployment values guide | `deployment-guide-R2.2.0.pdf` | All command prompts |
| Installer user guide | this document | Reference |

> **Recommendation:** Keep all release artifacts in a version-isolated folder (e.g., `/releases/R2.2.0/`) and never overwrite previous release artifacts.

### 4.2 Pre-run checklist

Complete these steps before starting either flow:

- [ ] Release artifacts downloaded and checksums validated
- [ ] Installer binary is executable: `chmod +x ./slingshot`
- [ ] Cloud CLI login completed and session active:
  - Azure: `az login` then `az account set --subscription <id>`
  - AWS: `aws configure` or `aws sso login`
  - GCP: `gcloud auth login && gcloud config set project <project>`
- [ ] Kubernetes context points to the correct cluster: `kubectl cluster-info`
- [ ] Target namespace confirmed (e.g., `slingshot-ns`)
- [ ] Database endpoints and credentials ready (MongoDB connection string, PostgreSQL host/port/user)
- [ ] Key vault / secrets manager access permissions verified
- [ ] Release deployment values guide available for data entry
- [ ] Network connectivity verified from the deployment machine to:
  - Cloud API endpoints
  - Kubernetes API server
  - Container registry
  - Database servers (MongoDB, PostgreSQL)
  - Key vault / secrets manager

### 4.3 Installer binary setup

```bash
# Make the binary executable
chmod +x ./slingshot

# Verify the binary works
./slingshot --version

# (Optional) Move to a directory in PATH for convenience
mv ./slingshot /usr/local/bin/slingshot
```

### 4.4 Installer navigation

The installer uses interactive menus throughout. Navigation controls:

| Key | Action |
|---|---|
| `↑` / `↓` arrows | Navigate menu items |
| `Enter` | Select highlighted option |
| `0` or type `back` | Go back (in submenus) / Quit (in main menu) |
| `q`, `x`, `exit` | Exit the application |
| `Ctrl+C` | Cancel current operation |

---

## 5. Fresh Installation Runbook

Run these commands **in order**. Do not skip steps unless explicitly noted.

```
Step 1:  config init       — Create deployment configuration
Step 2:  setup-tools       — Install and verify required CLI tools
Step 3:  provision-infra   — Provision Kubernetes cluster (if installer-managed)
Step 4:  validate-infra    — Validate infrastructure readiness
Step 5:  vault-sync        — Scan Helm charts for secrets, upload missing secrets
Step 6:  package-pull      — Pull and sync container images to target registry
Step 7:  install-app       — Deploy all application groups
Step 8:  validate-app      — Validate deployment health
```

---

### Step 1 — Initialize Configuration

```bash
./slingshot config init
```

**What this does:** Launches an interactive wizard that collects all deployment parameters and saves them to a JSON configuration file. This file is the source of truth for every subsequent command.

**Configuration file location:**
```
.slingshot/config/ss-installer-config-<cloud>.json
```
For example: `.slingshot/config/ss-installer-config-azure.json`

> **Important:** If a config file already exists from a previous run, the installer creates a timestamped backup (`.bak`) before overwriting. You can restore it if needed.

**Wizard steps — what you will be asked:**

| Step | What you configure | Source of values |
|---|---|---|
| 1. Installation type | Fresh Install or Upgrade | Select "Fresh Install" |
| 2. Cloud provider | Azure / AWS / GCP / OpenShift | Your target environment |
| 3. Slingshot version | Select from list or enter custom | Release deployment values guide |
| 4. Environment details | Subscription, resource group, cluster name, namespace | Platform team / release guide |
| 5. Package pull | Enable/disable image sync, registry details | Release deployment values guide |
| 6. Infrastructure provisioning | Terraform source (tarball or Git), paths | Release deployment values guide |
| 7. Application deployment | Helm charts source, deployment scripts source | Release deployment values guide |
| 8. Secrets/vault | Vault type and connection details | Platform team |

**Sensitive values — never saved to disk:**
- Service principal secrets
- Git tokens and passwords
- Registry passwords and tokens

Instead, use environment variable references in the format `${MY_SECRET_VAR}` when prompted, or enter them interactively (they are kept in memory only for the current session).

**Expected result:**
```
Configuration saved to: .slingshot/config/ss-installer-config-azure.json
Next steps:
  1. Run 'slingshot setup-tools' to install required CLI tools
  2. Run 'slingshot provision-infra' to create infrastructure (if needed)
```

---

### Step 2 — Setup Tools

```bash
./slingshot setup-tools
```

**What this does:** Detects which required CLI tools are already installed, verifies their versions, and installs any that are missing.

**What you will see:**
- A checklist of required tools for your cloud provider
- Green checkmarks for tools already installed and version-verified
- Prompts to install missing tools

**Installation location:** `~/.ss-tools/bin` (no sudo required by default)

**If auto-install fails due to permissions:**

The installer will display the manual installation command. Run it separately, then re-run `setup-tools` to verify.

```bash
# Example: manual crane install on Linux
curl -L https://github.com/google/go-containerregistry/releases/latest/download/go-containerregistry_Linux_x86_64.tar.gz | tar -xz -C ~/.ss-tools/bin crane
```

**Expected result:**
```
Tool verification complete
  ✓ kubectl    v1.29.2
  ✓ helm       v3.14.0
  ✓ terraform  v1.7.4
  ✓ az         2.57.0
  ✓ crane      v0.19.1
All required tools are installed and verified.
```

---

### Step 3 — Provision Infrastructure _(skip if cluster already exists)_

```bash
./slingshot provision-infra
```

**What this does:** Uses Terraform to create or update the Kubernetes cluster and associated infrastructure. Skip this step if the cluster has already been provisioned externally.

**Menu options:**

```
1. Configure Infrastructure Provisioning
2. Execute Infrastructure Provisioning
3. View Status
0. Back to Main Menu
```

**Terraform source — you will be asked to choose:**

| Option | When to use | Required inputs |
|---|---|---|
| Tarball / zip | Client receives Terraform artifacts zip | Path to `terraform-R2.2.0.zip` |
| Git repository | Terraform code is in a Git repo | Repo URL, branch, auth token |

**Artifact extraction:**
- Tarball extracts to: `.slingshot/terraform/`
- The installer looks for the Terraform root folder (e.g., `tf/`) inside the archive
- You specify the subfolder containing the target environment code (e.g., `ps` for platform services)

**Execution steps:** The installer runs `terraform init`, `terraform plan`, then `terraform apply` in sequence. You will see real-time output.

> **Warning:** `provision-infra` provisions real cloud resources. Verify the Terraform variables and subscription before executing apply. This action incurs cloud costs and is not immediately reversible.

**Expected result:**
```
Infrastructure provisioning complete.
Kubernetes cluster is ready.
Run 'slingshot validate-infra' to verify cluster readiness.
```

---

### Step 4 — Validate Infrastructure

```bash
./slingshot validate-infra
```

**What this does:** Runs a series of health checks against the cluster and cloud environment to confirm everything is ready for application deployment.

**Checks performed:**

| Category | What is checked |
|---|---|
| Cloud authentication | Active session with correct account/subscription |
| Kubernetes connectivity | API server reachable, kubeconfig valid |
| Node health | All nodes Ready, no NotReady/SchedulingDisabled |
| Storage | Required storage classes present, PVCs bindable |
| Networking | DNS resolution, service connectivity |
| Azure Key Vault | Access permissions verified (Azure only) |
| Container registry | Connectivity and pull permissions |

**Expected result:**
```
Infrastructure Validation: PASSED
  ✓ Cloud authentication
  ✓ Kubernetes connectivity
  ✓ Node health (3/3 nodes ready)
  ✓ Storage classes available
  ✓ Network configuration
  ✓ Key Vault access
```

If any check fails, the installer shows the specific error and a remediation suggestion. Fix the issue and re-run before proceeding.

---

### Step 5 — Vault Sync

```bash
./slingshot vault-sync
```

**What this does:** Scans the Helm charts for required secrets, compares them against the vault, identifies missing secrets, and allows you to upload them interactively or via Excel import.

**Typical fresh install usage:**
1. Select **Helm Secrets Scan** from the menu
2. The installer scans the Helm charts and lists all required secrets
3. For each missing secret, you are prompted to enter the value (value is masked and never logged)
4. Secrets are uploaded to the target vault (Azure Key Vault / AWS Secrets Manager / GCP Secret Manager / HashiCorp Vault)

**Menu options:**

```
1. Excel Import          — Bulk import secrets from KeyVaultSecrets.xlsx
2. Vault Copy / Move     — Migrate secrets between vaults
3. Helm Secrets Scan     — Scan charts, find missing secrets, upload interactively
4. View Configuration
0. Back to Main Menu
```

**Excel import format:**
If the release team provides a pre-filled `KeyVaultSecrets.xlsx`, use option 1 to bulk-import secrets instead of entering them one by one.

**Audit log:** Each vault-sync run saves an audit report to `./reports/vault-sync-<timestamp>.json`. Secret values are never included in the report.

**Expected result:**
```
Vault Sync Complete
  Secrets scanned: 42
  Secrets present: 38
  Secrets added:    4
  Secrets missing:  0
All required secrets are present in the vault.
```

---

### Step 6 — Package Pull

```bash
./slingshot package-pull
```

**What this does:** Copies container images from the Slingshot source registry to your target registry (ACR, ECR, GCR, or other OCI-compatible registry). This is required so that the Kubernetes cluster can pull images from your approved private registry.

**Configuration (set during `config init`):**
- Source registry: Slingshot delivery registry
- Target registry: Your environment's private registry
- Selected deployment groups: Which application groups to pull images for
- Parallel workers: Default 3
- Timeout per image: Default 10 minutes
- Retry attempts: Default 3

**What you will see:**
- Real-time progress with image count and ETA
- Per-image status (copied / skipped / failed)
- Summary report at the end

**Copy tool selection:**
The installer automatically selects the best available tool: `crane` → `skopeo` → `docker` → `podman`. No action required from you unless none are installed (in which case run `setup-tools` first).

**Air-gapped environments:**
If the deployment machine has no internet access, images must be pre-staged. See the release deployment values guide for the air-gapped image transfer procedure.

**Expected result:**
```
Package Pull Complete
  Images processed: 87
  Images copied:    85
  Images skipped:    2  (already exist in target registry)
  Images failed:     0
All images successfully synced to target registry.
```

---

### Step 7 — Install App

```bash
./slingshot install-app
```

**What this does:** Deploys all Slingshot application groups to the Kubernetes cluster using Helm. This is the main deployment step.

> **Warning:** This command deploys to the configured namespace. Verify the namespace and cluster context before proceeding. Running against the wrong cluster or namespace can cause unintended service disruption.

**What happens inside install-app:**

1. **Select application groups** — Choose which groups to deploy (or deploy all in order)
2. **Configure Helm charts source** — Tarball, Git repo, or local directory
3. **Configure deployment scripts** — Pre/post deployment automation scripts
4. **Configure Docker images source** — Registry or tarball
5. **Execute pre-deployment scripts** — Database setup, configuration seeding
6. **Deploy applications via Helm** — Groups deployed in dependency order
7. **Execute post-deployment scripts** — Data migration, post-configuration
8. **Health checks** — Verify each group is running correctly

**Deployment group order (fixed, dependency-enforced):**

| Order | Group | Description |
|---|---|---|
| 0 | `common-config` | Shared ConfigMap — required by all apps |
| 1 | `common-secrets` | Shared Secrets — required by all apps |
| 2 | `infrastructure-components` | RabbitMQ, Milvus, other infra services |
| 3 | `enterprise-apis` | Core enterprise API services |
| 4 | `portal-components` | Portal frontend and related services |
| 5 | `core-nexa-components` | Core Nexa platform services |
| 6 | `backlog-ai-components` | Backlog AI services |
| 7 | `aipp-services` | AIPP services |
| 8 | `app-modernization-suite` | App modernization tools |
| 9 | `agent-builder` | Agent builder services |
| 10 | `aism-services` | AISM services |

**Helm charts source prompt:**

```
Select Helm charts source:
  > Tarball / zip archive     ← use for release artifacts zip
    Git repository
    Local directory
```

Point to the Helm charts zip from your release package (e.g., `helm-charts-R2.2.0.zip`). The installer extracts it to `.slingshot/helm-charts/`.

**Deployment scripts source prompt:**

```
Select deployment scripts source:
  > Tarball / zip archive     ← use for release artifacts zip
    Git repository
    Local directory
```

Point to the deployment scripts zip from your release package (e.g., `deployment-scripts-R2.2.0.zip`). The installer extracts it to `.slingshot/deployment-scripts/`.

**Pre-deployment script examples (run automatically):**
- Database schema creation
- Initial data seeding
- Configuration server population

**Post-deployment script examples (run automatically):**
- Prompt migration
- Portal configuration
- Admin user creation

**Each group prompts before deployment:**
```
Proceed with deploying <group-name>?
  > Yes, deploy this group
    No, skip this group
    Cancel deployment
```

**Expected result:**
```
Deployment Summary
  ✓ common-config             Deployed (ConfigMap)
  ✓ common-secrets            Deployed (Secret)
  ✓ infrastructure-components Deployed (4/4 pods running)
  ✓ enterprise-apis           Deployed (8/8 pods running)
  ✓ portal-components         Deployed (3/3 pods running)
  ...
All groups deployed successfully.
Run 'slingshot validate-app' to confirm health.
```

---

### Step 8 — Validate App

```bash
./slingshot validate-app
```

**What this does:** Runs post-deployment health checks across all deployed application groups to confirm they are running and healthy.

**Checks performed:**

| Check | What is validated |
|---|---|
| Helm release status | All releases in `deployed` state |
| Pod health | All pods Running/Completed, readiness probes passing |
| Service endpoints | Services have endpoints registered |
| Ingress / Load Balancer | External IPs or hostnames assigned |
| Database connectivity | Application can connect to MongoDB / PostgreSQL |
| Application health endpoints | HTTP health checks return 200 |
| Inter-service communication | Services can reach their dependencies |

**Expected result:**
```
Application Validation: PASSED
  ✓ Helm releases     (11/11 deployed)
  ✓ Pod health        (87/87 running)
  ✓ Services          (all endpoints ready)
  ✓ Ingress           (hostname assigned)
  ✓ Health endpoints  (all returning 200)
Slingshot is ready.
```

---

## 6. Upgrade Runbook

Run these commands **in order** for every version upgrade. The upgrade flow preserves existing data and configuration.

```
Step 1:  config init       — Re-initialize config for new release version
Step 2:  setup-tools       — Verify / update CLI tools
Step 3:  vault-sync        — Verify secrets, add any new release-required secrets
Step 4:  package-pull      — Pull new version images to target registry
Step 5:  upgrade-app       — Execute 3-phase upgrade
Step 6:  validate-app      — Validate upgraded deployment
```

---

### Step 1 — Re-initialize Configuration

```bash
./slingshot config init
```

Re-run `config init` at the start of every upgrade to update the configuration with the new release version, new artifact paths (new zip files), and any changed values from the release deployment values guide.

The previous configuration is backed up automatically before being updated.

**Key values to update for every upgrade:**

| Prompt | Action |
|---|---|
| Slingshot version | Select or enter the new release version (e.g., `2.2.0`) |
| Helm charts tarball path | Update to point to new release Helm zip |
| Deployment scripts tarball path | Update to point to new release scripts zip |
| Installation type | Confirm "Upgrade" is selected |

---

### Step 2 — Setup Tools

```bash
./slingshot setup-tools
```

Run to verify tools are still present and version-compatible. Upgrades to new versions of `helm`, `kubectl`, or `terraform` may be required by the new release. Refer to the release deployment values guide for minimum version requirements.

---

### Step 3 — Vault Sync

```bash
./slingshot vault-sync
```

New releases may introduce new required secrets. Run vault-sync to scan the new Helm charts and identify any secrets that need to be added before deployment.

Use the release deployment values guide to identify and provide values for any new secrets.

---

### Step 4 — Package Pull

```bash
./slingshot package-pull
```

Pull the new release images to your target registry. The installer only copies images that do not already exist in the target registry (by digest), so previously synced images are not re-transferred.

---

### Step 5 — Upgrade App

```bash
./slingshot upgrade-app
```

**What this does:** Performs an in-place upgrade across three sequential phases. Each phase must complete before the next begins.

> **Warning:** The upgrade modifies live application deployments. Ensure database backups are complete and a rollback plan is confirmed before starting Phase 2.

#### Phase 1 — Pre-Deployment Activities

**Purpose:** Prepare the environment before any new code is deployed.

**Typical Phase 1 activities run by deployment scripts:**
- Database backups (MongoDB, PostgreSQL)
- Secrets backup
- Configuration server updates
- Portal pre-migration scripts
- Backlog pre-migration scripts

**Artifact prompt — deployment scripts:**

When Phase 1 begins, the installer checks if deployment scripts are already extracted locally:

```
Phase 1 - extract deployment scripts?
  > ✓ Continue (extract deployment scripts)    ← recommended for first run of a release
    ⏭  Skip (preserve local changes)           ← use if you have intentional local edits
```

| Choice | Effect |
|---|---|
| **Continue** | Extracts fresh copy from the release zip, overwrites existing `.slingshot/deployment-scripts/` |
| **Skip** | Uses the existing `.slingshot/deployment-scripts/` content without re-extracting |

**Recommendation:** Always choose **Continue** for the first run of a new release. Choose **Skip** only if you have intentionally modified the local scripts.

If the configured zip path is missing or invalid, the installer re-prompts:
```
⚠️  Deployment scripts tarball not found: /path/to/old-scripts.zip
Deployment scripts tarball/zip path: [enter new path]
```

Enter the correct path to the new release scripts zip.

**Expected Phase 1 result:**
```
Phase 1 Complete — Pre-deployment activities finished.
  ✓ Database backup completed
  ✓ Secrets backed up
  ✓ Configuration server updated
Ready for Phase 2.
```

---

#### Phase 2 — Deployment Activities

**Purpose:** Deploy the new Helm charts and upgrade application components.

**Artifact prompt — Helm charts:**

```
Phase 2 - extract Helm charts?
  > ✓ Continue (extract Helm charts)    ← recommended for first run of a release
    ⏭  Skip (preserve local changes)    ← use if you have intentional local edits
```

Behavior is identical to the deployment scripts prompt above.

**What happens during deployment:**

1. Installer reads current values from the live cluster (Kubernetes ConfigMaps and Secrets) to pre-populate upgrade parameters — no manual re-entry needed for values already in the cluster.
2. Installer runs **pre-upgrade hooks** (Helm pre-upgrade hooks from the chart).
3. Each application group is upgraded in order using `helm upgrade`.
4. Installer runs **post-upgrade hooks**.

**Values automatically fetched from the cluster:**

| Value | Source (Kubernetes resource) |
|---|---|
| Domain | `common-configmap-cm` → `DOMAIN` |
| PostgreSQL username | `common-configmap-cm` → `SS_POSTGRES__USERNAME` |
| PostgreSQL host | `common-configmap-cm` → `SS_POSTGRES__HOST` |
| MongoDB connection string | `common-secrets` → `SS_MONGODB__URL` |
| Default admin user | `api-oauth` secret → `SLINGSHOT-API-OAUTH-COREAI-ADMINS-EMAIL` |
| Portal service client ID | `common-secrets` → `SS_SVC_APP__PORTAL_ID` |
| Portal service client secret | `common-secrets` → `SS_SVC_APP__PORTAL_SECRET` |
| Enterprise Industry ID | `common-configmap-cm` → `ENT__INDUSTRY_ID_SLINGSHOT` |

> If any value cannot be fetched from the cluster, the installer prompts you to enter it manually.

**Expected Phase 2 result:**
```
Phase 2 Complete — Deployment finished.
  ✓ Helm charts extracted
  ✓ Pre-upgrade hooks executed
  ✓ enterprise-apis     upgraded to v2.2.0
  ✓ portal-components   upgraded to v2.2.0
  ✓ core-nexa-components upgraded to v2.2.0
  ... (all groups)
  ✓ Post-upgrade hooks executed
Ready for Phase 3.
```

---

#### Phase 3 — Post-Deployment Activities

**Purpose:** Run post-deployment migrations, configuration updates, and final cleanup.

**Typical Phase 3 activities:**
- Prompt migration scripts
- Portal post-deployment scripts
- Backlog post-migration scripts
- Service restarts
- Health verifications

**Artifact prompt:** Same Continue / Skip pattern as Phase 1.

**Expected Phase 3 result:**
```
Phase 3 Complete — Post-deployment activities finished.
  ✓ Prompt migration completed
  ✓ Portal scripts executed
  ✓ Service health checks passed
Upgrade complete. Run 'slingshot validate-app' to confirm.
```

---

#### Rollback

If any phase fails and the system is in an unhealthy state, the installer supports rollback via Helm release history.

**Manual rollback:**
```bash
# Check current Helm release history
helm history <release-name> -n slingshot-ns

# Rollback to previous version
helm rollback <release-name> <revision> -n slingshot-ns
```

> Contact the Slingshot support team before performing a rollback in a production environment.

---

### Step 6 — Validate App (Upgrade)

```bash
./slingshot validate-app
```

Run the same validation as the fresh install flow. All checks must pass before the upgrade is considered complete.

---

## 7. Artifact Handling

### 7.1 Supported archive formats

The installer accepts these archive formats for all artifact zips/tarballs:

- `.zip`
- `.tar.gz`
- `.tgz`
- `.tar`

### 7.2 Extraction locations

| Artifact | Extracted to |
|---|---|
| Terraform artifacts | `.slingshot/terraform/` |
| Helm charts | `.slingshot/helm-charts/` |
| Deployment scripts | `.slingshot/deployment-scripts/` |

The installer resolves the correct subfolder automatically by:
1. Matching the archive filename (minus extension) as a subdirectory name
2. If only one non-hidden, non-`__MACOSX` directory exists, using that
3. Falling back to the root of the extract path

### 7.3 Continue vs Skip — decision table

This prompt appears when artifacts are already present locally (from a previous run):

```
<Phase> - extract <artifact>?
  > ✓ Continue (extract <artifact>)
    ⏭  Skip (preserve local changes)
```

| Situation | Recommended choice |
|---|---|
| First run of a new release | **Continue** — always re-extract from the new release zip |
| Re-running a failed step in the same release | **Continue** — ensure a clean state |
| Re-running after intentionally editing local scripts | **Skip** — preserve your changes |
| Not sure | **Continue** — safer default; re-extraction is non-destructive to cluster state |

### 7.4 Missing archive path behavior

If the configured archive path does not exist (e.g., the path still points to a previous release zip), the installer warns and re-prompts:

```
⚠️  Deployment scripts tarball not found: /releases/R2.1.2/deployment-scripts-R2.1.2.zip
Deployment scripts tarball/zip path: [enter new path]
```

Enter the correct path to the current release zip. The installer will not proceed until a valid file is provided.

### 7.5 Reuse behavior

When **Skip** is selected, the installer derives the existing scripts directory from the configuration without re-extracting. The derivation priority is:

1. `artifacts.upgrade.scriptsDir` saved from the previous phase run
2. Derived from `artifacts.deploymentScripts.path` (archive filename → subdirectory)
3. Single subdirectory under `.slingshot/deployment-scripts/`

---

## 8. Prompt Reference

This section describes every major prompt across all commands, what value is expected, and how the default is resolved.

### 8.1 `config init` prompts

| Prompt | Purpose | Accepted input | Default source | Typical value |
|---|---|---|---|---|
| Installation type | Determines fresh install vs upgrade flow | Select from list | None | `Fresh Install` or `Upgrade` |
| Cloud provider | Target cloud environment | Select: Azure / AWS / GCP / OpenShift | Detected from existing config | Match target environment |
| Slingshot version | Version to deploy | Select from list or enter custom (e.g., `2.2.0`) | Recommended version from list | As specified in release guide |
| Azure Subscription ID | Azure subscription for cluster | GUID string | Existing config value | From platform team |
| Tenant ID | Azure Entra tenant | GUID string | Existing config value | From platform team |
| Resource Group | RG containing the AKS cluster | String | Existing config value | From platform team |
| AKS Cluster Name | Name of target AKS cluster | String | Existing config value | From platform team |
| Kubernetes namespace | Namespace to deploy into | String | `slingshot-ns` (hard-coded) | `slingshot-ns` |
| Helm charts source type | How Helm charts are provided | Select: tarball / Git / local | Existing config value | `tarball` for release artifacts |
| Helm charts tarball path | Path to Helm charts zip | File path | Existing config path | `/releases/R2.2.0/helm-charts-R2.2.0.zip` |
| Deployment scripts source type | How deployment scripts are provided | Select: tarball / Git / local | Existing config value | `tarball` for release artifacts |
| Deployment scripts tarball path | Path to deployment scripts zip | File path | Existing config path | `/releases/R2.2.0/deployment-scripts-R2.2.0.zip` |
| Source registry URL | Registry where release images are hosted | URL string | Existing config value | From release guide |
| Target registry URL | Your private registry URL | URL string | Existing config value | From platform team |
| Parallel workers | Number of concurrent image transfers | Integer | `3` (hard-coded) | Keep default |
| Timeout per image | Max time per image transfer | Duration string | `10m` (hard-coded) | Keep default |
| Retry attempts | Retries on transfer failure | Integer | `3` (hard-coded) | Keep default |
| Terraform source type | How Terraform code is provided | Select: tarball / Git | Existing config value | `tarball` for release artifacts |
| Terraform tarball path | Path to Terraform artifacts zip | File path | Existing config path | `/releases/R2.2.0/terraform-R2.2.0.zip` |
| Terraform root folder | Top-level folder in Terraform archive | String | Existing config value | `tf` |
| Terraform subfolder | Subfolder for target environment | String | Existing config value | From release guide (e.g., `ps`) |

### 8.2 `vault-sync` prompts

| Prompt | Purpose | Accepted input | Default source | Typical value |
|---|---|---|---|---|
| Vault type | Target secrets store | Select: Azure Key Vault / AWS / GCP / HashiCorp | Existing config | Match target environment |
| Azure authentication method | Auth mode for vault operations | Select from list (CLI / Service Principal / Managed Identity) | Existing config | Align with security policy |
| Tenant ID | Entra tenant ID | GUID string | Existing config | From platform team |
| Client ID | Service principal / app registration ID | GUID string | Existing config | From platform team |
| Client Secret | Service principal secret | Masked input | Never stored | Enter securely |
| Key Vault URL | Azure Key Vault endpoint | HTTPS URL | Existing config | From platform team |
| Secret name | Name for a specific secret being uploaded | String | None | Per release guide |
| Secret value | Value for secret being uploaded | Masked input | Never stored | Per release guide |

### 8.3 `package-pull` prompts

| Prompt | Purpose | Accepted input | Default source | Typical value |
|---|---|---|---|---|
| Source registry authentication | Auth method for source registry | Select: CLI / token / credential file | Existing config | Use approved enterprise method |
| Target registry authentication | Auth method for target registry | Select: CLI / token / credential file | Existing config | Use approved enterprise method |
| Deployment groups to pull | Which app groups to sync images for | Multi-select from list | All groups | Select all unless partial pull needed |
| Continue on error | Whether to continue if an image fails | Yes / No | `Yes` (hard-coded) | Keep default |

### 8.4 `install-app` / `upgrade-app` artifact prompts

| Prompt | Purpose | Accepted input | Default source | Typical value |
|---|---|---|---|---|
| Helm charts tarball/zip path | Location of Helm charts archive | File path | Last saved config path | `/releases/R2.2.0/helm-charts-R2.2.0.zip` |
| Deployment scripts tarball/zip path | Location of scripts archive | File path | Last saved config path | `/releases/R2.2.0/deployment-scripts-R2.2.0.zip` |
| Extract deployment scripts? | Continue (re-extract) or Skip | Select from menu | None (shown only if scripts exist locally) | **Continue** for first run |
| Extract Helm charts? | Continue (re-extract) or Skip | Select from menu | None (shown only if charts exist locally) | **Continue** for first run |
| Proceed with deploying `<group>`? | Confirm deployment of each group | Select: Yes / Skip / Cancel | None | Yes for all groups |

### 8.5 `upgrade-app` phase-specific prompts

| Prompt | Purpose | Accepted input | Default source | Typical value |
|---|---|---|---|---|
| Select upgrade phase | Choose Phase 1, 2, or 3 | Select from list | None | Run in order: 1 → 2 → 3 |
| Execute pre-deployment hook: `<name>`? | Confirm each pre-deployment hook | Continue / Skip | None | Continue unless advised otherwise |
| Execute post-deployment hook: `<name>`? | Confirm each post-deployment hook | Continue / Skip | None | Continue unless advised otherwise |
| `<field>` value | Manual entry if cluster fetch fails | String | Fetched from cluster | From cluster or release guide |

---

## 9. Release Values Mapping

Use this table with your release deployment values guide to identify where each value is entered in the installer.

| Value in release guide | Entered at | Command / Prompt | Required? | If not provided | Example format |
|---|---|---|---|---|---|
| Target release version | `config init` | Slingshot version selector | Yes | Deployment targets wrong version | `2.2.0` |
| Helm charts archive path | `config init` | Helm charts tarball path | Yes | Installer re-prompts until valid | `/releases/R2.2.0/helm-charts-R2.2.0.zip` |
| Deployment scripts archive path | `config init` | Deployment scripts tarball path | Yes (if scripts used) | Script steps fail | `/releases/R2.2.0/deployment-scripts-R2.2.0.zip` |
| Terraform archive path | `config init` | Terraform tarball path | Conditional (infra provisioning only) | Infra step cannot proceed | `/releases/R2.2.0/terraform-R2.2.0.zip` |
| Terraform root folder | `config init` | Terraform root folder | Conditional | Installer uses wrong directory | `tf` |
| Terraform subfolder | `config init` | Terraform subfolder | Conditional | Installer uses wrong environment | `ps` |
| Azure subscription ID | `config init` | Azure Subscription ID | Azure only | Context / AKS errors | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| Azure tenant ID | `config init` | Tenant ID | Azure only | Auth failures | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| Resource group | `config init` | Resource Group | Azure only | Lookup / auth failures | `rg-prod-aks-01` |
| Cluster name | `config init` | AKS / EKS / GKE Cluster Name | Yes | Kube context setup failure | `aks-prod-slingshot-01` |
| Kubernetes namespace | `config init` | Kubernetes namespace | Yes | Resources deployed to wrong namespace | `slingshot-ns` |
| Source registry URL | `config init` | Source registry URL | Yes (for package-pull) | Image sync fails | `slingshot.azurecr.io` |
| Target registry URL | `config init` | Target registry URL | Yes (for package-pull) | Images not available to cluster | `clientacr.azurecr.io` |
| Key vault URL | `vault-sync` | Key Vault URL | Yes (for vault-sync) | Secrets cannot be read/written | `https://client-kv.vault.azure.net` |
| New release secrets | `vault-sync` | Secret value prompts | Yes (for new secrets) | Deployment fails at runtime | Per release guide |
| Service principal client ID | `vault-sync` / `config init` | Client ID | Conditional | Auth failures | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| Service principal secret | `vault-sync` session | Client Secret (masked) | Conditional | Auth failures | Masked — enter securely |

---

## 10. Validation Checklist

### 10.1 Before starting any deployment

- [ ] Release artifacts downloaded and verified (checksums match)
- [ ] Installer binary executable and version confirmed
- [ ] Cloud CLI login active and pointing to correct subscription/account
- [ ] Kubernetes context confirmed: `kubectl config current-context`
- [ ] Cluster accessibility confirmed: `kubectl cluster-info`
- [ ] Target namespace exists or will be created: `kubectl get namespace slingshot-ns`
- [ ] Database endpoints reachable from deployment machine
- [ ] Key vault / secrets manager access permissions verified
- [ ] Release deployment values guide available

### 10.2 After `config init`

- [ ] Config file exists: `.slingshot/config/ss-installer-config-<cloud>.json`
- [ ] Installation type is set correctly (Fresh Install or Upgrade)
- [ ] Version number matches the release being deployed
- [ ] Artifact paths point to the current release files (not previous release)
- [ ] Registry URLs are correct for the environment

### 10.3 After `vault-sync`

- [ ] Vault sync report shows 0 missing secrets
- [ ] All new release-required secrets are present
- [ ] Vault sync audit log saved

### 10.4 After `package-pull`

- [ ] All images show as copied or already present
- [ ] 0 failed images
- [ ] Images are accessible from the Kubernetes cluster (spot check: `kubectl run test --image=<target-registry>/image:tag --rm -it`)

### 10.5 After `install-app` / `upgrade-app`

- [ ] All Helm releases in `deployed` state: `helm list -n slingshot-ns`
- [ ] All pods Running/Completed: `kubectl get pods -n slingshot-ns`
- [ ] No pods in CrashLoopBackOff or Error state
- [ ] Ingress / Load Balancer has external IP or hostname
- [ ] `validate-app` passes all checks

### 10.6 Go-live sign-off

- [ ] Functional smoke test completed (login, basic navigation)
- [ ] No stale test archives used — all artifact paths point to current release
- [ ] Audit logs preserved (`./reports/` directory)
- [ ] Screenshots of key steps captured (mask all secrets)
- [ ] Rollback contacts and procedure confirmed with team

---

## 11. Common Issues and Troubleshooting

### Run diagnostics first

```bash
./slingshot troubleshoot
```

This runs a full multi-cloud diagnostic and reports color-coded results (green/yellow/red) with specific remediation suggestions.

### Common issues

| Symptom | Likely cause | Resolution |
|---|---|---|
| `Configuration required` error at startup | No config file found | Run `./slingshot config init` first |
| Old default values shown in prompts | Previous release paths still in config | Run `config init`, update all paths to current release |
| Archive not found during extraction | Path points to wrong or missing zip | Re-enter correct path when prompted; verify file exists |
| `Continue` selected but scripts not refreshed | Archive path still points to old release zip | Update archive path in config before running |
| `crane` missing | Tool not installed | Run `./slingshot setup-tools` to install, or use docker fallback |
| Registry authentication errors | Wrong auth method or expired credentials | Re-authenticate: `az acr login`, `aws ecr get-login-password`, or `gcloud auth configure-docker` |
| Phase script failures | Wrong or missing deployment scripts archive | Re-extract scripts by selecting **Continue** and providing correct zip path |
| Pod in `CrashLoopBackOff` | Application misconfiguration or missing secret | Check pod logs: `kubectl logs <pod> -n slingshot-ns`; run `vault-sync` to verify secrets |
| Helm upgrade fails — resource conflict | Previous release in failed/pending state | Run `helm rollback <release> -n slingshot-ns` then retry |
| `kubectl` context wrong cluster | Stale kubeconfig | Run `kubectl config use-context <correct-context>` |
| Key Vault access denied | Missing role assignment | Assign `Key Vault Secrets Officer` role to the identity being used |
| Node not ready | Cluster node issue | Check node status: `kubectl get nodes`; refer to cloud console |
| Vault sync shows missing secrets | New release added secrets | Check release deployment values guide for new secrets, enter them via vault-sync |
| `No configuration file found` after moving directories | Config path is absolute | Re-run `config init` from the correct working directory |

### Viewing logs

```bash
# View pod logs
kubectl logs <pod-name> -n slingshot-ns

# View logs for all pods in a deployment
kubectl logs deployment/<deployment-name> -n slingshot-ns --all-containers

# Describe a pod for events and error details
kubectl describe pod <pod-name> -n slingshot-ns

# List all events in namespace (sorted)
kubectl get events -n slingshot-ns --sort-by='.lastTimestamp'
```

### Resuming an interrupted operation

If an operation was interrupted (network drop, Ctrl+C), you can resume from the last checkpoint:

```bash
./slingshot install-app --resume
./slingshot upgrade-app --resume
```

The installer reads the state file at `.slingshot/state.json` and resumes from the last successfully completed phase.

---

## 12. Operational Best Practices

1. **Version-isolate all release artifacts.** Store each release's zips in a dedicated folder (e.g., `/releases/R2.2.0/`). Never overwrite artifacts from a previous release.

2. **Always choose Continue on first run of a release.** When prompted to extract artifacts, select **Continue** to ensure you are working with the release-provided files, not leftover local changes.

3. **Run `vault-sync` before every deployment.** Secrets drift can cause hard-to-diagnose failures during deployment. Confirm 0 missing secrets before proceeding.

4. **Verify cloud CLI session before starting.** Cloud CLI sessions expire. If a session expires mid-deployment, operations will fail. Re-authenticate and resume.

5. **Keep the deployment machine network-stable.** Long-running operations (package-pull, install-app) can be interrupted by network drops. Run from a stable network or a jump server/bastion host co-located with the cluster.

6. **Never commit config files containing secrets.** The config file may contain registry URLs and paths that hint at internal infrastructure. Treat it as sensitive. Secrets are never written to config by design, but the file should still be handled carefully.

7. **Preserve audit logs.** The `./reports/` directory contains vault-sync audit logs. Keep these for compliance and troubleshooting.

8. **Capture screenshots for audit and support.** During a delivery engagement, capture screenshots of:
   - Main menu
   - `setup-tools` summary
   - `package-pull` source/auth selection and completion summary
   - Upgrade phase selection and completion
   - Continue vs Skip extraction prompt
   - `validate-app` result summary
   Mask all secret values before sharing.

9. **Do not include secrets in screenshots or shared notes.** The installer masks secrets in all output. Ensure your screenshots do not capture terminal history showing unmasked values.

10. **Test rollback before go-live.** For production upgrades, confirm the rollback procedure and Helm release history before starting Phase 2.

---

## 13. Appendix: Command Quick Reference

### All commands

```bash
# Show all commands and options
./slingshot --help

# Run interactive main menu
./slingshot

# Configuration management
./slingshot config init                    # Create new configuration (wizard)
./slingshot config update                  # Update existing configuration
./slingshot config validate                # Validate configuration file
./slingshot config generate-template       # Generate cloud-specific template

# Tool management
./slingshot setup-tools                    # Install and verify required CLI tools

# Infrastructure
./slingshot provision-infra                # Provision Kubernetes cluster via Terraform
./slingshot validate-infra                 # Validate infrastructure readiness

# Secrets
./slingshot vault-sync                     # Scan secrets and sync with vault

# Images
./slingshot package-pull                   # Sync container images to target registry

# Deployment (Fresh Install)
./slingshot install-app                    # Deploy applications to new cluster
./slingshot install-app --dry-run          # Preview deployment without changes
./slingshot install-app --resume           # Resume interrupted deployment
./slingshot install-app --skip-health-check  # Skip post-deploy health checks

# Deployment (Upgrade)
./slingshot upgrade-app                    # Upgrade to new version (3-phase)
./slingshot upgrade-app --resume           # Resume interrupted upgrade

# Validation
./slingshot validate-app                   # Validate application health

# Azure-specific
./slingshot validate-aks                   # Validate AKS cluster specifically

# Diagnostics
./slingshot troubleshoot                   # Run multi-cloud diagnostic checks

# Help
./slingshot help                           # Interactive help menu
```

### Global flags

| Flag | Description | Example |
|---|---|---|
| `--config <path>` | Use custom config file path | `--config /path/to/my-config.json` |
| `--dry-run` | Preview changes without applying | `./slingshot install-app --dry-run` |
| `--resume` | Resume from last checkpoint | `./slingshot upgrade-app --resume` |
| `--verbose` / `-v` | Enable verbose logging | `./slingshot install-app -v` |
| `--version` | Show installer version | `./slingshot --version` |
| `--help` / `-h` | Show help for any command | `./slingshot upgrade-app --help` |

### Useful kubectl commands for verification

```bash
# Check all pods in Slingshot namespace
kubectl get pods -n slingshot-ns

# Check Helm releases
helm list -n slingshot-ns

# Check Helm release history (for rollback)
helm history <release-name> -n slingshot-ns

# Check services and endpoints
kubectl get svc -n slingshot-ns

# Check ingress
kubectl get ingress -n slingshot-ns

# Check ConfigMaps
kubectl get configmap -n slingshot-ns

# Check Secrets (names only)
kubectl get secrets -n slingshot-ns

# View recent cluster events
kubectl get events -n slingshot-ns --sort-by='.lastTimestamp' | tail -20
```

### Environment variables reference

The installer supports environment variable substitution in all prompted values. Use the format `${VARIABLE_NAME}` when entering paths or credentials in prompts.

| Example variable | Usage |
|---|---|
| `${AZURE_CLIENT_SECRET}` | Service principal secret |
| `${REGISTRY_TOKEN}` | Container registry token |
| `${GIT_TOKEN}` | Git personal access token |
| `${HELM_CHARTS_PATH}` | Path to Helm charts archive |

---

*For issues not covered in this guide, run `./slingshot troubleshoot` and contact the Slingshot delivery team with the diagnostic output.*
