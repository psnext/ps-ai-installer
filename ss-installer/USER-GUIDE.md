# Slingshot Installer - Enterprise User Guide

> **Production-Ready Deployment Guide for Kubernetes Applications on Multi-Cloud Platforms**

**Version**: 2.0  
**Last Updated**: February 5, 2026  
**Supported Platforms**: Azure AKS, AWS EKS, GCP GKE, OpenShift  
**Target Users**: DevOps Engineers, Platform Engineers, Site Reliability Engineers

---

## 📑 Table of Contents

1. [Introduction](#1-introduction)
2. [Architecture Overview](#2-architecture-overview)
3. [Prerequisites](#3-prerequisites)
4. [Getting Started](#4-getting-started)
5. [What Will Be Deployed?](#5-what-will-be-deployed)
6. [Complete Workflow](#6-complete-workflow)
7. [Application Groups & Deployment Order](#7-application-groups--deployment-order)
8. [Pre & Post-Deployment Hooks](#8-pre--post-deployment-hooks)
9. [Troubleshooting](#9-troubleshooting)
10. [Security & Compliance](#10-security--compliance)
11. [Best Practices](#11-best-practices)
12. [Reference](#12-reference)

---

## 1. Introduction 🚀

### 1.1 What is Slingshot Installer?

Slingshot Installer is an enterprise-grade, production-ready CLI orchestration tool that enables you to deploy and manage the complete **Bodhi Agent Framework**, **Slingshot Portal**, and **Agent Studio** platform across Kubernetes environments.

> 💡 **For End Users**: This guide helps you deploy the complete AI platform using the pre-built `slingshot` binary. No programming or compilation required!

**What You Will Deploy:**
- 🔌 **Bodhi Agent Framework**: Core authentication, LLM APIs, and agent services
- 🌐 **Slingshot Portal**: User interface and management dashboard
- 🎯 **Agent Studio**: AI agent creation and management tools
- 🏗️ **Infrastructure**: RabbitMQ, databases, and supporting services

**Core Capabilities:**
- 🎯 Intelligent orchestration with dependency-aware deployment
- ☁️ Multi-cloud support (Azure AKS, AWS EKS, GCP GKE, OpenShift)
- 🔧 Automated tool installation (kubectl, helm, terraform, cloud CLIs)
- 🪝 Pre/post-deployment hooks for database setup and configuration
- ✅ Comprehensive validation framework
- 🔒 Secure credential management (in-memory only)
- 🩺 Built-in health diagnostics and troubleshooting
- 🛡️ Production-ready error handling and recovery

> ⚡ **Quick Start**: Receive binary → Make executable → Configure → Deploy → Validate. Simple as that!

### 1.2 Key Features ⭐

| Feature | Description | Benefit |
|---------|-------------|---------|
| **Ready-to-Use Binary** | Pre-built executable, no compilation needed | ⚡ Start deploying immediately |
| **One-Click Deployment** | Single command deploys entire platform | 🚀 Simple and fast |
| **Automated Tool Setup** | Auto-installs kubectl, helm, terraform | 🛠️ No manual configuration |
| **Smart Configuration** | Interactive wizard with defaults | 📝 Easy to configure |
| **Complete Platform** | Bodhi + Slingshot + Agent Studio | 🎯 Everything you need |
| **Health Diagnostics** | Built-in troubleshooting command | 🩺 Easy problem resolution |
| **Multi-Cloud Ready** | Works on Azure, AWS, GCP, OpenShift | ☁️ Deploy anywhere |
| **Production-Grade** | Enterprise security and reliability | 🛡️ Mission-critical ready |

### 1.3 Who Should Use This Guide 👥

**✅ This guide is for you if:**
- 🎯 You want to deploy Bodhi Agent Framework for your organization
- 🌐 You need to set up Slingshot Portal for your team
- 🤖 You want to enable Agent Studio capabilities
- ☁️ You have access to a Kubernetes cluster (Azure AKS, AWS EKS, GCP GKE)
- 🗄️ You have PostgreSQL and MongoDB databases ready
- 📦 You received the `slingshot` binary from your vendor

**❌ This guide is NOT for:**
- 👨‍💻 Developing or modifying the Slingshot installer itself
- 🔨 Building the binary from source code
- 🧪 Contributing to the Slingshot codebase

> 💡 **Expected Skill Level**: Basic knowledge of Kubernetes, cloud platforms, and command-line tools. No programming required!

---

## 2. Architecture Overview 🏛️

### 2.1 What Gets Deployed 📊

> ⚠️ **Important**: Slingshot deploys applications in a specific order. The installer handles all dependencies automatically.

When you deploy using Slingshot, **10 deployment groups** are installed in sequence:

```
1. 📋 Common Configuration     → Shared settings for all applications
2. 🔐 Common Secrets          → Secure credentials and keys
3. 🏗️ Infrastructure          → Messaging (RabbitMQ) and monitoring
4. 🔌 Bodhi Agent Framework   → Core AI and authentication services ⭐
5. 🌐 Slingshot Portal        → User interface and management ⭐
6. 🧠 NEXA Intelligence       → AI execution and orchestration
7. 📝 Backlog AI Platform     → Project management features
8. 🤖 AIPP Services           → Additional integration services
9. 🔄 App Modernization       → Code transformation tools
10. 🎯 Agent Studio           → AI agent creation tools ⭐
```

**⭐ Essential Groups:**
- **Groups 1-5**: Required for basic Bodhi + Slingshot functionality
- **Groups 6-10**: Optional features (can be enabled/disabled)

> 💡 **Typical Deployment**: Most users deploy Groups 1-5 plus Group 10 (Agent Studio)

### 2.2 Deployment Timeline ⏱️

**Expected Duration for Full Deployment:**

| Phase | Duration | What Happens |
|-------|----------|--------------|
| 📋 **Configuration** | 5-10 min | Interactive setup wizard |
| 🛠️ **Tool Setup** | 5-10 min | Install kubectl, helm, terraform |
| 📦 **Package Pull** | 10-30 min | Download container images |
| 🏗️ **Infrastructure** | 15-30 min | Provision Kubernetes cluster |
| ✅ **Validation** | 2-5 min | Verify infrastructure ready |
| 🚀 **Application Install** | 30-60 min | Deploy all 10 groups |
| ✔️ **Final Validation** | 5-10 min | Health checks and verification |
| **TOTAL** | **1.5-3 hours** | Complete end-to-end deployment |

> ⏰ **Plan Accordingly**: Schedule a 3-hour window for your first deployment. Subsequent deployments are faster (skip infrastructure provisioning).

### 2.3 System Architecture 🏗️

**High-Level Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR ORGANIZATION                        │
│                                                             │
│  ┌─────────────┐      ┌──────────────┐      ┌───────────┐ │
│  │   Users     │─────▶│   Slingshot  │─────▶│  Agent    │ │
│  │             │      │   Portal     │      │  Studio   │ │
│  └─────────────┘      └──────────────┘      └───────────┘ │
│         │                     │                     │      │
│         └─────────────────────┼─────────────────────┘      │
│                               │                            │
│                               ▼                            │
│                  ┌─────────────────────────┐              │
│                  │  Bodhi Agent Framework  │              │
│                  │  - Authentication       │              │
│                  │  - LLM APIs            │              │
│                  │  - Agent Services      │              │
│                  └─────────────────────────┘              │
│                               │                            │
└───────────────────────────────┼────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
              ┌─────▼─────┐         ┌──────▼──────┐
              │ PostgreSQL │         │   MongoDB   │
              │ (Metadata) │         │ (Documents) │
              └────────────┘         └─────────────┘
```

> 📝 **Note**: All components run in your Kubernetes cluster. Databases can be managed services (Azure/AWS) or in-cluster.

### 2.4 Automated Setup (Hooks) 🪝

> 💡 **Automation**: The installer automatically sets up databases and configurations. You just provide credentials!

During deployment, Slingshot runs **automated setup scripts** (called "hooks"):

**🌐 Portal Setup (Automated):**
- ✅ Creates MongoDB database collections
- ✅ Sets up default client and project
- ✅ Initializes PostgreSQL schemas
- ✅ Configures service URLs

**🧠 NEXA Setup (Automated):**
- ✅ Configures AI service settings
- ✅ Sets up configuration server

**📝 Backlog AI Setup (Automated):**
- ✅ Creates PostgreSQL database
- ✅ Runs database migrations
- ✅ Sets up authentication

> ⚙️ **What You Do**: Provide database credentials during configuration wizard  
> 🤖 **What Slingshot Does**: Automatically creates all databases, tables, and initial data

**Typical Setup Questions:**
```
🔐 PostgreSQL Password: ********
📝 PostgreSQL Host: your-db-server.postgres.database.azure.com
📝 MongoDB Connection String: mongodb://your-mongo-server:27017
📝 Domain URL: your-company.sapientslingshot.com
📝 Admin Email: admin@your-company.com
```

> 🎉 **Benefit**: No manual database setup required! The installer handles everything.

---

## 3. Prerequisites ✋

> ⚠️ **Production Deployment**: Ensure all prerequisites are met before starting deployment. Missing prerequisites will cause deployment failures.

### 3.1 System Requirements 💻

| Requirement | Minimum | Recommended | Production |
|-------------|---------|-------------|------------|
| **OS** | Linux, macOS, Windows WSL2 | Ubuntu 20.04+ | Ubuntu 22.04 LTS |
| **CPU** | 2 cores | 4+ cores | 8+ cores |
| **Memory** | 4 GB | 8+ GB | 16+ GB |
| **Disk** | 20 GB free | 50+ GB free | 100+ GB free |

> 💡 **Tip**: Production deployments should use the 'Production' column specifications for optimal performance.

### 3.2 Required Access 🔑

> 🔒 **Security Note**: Never commit credentials to version control. Always use environment variables or secret managers.

#### ☁️ Cloud Provider Authentication

**Azure:**
```bash
az login
az account set --subscription "<subscription-id>"
kubectl config use-context <cluster-context>
```

**AWS:**
```bash
aws configure
aws eks update-kubeconfig --name <cluster> --region <region>
```

**GCP:**
```bash
gcloud auth login
gcloud config set project <project-id>
gcloud container clusters get-credentials <cluster>
```

#### 🗄️ Database Connectivity

> ⚠️ **Critical**: Database connectivity is required BEFORE starting deployment. Test connections first!

**🐘 PostgreSQL** (Required for Portal, Backlog AI):
```bash
# Test connectivity
psql -h <host> -p 5432 -U <user> -d <database>
```

> 💡 **Tip**: If connection fails, check firewall rules and ensure your IP is whitelisted.

**🍃 MongoDB** (Required for Bodhi Agent Framework, Portal):
```bash
mongosh "mongodb://<host>:27017"
```

#### Container Registry

**Azure ACR:**
```bash
export ACR_USERNAME="<registry-name>"
export ACR_PASSWORD="<access-token>"
```

**AWS ECR:**
```bash
export AWS_ACCESS_KEY_ID="<key>"
export AWS_SECRET_ACCESS_KEY="<secret>"
```

### 3.3 Pre-Deployment Checklist ✅

> ⚠️ **Production Ready**: Complete ALL checklist items before proceeding. Skipping items will cause failures.

- [ ] ☁️ Cloud CLI authenticated
- [ ] ☸️ Kubernetes cluster accessible
- [ ] 🔐 RBAC permissions granted
- [ ] 🐘 PostgreSQL accessible and tested
- [ ] 🍃 MongoDB accessible and tested
- [ ] 📦 Container registry credentials configured
- [ ] 🌐 Network connectivity verified
- [ ] 💾 20+ GB disk space available (100+ GB for production)
- [ ] 📋 Configuration backup created
- [ ] 👥 Team notified (production deployments)

> 💡 **Best Practice**: Run `kubectl cluster-info` and `kubectl get nodes` to verify cluster access before proceeding.

---

## 4. Getting Started 🚀

> 💡 **Quick Start**: You will receive a pre-built `slingshot` binary. Simply make it executable and start using it!

### 4.1 Receiving the Slingshot Binary 📦

Your deployment package includes:
- ✅ **Pre-built binary**: `slingshot` (for your operating system)
- 📚 **User Guide**: This documentation
- ⚙️ **Configuration templates**: Sample configuration files

> 🎯 **No Compilation Required**: The binary is ready to use immediately.

### 4.2 Making the Binary Executable 🔧

**🌍 Linux/macOS:**
```bash
chmod +x slingshot
```

**🪟 Windows:**
```powershell
# No additional steps needed - run slingshot.exe directly
```

### 4.3 Verify Installation ✅

Test that the binary works:

```bash
./slingshot version
```

**Expected Output:**
```
Slingshot Installer v2.0.0
Build Date: 2026-02-05
Platform: linux/amd64
```

> ✅ **Success**: If you see version information, you're ready to proceed!

### 4.4 Optional: Add to System PATH 📍

For easier access, add `slingshot` to your system PATH:

**🌍 Linux/macOS (System-Wide):**
```bash
sudo mv slingshot /usr/local/bin/
sudo chmod +x /usr/local/bin/slingshot
```

**👤 Linux/macOS (User-Only):**
```bash
mkdir -p ~/bin
mv slingshot ~/bin/
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc
source ~/.bashrc  # or source ~/.zshrc
```

**🪟 Windows:**
```powershell
# Move to a directory in your PATH, for example:
Move-Item slingshot.exe C:\Windows\System32\
```

> 💡 **Benefit**: After adding to PATH, run `slingshot` from any directory without `./`

### 4.5 Quick Help Reference 📖

View available commands:

```bash
slingshot --help
```

**Common Commands:**
```bash
slingshot config init        # Initialize configuration
slingshot setup-tools        # Install required tools (kubectl, helm, etc.)
slingshot install-app        # Deploy applications
slingshot troubleshoot       # Health diagnostics
slingshot --version          # Show version
```

---

## 5. What Will Be Deployed? 🎯

> 📋 **Deployment Overview**: Slingshot will deploy the complete platform including Bodhi Agent Framework, Portal, and Agent Studio.

When you run the deployment, Slingshot will install:

### Core Platform Components:

1. **🔌 Bodhi Agent Framework** (Essential)
   - Authentication API v2
   - LLM API
   - User Profile API
   - Project API v2
   - Configuration Service
   - State API
   - Workflow API
   - Notification API
   - Graph API
   - Embeddings API

2. **🌐 Slingshot Portal** (Essential)
   - Portal Backend Service
   - Portal Frontend UI
   - Authentication Service
   - Database: PostgreSQL + MongoDB

3. **🎯 Agent Studio** (Optional)
   - Agent Builder UI
   - Save API
   - Worker Services

4. **🏗️ Infrastructure Services** (Essential)
   - RabbitMQ (Message Broker)
   - Configuration Management
   - Secrets Management

5. **🧠 NEXA AI Components** (Optional)
   - AI Execution Service
   - AI Orchestration
   - Prompt Library
   - Ingestion Service

6. **📝 Additional Services** (Optional)
   - Backlog AI
   - App Modernization Suite
   - Service Management

> ⏱️ **Total Deployment Time**: 30-60 minutes for complete deployment

---

## 6. Complete Workflow 🔄

> 📋 **Workflow Overview**: Follow steps 6.1 → 6.2 → 6.3 → 6.4 → 6.5 → 6.6 → 6.7 in order for successful deployment.

### 6.1 Configuration Initialization ⚙️

```bash
slingshot config init
```

**🧙 Interactive Wizard:**
1. ☁️ Select cloud provider (Azure/AWS/GCP/OpenShift)
2. 🌍 Choose region
3. 📦 Define namespace
4. 🗄️ Configure container registry
5. 📋 Select application groups
6. 🏷️ Specify Slingshot version

> 💡 **Tip**: Use descriptive namespace names like `slingshot-prod` or `slingshot-dev` to avoid confusion.

**Configuration File:**
```
.slingshot/config/ss-installer-config-azure.json
```

### 6.2 Setup Tools 🛠️

```bash
slingshot setup-tools
```

> ⏱️ **Time Estimate**: 5-10 minutes depending on internet speed.

**📦 Installs:**
- ☸️ kubectl v1.28+
- ⎈ helm v3.12+
- 🏗️ terraform v1.5+
- ☁️ Cloud CLI (az/aws/gcloud)

> ✅ **Verification**: Run `kubectl version`, `helm version`, and `terraform version` to confirm installation.

### 6.3 Pull Packages 📦

> � **Purpose**: Downloads all required container images, Helm charts, and deployment scripts from your container registry.

#### Command Usage

**Basic Usage:**
```bash
slingshot package-pull
```

**⚙️ Prerequisites:**
- ✅ Configuration initialized (`config init` completed)
- ✅ Container registry credentials configured
- ✅ Network connectivity to registry
- ✅ Sufficient disk space (20+ GB recommended)

#### Setting Up Registry Credentials 🔐

> 🔒 **Security Best Practice**: Always use environment variables for credentials, never hardcode them!

**Azure Container Registry (ACR):**
```bash
export ACR_USERNAME="myregistry"
export ACR_PASSWORD="<your-access-token>"

# Or use Azure CLI to get token
export ACR_PASSWORD=$(az acr login --name myregistry --expose-token --query accessToken -o tsv)
```

**AWS Elastic Container Registry (ECR):**
```bash
export AWS_ACCESS_KEY_ID="<your-access-key>"
export AWS_SECRET_ACCESS_KEY="<your-secret-key>"
export AWS_DEFAULT_REGION="us-east-1"

# ECR login handled automatically by slingshot
```

**GCP Artifact Registry:**
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"

# Or use gcloud
gcloud auth configure-docker <region>-docker.pkg.dev
```

**Docker Hub / Private Registry:**
```bash
export DOCKER_USERNAME="<your-username>"
export DOCKER_PASSWORD="<your-password>"
export DOCKER_REGISTRY="registry.example.com"  # Optional, defaults to Docker Hub
```

#### Command Options

```bash
slingshot package-pull [flags]

Flags:
  --skip-images          Skip container image downloads (charts only)
  --skip-charts          Skip Helm chart downloads (images only)
  --skip-scripts         Skip deployment scripts download
  --export PATH          Export packages for air-gapped deployment
  --validate-checksums   Verify package integrity after download
  --parallel int         Number of parallel downloads (default: 3)
  --retry int            Number of retry attempts for failed downloads (default: 3)
  -h, --help            Help for package-pull
```

#### What Gets Downloaded 📥

**🐳 Container Images:**
- Bodhi services (auth, llm, userprofile, project, etc.)
- Portal services (backend, frontend, auth)
- Infrastructure (RabbitMQ, Chroma, Grafana)
- NEXA services (config-server, api, indexer)
- Agent Studio components
- Database clients and utilities

**⎈ Helm Charts:**
- Application deployment charts
- Configuration templates
- Service definitions
- Ingress rules

**📜 Deployment Scripts:**
- Pre-deployment hooks
- Post-deployment hooks
- Database setup scripts
- Migration scripts

#### Expected Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 Package Pull
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Reading configuration...
   • Registry: myregistry.azurecr.io
   • Version: v2.0.0
   • Selected groups: 10

🐳 Downloading container images...
   [1/45] myregistry.azurecr.io/api-auth-v2:v2.0.0 ✓ (2.1 GB)
   [2/45] myregistry.azurecr.io/api-llm:v2.0.0 ✓ (1.8 GB)
   [3/45] myregistry.azurecr.io/portal-backend:v2.0.0 ✓ (1.5 GB)
   ...
   [45/45] myregistry.azurecr.io/agent-studio-ui:v2.0.0 ✓ (987 MB)

⎈ Downloading Helm charts...
   [1/10] bodhi-auth-v2-chart-2.0.0.tgz ✓
   [2/10] portal-platform-chart-2.0.0.tgz ✓
   ...
   [10/10] agent-studio-chart-2.0.0.tgz ✓

📜 Downloading deployment scripts...
   ✓ Portal platform hooks
   ✓ NEXA configuration hooks
   ✓ Backlog AI setup hooks
   ✓ App Modernization hooks

✅ Package pull completed successfully!
   • Images: 45 downloaded (total: 68.5 GB)
   • Charts: 10 downloaded
   • Scripts: 4 packages downloaded
   • Duration: 12m 34s

📂 Files location: .slingshot/
   ├── helm-charts/
   ├── images/
   └── deployment-scripts/

✅ Ready for infrastructure provisioning!
```

> ⏱️ **Time Estimate**: 10-30 minutes depending on:
> - Package size (typically 50-80 GB total)
> - Network bandwidth
> - Registry location/latency
> - Number of selected deployment groups

#### Air-Gapped Deployment 🔌

> 💡 **Use Case**: For environments without internet access (air-gapped, secure networks).

**Step 1: Export packages (on internet-connected machine):**
```bash
slingshot package-pull --export /path/to/export/directory
```

**This creates:**
```
package-export-v2.0.0.tar.gz  (compressed archive)
├── images/
│   ├── api-auth-v2-v2.0.0.tar
│   ├── api-llm-v2.0.0.tar
│   └── ...
├── charts/
│   ├── bodhi-auth-v2-chart-2.0.0.tgz
│   └── ...
├── scripts/
│   └── ...
└── manifest.json  (checksums and metadata)
```

**Step 2: Transfer to air-gapped environment:**
```bash
# Copy to USB drive, secure transfer, etc.
cp package-export-v2.0.0.tar.gz /media/usb-drive/
```

**Step 3: Import on air-gapped machine:**
```bash
slingshot package-import --file package-export-v2.0.0.tar.gz
```

> ✅ **Verification**: The import process validates checksums to ensure package integrity.

#### Troubleshooting Package Pull 🔧

**❌ Issue: "Authentication failed"**
```
Error: failed to authenticate to registry myregistry.azurecr.io
```
**✅ Solution:**
```bash
# Verify credentials are set
echo $ACR_USERNAME
echo $ACR_PASSWORD | head -c 20  # Show first 20 chars only

# Test registry access
az acr login --name myregistry

# Or manually test
docker login myregistry.azurecr.io -u $ACR_USERNAME -p $ACR_PASSWORD
```

**❌ Issue: "Disk space insufficient"**
```
Error: not enough disk space (available: 15 GB, required: 70 GB)
```
**✅ Solution:**
```bash
# Check available space
df -h .

# Clean up Docker cache (if using Docker)
docker system prune -a --volumes

# Or specify different download location
mkdir -p /mnt/large-disk/.slingshot
ln -s /mnt/large-disk/.slingshot ~/.slingshot
```

**❌ Issue: "Image pull timeout"**
```
Error: timeout pulling image myregistry.azurecr.io/api-auth-v2:v2.0.0
```
**✅ Solution:**
```bash
# Reduce parallel downloads
slingshot package-pull --parallel 1

# Or increase retry attempts
slingshot package-pull --retry 5

# Check network connectivity
curl -I https://myregistry.azurecr.io/v2/
```

**❌ Issue: "Some images failed to download"**
**✅ Solution:**
```bash
# Re-run package-pull (skips already downloaded packages)
slingshot package-pull

# Or validate and redownload corrupted packages
slingshot package-pull --validate-checksums
```

#### Best Practices 💡

- ✅ **Verify credentials** before starting long downloads
- ✅ **Check disk space** (need 2x package size for extraction)
- ✅ **Use wired connection** for large downloads (avoid Wi-Fi)
- ✅ **Downloads are resumable** - safe to re-run if interrupted
- ✅ **Keep packages** for quick reinstalls or rollbacks
- ⚠️ **Production**: Download during off-peak hours
- ⚠️ **Multiple regions**: Consider using a registry in same region as cluster

### 6.4 Provision Infrastructure 🏗️

> 🏗️ **Purpose**: Provisions Kubernetes cluster and all required cloud infrastructure using Terraform.

> ⚠️ **IMPORTANT**: This command creates cloud resources that **incur costs**. Ensure you have:
> - 💳 Appropriate cloud credits or budget approval
> - 👥 Authority to create resources in your cloud account
> - 📋 Cost estimation reviewed with your team

#### Command Usage

**Basic Usage:**
```bash
slingshot provision-infra
```

**⚙️ Prerequisites:**
- ✅ Configuration initialized (`config init` completed)
- ✅ Packages pulled (`package-pull` completed)
- ✅ Cloud CLI authenticated (az/aws/gcloud logged in)
- ✅ Sufficient cloud permissions (Owner or Contributor)
- ✅ Terraform templates available

#### Interactive Menu 🧙

When you run the command, you'll see:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏗️ Infrastructure Provisioning
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Select an option:
  1. ⚙️  Configure Terraform Source
  2. ▶️  Execute Infrastructure Provisioning
  3. 📊 View Provisioning Status
  4. 🔄 Retry Failed Provisioning
  5. 🗑️  Destroy Infrastructure (WARNING)
  6. ◀️  Back to Main Menu

Choice [1-6]: 
```

#### Option 1: Configure Terraform Source ⚙️

**Purpose**: Specify where your Terraform infrastructure-as-code (IaC) templates are located.

**Prompts:**
```
📝 Select Terraform source:
  1. Tarball (pre-packaged .tar.gz file)
  2. Git Repository

Choice [1-2]: 1

📝 Terraform tarball path: /path/to/terraform-templates.tar.gz

✅ Terraform source configured!
   • Source type: Tarball
   • Path: /path/to/terraform-templates.tar.gz
   • Extraction: .slingshot/terraform/
```

**For Git Repository:**
```
📝 Git repository URL: https://github.com/yourorg/terraform-infra.git
📝 Branch (default: main): 
📝 Path within repo (default: /): terraform/azure/
🔐 Git token (if private repo): ********

✅ Terraform source configured!
   • Source: Git repository
   • URL: https://github.com/yourorg/terraform-infra.git
   • Branch: main
   • Path: terraform/azure/
```

> 💡 **Tip**: Most users receive a pre-packaged tarball with tested templates.

#### Option 2: Execute Infrastructure Provisioning ▶️

**What Happens:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶️ Infrastructure Provisioning
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  COST WARNING:
   This will create cloud resources that incur costs.
   Estimated monthly cost: $500-2000 (depending on cluster size)

? Proceed with provisioning? (yes/no): yes

🔍 Phase 1: Preparing Terraform workspace...
   • Extracting templates to .slingshot/terraform/
   • Loading configuration from ss-installer-config-azure.json
   • Generating terraform.tfvars
   ✅ Workspace prepared

🔍 Phase 2: Terraform init...
   • Initializing Terraform providers
   • Downloading Azure provider v3.85.0
   • Configuring backend
   ✅ Terraform initialized

🔍 Phase 3: Terraform plan...
   • Analyzing infrastructure changes
   • Resources to create: 42
   • Resources to modify: 0
   • Resources to destroy: 0
   ✅ Plan generated

📋 Resources to be created:
   • AKS Cluster (aks-slingshot-prod-01)
   • Node Pool (3 nodes, Standard_D4s_v3)
   • Virtual Network (10.0.0.0/16)
   • Subnet (10.0.1.0/24)
   • Network Security Group
   • Azure Container Registry
   • Azure PostgreSQL Server
   • Azure Key Vault
   • Public IP addresses (2)
   • Load Balancer
   • Storage Account
   ... (32 more resources)

? Review looks good. Apply? (yes/no): yes

🔍 Phase 4: Terraform apply...
   [00:01:23] Creating resource group rg-slingshot-prod-eus2... ✓
   [00:01:45] Creating virtual network vnet-slingshot-prod... ✓
   [00:02:10] Creating network security group nsg-slingshot-prod... ✓
   [00:03:30] Creating Azure Container Registry acr-slingshot-prod... ✓
   [00:15:20] Creating AKS cluster aks-slingshot-prod-01... ✓
   [00:16:45] Creating node pool agentpool... ✓
   [00:18:30] Creating Azure PostgreSQL server pgsql-slingshot-prod... ✓
   [00:19:15] Creating Azure Key Vault kv-slingshot-prod... ✓
   [00:20:05] Configuring networking... ✓
   [00:21:30] Setting up IAM roles... ✓
   [00:22:00] Applying tags and labels... ✓

✅ Infrastructure provisioned successfully!

📊 Summary:
   • Cluster: aks-slingshot-prod-01
   • Region: East US 2
   • Nodes: 3 (Standard_D4s_v3)
   • Network: 10.0.0.0/16
   • Duration: 22m 00s

📋 Next steps:
   1. Run: slingshot validate-infra
   2. Configure kubectl access
   3. Proceed with application installation

🔧 kubectl configuration:
   az aks get-credentials --resource-group rg-slingshot-prod-eus2 \
     --name aks-slingshot-prod-01
```

> ⏱️ **Time Estimate**: 15-30 minutes
> - Terraform init: 2-3 minutes
> - Terraform plan: 1-2 minutes
> - Terraform apply: 12-25 minutes
> - **Total**: 15-30 minutes

#### What Gets Provisioned ☁️

**Azure (AKS):**
- ☸️ AKS Kubernetes Cluster (1.28+)
- 🖥️ Node Pools (3+ nodes)
- 🌐 Virtual Network + Subnets
- 🔒 Network Security Groups
- 📦 Azure Container Registry
- 🗄️ Azure PostgreSQL Flexible Server
- 🔑 Azure Key Vault
- 💾 Storage Accounts
- 🌍 Public IP + Load Balancer
- 📊 Log Analytics Workspace

**AWS (EKS):**
- ☸️ EKS Kubernetes Cluster
- 🖥️ EC2 Node Groups (or Fargate)
- 🌐 VPC + Subnets + Route Tables
- 🔒 Security Groups
- 📦 ECR Registry
- 🗄️ RDS PostgreSQL
- 🔑 Secrets Manager
- 💾 EBS Volumes
- 🌍 Elastic Load Balancer
- 📊 CloudWatch Logs

**GCP (GKE):**
- ☸️ GKE Kubernetes Cluster
- 🖥️ Node Pools
- 🌐 VPC + Subnets
- 🔒 Firewall Rules
- 📦 Artifact Registry
- 🗄️ Cloud SQL PostgreSQL
- 🔑 Secret Manager
- 💾 Persistent Disks
- 🌍 Cloud Load Balancer
- 📊 Cloud Logging

#### Option 3: View Provisioning Status 📊

**Check current provisioning state:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Provisioning Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Checking Terraform state...

✅ Infrastructure Status: Provisioned

📋 Resources:
   Resource Group:     rg-slingshot-prod-eus2
   AKS Cluster:        aks-slingshot-prod-01 (Running)
   Node Count:         3/3 Ready
   Kubernetes Version: v1.28.5
   PostgreSQL Server:  pgsql-slingshot-prod.postgres.database.azure.com (Ready)
   Key Vault:          kv-slingshot-prod (Accessible)
   ACR:               acrslingshot, prod.azurecr.io (Active)

⏱️  Provisioned: 2026-02-05 14:30:00 UTC (2 hours ago)
📂 State file: .slingshot/terraform/terraform.tfstate

✅ Infrastructure is healthy and ready for deployment!
```

#### Command Options

```bash
slingshot provision-infra [flags]

Flags:
  --auto-approve         Skip interactive approval prompts
  --skip-plan            Skip terraform plan (not recommended)
  --var-file PATH        Custom terraform variables file
  --destroy              Destroy infrastructure instead of creating
  --state-file PATH      Custom terraform state file location
  -h, --help            Help for provision-infra
```

**Automated Provisioning (CI/CD):**
```bash
# Non-interactive mode
slingshot provision-infra --auto-approve
```

> ⚠️ **Warning**: Use `--auto-approve` only in automated pipelines with proper safeguards!

#### Troubleshooting Infrastructure Provisioning 🔧

**❌ Issue: "Insufficient permissions"**
```
Error: authorization failed: user lacks permissions to create resource group
```
**✅ Solution:**
```bash
# Check current account
az account show

# Verify role assignment
az role assignment list --assignee <your-user-id> --subscription <sub-id>

# Required roles: Contributor or Owner
# Contact your Azure administrator to grant permissions
```

**❌ Issue: "Quota exceeded"**
```
Error: The operation could not be completed because it results in exceeding approved quota
```
**✅ Solution:**
```bash
# Check quotas
az vm list-usage --location eastus --output table

# Request quota increase in Azure Portal:
# Support > New support request > Quota

# Or use smaller VM sizes in terraform.tfvars
```

**❌ Issue: "Resource name already exists"**
```
Error: A resource with the ID '/subscriptions/.../aks-slingshot-prod-01' already exists
```
**✅ Solution:**
```bash
# Option 1: Import existing resource
cd .slingshot/terraform
terraform import azurerm_kubernetes_cluster.main /subscriptions/.../aks-slingshot-prod-01

# Option 2: Use different name in config
slingshot config init
# Change cluster name when prompted
```

**❌ Issue: "Terraform state locked"**
```
Error: Error acquiring the state lock
```
**✅ Solution:**
```bash
# Check for stale locks
cd .slingshot/terraform
terraform force-unlock <lock-id>

# Or wait for previous operation to complete
```

**❌ Issue: "Network CIDR overlap"**
```
Error: The specified subnet address range overlaps with existing network
```
**✅ Solution:**
```bash
# Edit terraform.tfvars
vim .slingshot/terraform/terraform.tfvars

# Change CIDR blocks:
vnet_address_space = ["10.1.0.0/16"]  # Different range
subnet_address_prefix = "10.1.1.0/24"

# Re-run provisioning
slingshot provision-infra
```

#### Destroying Infrastructure 🗑️

> ⚠️ **DANGER**: This permanently deletes ALL provisioned resources!

**To destroy infrastructure:**
```
Select option 5 from menu:
  5. 🗑️  Destroy Infrastructure (WARNING)

⚠️  CRITICAL WARNING:
   This will PERMANENTLY DELETE all infrastructure resources:
   • AKS cluster aks-slingshot-prod-01
   • All deployed applications and data
   • PostgreSQL database server
   • Storage accounts and volumes
   • Network configuration
   • All associated resources (42 total)

❗ DATA LOSS WARNING:
   • All databases will be deleted
   • All persistent volumes will be deleted
   • This action CANNOT be undone

? Type 'DESTROY' to confirm: DESTROY

? Are you absolutely sure? (yes/no): yes

🗑️  Destroying infrastructure...
   [00:00:30] Draining node pools... ✓
   [00:02:15] Deleting AKS cluster... ✓
   [00:03:45] Deleting PostgreSQL server... ✓
   [00:04:20] Deleting network resources... ✓
   [00:05:10] Deleting resource group... ✓

✅ Infrastructure destroyed
```

> 💡 **Before Destroying**: Always backup databases and export important data!

#### Best Practices 💡

- ✅ **Review terraform plan** carefully before applying
- ✅ **Use consistent naming** for easy resource identification
- ✅ **Tag all resources** with project, environment, owner
- ✅ **Keep state files secure** (contains sensitive data)
- ✅ **Document changes** to infrastructure
- ✅ **Test in dev environment** before production provisioning
- ⚠️ **Monitor costs** using cloud cost management tools
- ⚠️ **Set up budgets** and alerts for unexpected cost spikes
- 🔒 **Enable encryption** for databases and storage
- 🔒 **Use private endpoints** for production databases
- 📊 **Enable logging** and monitoring from day one

### 6.5 Validate Infrastructure ✅

```bash
slingshot validate-infra
```

> 🔍 **Critical Step**: Do NOT skip validation! It ensures infrastructure is ready for deployment.

**🔍 Checks:**
- 💚 Kubernetes API health
- 🖥️ Node status
- 💾 Storage classes
- 🌐 DNS resolution

> ⚠️ **Action Required**: Fix any validation failures before proceeding to application installation.

### 6.6 Install Applications 🚀

> 🚀 **Purpose**: Deploys all application groups (Bodhi, Portal, Agent Studio, etc.) to your Kubernetes cluster.

#### Command Usage

**Basic Usage:**
```bash
slingshot install-app
```

**⚙️ Prerequisites:**
- ✅ Infrastructure provisioned (`provision-infra` completed)
- ✅ Infrastructure validated (`validate-infra` passed)
- ✅ Packages pulled (`package-pull` completed)
- ✅ kubectl configured and cluster accessible
- ✅ Database servers accessible (PostgreSQL, MongoDB)
- ✅ Cluster has sufficient resources (CPU, memory, storage)

> ⏱️ **Time Estimate**: 30-60 minutes for complete deployment (all 10 groups).
> ⚠️ **Production Warning**: Monitor the deployment. Do NOT close the terminal during installation.

#### Deployment Process 🔄

**For each application group (1-10):**
1. 🪝 **Execute pre-deployment hooks** (database setup, configuration)
2. ⎈ **Deploy applications via Helm** (with custom values)
3. ⏳ **Wait for pods ready** (health checks with retries)
4. 🪝 **Execute post-deployment hooks** (migrations, data seeding)
5. 💚 **Perform health checks** (API endpoints, service connectivity)

> 💡 **Tip**: Open a second terminal and run `kubectl get pods -n <namespace> -w` to watch pod status in real-time.

#### Interactive Prompts 🧙

During deployment, you'll be prompted for configuration values needed by pre/post-deployment hooks:

**Group 5: Portal Platform Hooks**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Installing Group 5: Portal Applications
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 Collecting configuration for pre-deployment hooks...

🔐 POSTGRES_PASSWORD: ********
📝 POSTGRES_HOST [pgsql-server.postgres.database.azure.com]: 
📝 POSTGRES_PORT [5432]: 
📝 POSTGRES_USER_NAME [admin]: 
📝 MONGO_CONN_STRING: mongodb://mongo-server:27017/portal
📝 DOMAIN [stg3.sapientslingshot.com]: 
📝 DEFAULT_ADMIN_USERS [admin@example.com]: 
📝 DEFAULT_CLIENT_ID (auto-generated): b97fbef7-cad9-4e56-9850-39e86c8fdf61

✅ Configuration collected
```

> 🔐 **Security**: Passwords are masked during input and stored in-memory only. Never written to logs.

**Group 7: Backlog AI Fresh Setup**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Installing Group 7: Backlog AI & Platform
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔧 Pre-Deployment Hook: Backlog Fresh Setup

📝 SCRIPTS_LIST (default: f001_setup_postgres.py): [Enter]

Step 1: Parsing environment files...
   Found 8 configuration fields

Step 2: Collecting configuration values...
   🔐 POSTGRES_PASSWORD: ******** (using saved value)
   📝 POSTGRES_HOST: pgsql-server.postgres.database.azure.com
   📝 DEFAULT_CLIENT_ID: b97fbef7-cad9-4e56-9850-39e86c8fdf61
   ℹ️  Using defaults from previous configuration

Step 3: Writing environment files...
   ✅ .env_common created
   ✅ .env_main created

Step 4: Configuring .env_common...
   ℹ️  Auto-generated 10 service URLs from DOMAIN:
   • ENT_AUTH_URL: https://stg3.sapientslingshot.com/api/v2/auth
   • SS_BACKLOG_URL: https://stg3.sapientslingshot.com/api/v1/ai-backlog
   • SS_PORTAL_URL: https://stg3.sapientslingshot.com
   ... (7 more URLs)

Step 5: Executing setup script...
   ⏳ Running: run_modules.sh
   
   2026-02-05 14:35:00 - INFO - Starting PostgreSQL setup
   2026-02-05 14:35:10 - INFO - Creating database: ai_backlog
   2026-02-05 14:35:12 - INFO - Running migrations: version 001
   2026-02-05 14:35:15 - INFO - Running migrations: version 002
   2026-02-05 14:35:18 - INFO - Seeding default data
   2026-02-05 14:35:45 - INFO - EXECUTION COMPLETED

✅ Backlog Fresh Setup completed successfully
✅ SCRIPTS_LIST commented out (prevents re-execution)
```

#### Expected Output 📊

**Complete Deployment:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 Application Installation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Loading configuration...
   • Namespace: slingshot-ns
   • Groups to deploy: 10
   • Helm release name prefix: slingshot

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Group 1/10: Common Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⎈ Deploying application: common-config
   • Helm chart: common-config-v2.0.0
   • Release: slingshot-common-config
   [00:00:15] Installing chart... ✓
   [00:00:30] Waiting for ConfigMap ready... ✓

✅ Group 1 completed in 0m 35s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 Group 2/10: Common Secrets
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⎈ Deploying application: common-secrets
   • Helm chart: common-secrets-v2.0.0
   • Release: slingshot-common-secrets
   [00:00:10] Installing chart... ✓
   [00:00:15] Waiting for Secret ready... ✓

✅ Group 2 completed in 0m 20s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏗️ Group 3/10: Infrastructure Components
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⎈ Deploying application 1/3: rabbitmq
   [00:01:30] Installing chart... ✓
   [00:03:00] Waiting for pods (3 replicas)... ✓
   ✓ Pod: rabbitmq-0 (Running)
   ✓ Pod: rabbitmq-1 (Running)
   ✓ Pod: rabbitmq-2 (Running)

⎈ Deploying application 2/3: chroma
   [00:00:45] Installing chart... ✓
   [00:01:20] Waiting for pods (1 replica)... ✓
   ✓ Pod: chroma-6d8f9b7c-xyz (Running)

⎈ Deploying application 3/3: grafana
   [00:00:30] Installing chart... ✓
   [00:01:00] Waiting for pods (1 replica)... ✓
   ✓ Pod: grafana-5c7d8e9f-abc (Running)

✅ Group 3 completed in 6m 45s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔌 Group 4/10: Bodhi Agent Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⎈ Deploying application 1/5: api-auth-v2
   [00:01:00] Installing chart... ✓
   [00:02:30] Waiting for pods (2 replicas)... ✓
   [00:02:45] Health check: /api/v2/auth/health... ✓

⎈ Deploying application 2/5: api-llm
   [00:00:50] Installing chart... ✓
   [00:02:00] Waiting for pods (2 replicas)... ✓

... (3 more applications)

✅ Group 4 completed in 12m 15s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Group 5/10: Portal Applications
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🪝 Executing pre-deployment hooks...
   ✓ MongoDB collection setup
   ✓ PostgreSQL schema creation
   ✓ Default client creation
   Duration: 2m 10s

⎈ Deploying application 1/12: portal-backend
   [00:01:20] Installing chart... ✓
   [00:03:00] Waiting for pods (3 replicas)... ✓

... (11 more applications)

🪝 Executing post-deployment hooks...
   ✓ Configuration server update
   ✓ Admin user creation
   Duration: 1m 30s

✅ Group 5 completed in 18m 45s

... (Groups 6-10)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Installation Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Deployment completed successfully!

📈 Statistics:
   • Groups deployed: 10/10
   • Applications installed: 45
   • Pods running: 87
   • Services created: 45
   • ConfigMaps: 12
   • Secrets: 8
   • Total duration: 47m 32s

📋 Deployed Applications by Group:
   ✓ Group 1: Common Configuration (1)
   ✓ Group 2: Common Secrets (1)
   ✓ Group 3: Infrastructure (3)
   ✓ Group 4: Bodhi Agent Framework (5)
   ✓ Group 5: Portal Applications (12)
   ✓ Group 6: NEXA Intelligence (3)
   ✓ Group 7: Backlog AI Platform (8)
   ✓ Group 8: AIPP Services (4)
   ✓ Group 9: App Modernization (6)
   ✓ Group 10: AISM Services (2)

🔗 Access URLs:
   🌐 Portal: https://stg3.sapientslingshot.com
   🔐 Auth API: https://stg3.sapientslingshot.com/api/v2/auth
   🤖 LLM API: https://stg3.sapientslingshot.com/api/v1/llm
   🎯 Agent Studio: https://stg3.sapientslingshot.com/agent-studio

📝 Next Steps:
   1. Run: slingshot validate-app
   2. Run: slingshot troubleshoot
   3. Test portal access: https://stg3.sapientslingshot.com
   4. Review deployment logs: .slingshot/logs/

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Command Options

```bash
slingshot install-app [flags]

Flags:
  --group string         Install specific group only (e.g., "portal-platform")
  --skip-hooks           Skip pre/post-deployment hooks (not recommended)
  --skip-wait            Don't wait for pods to be ready (not recommended)
  --timeout duration     Per-deployment timeout (default: 10m)
  --parallel            Deploy applications in parallel where possible
  --dry-run             Show what would be deployed without deploying
  --force               Force reinstall even if already deployed
  -h, --help            Help for install-app
```

**Examples:**

```bash
# Install specific group only
slingshot install-app --group portal-platform

# Dry run to see what will be deployed
slingshot install-app --dry-run

# Force reinstall (upgrades existing deployments)
slingshot install-app --force

# Install with longer timeout for slow clusters
slingshot install-app --timeout 20m
```

#### Selective Group Deployment 🎯

**Install individual groups:

**
```bash
# Deploy only essential infrastructure
slingshot install-app --group common-config
slingshot install-app --group common-secrets
slingshot install-app --group infrastructure

# Deploy Bodhi and Portal only
slingshot install-app --group enterprise-apis
slingshot install-app --group portal-platform

# Add optional features later
slingshot install-app --group nexa-services
slingshot install-app --group agent-studio
```

> 💡 **Use Case**: Incremental deployment or testing individual groups.

#### Monitoring Deployment Progress 📊

**Terminal 1 - Run installer:**
```bash
slingshot install-app
```

**Terminal 2 - Watch pods:**
```bash
# Watch all pods in namespace
kubectl get pods -n slingshot-ns -w

# Watch specific deployment
kubectl get pods -n slingshot-ns -l app=api-auth-v2 -w

# Watch events
kubectl get events -n slingshot-ns --sort-by='.lastTimestamp' -w
```

**Terminal 3 - Monitor logs:**
```bash
# Follow logs of specific pod
kubectl logs -f -n slingshot-ns <pod-name>

# Follow logs with label selector
kubectl logs -f -n slingshot-ns -l app=portal-backend --tail=50
```

#### Troubleshooting Installation 🔧

**❌ Issue: "Pod stuck in Pending state"**
```
⚠️  Waiting for pod portal-backend-6d8f9b7c-xyz... (5/10 minutes)
   Status: Pending
```
**✅ Solution:**
```bash
# Check pod details
kubectl describe pod portal-backend-6d8f9b7c-xyz -n slingshot-ns

# Common causes:
# 1. Insufficient cluster resources
kubectl top nodes
kubectl describe nodes

# 2. ImagePullBackOff - check image name and registry credentials
kubectl get events -n slingshot-ns | grep -i pull

# 3. Volume mount issues
kubectl get pvc -n slingshot-ns
```

**❌ Issue: "Pod in CrashLoopBackOff"**
```
❌ Pod api-llm-7e9f8g0h-abc failed to start (CrashLoopBackOff)
```
**✅ Solution:**
```bash
# Check pod logs
kubectl logs api-llm-7e9f8g0h-abc -n slingshot-ns
kubectl logs api-llm-7e9f8g0h-abc -n slingshot-ns --previous

# Common causes:
# 1. Database connection failed - check connection strings
# 2. Missing environment variables - check ConfigMaps/Secrets
kubectl get configmap -n slingshot-ns
kubectl get secret -n slingshot-ns

# 3. Application error - review logs for stack traces
```

**❌ Issue: "Hook execution timeout"**
```
⏳ Executing pre-deployment hook: portal-db-setup
   Running for 10 minutes... timeout
❌ Hook execution failed
```
**✅ Solution:**
```bash
# Check hook pod logs
kubectl logs -n slingshot-ns -l job-name=portal-db-setup

# Manually run hook to see full output
kubectl get jobs -n slingshot-ns
kubectl logs -n slingshot-ns job/portal-db-setup-<hash>

# Common causes:
# 1. Database connectivity - verify firewall rules
# 2. Slow database - increase timeout with --timeout flag
# 3. Hook script error - check logs for Python/Shell errors
```

**❌ Issue: "Helm release failed"**
```
Error: UPGRADE FAILED: cannot patch "portal-backend" with kind Deployment
```
**✅ Solution:**
```bash
# Check Helm release status
helm list -n slingshot-ns
helm status slingshot-portal-backend -n slingshot-ns

# Option 1: Rollback to previous version
helm rollback slingshot-portal-backend -n slingshot-ns

# Option 2: Force reinstall
slingshot install-app --group portal-platform --force

# Option 3: Manual cleanup and retry
helm delete slingshot-portal-backend -n slingshot-ns
kubectl delete deployment portal-backend -n slingshot-ns
slingshot install-app --group portal-platform
```

**❌ Issue: "Health check failed"**
```
⚠️  Health check failed for api-auth-v2
   URL: http://api-auth-v2-svc:8080/api/v2/auth/health
   Response: Connection timeout
```
**✅ Solution:**
```bash
# Test service connectivity from within cluster
kubectl run test-curl --rm -i --restart=Never \
  --image=curlimages/curl:latest -n slingshot-ns -- \
  curl -v http://api-auth-v2-svc:8080/api/v2/auth/health

# Check service endpoints
kubectl get endpoints api-auth-v2-svc -n slingshot-ns

# Check pod readiness
kubectl get pods -n slingshot-ns -l app=api-auth-v2

# Review pod logs for startup errors
kubectl logs -n slingshot-ns -l app=api-auth-v2
```

#### Rollback Strategy 🔄

**If deployment fails mid-way:**

```bash
# Check what was deployed
helm list -n slingshot-ns

# Rollback specific releases
helm rollback slingshot-portal-backend -n slingshot-ns
helm rollback slingshot-api-auth-v2 -n slingshot-ns

# Or uninstall failed deployments
helm delete slingshot-portal-backend -n slingshot-ns
kubectl delete all -n slingshot-ns -l app=portal-backend

# Fix issues, then retry
slingshot install-app --group portal-platform
```

> 💡 **Tip**: Helm maintains release history. Use `helm history <release> -n <namespace>` to view all revisions.

#### Best Practices 💡

- ✅ **Run validate-infra first** - ensure cluster is ready
- ✅ **Monitor during deployment** - watch pods and logs
- ✅ **Deploy incrementally** - test each group before proceeding
- ✅ **Keep terminal open** - do NOT close during installation
- ✅ **Check database connectivity** - test before deploying
- ✅ **Review configuration** - verify URLs and credentials
- ⚠️ **Production deployments** - schedule during maintenance window
- ⚠️ **Backup before rollback** - export data if needed
- 🔒 **Secure credentials** - use environment variables, not files
- 📊 **Save logs** - keep deployment logs for troubleshooting

**Example: Backlog AI Fresh Setup Hook**

```
🔧 Pre-Deployment: Backlog Fresh Setup

📝 SCRIPTS_LIST (default: f001_setup_postgres.py): [Enter]

Step 1: Parsing environment files...
Found 8 configuration fields

Step 2: Collecting configuration values...
🔐 POSTGRES_PASSWORD: ********
📝 POSTGRES_HOST: pgsql-server.postgres.database.azure.com
📝 DEFAULT_CLIENT_ID: b97fbef7-cad9-4e56-9850-39e86c8fdf61
...

📋 Fresh Setup Critical Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  Review and confirm critical configuration:

Database Configuration:
• POSTGRES_HOST: pgsql-server.postgres.database.azure.com
• POSTGRES_PORT: 5432
• POSTGRES_USER_NAME: admin
• MONGO_CONN_STRING: ********

Client & Authentication:
• DEFAULT_CLIENT_ID: b97fbef7-cad9-4e56-9850-39e86c8fdf61
• GROUPE_CLIENT_ID: b97fbef7-cad9-4e56-9850-39e86c8fdf61

Environment:
• DOMAIN: stg3.sapientslingshot.com

✅ Critical configuration confirmed

Step 3: Writing environment files...
✅ .env_common created
✅ .env_main created

Step 4: Configuring .env_common...
ℹ️  Auto-generated service URLs from DOMAIN
   • ENT_AUTH_URL: https://stg3.sapientslingshot.com/api/v2/auth
   • SS_BACKLOG_URL: https://stg3.sapientslingshot.com/api/v1/ai-backlog
   ...

Step 5: Executing setup script...
⏳ Running: run_modules.sh

2026-01-30 14:35:00 - INFO - Starting PostgreSQL setup
2026-01-30 14:35:45 - INFO - EXECUTION COMPLETED

✅ Backlog Fresh Setup completed successfully
✅ SCRIPTS_LIST commented out
```

### 6.7 Validate Applications ✔️

```bash
slingshot validate-app
```

> 🎉 **Almost Done!** Final validation ensures everything is working correctly.

**🔍 Validation Checks:**
- 💚 Pod health (45/45 running)
- 🔌 Service endpoints
- 🌐 API health checks
- 🗄️ Database connectivity
- ⚡ Performance metrics

> ✅ **Success Criteria**: All checks should pass with 0 failures. Review any warnings carefully.

### 6.8 Upgrade Application 🔄

> 🔄 **Purpose**: Upgrades an already deployed application to a newer version or applies configuration changes.

#### Command Usage

**Basic Usage:**
```bash
slingshot upgrade-app
```

**⚙️ Prerequisites:**
- ✅ Applications already deployed (`install-app` completed previously)
- ✅ New packages pulled for target version (if upgrading version)
- ✅ Configuration updated (if applying config changes)
- ✅ kubectl configured and cluster accessible

> ⚠️ **Important**: This performs an in-place upgrade. Pods will be rolling-updated with zero downtime for most services.

#### When to Use Upgrade 📝

**Use `upgrade-app` when:**
- 🆕 Deploying a new version of the application
- ⚙️ Applying configuration changes to existing deployments
- 🔧 Updating resource limits (CPU, memory)
- 🔒 Applying security patches
- 🌐 Changing service URLs or endpoints
- 📊 Modifying replica counts

**Do NOT use `upgrade-app` for:**
- ❌ First-time installation (use `install-app` instead)
- ❌ Major version upgrades requiring database migrations (follow specific upgrade guide)
- ❌ Changing deployment groups (redeploy with `install-app`)

#### Upgrade Process 🔄

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 Application Upgrade
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Detecting deployed applications...
   Found 45 deployed applications across 10 groups

📝 Select upgrade scope:
   1. Upgrade all applications
   2. Upgrade specific application group
   3. Upgrade single application

Choice [1-3]: 2

📝 Select application group:
   1. Common Configuration
   2. Common Secrets
   3. Infrastructure
   4. Bodhi Agent Framework
   5. Portal Applications
   6. NEXA Intelligence
   7. Backlog AI Platform
   8. AIPP Services
   9. App Modernization
   10. AISM Services

Choice [1-10]: 5

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Upgrading Group 5: Portal Applications
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📦 Current version: v1.9.5
📦 Target version: v2.0.0

⚠️  Upgrade Summary:
   • Applications to upgrade: 12
   • Pods to restart: 28
   • Downtime: Rolling update (zero downtime)
   • Hooks: 2 pre-upgrade, 1 post-upgrade

? Proceed with upgrade? (yes/no): yes

🪝 Executing pre-upgrade hooks...
   ✓ Database migration check
   ✓ Configuration backup
   Duration: 1m 20s

⎈ Upgrading application 1/12: portal-backend
   • Current: v1.9.5 → Target: v2.0.0
   [00:00:30] Helm upgrade... ✓
   [00:01:45] Rolling update: 3/3 pods replaced... ✓
   ✓ Pod: portal-backend-7f8g9h0i-new1 (Running)
   ✓ Pod: portal-backend-7f8g9h0i-new2 (Running)
   ✓ Pod: portal-backend-7f8g9h0i-new3 (Running)
   [00:02:00] Health check... ✓

⎈ Upgrading application 2/12: portal-frontend
   • Current: v1.9.5 → Target: v2.0.0
   [00:00:25] Helm upgrade... ✓
   [00:01:20] Rolling update: 2/2 pods replaced... ✓
   [00:01:35] Health check... ✓

... (10 more applications)

🪝 Executing post-upgrade hooks...
   ✓ Configuration sync
   Duration: 45s

✅ Upgrade completed successfully!

📊 Upgrade Summary:
   • Applications upgraded: 12/12
   • Pods restarted: 28
   • Failed upgrades: 0
   • Duration: 14m 32s

📝 Changes:
   • Portal backend: v1.9.5 → v2.0.0
   • Portal frontend: v1.9.5 → v2.0.0
   • Auth service: v1.9.5 → v2.0.0
   ... (9 more)

🔗 Application URLs (unchanged):
   🌐 Portal: https://stg3.sapientslingshot.com
   🔐 Auth API: https://stg3.sapientslingshot.com/api/v2/auth

📝 Next Steps:
   1. Run: slingshot validate-app
   2. Test application functionality
   3. Monitor logs for errors

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Command Options

```bash
slingshot upgrade-app [flags]

Flags:
  --group string         Upgrade specific group only
  --app string           Upgrade specific application only
  --version string       Target version to upgrade to
  --skip-hooks           Skip pre/post-upgrade hooks (not recommended)
  --force                Force upgrade even if version is same
  --rollback-on-failure  Auto-rollback if upgrade fails
  --dry-run              Show what would be upgraded without upgrading
  --timeout duration     Per-upgrade timeout (default: 10m)
  -h, --help            Help for upgrade-app
```

**Examples:**

```bash
# Upgrade specific group
slingshot upgrade-app --group portal-platform

# Upgrade specific application
slingshot upgrade-app --app portal-backend

# Upgrade to specific version
slingshot upgrade-app --version v2.1.0

# Dry run to see what will change
slingshot upgrade-app --dry-run

# Upgrade with auto-rollback on failure
slingshot upgrade-app --rollback-on-failure
```

#### Rolling Update Strategy 🔄

**How upgrades work:**

```
Before Upgrade:
┌─────────────────────────────┐
│ portal-backend-old-pod1 ✓   │
│ portal-backend-old-pod2 ✓   │
│ portal-backend-old-pod3 ✓   │
└─────────────────────────────┘

During Upgrade (Rolling):
┌─────────────────────────────┐
│ portal-backend-old-pod1 ✓   │  ← Still serving traffic
│ portal-backend-old-pod2 ✓   │  ← Still serving traffic
│ portal-backend-NEW-pod1 ⚡  │  ← New version starting
└─────────────────────────────┘

After Upgrade:
┌─────────────────────────────┐
│ portal-backend-NEW-pod1 ✓   │
│ portal-backend-NEW-pod2 ✓   │
│ portal-backend-NEW-pod3 ✓   │
└─────────────────────────────┘
```

> ✅ **Zero Downtime**: Old pods remain running until new pods are ready.

#### Rollback After Upgrade 🔙

**If issues detected after upgrade:**

```bash
# Check Helm release history
helm history slingshot-portal-backend -n slingshot-ns

REVISION  STATUS      CHART               DESCRIPTION
1         superseded  portal-backend-1.9  Initial install
2         superseded  portal-backend-1.9  Config update
3         deployed    portal-backend-2.0  Upgrade to v2.0.0

# Rollback to previous revision
helm rollback slingshot-portal-backend 2 -n slingshot-ns

# Or rollback to specific revision
helm rollback slingshot-portal-backend 1 -n slingshot-ns
```

**Using Slingshot rollback:**
```bash
# Rollback specific group to previous version
slingshot upgrade-app --group portal-platform --version v1.9.5

# Rollback all applications
slingshot upgrade-app --version v1.9.5
```

#### Troubleshooting Upgrades 🔧

**❌ Issue: "Upgrade failed mid-way"**
```
❌ Application portal-backend upgrade failed
   Error: pod portal-backend-7f8g9h0i-new1 failed health check
```
**✅ Solution:**
```bash
# Check what was upgraded
helm list -n slingshot-ns

# Check pod status
kubectl get pods -n slingshot-ns -l app=portal-backend

# Check logs
kubectl logs -n slingshot-ns -l app=portal-backend --tail=100

# Rollback if needed
helm rollback slingshot-portal-backend -n slingshot-ns

# Or use auto-rollback flag next time
slingshot upgrade-app --group portal-platform --rollback-on-failure
```

**❌ Issue: "Database migration required"**
```
⚠️  Warning: Version v2.0.0 requires database migration
   Current DB schema: v1.9
   Required DB schema: v2.0
```
**✅ Solution:**
```bash
# This is expected for major versions
# Migration hooks should handle this automatically

# Verify hooks are not skipped
slingshot upgrade-app --group portal-platform
# (Do NOT use --skip-hooks)

# Check hook execution logs
kubectl logs -n slingshot-ns -l job-name=portal-migration

# Manual migration if needed
kubectl exec -it -n slingshot-ns <portal-pod> -- bash
# Run migration scripts manually
```

**❌ Issue: "Configuration mismatch"**
```
⚠️  Configuration has changed but not upgraded
   • DOMAIN updated in config
   • Service URLs need regeneration
```
**✅ Solution:**
```bash
# Force upgrade to apply config changes
slingshot upgrade-app --group portal-platform --force

# This will:
# 1. Re-run hooks (regenerate service URLs)
# 2. Update ConfigMaps/Secrets
# 3. Restart pods to pick up new config
```

#### Best Practices 💡

- ✅ **Test in non-production first** - always test upgrades in dev/staging
- ✅ **Read release notes** - understand breaking changes
- ✅ **Backup databases** - before major version upgrades
- ✅ **Monitor during upgrade** - watch pods and logs
- ✅ **Verify after upgrade** - run `validate-app` and `troubleshoot`
- ✅ **Plan rollback** - know how to rollback if issues occur
- ⚠️ **Schedule maintenance** - upgrade production during low-traffic periods
- ⚠️ **Communicate upgrades** - notify users of scheduled maintenance
- 🔒 **Keep history** - Helm maintains last 10 revisions automatically
- 📊 **Monitor metrics** - track error rates and performance after upgrade

---

### 6.9 Getting Help 📖

> 📖 **Purpose**: View comprehensive help for Slingshot commands and options.

#### Command Usage

**View main help:**
```bash
slingshot --help
```

**View specific command help:**
```bash
slingshot <command> --help
```

**Examples:**
```bash
slingshot install-app --help
slingshot provision-infra --help
slingshot troubleshoot --help
```

#### Main Help Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 Slingshot Installer v2.0.0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Enterprise-grade orchestration tool for deploying Bodhi Agent Framework,
Slingshot Portal, and Agent Studio to Kubernetes clusters.

USAGE:
  slingshot [command] [flags]

CORE COMMANDS:
  config init         Initialize configuration for deployment
  setup-tools         Install required tools (kubectl, helm, terraform)
  package-pull        Download container images, charts, and scripts
  provision-infra     Provision Kubernetes cluster and infrastructure
  validate-infra      Validate infrastructure readiness
  install-app         Install applications to cluster
  validate-app        Validate deployed applications
  upgrade-app         Upgrade existing applications
  troubleshoot        Run health diagnostics and troubleshooting

MANAGEMENT COMMANDS:
  vault-sync          Sync secrets with Azure Key Vault
  package-import      Import packages from air-gapped export
  package-export      Export packages for air-gapped deployment

UTILITY COMMANDS:
  help                Show help for any command
  version             Show version information

GLOBAL FLAGS:
  --config string     Path to config file (default: .slingshot/config/ss-installer-config-azure.json)
  --namespace string  Kubernetes namespace (default: from config)
  --verbose           Enable verbose logging
  --debug             Enable debug logging
  -h, --help         Help for slingshot

EXAMPLES:
  # Quick start workflow
  slingshot config init
  slingshot setup-tools
  slingshot package-pull
  slingshot provision-infra
  slingshot install-app

  # Troubleshoot deployment
  slingshot troubleshoot

  # Upgrade applications
  slingshot upgrade-app --group portal-platform

  # Get help for specific command
  slingshot install-app --help

DOCUMENTATION:
  📖 User Guide: ./USER-GUIDE.md
  🌐 Online Docs: https://docs.sapientslingshot.com
  💬 Support: slingshot-support@publicissapient.com

Use "slingshot [command] --help" for more information about a command.
```

#### Command-Specific Help Examples

**install-app help:**
```bash
slingshot install-app --help
```

**Output:**
```
Install applications to Kubernetes cluster

USAGE:
  slingshot install-app [flags]

FLAGS:
  --group string         Install specific group only (e.g., "portal-platform")
  --skip-hooks           Skip pre/post-deployment hooks (not recommended)
  --skip-wait            Don't wait for pods to be ready (not recommended)
  --timeout duration     Per-deployment timeout (default: 10m)
  --parallel            Deploy applications in parallel where possible
  --dry-run             Show what would be deployed without deploying
  --force               Force reinstall even if already deployed
  -h, --help            Help for install-app

EXAMPLES:
  # Install all applications
  slingshot install-app

  # Install specific group
  slingshot install-app --group portal-platform

  # Dry run
  slingshot install-app --dry-run

  # Install with custom timeout
  slingshot install-app --timeout 20m
```

**troubleshoot help:**
```bash
slingshot troubleshoot --help
```

**Output:**
```
Run comprehensive health diagnostics on deployed infrastructure and applications

USAGE:
  slingshot troubleshoot [flags]

DESCRIPTION:
  Performs 8-phase health check:
  1. Cloud authentication
  2. Kubernetes cluster connectivity
  3. Pod status and health
  4. Inter-pod connectivity
  5. Database connectivity
  6. External connectivity
  7. Azure resources (Azure only)
  8. Application health endpoints

FLAGS:
  --skip-interactive    Skip interactive prompts, use config defaults
  --component string    Specific component to troubleshoot (empty = all)
  --collect-logs        Collect detailed logs for support
  -h, --help           Help for troubleshoot

EXAMPLES:
  # Interactive mode (recommended)
  slingshot troubleshoot

  # Automated/CI mode
  slingshot troubleshoot --skip-interactive

  # Collect logs for support
  slingshot troubleshoot --collect-logs
```

---

### 6.10 Vault Sync (Azure Key Vault) 🔐

> 🔐 **Purpose**: Synchronizes secrets between Azure Key Vault and Kubernetes secrets for secure credential management.

> ⚠️ **Azure Only**: This command is only available when using Azure cloud provider.

#### Command Usage

**Basic Usage:**
```bash
slingshot vault-sync
```

**⚙️ Prerequisites:**
- ✅ Azure Key Vault created and configured
- ✅ Managed Identity or Service Principal with Key Vault access
- ✅ Applications deployed (`install-app` completed)
- ✅ Key Vault secrets defined for your applications

#### What is Vault Sync? 🔄

**Problem:**
Applications need sensitive credentials (database passwords, API keys, etc.) but:
- ❌ Environment variables in pod specs are visible
- ❌ ConfigMaps are not encrypted
- ❌ Manual secret management is error-prone

**Solution:**
Vault sync automatically:
- ✅ Pulls secrets from Azure Key Vault
- ✅ Creates/updates Kubernetes secrets
- ✅ Triggers pod restarts to use new secrets
- ✅ Maintains audit trail of secret access

#### Interactive Process 🧙

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 Azure Key Vault Sync
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Detecting Azure Key Vault...
   • Key Vault: kv-slingshot-prod
   • Resource Group: rg-slingshot-prod-eus2
   • Region: East US 2

📝 Select sync mode:
   1. Sync all secrets
   2. Sync specific secret
   3. List available secrets
   4. Validate Key Vault access

Choice [1-4]: 1

🔐 Checking Key Vault access...
   ✓ Managed identity authenticated
   ✓ Key Vault access granted

📋 Discovered secrets in Key Vault:
   1. postgres password
   2. mongodb-connection-string
   3. rabbitmq-admin-password
   4. api-auth-jwt-secret
   5. portal-session-secret
   6. llm-api-key
   ... (12 secrets total)

🔄 Syncing secrets to Kubernetes...
   [1/12] postgres-password → Secret/portal-db-creds... ✓
   [2/12] mongodb-connection-string → Secret/portal-mongo-creds... ✓
   [3/12] rabbitmq-admin-password → Secret/rabbitmq-creds... ✓
   [4/12] api-auth-jwt-secret → Secret/auth-secrets... ✓
   [5/12] portal-session-secret → Secret/portal-secrets... ✓
   [6/12] llm-api-key → Secret/llm-secrets... ✓
   ... (6 more)

✅ All secrets synced successfully!

⚠️  Some pods need restart to use new secrets:
   • portal-backend (3 pods)
   • api-auth-v2 (2 pods)
   • portal-frontend (2 pods)

? Restart pods now? (yes/no): yes

🔄 Restarting pods...
   ✓ Restarted portal-backend (3 pods)
   ✓ Restarted api-auth-v2 (2 pods)
   ✓ Restarted portal-frontend (2 pods)

✅ Vault sync completed!

📊 Summary:
   • Secrets synced: 12
   • Kubernetes secrets updated: 8
   • Pods restarted: 7
   • Duration: 2m 45s

📝 Next steps:
   1. Run: slingshot troubleshoot
   2. Verify applications are healthy
   3. Test authentication

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Command Options

```bash
slingshot vault-sync [flags]

Flags:
  --keyvault string      Azure Key Vault name
  --secret string        Sync specific secret only
  --namespace string     Kubernetes namespace (default: from config)
  --list                 List available secrets without syncing
  --validate             Validate Key Vault access only
  --skip-restart         Don't restart pods after sync
  --dry-run              Show what would be synced without syncing
  -h, --help            Help for vault-sync
```

**Examples:**

```bash
# Sync all secrets
slingshot vault-sync

# Sync specific secret
slingshot vault-sync --secret postgres-password

# List available secrets
slingshot vault-sync --list

# Validate access without syncing
slingshot vault-sync --validate

# Dry run
slingshot vault-sync --dry-run
```

#### Secret Mapping Configuration 📝

Create `.slingshot/vault-mapping.yaml` to define how Key Vault secrets map to Kubernetes secrets:

```yaml
secretMappings:
  # PostgreSQL credentials
  - keyVaultSecret: postgres-password
    kubernetesSecret: portal-db-creds
    namespace: slingshot-ns
    keys:
      - keyVaultKey: postgres-password
        secretKey: password

  # MongoDB connection string
  - keyVaultSecret: mongodb-connection-string
    kubernetesSecret: portal-mongo-creds
    namespace: slingshot-ns
    keys:
      - keyVaultKey: mongodb-connection-string
        secretKey: connectionString

  # JWT secret for auth
  - keyVaultSecret: api-auth-jwt-secret
    kubernetesSecret: auth-secrets
    namespace: slingshot-ns
    keys:
      - keyVaultKey: api-auth-jwt-secret
        secretKey: jwtSecret

  # LLM API key
  - keyVaultSecret: llm-api-key
    kubernetesSecret: llm-secrets
    namespace: slingshot-ns
    keys:
      - keyVaultKey: llm-api-key
        secretKey: apiKey

# Pods to restart after sync
podRestarts:
  - deployment: portal-backend
    namespace: slingshot-ns
  - deployment: api-auth-v2
    namespace: slingshot-ns
  - deployment: api-llm
    namespace: slingshot-ns
```

#### Setting Up Azure Key Vault 🔧

**1. Create Key Vault:**
```bash
# Create Key Vault
az keyvault create \
  --name kv-slingshot-prod \
  --resource-group rg-slingshot-prod-eus2 \
  --location eastus2

# Enable RBAC
az keyvault update \
  --name kv-slingshot-prod \
  --enable-rbac-authorization true
```

**2. Create and Store Secrets:**
```bash
# Store PostgreSQL password
az keyvault secret set \
  --vault-name kv-slingshot-prod \
  --name postgres-password \
  --value "YourSecurePassword123!"

# Store MongoDB connection string
az keyvault secret set \
  --vault-name kv-slingshot-prod \
  --name mongodb-connection-string \
  --value "mongodb://mongo-server:27017/portal"

# Store JWT secret
az keyvault secret set \
  --vault-name kv-slingshot-prod \
  --name api-auth-jwt-secret \
  --value "$(openssl rand -base64 32)"
```

**3. Grant Access to AKS:**
```bash
# Get AKS managed identity
AKS_IDENTITY=$(az aks show \
  --resource-group rg-slingshot-prod-eus2 \
  --name aks-slingshot-prod-01 \
  --query identityProfile.kubeletidentity.objectId -o tsv)

# Grant Key Vault Secrets Officer role
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee $AKS_IDENTITY \
  --scope /subscriptions/<sub-id>/resourceGroups/rg-slingshot-prod-eus2/providers/Microsoft.KeyVault/vaults/kv-slingshot-prod
```

#### Troubleshooting Vault Sync 🔧

**❌ Issue: "Access denied to Key Vault"**
```
Error: The user, group or application does not have secrets get permission on key vault
```
**✅ Solution:**
```bash
# Verify managed identity
az aks show \
  --resource-group <rg> \
  --name <cluster> \
  --query identityProfile.kubeletidentity.objectId -o tsv

# Check role assignments
az role assignment list \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<kv> \
  --output table

# Grant permissions
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee <identity-object-id> \
  --scope <keyvault-resource-id>
```

**❌ Issue: "Secret not found in Key Vault"**
```
Error: Secret 'postgres-password' not found in Key Vault 'kv-slingshot-prod'
```
**✅ Solution:**
```bash
# List secrets
az keyvault secret list --vault-name kv-slingshot-prod --output table

# Create missing secret
az keyvault secret set \
  --vault-name kv-slingshot-prod \
  --name postgres-password \
  --value "YourPassword"
```

**❌ Issue: "Pod still using old secret"**
```
⚠️  Pod portal-backend-xyz still has old secret values
```
**✅ Solution:**
```bash
# Force pod restart
kubectl rollout restart deployment/portal-backend -n slingshot-ns

# Verify new secret mounted
kubectl describe pod portal-backend-xxx -n slingshot-ns | grep -A5 "Mounts:"

# Check secret updated
kubectl get secret portal-db-creds -n slingshot-ns -o yaml
```

#### Best Practices 💡

- ✅ **Use managed identity** - more secure than service principals
- ✅ **Enable RBAC on Key Vault** - for fine-grained access control
- ✅ **Rotate secrets regularly** - sync after rotation
- ✅ **Audit secret access** - enable Key Vault logging
- ✅ **Test in non-production** - before syncing production secrets
- 🔒 **Restrict Key Vault network** - use private endpoints for production
- 🔒 **Enable soft delete** - protect against accidental deletion
- 📊 **Monitor sync operations** - track sync history and failures

---

## 7. Application Groups & Deployment Order 📊

> 📋 **Reference Guide**: Use this section to understand what each deployment group does.

### Group 1: 📋 Common Configuration
- **Applications**: 1
- **Essential**: ⚠️ Yes (Required)
- **Hooks**: ❌ No hooks

### Group 2: 🔐 Common Secrets
- **Applications**: 1
- **Essential**: ⚠️ Yes (Required)
- **Hooks**: ❌ No hooks

### Group 3: 🏗️ Infrastructure Components
- **Applications**: 3 (RabbitMQ, Chroma, Grafana)
- **Essential**: ⚠️ Yes (Required)
- **Hooks**: ❌ No hooks

### Group 4: 🔌 Bodhi Agent Framework
- **Applications**: 5
- **Essential**: ⚠️ Yes (Required)
- **Hooks**: ❌ No hooks

### Group 5: 🌐 Portal Applications
- **Applications**: 12 (Portal DB, Auth, Backend, Frontend)
- **Essential**: ✅ Yes (if using Portal UI)
- **Pre-hooks**: 🗄️ Database collections + 👤 default client
- **Post-hooks**: ⚙️ Config server update
- **Critical Fields**: 🔑 21 fields

> 💡 **Tip**: Portal is the main user interface for Slingshot. Essential for most deployments.

### Group 6: 🧠 NEXA Intelligence
- **Applications**: 3 (Config Server, API, Indexer)
- **Essential**: ✅ Yes (if using NEXA AI features)
- **Pre-hooks**: ⚙️ Config server setup
- **Features**: 🔍 Completion detection, 🌍 cross-platform

> 💡 **Tip**: CONFIG_SERVER_BASE_DIR_NAME must match your config repository structure.

### Group 7: 📝 Backlog AI & Platform
- **Applications**: 8 (Backend, Frontend, Workers, APIs)
- **Essential**: 🟡 Optional (AI-powered project management features)
- **Pre-hooks**: 🆕 Fresh setup + 🗄️ Database setup
- **Features**: 
  * ✍️ SCRIPTS_LIST prompting
  * ✅ 21 critical field validation
  * 🔗 Service URL auto-generation
  * 💬 Automatic SCRIPTS_LIST commenting
  * 🆔 DEFAULT_CLIENT_ID special handling

> ⚠️ **Important**: Requires comprehensive database and authentication configuration.

### Group 8: 🔄 AIPP Services
- **Applications**: Variable (Platform integration services)
- **Essential**: 🟡 Optional
- **Hooks**: ❌ No hooks currently

### Group 9: 🤖 App Modernization Suite
- **Applications**: Variable (Modernization Service, UI, Workers)
- **Essential**: 🟡 Optional (Legacy application modernization workflows)
- **Pre-hooks**: 🗄️ MongoDB + ⚙️ Config server
- **Features**: ✅ Completion detection

> 💡 **Use Case**: Enable this group if you need AI-assisted application modernization capabilities.

### Group 10: 🎯 AISM Services
- **Applications**: Variable (Service mesh and observability)
- **Essential**: 🟡 Optional (Enhanced monitoring and tracing)
- **Hooks**: ❌ No hooks currently

> 💡 **Tip**: Enable for production environments requiring advanced observability.

---

## 8. Pre & Post-Deployment Hooks 🪝

### 8.1 Hook Execution Architecture ⚙️

> 💡 **Smart Execution**: All hooks use intelligent completion detection for reliable automation.

All hooks use **runCommandWithIdleTimeout()** with:
- ✅ **Completion Detection**: Monitors for "EXECUTION COMPLETED"
- ⚡ **Immediate Termination**: Kills process after completion
- 🛠️ **Error Filtering**: Filters "Input redirection" errors
- 🌍 **Cross-Platform**: Linux, macOS, Windows support
- ⏱️ **Timeout**: 3-minute idle timeout

### 8.2 Critical Fields (21 Total) 🔑

> ⚠️ **Production Critical**: All 21 fields must be properly configured for system functionality.

**🗄️ Database:**
- POSTGRES_HOST, POSTGRES_PORT, POSTGRES_USER_NAME, POSTGRES_PASSWORD
- MONGO_CONN_STRING

**👤 Client & Auth:**
- DEFAULT_CLIENT_ID, PORTAL_SVC_ADMIN_CLIENT_ID
- PORTAL_SVC_ADMIN_CLIENT_SECRET, PORTAL_SVC_ADMIN_APP_SECRET

**🌍 Environment:**
- DOMAIN, DEFAULT_ACCOUNT_NAME, DEFAULT_PROJECT_NAME
- ENT_INDUSTRY_ID_SLINGSHOT, ENT_LOGIN_SOURCE

**📊 Enterprise DB:**
- ENT_POSTGRES_USER, ENT_POSTGRES_DB_NAME
- ENT_POSTGRES_DB_HOST, ENT_POSTGRES_DB_PORT

**👨‍💼 Admin:**
- UPDATED_BY, DEFAULT_ADMIN_USERS

### 8.3 Service URL Auto-Generation 🔗

> ⚙️ **Automatic**: From DOMAIN field, 10 service URLs are automatically generated.

From DOMAIN field, 10 service URLs are automatically generated:
```
🔐 ENT_AUTH_URL
⚙️ ENT_CONFIG_SERVICE_URL
📊 ENT_PROJECT_URL
👤 ENT_USER_PROFILE_URL
🔑 SS_AUTH_URL
📝 SS_BACKLOG_URL
🌐 SS_PORTAL_URL
📨 SS_EVENT_CONSUMER_URL
🏷️ SS_LICENSE_URL
💬 SS_WEBCHAT_URL
```

> ✅ **Benefit**: No manual configuration needed. URLs follow Kubernetes service discovery pattern.

### 8.4 SCRIPTS_LIST Management 📜

> 💡 **Smart Prompting**: Interactive prompt allows customization or quick acceptance of defaults.

**✍️ Prompting:**
```
📝 SCRIPTS_LIST (default: f001_setup_postgres.py): 
```

**💬 After Execution:**
```ini
# ✅ Before:
SCRIPTS_LIST="f001_setup_postgres.py"

# ✅ After (auto-commented):
# SCRIPTS_LIST="f001_setup_postgres.py"
```

> ⚠️ **Important**: Auto-commenting prevents re-execution on subsequent deployments.

---

## 9. Troubleshooting & Health Diagnostics 🔍

> 💡 **New in v2.0**: Comprehensive automated diagnostics tool for deployment troubleshooting

### 9.1 Troubleshoot Command Overview 🩺

The `slingshot troubleshoot` command is your first-line diagnostic tool for identifying and resolving deployment issues. It performs **8 comprehensive health check phases** from your cluster's perspective, testing everything from cloud authentication to application health endpoints.

> ⚡ **Quick Start**: Run `slingshot troubleshoot` after deployment to verify system health.

**🎯 When to Use:**
- ✅ After completing `install-app` to verify deployment health
- 🔴 When applications are not accessible or responding
- ⚠️ When pods are restarting or failing
- 🗄️ When experiencing database connectivity issues
- 🌐 When users report intermittent access problems
- 📊 Before promoting to production (pre-flight check)

**⏱️ Duration:** 3-8 minutes (depending on cluster size and enabled checks)

---

### 9.2 Running Troubleshoot - Interactive Mode 🧙

> 📝 **Note**: Interactive mode guides you through configuration with smart defaults from your installer config.

**Step 1: Launch Troubleshoot**
```bash
slingshot troubleshoot
```

**Step 2: Configuration Prompts**

The wizard loads defaults from your `ss-installer-config-azure.json` and prompts for any missing values:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🩺 Deployment Troubleshooting
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 Subscription ID [00000000-0000-0000-0000-000000000000]: 
```
> **💡 Action**: Review the default value in brackets. If correct, press **Enter**. Otherwise, type the correct value.

```
📝 Resource Group [rg-slingshot-prod-eus2]: 
```
> **💡 Action**: Press **Enter** to use default, or type your resource group name.

```
📝 Region [eastus]: 
```
> **💡 Action**: Press **Enter** for default region, or type your region (e.g., `westus2`, `centralus`).

```
📝 Kubernetes Cluster Name [aks-slingshot-prod-01]: 
```
> **💡 Action**: Press **Enter** to accept, or type your cluster name.

```
📝 Kubernetes Namespace [slingshot-ns]: 
```
> **💡 Action**: Press **Enter** for default namespace, or type your namespace.

```
📝 Domain URL (optional) [stg3.sapientslingshot.com]: 
```
> **💡 Action**: Press **Enter** to use configured domain, or type your domain without `https://`.

```
📝 Azure KeyVault Name (optional) [kv-slingshot-prod]: 
```
> **💡 Action**: Press **Enter** to skip, or type your KeyVault name for Azure resource checks.

**Step 3: Database Testing (Optional)**

```
? Test database connectivity?
  ▸ Yes
    No
```
> **💡 Action**: Use **arrow keys** to select. Press **Enter**.
> 
> ⚠️ **Choose "Yes"** if you want to validate MongoDB and PostgreSQL connectivity from within your cluster.
> 
> ⚠️ **Choose "No"** to skip database tests (useful for quick infrastructure checks).

**If you select "Yes":**

```
📝 MongoDB Connection URL (optional) [mongodb://mongo-server:27017/database]: 
```
> **💡 Action**: Press **Enter** to use default from config, or paste your MongoDB connection string.
> 
> **Format**: `mongodb://host:port/database` or `mongodb+srv://...`

```
📝 PostgreSQL Connection URL (optional) [pgsql-server.postgres.database.azure.com]: 
```
> **💡 Action**: Press **Enter** to use default, or type your PostgreSQL hostname.

```
📝 PostgreSQL User [postgres]: 
```
> **💡 Action**: Press **Enter** for default user, or type your PostgreSQL username.

```
📝 PostgreSQL Database [slingshot]: 
```
> **💡 Action**: Press **Enter** for default database name, or type your database name.

```
🔒 PostgreSQL Password: 
```
> **💡 Action**: Type your PostgreSQL password. **Input is masked** for security.
> 
> **🔐 Security**: Password is stored in-memory only and never written to disk.

---

### 9.3 Understanding the 8 Health Check Phases 📊

> 🔍 **Real-Time Progress**: Watch as troubleshoot executes each phase with live status updates.

The troubleshoot command performs **8 sequential phases**, each testing critical deployment components:

---

#### **Phase 1: 🔑 Cloud Authentication**

**What it checks:**
- ✅ Azure CLI login status (`az account show`)
- ✅ Access to specified subscription
- ✅ Active account details

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔑 Phase 1: Cloud Authentication
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Cloud Login Status: Azure CLI authenticated successfully
   • Azure account is active
   Duration: 1.2s

✅ Subscription Access: Subscription 00000000-... is accessible
   Duration: 0.8s
```

**Common Issues:**
- 🔴 **Not logged in**: Run `az login` before troubleshoot
- ⚠️ **Wrong subscription**: Use `az account set --subscription <id>`

---

#### **Phase 2: 🚀 Kubernetes Cluster Connectivity**

**What it checks:**
- ✅ kubectl configuration and cluster connection
- ✅ Kubernetes API server health
- ✅ Node status and readiness
- ✅ Available node resources (CPU, memory)

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 Phase 2: Kubernetes Cluster Connectivity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Cluster Connection: Connected to aks-slingshot-prod-01
   • API Server: https://aks-prod-xxxxx.hcp.eastus.azmk8s.io:443
   • Version: v1.28.5
   Duration: 0.9s

✅ Node Status: 3/3 nodes ready
   • aks-agentpool-12345678-vmss000000 (Ready)
   • aks-agentpool-12345678-vmss000001 (Ready)
   • aks-agentpool-12345678-vmss000002 (Ready)
   Duration: 1.4s
```

**Common Issues:**
- 🔴 **Cannot connect**: Run `az aks get-credentials --resource-group <rg> --name <cluster>`
- 🔴 **Nodes not ready**: Check node logs (`kubectl describe node <name>`)

---

#### **Phase 3: 💚 Pod Status and Health**

**What it checks:**
- ✅ All pods in specified namespace
- ✅ Running vs. Total pod count
- ✅ Pod readiness status
- ✅ Recent restarts (warning if pods restarting frequently)

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💚 Phase 3: Pod Status and Health
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Pod Status Check: 45/45 pods running in namespace slingshot-ns
   Duration: 2.1s
```

**Warning Example:**
```
⚠️  Pod Restart Warning: Pod api-llm-6d8f9b7c-xyz has 3 restarts
   • Check pod logs: kubectl logs api-llm-6d8f9b7c-xyz -n slingshot-ns
   • Describe pod: kubectl describe pod api-llm-6d8f9b7c-xyz -n slingshot-ns
```

**Common Issues:**
- 🔴 **CrashLoopBackOff**: Check logs with `kubectl logs <pod-name> -n slingshot-ns`
- 🔴 **ImagePullBackOff**: Verify registry credentials and image name
- 🔴 **Pending**: Check resource requests (`kubectl describe pod <name>`)

---

#### **Phase 4: 🔗 Inter-Pod Connectivity**

**What it checks:**
- ✅ DNS resolution between pods (using busybox test pod)
- ✅ Service discovery within cluster
- ✅ Health endpoint accessibility (tests `/api/v2/auth/health` on api-auth-v2 service)

> 🎯 **Cluster Perspective**: Tests run FROM within the cluster using temporary pods, validating the same network path your applications use.

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔗 Phase 4: Inter-Pod Connectivity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Service Discovery: DNS resolution working
   • kubernetes.default.svc.cluster.local resolves correctly
   • api-auth-v2-svc.slingshot-ns.svc.cluster.local resolves correctly
   Duration: 2.3s

✅ Health Endpoint Check: api-auth-v2 service is healthy
   • URL: http://api-auth-v2-svc.slingshot-ns.svc.cluster.local:8080/api/v2/auth/health
   • Response: HTTP 200 OK
   Duration: 1.8s
```

**Common Issues:**
- 🔴 **DNS resolution failed**: Check CoreDNS pods in kube-system namespace
- 🔴 **Health endpoint timeout**: Service may not be ready yet, check pod logs
- ⚠️ **HTTP 503**: Service is starting, wait and retry

---

#### **Phase 5: 🗄️ Database Connectivity**

> 🎯 **Cluster Perspective**: Tests run FROM within the cluster using temporary pods with database clients.

**What it checks:**
- ✅ MongoDB connection from cluster (using `python:3.11-slim` pod with pymongo)
- ✅ PostgreSQL connection from cluster (using `postgres:15-alpine` pod with psql)
- ✅ Database authentication
- ✅ Basic query execution

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🗄️ Phase 5: Database Connectivity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ MongoDB Connection (from cluster): MongoDB is accessible from cluster
   • Databases: ['admin', 'config', 'portal', 'authorization']
   Duration: 5.2s

✅ PostgreSQL Connection (from cluster): PostgreSQL is accessible from cluster
   • Version: PostgreSQL 15.3 on x86_64-pc-linux-gnu
   Duration: 3.8s
```

**Common Issues:**
- 🔴 **Connection timeout**: Check database firewall rules
  - **Azure**: Add AKS egress IP to PostgreSQL firewall
  - Run: `az postgres server firewall-rule create --resource-group <rg> --server-name <server> --name AllowAKS --start-ip-address <ip> --end-ip-address <ip>`
- 🔴 **Authentication failed**: Verify username and password
- 🔴 **Database not found**: Check database name spelling
- ⚠️ **SSL required**: Add `?ssl=true` to connection string

---

#### **Phase 6: 🌐 External Connectivity**

**What it checks:**
- ✅ Egress connectivity from cluster to internet
- ✅ DNS resolution for external domains
- ✅ HTTPS connectivity to external services

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Phase 6: External Connectivity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ External DNS: External DNS resolution working
   • Successfully resolved: www.google.com
   Duration: 0.7s

✅ Egress Connectivity: Cluster can reach external endpoints
   • Test endpoint: https://www.google.com
   Duration: 1.2s
```

**Common Issues:**
- 🔴 **Egress blocked**: Check network security groups (NSGs) or firewall rules
- 🔴 **DNS resolution failed**: Check cluster DNS configuration

---

#### **Phase 7: 🔐 Azure Resources** *(Azure only)*

**What it checks:**
- ✅ Azure KeyVault connectivity
- ✅ Access to KeyVault secrets
- ✅ Managed identity configuration (if applicable)

**Success Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 Phase 7: Azure Resources
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ KeyVault Access: Can access KeyVault kv-slingshot-prod
   Duration: 1.5s

✅ Secret Access: Can read secrets from KeyVault
   • Test secret accessible
   Duration: 0.9s
```

**Common Issues:**
- 🔴 **Access denied**: Verify KeyVault access policies or RBAC permissions
- 🔴 **KeyVault not found**: Check KeyVault name and subscription

---

### 9.4 Understanding the Enhanced Summary Report 📈

> 🎉 **New in v2.0**: Comprehensive summary with category breakdown, timing, and recommendations.

After completing all phases, troubleshoot displays a **detailed summary report**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Troubleshooting Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ╔══════════════════════════════════════════════════════════════╗
  ║                    OVERALL HEALTH STATUS                     ║
  ╚══════════════════════════════════════════════════════════════╝

  Total Checks:  14
  ✅ Passed:     12 (85.7%)
  ⚠️  Warnings:   1 (7.1%)
  ❌ Failed:      1 (7.1%)

  ⏱️  Total Time: 18.4s
  📊 Average:    1.3s per check

  ╔══════════════════════════════════════════════════════════════╗
  ║                  DETAILED CATEGORY BREAKDOWN                 ║
  ╚══════════════════════════════════════════════════════════════╝

  ✅ Cloud Authentication
     ✓: 2  

  ✅ Kubernetes Connectivity
     ✓: 2  

  ⚠️  Pod Status
     ✓: 1  ⚠: 1  

  ✅ Inter-Pod Connectivity
     ✓: 2  

  ❌ Database Connectivity
     ✓: 1  ✗: 1

  ✅ External Connectivity
     ✓: 2  

  ✅ Azure Resources
     ✓: 2  
```

**Reading the Summary:**

1. **Overall Health Status** - Quick snapshot:
   - **85%+ passed**: ✅ System healthy
   - **70-85% passed**: ⚠️ Review warnings
   - **<70% passed**: ❌ Troubleshooting required

2. **Category Breakdown** - See which areas need attention:
   - ✅ **Green categories**: All checks passed
   - ⚠️ **Yellow categories**: Some warnings (review recommended)
   - ❌ **Red categories**: Failed checks (immediate action required)

3. **Timing Information**:
   - **Total Time**: Full troubleshoot duration
   - **Average**: Performance baseline (typical: 1-2s per check)

---

**Failed Checks Section** (if any failures):

```
  ╔══════════════════════════════════════════════════════════════╗
  ║                       FAILED CHECKS                          ║
  ╚══════════════════════════════════════════════════════════════╝

    ❌ [Database Connectivity] PostgreSQL Connection (from cluster)
       ↳ Cannot connect to PostgreSQL from cluster

       ⚠ Error: dial tcp 10.0.2.15:5432: i/o timeout

       💡 Details:
          • Check database firewall rules
          • Verify AKS egress IP is whitelisted
          • Confirm connection string is correct
          • Test direct connectivity: kubectl run test-pg --rm -i --restart=Never --image=postgres:15-alpine -- psql -h <host> -U <user>
```

> 💡 **Action Required**: Address all failed checks before proceeding to production.

---

**Warnings Section** (if any warnings):

```
  ╔══════════════════════════════════════════════════════════════╗
  ║                         WARNINGS                             ║
  ╚══════════════════════════════════════════════════════════════╝

    ⚠️  [Pod Status] Pod Restart Warning
       ↳ Pod api-llm-6d8f9b7c-xyz has 3 restarts

          • Check logs: kubectl logs api-llm-6d8f9b7c-xyz -n slingshot-ns
          • Describe pod: kubectl describe pod api-llm-6d8f9b7c-xyz -n slingshot-ns
```

> 💡 **Recommendation**: Investigate warnings to prevent future issues.

---

**Final Verdict:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  🎉 All systems operational! Deployment environment is healthy.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Possible verdicts:**
- 🎉 **All systems operational**: 0 failures, 0 warnings
- ✅ **System functional**: 0 failures, some warnings (review recommended)
- ⚠️ **System degraded**: 1-2 critical issues (immediate attention required)
- ❌ **System unhealthy**: 3+ critical issues (troubleshooting required)

---

### 9.5 Non-Interactive Mode (CI/CD) 🤖

> 🔄 **Automation**: Use `--skip-interactive` for automated pipelines.

For automated troubleshooting in CI/CD pipelines:

```bash
slingshot troubleshoot --skip-interactive
```

**Behavior:**
- ✅ Uses all values from configuration file
- ❌ No prompts (fails if required values missing)
- 📊 Returns exit code 0 (success) or 1 (failure)
- 📝 Full output to stdout for log capture

**Example CI/CD Usage:**
```yaml
# Azure DevOps Pipeline
- task: Bash@3
  displayName: 'Post-Deployment Health Check'
  inputs:
    targetType: 'inline'
    script: |
      ./slingshot troubleshoot --skip-interactive
      if [ $? -ne 0 ]; then
        echo "##vso[task.logissue type=error]Health check failed"
        exit 1
      fi
```

---

### 9.6 Common Troubleshooting Scenarios 🔧

#### **Scenario 1: Pods Not Ready After Deployment**

**Symptoms:**
- ⚠️ Phase 3 shows warnings like "Pod not ready after 5 minutes"
- ❌ Applications not accessible

**Investigation Steps:**

1. **Check pod status:**
   ```bash
   kubectl get pods -n slingshot-ns
   ```

2. **Check specific pod:**
   ```bash
   kubectl describe pod <pod-name> -n slingshot-ns
   ```

3. **Check logs:**
   ```bash
   kubectl logs <pod-name> -n slingshot-ns
   kubectl logs <pod-name> -n slingshot-ns --previous  # if pod restarted
   ```

**Common Fixes:**
- **ImagePullBackOff**: Verify registry credentials (`kubectl get secret -n slingshot-ns`)
- **CrashLoopBackOff**: Check application logs for errors
- **Pending**: Check node resources (`kubectl describe node`)

---

#### **Scenario 2: Database Connectivity Failures**

**Symptoms:**
- ❌ Phase 5 shows "Cannot connect to PostgreSQL from cluster"
- ❌ Phase 5 shows "MongoDB connection failed"

**Investigation Steps:**

1. **Verify connection string:**
   ```bash
   # Check your installer config
   cat .slingshot/config/ss-installer-config-azure.json | grep -i postgres
   ```

2. **Test from cluster (PostgreSQL):**
   ```bash
   kubectl run test-pg --rm -i --restart=Never \
     --image=postgres:15-alpine -n slingshot-ns -- \
     psql -h <host> -U <user> -d <database> -c "SELECT version();"
   ```

3. **Test from cluster (MongoDB):**
   ```bash
   kubectl run test-mongo --rm -i --restart=Never \
     --image=python:3.11-slim -n slingshot-ns -- \
     sh -c "pip install -q pymongo && python3 -c 'from pymongo import MongoClient; client = MongoClient(\"<connection-string>\"); print(client.server_info())'"
   ```

**Common Fixes:**

**For Azure PostgreSQL:**
```bash
# Add firewall rule for AKS cluster
az postgres server firewall-rule create \
  --resource-group <rg> \
  --server-name <server> \
  --name AllowAKS \
  --start-ip-address <aks-egress-ip> \
  --end-ip-address <aks-egress-ip>
```

**For MongoDB:**
- Verify connection string format: `mongodb://host:27017/database` or `mongodb+srv://...`
- Check network security groups allow port 27017
- Verify authentication database if using MongoDB Atlas

---

#### **Scenario 3: Health Endpoint Not Responding**

**Symptoms:**
- ❌ Phase 4 shows "Health endpoint check failed"
- ⚠️ HTTP 503 or timeout errors

**Investigation Steps:**

1. **Check service exists:**
   ```bash
   kubectl get svc -n slingshot-ns | grep api-auth-v2
   ```

2. **Check pod is ready:**
   ```bash
   kubectl get pods -n slingshot-ns | grep api-auth-v2
   ```

3. **Test endpoint manually:**
   ```bash
   kubectl run test-curl --rm -i --restart=Never \
     --image=curlimages/curl:latest -n slingshot-ns -- \
     curl -v http://api-auth-v2-svc.slingshot-ns.svc.cluster.local:8080/api/v2/auth/health
   ```

**Common Fixes:**
- **HTTP 503**: Service is starting, wait 2-3 minutes and retry
- **Connection refused**: Pod not ready, check pod logs
- **Timeout**: Service may be misconfigured, check service ports

---

#### **Scenario 4: External Connectivity Issues**

**Symptoms:**
- ❌ Phase 6 shows "External DNS resolution failed"
- ❌ Unable to reach external APIs

**Investigation Steps:**

1. **Test DNS resolution:**
   ```bash
   kubectl run test-dns --rm -i --restart=Never \
     --image=busybox:latest -n slingshot-ns -- \
     nslookup www.google.com
   ```

2. **Check CoreDNS:**
   ```bash
   kubectl get pods -n kube-system | grep coredns
   kubectl logs -n kube-system <coredns-pod>
   ```

**Common Fixes:**
- **DNS issues**: Restart CoreDNS pods
- **Egress blocked**: Check NSG/firewall rules
- **Azure**: Verify NAT Gateway or Azure Firewall configuration

---

### 9.7 Advanced Troubleshooting Tips 💡

**Enable Verbose Logging:**
```bash
slingshot troubleshoot --verbose
```

**Skip Database Tests (Faster):**
- When prompted "Test database connectivity?", select **No**
- Useful for quick infrastructure checks

**Test Specific Components:**
```bash
# Quick kubernetes check
kubectl cluster-info
kubectl get nodes
kubectl get pods -n slingshot-ns

# Quick database check from your machine
psql -h <host> -U <user> -d postgres
mongo <connection-string> --eval "db.adminCommand('ping')"
```

**Collect Logs for Support:**
```bash
# Collect all pod logs
kubectl logs -n slingshot-ns --all-containers=true --selector='app=api-auth-v2' > auth-logs.txt

# Describe all resources
kubectl describe all -n slingshot-ns > resources.txt

# Get events
kubectl get events -n slingshot-ns --sort-by='.lastTimestamp' > events.txt
```

---

### 9.8 Known Issues & Resolutions ✅

#### **Issue 1: Hook Hangs (RESOLVED)**

**❌ Previous Issue:** Scripts hanging with "Input redirection" errors

**✅ Solution (Implemented in v2.0):**
- All hooks use completion detection
- Process terminates on "EXECUTION COMPLETED"
- Error messages filtered
- Works on all platforms

---

#### **Issue 2: Service URLs Not Generated (RESOLVED)**

**❌ Previous Issue:** Service URLs showing `<<DOMAIN_URL>>` placeholder

**✅ Solution (Implemented in v2.0):**
- All hooks auto-generate service URLs from DOMAIN field
- Format: `https://<domain>/api/v2/auth`

**🔍 Verification:**
```bash
cat .slingshot/deployment-scripts/*/portal_platform/.env_main | grep ENT_AUTH_URL
# Should show: ENT_AUTH_URL="https://stg3.sapientslingshot.com/api/v2/auth"
```

---

#### **Issue 3: Multiple Password Prompts (RESOLVED)**

**❌ Previous Issue:** PostgreSQL password prompted 7+ times during troubleshoot

**✅ Solution (Implemented in v2.0):**
- Password prompted once and stored in-memory
- Secure handling with wizard.PromptPasswordWithDefault()
- Never written to disk

---

### 9.9 When to Contact Support 📞

Contact support if troubleshoot shows:

- ❌ **Persistent database failures** after firewall configuration
- ❌ **Cluster connectivity issues** after credential refresh
- ❌ **Multiple pod failures** in essential services
- ❌ **Azure KeyVault access denied** after verifying permissions

**Include in Support Request:**
1. Troubleshoot command output (full summary)
2. Cloud provider and region
3. Kubernetes version (`kubectl version`)
4. Pod logs for failing services
5. Recent changes to infrastructure or configuration

---

## 10. Security & Compliance 🔒

### 10.1 Credential Management 🔑

> ⚠️ **Security First**: Never expose credentials in code or logs!

**✅ Best Practices:**
- 📝 Use environment variables for secrets
- ❌ Never commit credentials to Git
- 🔄 Rotate passwords regularly
- ☁️ Use cloud secret managers

**📝 Example:**
```bash
# Azure Key Vault
az keyvault secret show --vault-name <vault> --name postgres-password --query value -o tsv

# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id postgres-password --query SecretString --output text
```

### 10.2 RBAC Configuration 🛡️

> 👥 **Access Control**: Ensure proper Kubernetes permissions for deployment.

Ensure proper Kubernetes permissions:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: slingshot-deployer
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["create", "update", "patch", "delete", "get", "list"]
```

### 10.3 Audit Logging 📊

> 📝 **Traceability**: All operations are logged for audit and troubleshooting.

All operations logged to:
```
.slingshot/logs/
├── deployment-2026-01-30-14-00-00.log
├── validation-2026-01-30-14-30-00.log
└── errors-2026-01-30.log
```

---

## 11. Best Practices ⭐

### 11.1 Deployment Best Practices 🚀

**✅ DO:**
- 📖 Always run workflow in order
- 🧪 Test in non-production first
- 💾 Backup before changes
- 👀 Monitor during deployment
- ✅ Run validation after deployment

**❌ DON'T:**
- ⛔ Skip validation steps
- 🔐 Hardcode credentials
- 🚫 Run parallel deployments to same cluster
- 🙅 Ignore errors

### 11.2 Configuration Management 🗂️

**📦 Version Control:**
```bash
git add .slingshot/config/
git commit -m "Add production configuration"
git push
```

**Environment Separation:**
```
configs/
├── dev.json
├── staging.json
└── production.json
```

### 11.3 Maintenance Windows 🗓️

> ⏰ **Timing**: Schedule deployments during low-traffic periods to minimize impact.

**🏭 Production Schedule:**
- ⏰ Window: Off-peak hours (2-4 AM)
- ⏱️ Duration: 2-3 hours
- ✅ Prerequisites: Backup verified, rollback plan ready

---

## 12. Reference 📚

### 12.1 Command Reference ⌨️

> 📖 **Quick Reference**: All available Slingshot commands with detailed usage information.

#### Core Deployment Commands

| Command | Purpose | Documentation | Typical Duration |
|---------|---------|---------------|------------------|
| `config init` | Initialize deployment configuration with interactive wizard | [Section 6.1](#61-configuration-initialization-️) | 5-10 minutes |
| `setup-tools` | Install kubectl, helm, terraform, and cloud CLIs | [Section 6.2](#62-setup-tools-️) | 5-10 minutes |
| `package-pull` | Download container images, Helm charts, and scripts | [Section 6.3](#63-pull-packages-) | 10-30 minutes |
| `provision-infra` | Provision Kubernetes cluster and cloud infrastructure | [Section 6.4](#64-provision-infrastructure-️) | 15-30 minutes |
| `validate-infra` | Validate infrastructure readiness for deployment | [Section 6.5](#65-validate-infrastructure-) | 2-5 minutes |
| `install-app` | Deploy all application groups to Kubernetes cluster | [Section 6.6](#66-install-applications-) | 30-60 minutes |
| `validate-app` | Validate deployed applications and health | [Section 6.7](#67-validate-applications-️) | 5-10 minutes |

#### Management Commands

| Command | Purpose | Documentation | When to Use |
|---------|---------|---------------|-------------|
| `upgrade-app` | Upgrade applications to newer version or apply config changes | [Section 6.8](#68-upgrade-application-) | After deployment, for updates |
| `troubleshoot` | Run comprehensive 8-phase health diagnostics | [Section 9](#9-troubleshooting--health-diagnostics-) | After deployment, for issues |
| `vault-sync` | Sync secrets from Azure Key Vault to Kubernetes | [Section 6.10](#610-vault-sync-azure-key-vault-) | For secure credential management |

#### Utility Commands

| Command | Purpose | Documentation | Common Usage |
|---------|---------|---------------|--------------|
| `help` | Display help for commands and options | [Section 6.9](#69-getting-help-) | Getting command syntax |
| `version` | Show Slingshot version information | - | Version verification |
| `package-import` | Import packages for air-gapped deployment | [Section 6.3](#air-gapped-deployment-) | Offline installations |
| `package-export` | Export packages for air-gapped deployment | [Section 6.3](#air-gapped-deployment-) | Creating offline bundles |

---

#### Detailed Command Usage

**config init**
```bash
slingshot config init

# Interactive wizard for:
# - Cloud provider selection (Azure/AWS/GCP)
# - Region and resource configuration
# - Container registry setup
# - Application group selection
# - Slingshot version specification
```

**setup-tools**
```bash
slingshot setup-tools

# Installs:
# - kubectl v1.28+
# - helm v3.12+
# - terraform v1.5+
# - Cloud CLI (az/aws/gcloud)
```

**package-pull**
```bash
# Set registry credentials first
export ACR_USERNAME="myregistry"
export ACR_PASSWORD="<token>"

slingshot package-pull [flags]

Flags:
  --skip-images          Skip container image downloads
  --skip-charts          Skip Helm chart downloads
  --skip-scripts         Skip deployment scripts
  --export PATH          Export for air-gapped deployment
  --validate-checksums   Verify package integrity
  --parallel int         Parallel downloads (default: 3)
  --retry int            Retry attempts (default: 3)
```

**provision-infra**
```bash
slingshot provision-infra [flags]

Flags:
  --auto-approve         Skip interactive approval prompts
  --skip-plan            Skip terraform plan (not recommended)
  --var-file PATH        Custom terraform variables file
  --destroy              Destroy infrastructure
  --state-file PATH      Custom terraform state file
```

**validate-infra**
```bash
slingshot validate-infra

# Validates:
# - Kubernetes API health
# - Node status and resources
# - Storage classes
# - DNS resolution
```

**install-app**
```bash
slingshot install-app [flags]

Flags:
  --group string         Install specific group only
  --skip-hooks           Skip pre/post-deployment hooks
  --skip-wait            Don't wait for pods ready
  --timeout duration     Per-deployment timeout (default: 10m)
  --parallel            Deploy applications in parallel
  --dry-run             Preview deployment without executing
  --force               Force reinstall

Examples:
  slingshot install-app
  slingshot install-app --group portal-platform
  slingshot install-app --dry-run
```

**validate-app**
```bash
slingshot validate-app

# Validates:
# - Pod health status
# - Service endpoints
# - API health checks
# - Database connectivity
# - Performance metrics
```

**upgrade-app**
```bash
slingshot upgrade-app [flags]

Flags:
  --group string         Upgrade specific group
  --app string           Upgrade specific application
  --version string       Target version
  --skip-hooks           Skip pre/post-upgrade hooks
  --force                Force upgrade if version same
  --rollback-on-failure  Auto-rollback on failure
  --dry-run              Preview upgrade
  --timeout duration     Per-upgrade timeout (default: 10m)

Examples:
  slingshot upgrade-app
  slingshot upgrade-app --group portal-platform
  slingshot upgrade-app --version v2.1.0
  slingshot upgrade-app --dry-run
  slingshot upgrade-app --rollback-on-failure
```

**troubleshoot**
```bash
slingshot troubleshoot [flags]

Flags:
  --skip-interactive    Non-interactive mode (use config defaults)
  --component string    Troubleshoot specific component
  --collect-logs        Collect detailed logs for support

Performs 8-phase diagnostics:
  1. Cloud authentication
  2. Kubernetes cluster connectivity
  3. Pod status and health
  4. Inter-pod connectivity (cluster-based)
  5. Database connectivity (from cluster)
  6. External connectivity
  7. Azure resources (Azure only)
  8. Application health endpoints

Examples:
  slingshot troubleshoot                    # Interactive mode
  slingshot troubleshoot --skip-interactive # CI/CD mode
  slingshot troubleshoot --collect-logs     # Support bundle
```

**vault-sync** (Azure only)
```bash
slingshot vault-sync [flags]

Flags:
  --keyvault string      Azure Key Vault name
  --secret string        Sync specific secret only
  --namespace string     Kubernetes namespace
  --list                 List available secrets
  --validate             Validate Key Vault access
  --skip-restart         Don't restart pods after sync
  --dry-run              Preview sync operations

Examples:
  slingshot vault-sync                              # Sync all secrets
  slingshot vault-sync --secret postgres-password   # Sync specific secret
  slingshot vault-sync --list                       # List available secrets
  slingshot vault-sync --validate                   # Test access
```

**help**
```bash
slingshot --help                    # Main help
slingshot <command> --help          # Command-specific help

Examples:
  slingshot install-app --help
  slingshot provision-infra --help
  slingshot troubleshoot --help
```

**package-import / package-export** (Air-gapped)
```bash
# Export packages (internet-connected machine)
slingshot package-pull --export /path/to/export

# Transfer package-export-v2.0.0.tar.gz to air-gapped environment

# Import packages (air-gapped machine)
slingshot package-import --file package-export-v2.0.0.tar.gz
```

---

#### Command Workflow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   DEPLOYMENT WORKFLOW                        │
└─────────────────────────────────────────────────────────────┘

1. config init          → Initialize configuration (5-10 min)
         ↓
2. setup-tools          → Install CLI tools (5-10 min)
         ↓
3. package-pull         → Download packages (10-30 min)
         ↓
4. provision-infra      → Create infrastructure (15-30 min)
         ↓
5. validate-infra       → Verify infrastructure (2-5 min)
         ↓
6. install-app          → Deploy applications (30-60 min)
         ↓
7. validate-app         → Validate deployment (5-10 min)
         ↓
8. troubleshoot         → Health diagnostics (3-8 min)

         ↓ (Optional)

9. upgrade-app          → Update applications (varies)
10. vault-sync          → Sync secrets (2-5 min)

TOTAL TIME: 1.5-3 hours for initial deployment
```

---

#### Global Flags (Available for All Commands)

```bash
--config string        Path to config file
                       (default: .slingshot/config/ss-installer-config-azure.json)

--namespace string     Kubernetes namespace override
                       (default: from config)

--verbose              Enable verbose logging (detailed output)

--debug                Enable debug logging (for troubleshooting)

-h, --help            Show help for command
```

### 12.2 Environment Variables 📝

> 🔐 **Secrets**: All sensitive values should be passed via environment variables.

| Variable | Description | Icon |
|----------|-------------|------|
| `ACR_USERNAME` | Azure Container Registry username | 👤 |
| `ACR_PASSWORD` | Azure Container Registry password | 🔒 |
| `AWS_ACCESS_KEY_ID` | AWS access key | 🔑 |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | 🔐 |
| `POSTGRES_PASSWORD` | PostgreSQL password | 🗄️ |
| `MONGO_CONN_STRING` | MongoDB connection string | 🍃 |

### 12.3 File Locations 📂

> 📍 **Directory Structure**: Understanding where Slingshot stores files.

```
.slingshot/
├── config/
│   └── ss-installer-config-azure.json
├── deployment-scripts/
│   └── ai-client-artifacts-main/
│       └── portal_platform/
│           ├── .env_common
│           ├── .env_main
│           └── run_modules.sh
├── helm-charts/
├── tools/
└── logs/
```

---

## Appendix: Version History 📜

> 📅 **Changelog**: Track major feature additions and improvements.

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2026-02-05 | • ✅ Enhanced hook execution<br>• 🆕 Backlog Fresh Setup hook<br>• 💬 SCRIPTS_LIST auto-commenting<br>• 🔗 Service URL auto-generation<br>• 🔑 Critical fields validation<br>• 🌍 Cross-platform support<br>• 🩺 **NEW** Comprehensive troubleshoot command<br>• 📊 8-phase health diagnostics<br>• 🔍 Cluster-based database testing<br>• 📈 Enhanced summary with category breakdown<br>• ⏱️ Timing and performance metrics<br>• 💡 Actionable remediation suggestions |
| 1.9.0 | 2026-01-15 | • 🤖 App Modernization Suite<br>• ✅ Validation framework |
| 1.8.0 | 2026-01-01 | • ☁️ Multi-cloud support<br>• 🔒 Air-gapped mode |

---

**Document Version**: 2.0  
**Last Updated**: February 5, 2026  
**Maintained By**: Platform Engineering Team

---

## 📞 Support & Resources

### Getting Help 🆘
- 📧 **Email**: [slingshot-support@publicisspient.com](mailto:slingshot-support@publicisspient.com)
- 💬 **Slack**: #slingshot-support
- 📚 **Documentation**: [Internal Wiki](https://wiki.company.com/slingshot)
- 🐛 **Bug Reports**: [JIRA Board](https://jira.company.com/projects/SLING)

### Quick Tips 💡
- ⚡ **Performance**: Use SSD storage for better database performance
- 🔍 **Monitoring**: Enable Grafana dashboards for real-time observability
- 🔄 **Updates**: Check for Slingshot updates monthly
- 📊 **Metrics**: Review deployment logs after each installation
- 🛡️ **Security**: Scan container images regularly for vulnerabilities
- 🩺 **Health Checks**: Run `slingshot troubleshoot` after every deployment
- ✅ **Validation**: Always run troubleshoot before promoting to production
- 📈 **Diagnostics**: Use troubleshoot summary to track deployment health trends
- 🔧 **Quick Fixes**: Review troubleshoot remediation suggestions for common issues

---

**© 2026 Publicis Sapient. All rights reserved.**
