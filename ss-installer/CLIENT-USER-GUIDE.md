# Slingshot Installer - Client User Guide

> **Complete guide for deploying applications using Slingshot Installer across Windows, Linux, and macOS**

---

## Table of Contents

- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [System Requirements](#system-requirements)
- [Running the Installer](#running-the-installer)
- [Main Menu Overview](#main-menu-overview)
- [Complete Deployment Workflow](#complete-deployment-workflow)
  - [1. Help & Documentation](#1-help--documentation)
  - [2. Configuration Management](#2-configuration-management)
  - [3. Setup Tools](#3-setup-tools)
  - [4. Package Pull](#4-package-pull)
  - [5. Provision Infrastructure](#5-provision-infrastructure)
  - [6. Validate Infrastructure](#6-validate-infrastructure)
  - [7. Install Applications](#7-install-applications)
  - [8. Upgrade Applications](#8-upgrade-applications)
  - [9. Validate Applications](#9-validate-applications)
  - [10. Troubleshoot](#10-troubleshoot)
- [Menu Navigation Guide](#menu-navigation-guide)
- [Common Scenarios](#common-scenarios)
- [Best Practices](#best-practices)
- [Support & Resources](#support--resources)

---

## Introduction

Slingshot Installer is an enterprise-grade deployment automation tool designed to simplify Kubernetes application deployments across multiple cloud platforms including Azure, AWS, GCP, and OpenShift. It provides a user-friendly, menu-driven interface that guides you through every step of the deployment process.

### Key Features

- **Interactive Menu System** - No command-line expertise required
- **Multi-Cloud Support** - Works with Azure (AKS), AWS (EKS), GCP (GKE), and OpenShift
- **Automated Tool Installation** - Automatically sets up required tools
- **Guided Workflows** - Step-by-step deployment process
- **Validation Framework** - Pre and post-deployment verification
- **Secure Credential Management** - Credentials stored in memory only, never persisted to disk

---

## Getting Started

### System Requirements

**Operating Systems:**
- **Windows**: Windows 10/11 with WSL2 (Windows Subsystem for Linux)
- **Linux**: Any modern Linux distribution (Ubuntu, RHEL, CentOS, etc.)
- **macOS**: macOS 10.15 (Catalina) or later

**Hardware:**
- **Memory**: Minimum 4GB RAM (8GB recommended)
- **Disk Space**: At least 10GB free space
- **Network**: Active internet connection

**Access Requirements:**
- Cloud provider account (Azure, AWS, GCP, or OpenShift)
- Appropriate permissions to create and manage Kubernetes clusters
- Network access to your cloud provider's APIs

---

## Running the Installer

### For Windows Users (WSL2)

1. Open WSL2 terminal (Ubuntu or your preferred distribution)
2. Navigate to the installer directory
3. Run the installer:
   ```
   ./bin/slingshot
   ```

### For Linux Users

1. Open your terminal application
2. Navigate to the installer directory
3. Make the binary executable (first time only):
   ```
   chmod +x ./bin/slingshot
   ```
4. Run the installer:
   ```
   ./bin/slingshot
   ```

### For macOS Users

1. Open Terminal application
2. Navigate to the installer directory
3. Make the binary executable (first time only):
   ```
   chmod +x ./bin/slingshot
   ```
4. Run the installer:
   ```
   ./bin/slingshot
   ```

**Note:** On macOS, if you see a security warning, go to System Preferences > Security & Privacy and click "Allow" for the Slingshot application.

---

## Main Menu Overview

When you launch Slingshot, you'll see the main menu with the following options:

```
🎯 MAIN MENU - AVAILABLE COMMANDS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 1  help                  💡 Show comprehensive help and documentation
 2  config                ⚙️  Configuration management
 3  setup-tools           🔧 Setup and install required CLI tools
 4  package-pull          📦 Pull application packages and Helm charts
 5  provision-infra       🏗️  Provision Kubernetes infrastructure
 6  validate-infra        ✅ Validate infrastructure configuration
 7  install-app           🆕 Deploy applications to new cluster
 8  upgrade-app           ⬆️  Upgrade application to new version
 9  validate-app          🔍 Validate deployed applications
10  troubleshoot          🔧 Run system diagnostics and troubleshooting

 0  Quit                  Exit Slingshot Installer
```

Each menu item provides specific functionality for your deployment journey.

---

## Complete Deployment Workflow

Follow this workflow for a successful deployment:

### 1. Help & Documentation

**Menu Option: 1 - help**

Access comprehensive help and documentation directly from the installer.

#### Available Help Sections:

- **Menu Guide** - Learn how to navigate the interactive menu system
- **Overview** - Understand what Slingshot Installer does
- **Usage Sessions** - Learn about interactive vs. direct command mode
- **Deployment Workflow** - Step-by-step deployment process
- **Command Reference** - Detailed information about each command
- **Configuration Guide** - How to set up configuration files
- **Best Practices** - Recommendations and tips for successful deployments
- **Troubleshooting Guide** - Common issues and solutions
- **Environment Variables** - Configuration via environment variables
- **Generate User Guide** - Create a markdown guide file
- **Support & Resources** - Documentation and help resources

**When to Use:** Start here if you're new to Slingshot or need guidance on any feature.

---

### 2. Configuration Management

**Menu Option: 2 - config**

Create and manage your deployment configuration. This is the first step in your deployment journey.

#### Submenu Options:

**a) Initialize Configuration (config init)**
- Launch interactive wizard to create a new configuration file
- Guides you through all necessary settings
- Creates a configuration file with your cloud provider and application settings

**Steps in the Configuration Wizard:**

1. **Select Cloud Provider**
   - Choose from Azure, AWS, GCP, or OpenShift
   - Each provider has specific configuration requirements

2. **Choose Region(s)**
   - Select the geographic region(s) for deployment
   - Multiple regions can be selected for distributed deployments

3. **Define Namespace**
   - Specify Kubernetes namespace for your applications
   - Default is usually "default" but you can create custom namespaces

4. **Specify Cluster Name**
   - Enter a unique identifier for your Kubernetes cluster
   - Used for organizing and identifying your deployment

5. **Configure Container Registry**
   - Set up access to your container image registry
   - Required for pulling Docker images during deployment

6. **Select Applications**
   - Choose which applications to deploy
   - Multiple applications can be selected

7. **Choose Slingshot Version**
   - Select the version of Slingshot components to install

**b) Update Configuration (config update)**
- Update existing configuration file
- Preserves current values and allows modifications
- Useful when you need to change settings without starting from scratch

**c) Generate Template (config generate-template)**
- Creates a blank configuration template
- Useful for understanding configuration structure
- Can be manually edited and used for deployment

**d) Validate Configuration (config validate)**
- Checks your configuration file for errors
- Verifies all required fields are present
- Ensures values are in correct format

**Navigation:**
- Select **[0] Back** to return to the main menu

**When to Use:** Run this before any other deployment steps. You must have a valid configuration before proceeding.

---

### 3. Setup Tools

**Menu Option: 3 - setup-tools**

Automatically checks for and installs required command-line tools needed for deployment.

#### Tools Installed:

- **kubectl** - Kubernetes command-line tool
- **helm** - Kubernetes package manager
- **terraform** - Infrastructure as code tool
- **Cloud CLI tools**:
  - Azure CLI (`az`) for Azure deployments
  - AWS CLI (`aws`) for AWS deployments
  - Google Cloud SDK (`gcloud`) for GCP deployments
  - OpenShift CLI (`oc`) for OpenShift deployments

#### Submenu Options:

**a) Check Installation Status**
- Shows which tools are currently installed
- Displays version information for each tool
- Identifies missing tools

**Example Output:**
```
✅ kubectl v1.28.0 - installed
✅ helm v3.12.0 - installed
✅ terraform v1.5.0 - installed
❌ az - not installed
```

**b) Resume or Re-run Installation**
- Continues from where the installation previously stopped
- Useful if installation was interrupted
- Only installs missing tools

**c) Run Complete Installation**
- Fresh installation of all required tools
- Reinstalls even if tools are already present
- Recommended for first-time setup

**What Happens During Installation:**
1. System checks which tools are missing
2. Downloads appropriate versions for your operating system
3. Installs tools to system directories
4. Verifies successful installation
5. Displays summary of installed tools

**When to Use:** Run this after creating your configuration and before pulling packages or provisioning infrastructure.

**Platform-Specific Notes:**
- **Windows (WSL2)**: Tools are installed within the WSL environment
- **Linux**: May require sudo privileges for system-wide installation
- **macOS**: Uses Homebrew if available, otherwise direct installation

---

### 4. Package Pull

**Menu Option: 4 - package-pull**

Downloads application packages, container images, and Helm charts from your configured registry.

#### What This Does:

1. **Authenticates to Container Registry**
   - Connects to your configured registry (Azure ACR, AWS ECR, GCP Artifact Registry, etc.)
   - Uses credentials from configuration or environment variables

2. **Pulls Container Images**
   - Downloads all Docker images specified in your configuration
   - Images are pulled to local Docker cache or registry

3. **Downloads Helm Charts**
   - Retrieves Helm chart packages
   - Charts are stored locally for deployment

4. **Prepares Deployment Packages**
   - Organizes all artifacts for installation
   - Validates package integrity

#### Required Credentials by Cloud Provider:

**Azure Container Registry (ACR):**
- Username: Your ACR registry name
- Password/Token: Access token from Azure portal
- Set via environment variables or prompted during execution

**AWS Elastic Container Registry (ECR):**
- AWS Access Key ID
- AWS Secret Access Key
- Credentials can be configured via AWS CLI or environment variables

**GCP Artifact Registry:**
- GCP Project ID
- Service account credentials
- Configured through gcloud CLI or key file

**Generic Docker Registry:**
- Registry URL
- Username and password
- Configured in your Slingshot configuration file

#### Process Flow:

1. Select **package-pull** from main menu
2. System validates configuration
3. Prompts for credentials if not already set
4. Authenticates to registry
5. Begins downloading packages
6. Shows progress for each package
7. Displays completion summary

**Expected Output:**
```
📦 Pulling application packages...
✅ Authenticated to registry
🔄 Pulling image: myapp:v1.0... Done
🔄 Pulling chart: myapp-chart... Done
✅ All packages pulled successfully
```

**When to Use:** After setting up tools and before provisioning infrastructure. This ensures all necessary packages are available for deployment.

**Important Notes:**
- Requires active internet connection (unless using local/offline registry)
- May take several minutes depending on package size
- Ensure sufficient disk space for all packages

---

### 5. Provision Infrastructure

**Menu Option: 5 - provision-infra**

Creates or updates your Kubernetes cluster infrastructure using Terraform automation scripts.

#### Submenu Options:

**a) Configure Provisioning Scripts Source**
- Set up where Terraform scripts are located
- Two source options available:

**Source Option 1: Tarball**
- Use when scripts are packaged in a compressed archive
- Path entry includes tab completion for easy navigation
- Supports .tar, .tar.gz, .tgz formats

**Steps:**
1. Select "Configure Provisioning Scripts Source"
2. Choose "📦 Tarball"
3. Enter the path to your tarball file
   - Example: `/home/user/terraform/infra.tar.gz`
4. System confirms configuration
5. Subfolder path is automatically read from master configuration

**Source Option 2: Git Repository**
- Use when scripts are in a Git repository
- Supports authentication methods

**Steps:**
1. Select "Configure Provisioning Scripts Source"
2. Choose "🔗 Git"
3. Enter repository URL (must end with .git)
   - Example: `https://github.com/yourorg/terraform-scripts.git`
4. Enter branch name (default: main)
5. Select authentication method:
   - **Personal Access Token (Recommended)**: Most secure, single token entry
   - **Username & Password**: Traditional authentication
6. Enter credentials when prompted
7. System confirms configuration

**b) Execute Infrastructure Provisioning**
- Runs the actual provisioning process
- Creates or updates Kubernetes cluster

**What Happens:**
1. Provisioning scripts are extracted to `.slingshot/provisioning-infra/`
2. Terraform is initialized in the working directory
3. Terraform plan is generated (shows what will be created)
4. Terraform apply is executed (creates infrastructure)
5. Cluster is created and configured
6. Connection details are saved to kubeconfig

**Process Output:**
```
🚀 Provisioning Kubernetes Infrastructure
═══════════════════════════════════════

📦 Extracting provisioning scripts...
✅ Scripts ready at: .slingshot/provisioning-infra/

⚙️  Initializing Terraform...
✅ Terraform initialized

🏗️  Applying Terraform configuration...
Creating subnet...
Creating cluster...
Configuring nodes...
✅ Infrastructure provisioned successfully

🎉 Cluster is ready!
Cluster Name: my-cluster
API Server: https://my-cluster.region.provider.com
```

**c) View Configuration Status**
- Shows current provisioning script configuration
- Displays source type (Tarball or Git)
- Shows path or repository details
- Useful for verifying settings before execution

**When to Use:** After pulling packages and before validating infrastructure. This creates the Kubernetes cluster where your applications will be deployed.

**Important Notes:**
- Provisioning can take 10-30 minutes depending on cloud provider
- Requires valid cloud provider credentials (Azure login, AWS credentials, etc.)
- May incur costs from your cloud provider
- Ensure you have necessary permissions to create infrastructure resources

---

### 6. Validate Infrastructure

**Menu Option: 6 - validate-infra**

Verifies that your Kubernetes infrastructure is properly configured and ready for application deployment.

#### Submenu Options:

**a) Configure Validation Scripts Source**
- Set up where validation scripts are located
- Similar configuration to provisioning scripts

**Source Option 1: Tarball**
1. Select "Configure Validation Scripts Source"
2. Choose "📦 Tarball"
3. Enter path to validation scripts tarball
   - Example: `/path/to/infra-validation.tar.gz`
4. System confirms configuration

**Source Option 2: Git Repository**
1. Select "Configure Validation Scripts Source"
2. Choose "🔗 Git"
3. Enter repository URL
   - Example: `https://github.com/yourorg/validation-scripts.git`
4. Enter branch (default: main)
5. Select authentication:
   - Personal Access Token
   - Username & Password
6. Enter credentials
7. System confirms configuration

**b) Execute Infrastructure Validation**
- Runs comprehensive infrastructure checks
- Validates cluster readiness

**Validation Checks Performed:**

1. **Kubernetes API Server**
   - Verifies cluster is accessible
   - Checks API server health

2. **Node Health**
   - Ensures all nodes are ready
   - Checks node resources (CPU, memory)

3. **Storage Classes**
   - Validates storage provisioners
   - Checks default storage class exists

4. **DNS Resolution**
   - Tests cluster DNS functionality
   - Verifies internal DNS works correctly

5. **Network Policies**
   - Checks network connectivity
   - Validates pod-to-pod communication

6. **Resource Quotas**
   - Verifies namespace quotas if configured
   - Checks available resources

**Example Output:**
```
🔍 Running infrastructure validation checks...

✅ PASS - Kubernetes API Server
   - API server is healthy and responding
   
✅ PASS - Node Health
   - 3/3 nodes are ready
   - Sufficient CPU and memory available
   
✅ PASS - Storage Classes
   - Default storage class: standard
   - Provisioner: kubernetes.io/azure-disk
   
✅ PASS - DNS Resolution
   - CoreDNS pods are running
   - DNS queries successful
   
⚠️  WARN - Network Policies
   - Network policies not configured
   - Cluster is accessible without restrictions

═══════════════════════════════════════
Infrastructure Validation Summary
═══════════════════════════════════════
Total Checks: 5
Passed: 4
Failed: 0
Warnings: 1
Duration: 45 seconds
═══════════════════════════════════════
```

**c) View Validation Configuration Status**
- Shows current validation script configuration
- Displays source location
- Verifies scripts are accessible

**When to Use:** After provisioning infrastructure and before installing applications. This ensures your cluster is ready to host applications.

**What to Do:**
- **All Passed**: Proceed to application installation
- **Warnings**: Review warnings, most can be addressed after deployment
- **Failures**: Must be resolved before proceeding to ensure successful deployment

---

### 7. Install Applications

**Menu Option: 7 - install-app**

Deploys your applications to the Kubernetes cluster for the first time. This is for fresh, new deployments only.

#### Important Notes:

- **Use for NEW deployments only** - First-time installation
- **For UPGRADES**, use **upgrade-app** instead (Option 8)
- Deploys all applications specified in your configuration

#### Deployment Process:

**Phase 1: Pre-Installation Validation**
- Checks cluster connectivity
- Verifies namespace exists (creates if needed)
- Validates Helm charts are available
- Checks that required secrets/configs exist

**Phase 2: Pre-Deployment Scripts**
- Executes any pre-deployment automation
- May include:
  - Database initialization
  - Secret creation
  - Configuration setup
  - Environment preparation

**Phase 3: Application Deployment**
- Installs Helm charts in dependency order
- Creates Kubernetes resources:
  - Deployments
  - Services
  - Ingresses
  - ConfigMaps
  - Secrets
- Waits for pods to become ready
- Monitors deployment progress

**Phase 4: Post-Deployment Scripts**
- Executes post-deployment automation
- May include:
  - Data seeding
  - Initial configuration
  - Service verification
  - Smoke tests

**Phase 5: Deployment Verification**
- Verifies all pods are running
- Checks service endpoints
- Displays application URLs
- Shows access information

**Example Output:**
```
🆕 Installing Slingshot Applications
═══════════════════════════════════

📋 Pre-Installation Checks
✅ Cluster connectivity verified
✅ Namespace 'production' ready
✅ Helm charts validated

⚙️  Running Pre-Deployment Scripts
✅ Database initialized
✅ Secrets created

📦 Deploying Applications

Deploying app1...
  ✅ Helm release installed
  ⏳ Waiting for pods... 
  ✅ 3/3 pods running
  ✅ Service endpoints ready

Deploying app2...
  ✅ Helm release installed
  ⏳ Waiting for pods...
  ✅ 2/2 pods running
  ✅ Service endpoints ready

⚙️  Running Post-Deployment Scripts
✅ Data seeded successfully
✅ Configuration verified

═══════════════════════════════════
Deployment Summary
═══════════════════════════════════
Applications Deployed: 2
Status: SUCCESS
Duration: 5 minutes 23 seconds

Access URLs:
  app1: https://app1.example.com
  app2: https://app2.example.com
═══════════════════════════════════

✅ All applications deployed successfully!
```

**When to Use:** After validating infrastructure. This is typically Step 7 in your first deployment.

**Prerequisites:**
- Configuration completed (Step 2)
- Tools installed (Step 3)
- Packages pulled (Step 4)
- Infrastructure provisioned (Step 5)
- Infrastructure validated (Step 6)

**Typical Duration:** 5-15 minutes depending on number and size of applications

---

### 8. Upgrade Applications

**Menu Option: 8 - upgrade-app**

Upgrades existing applications to a new version. Use this for updating already-deployed applications.

#### When to Use:

- **DO USE** for updating existing deployments
- **DO NOT USE** for first-time installations (use install-app instead)

#### Status:

🚧 **Currently Under Development**

#### Planned Features:

**Rolling Updates**
- Gradual update of application instances
- Zero-downtime deployments
- Automatic rollback if issues detected

**Automated Rollback**
- Automatic reversion to previous version on failure
- Configurable failure thresholds
- Manual rollback options

**Blue-Green Deployments**
- Run old and new versions simultaneously
- Switch traffic between versions
- Quick rollback capability

**Canary Releases**
- Deploy to subset of instances first
- Monitor performance and errors
- Gradual rollout based on metrics

#### What Will Be Available:

1. **Version Selection**
   - Choose target version for upgrade
   - View current and available versions
   - See version changelog

2. **Pre-Upgrade Validation**
   - Check compatibility
   - Verify resources available
   - Test configuration changes

3. **Upgrade Execution**
   - Automated Helm upgrade
   - Rolling pod updates
   - Service continuity maintained

4. **Post-Upgrade Validation**
   - Verify new version running
   - Check application health
   - Validate functionality

**When to Use:** After initial installation when you need to update to a newer version.

**Note:** This feature is coming soon. For now, consult with your support team for upgrade procedures.

---

### 9. Validate Applications

**Menu Option: 9 - validate-app**

Validates that deployed applications are healthy, functional, and operating correctly.

#### Submenu Options:

**a) Configure Validation Scripts Source**
- Set up where application validation scripts are located

**Source Option 1: Tarball**
1. Select "Configure Validation Scripts Source"
2. Choose "📦 Tarball"
3. Enter path to validation scripts tarball
   - Example: `/path/to/app-validation.tar.gz`
4. System confirms configuration

**Source Option 2: Git Repository**
1. Select "Configure Validation Scripts Source"
2. Choose "🔗 Git"
3. Enter repository URL
   - Example: `https://github.com/yourorg/app-validation.git`
4. Enter branch (default: main)
5. Select authentication method:
   - Personal Access Token
   - Username & Password
6. Enter credentials
7. System confirms configuration

**b) Execute Application Validation**
- Runs comprehensive application health checks
- Validates application functionality

**Validation Checks Performed:**

1. **Pod Health Check**
   - Verifies all application pods are running
   - Checks pod restart count (should be low)
   - Validates container health probes

2. **Service Endpoints**
   - Confirms services are accessible
   - Checks endpoint configuration
   - Validates load balancer status

3. **Ingress Configuration**
   - Verifies ingress rules are configured
   - Tests external URL accessibility
   - Checks SSL/TLS certificates

4. **Database Connectivity**
   - Tests database connections
   - Verifies connection pools
   - Checks database health

5. **API Health**
   - Calls application health endpoints
   - Verifies API responses
   - Tests critical API paths

6. **Resource Utilization**
   - Checks CPU and memory usage
   - Verifies within acceptable limits
   - Identifies potential issues

**Example Output:**
```
🔍 Running application validation checks...

✅ PASS - Pod Health Check
   - All 10 pods are running and healthy
   - 0 restarts in the last hour
   - Liveness and readiness probes passing

✅ PASS - Service Endpoints
   - app1-service: 3 endpoints available
   - app2-service: 2 endpoints available
   - Load balancer IP: 52.123.45.67

✅ PASS - Ingress Configuration
   - app1.example.com - HTTP 200 OK
   - app2.example.com - HTTP 200 OK
   - SSL certificates valid

✅ PASS - Database Connectivity
   - MongoDB: Connected (10/10 connections)
   - PostgreSQL: Connected (5/5 connections)
   - Response time < 50ms

✅ PASS - API Health
   - app1/health: HTTP 200 OK
   - app2/health: HTTP 200 OK
   - Response time < 200ms

═══════════════════════════════════════
Application Validation Summary
═══════════════════════════════════════
Total Checks: 5
Passed: 5
Failed: 0
Warnings: 0
Duration: 32 seconds
═══════════════════════════════════════

✅ All applications are healthy and operational!
```

**c) View Validation Configuration Status**
- Shows current validation script configuration
- Displays source information
- Verifies scripts are accessible

**When to Use:** After installing or upgrading applications to ensure everything is working correctly.

**What to Do:**
- **All Passed**: Applications are healthy and ready for use
- **Warnings**: Review and monitor, may indicate potential issues
- **Failures**: Investigate and resolve before using applications in production

---

### 10. Troubleshoot

**Menu Option: 10 - troubleshoot**

Provides diagnostic tools and troubleshooting assistance for deployment issues.

#### Status:

🚧 **Work in Progress**

#### Planned Features:

**System Diagnostics**
- Check tool installations and versions
- Verify configuration files
- Test cloud provider connectivity
- Validate credentials

**Cluster Diagnostics**
- Check cluster health
- View pod status
- Review recent events
- Identify resource issues

**Application Diagnostics**
- Review application logs
- Check deployment status
- Analyze error messages
- Test service connectivity

**Guided Troubleshooting**
- Step-by-step problem resolution
- Common issue detection
- Suggested fixes
- Links to documentation

#### What Will Be Available:

1. **Diagnostic Report Generation**
   - Collects system information
   - Gathers cluster state
   - Captures logs
   - Creates comprehensive report

2. **Interactive Problem Solver**
   - Describes issues you're experiencing
   - Provides targeted diagnostics
   - Suggests solutions
   - Guides through resolution

3. **Log Collection**
   - Gathers relevant logs
   - Filters for errors and warnings
   - Formats for easy reading
   - Exports for support tickets

**When to Use:** When experiencing issues with any part of the deployment process.

**Note:** This feature is under development. For immediate help, see the [Common Scenarios](#common-scenarios) section below.

---

## Menu Navigation Guide

### How to Navigate Menus

**Using Your Keyboard:**

1. **Selecting Options**
   - Type the number of your choice
   - Press Enter to confirm
   - Examples: Type `1` and press Enter for first option

2. **Going Back**
   - Type `0` and press Enter to return to previous menu
   - Or type `back` and press Enter

3. **Quitting**
   - Type `0` from the main menu to exit
   - Or type any of: `q`, `quit`, `x`, `exit`
   - Press Ctrl+C at any time to cancel and exit

### Menu Levels

**Main Menu**
- Top level menu shown when you start Slingshot
- Shows all major commands
- Typing `0` exits the application

**Submenus**
- Shown when you select a command with multiple options
- Example: Selecting "config" shows configuration submenu
- Typing `0` returns to main menu

**Tips for Effective Navigation:**

- Take your time reading each option
- Use help menu to learn about unfamiliar commands
- Always check configuration status before proceeding
- Use the back option freely to explore without commitment
- Menu selections are safe - they won't make changes until you confirm

---

## Common Scenarios

### Scenario 1: First-Time Deployment to Azure

**Steps to Follow:**

1. **Launch Slingshot**
   - Run `./bin/slingshot`

2. **View Help (Optional)**
   - Select `1` for help
   - Review "Deployment Workflow"
   - Return to main menu

3. **Create Configuration**
   - Select `2` for config
   - Select `1` for "init"
   - Choose "Azure" as cloud provider
   - Select region (e.g., "eastus")
   - Enter namespace name
   - Enter cluster name
   - Configure Azure Container Registry
   - Select applications to deploy
   - Choose Slingshot version
   - Configuration saved

4. **Setup Tools**
   - Select `3` for setup-tools
   - Select `3` for "Run complete installation"
   - Wait for tools to install (5-10 minutes)
   - Verify all tools show ✅

5. **Pull Packages**
   - Select `4` for package-pull
   - Enter Azure Container Registry credentials when prompted
   - Wait for packages to download

6. **Provision Infrastructure**
   - Select `5` for provision-infra
   - Select `1` to configure scripts source
   - Choose Tarball or Git
   - Provide script location
   - Select `2` to execute provisioning
   - Wait for cluster creation (15-25 minutes)

7. **Validate Infrastructure**
   - Select `6` for validate-infra
   - Select `1` to configure validation scripts
   - Provide script location
   - Select `2` to execute validation
   - Verify all checks pass

8. **Install Applications**
   - Select `7` for install-app
   - Wait for deployment (5-15 minutes)
   - Note the access URLs provided

9. **Validate Applications**
   - Select `9` for validate-app
   - Select `1` to configure validation scripts
   - Provide script location
   - Select `2` to execute validation
   - Verify all checks pass

10. **Done!**
    - Access your applications using the URLs provided
    - Select `0` to quit Slingshot

---

### Scenario 2: Updating Configuration

**When:** You need to change deployment settings

**Steps:**

1. Launch Slingshot
2. Select `2` for config
3. Select `2` for "update"
4. Follow wizard to modify settings
5. Configuration is updated
6. Return to main menu

---

### Scenario 3: Adding New Applications

**When:** You want to deploy additional applications to existing cluster

**Steps:**

1. Update configuration to include new applications:
   - Select `2` (config)
   - Select `2` (update)
   - Add new applications to selection
   - Save configuration

2. Pull new packages:
   - Select `4` (package-pull)
   - New application packages are downloaded

3. Install new applications:
   - Select `7` (install-app)
   - Only new applications are deployed

4. Validate new applications:
   - Select `9` (validate-app)
   - Verify new applications are healthy

---

### Scenario 4: Recovering from Failed Installation

**When:** Installation failed partway through

**Steps:**

1. Review error messages from failed installation

2. Check configuration:
   - Select `2` (config)
   - Select `4` (validate)
   - Fix any validation errors

3. Verify tools:
   - Select `3` (setup-tools)
   - Select `1` (check status)
   - Re-run installation if needed

4. Verify infrastructure:
   - Select `6` (validate-infra)
   - Ensure cluster is healthy

5. Retry installation:
   - Select `7` (install-app)
   - System will resume from where it failed

---

## Best Practices

### Security Best Practices

**Credentials Management:**
- Never hardcode credentials in configuration files
- Use environment variables for sensitive information
- Rotate credentials regularly
- Use access tokens instead of passwords when possible

**Access Control:**
- Use least-privilege access for cloud accounts
- Enable RBAC (Role-Based Access Control) on clusters
- Regularly review and audit access permissions

### Configuration Best Practices

**Version Control:**
- Keep configuration files in version control (Git)
- Don't commit sensitive information
- Use separate configurations for different environments
- Document custom settings and changes

**Environment Separation:**
- Use different configurations for dev, staging, production
- Test deployments in non-production first
- Maintain naming conventions to avoid confusion

### Deployment Best Practices

**Follow the Workflow:**
- Always complete steps in order
- Don't skip validation steps
- Review output messages carefully
- Keep logs for troubleshooting

**Pre-Deployment:**
- Verify sufficient cloud resources and quotas
- Ensure all required credentials are available
- Check network connectivity
- Have rollback plan ready

**During Deployment:**
- Monitor deployment progress
- Don't interrupt running deployments
- Note any warnings or errors
- Save deployment logs

**Post-Deployment:**
- Always run application validation
- Test critical functionality
- Document any issues encountered
- Create backup/snapshot after successful deployment

### Operational Best Practices

**Maintenance:**
- Keep Slingshot updated to latest version
- Regularly update deployed applications
- Monitor cluster resource utilization
- Review logs periodically

**Documentation:**
- Document your deployment procedures
- Keep notes on custom configurations
- Record URLs and access information
- Maintain runbook for common tasks

**Monitoring:**
- Set up alerts for application issues
- Monitor cluster health regularly
- Track resource usage trends
- Plan capacity upgrades proactively

---

## Support & Resources

### Getting Help

**Built-in Help System:**
- Access comprehensive help: Select `1` from main menu
- View specific command help within each submenu
- Generate user guide: Help menu → Generate User Guide

**Documentation Files:**
- **README.md** - Quick start guide
- **QUICKSTART.md** - Fast track deployment
- **USER-GUIDE.md** - Detailed technical guide
- **ARCHITECTURE.md** - System architecture and design

### Common Issues and Solutions

**Issue: "Configuration file not found"**
- **Solution:** Run config init (Menu → 2 → 1)
- Must create configuration before other operations

**Issue: "kubectl not found" or other tool missing**
- **Solution:** Run setup-tools (Menu → 3 → 3)
- Select complete installation to install all required tools

**Issue: "Authentication failed" to container registry**
- **Solution:** Verify credentials in environment variables
- Check that you have permission to access the registry
- Re-enter credentials when prompted

**Issue: "Cluster not accessible"**
- **Solution:** Run validate-infra (Menu → 6)
- Check cloud provider authentication
- Verify cluster was created successfully
- Ensure kubeconfig is properly configured

**Issue: "Validation scripts not found"**
- **Solution:** Verify the path to tarball or Git repository
- Check file permissions and network access
- Use absolute paths instead of relative paths
- Ensure Git credentials are correct

**Issue: "Helm chart deployment failed"**
- **Solution:** Check cluster has sufficient resources
- Verify all required secrets and configmaps exist
- Review Helm chart values configuration
- Check application logs for specific errors

### Contact Support

For additional assistance:
- Review detailed logs in `.slingshot/logs/` directory
- Collect error messages and deployment output
- Document steps taken and issue encountered
- Contact your support team with collected information

---

## Appendix: Platform-Specific Notes

### Windows (WSL2) Notes

**WSL2 Setup:**
- Install WSL2 from Microsoft Store or PowerShell
- Install Ubuntu or preferred Linux distribution
- Update WSL: `wsl --update`
- Access files: `/mnt/c/` for C drive

**Common Issues:**
- **File permissions:** WSL may have different permissions than Windows
- **Network:** Ensure WSL can access internet
- **Path differences:** Use Linux paths within WSL

### Linux Notes

**Permissions:**
- May need sudo for tool installation
- Binary needs execute permission: `chmod +x bin/slingshot`
- Some operations may require elevated privileges

**Package Managers:**
- Tools can be installed via apt, yum, dnf depending on distribution
- Slingshot handles installation automatically

### macOS Notes

**Security:**
- First run may require security approval
- Go to System Preferences → Security & Privacy
- Click "Allow" for Slingshot application

**Architecture:**
- Use appropriate binary for your Mac (Intel vs Apple Silicon)
- `slingshot-darwin-amd64` for Intel Macs
- `slingshot-darwin-arm64` for Apple Silicon (M1/M2/M3)

**Homebrew:**
- Some tools may install via Homebrew if available
- Slingshot handles installation automatically

---

**End of Client User Guide**

*For technical details and advanced configuration, please refer to the technical documentation.*

**Powered by Publicis Sapient** 🚀
