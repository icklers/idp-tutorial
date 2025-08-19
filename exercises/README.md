# Tutorial Exercises

This directory contains all exercise files organized by tutorial section, providing hands-on YAML configurations and GitOps workflows for building an Internal Developer Platform.

## Directory Structure

```
exercises/
├── README.md                           # This file
├── getting-started/                    # Local setup and GitOps bootstrap
├── gitops-fundamentals/                # ArgoCD and GitOps patterns
├── crossplane-fundamentals/            # Provider setup and managed resources
├── crossplane-advanced-patterns-01/    # First composition and environment patterns
├── crossplane-advanced-patterns-02/    # Networking compositions
├── crossplane-advanced-patterns-03/    # Environment-based patterns
├── observability/                      # Monitoring, alerting, and health checks
├── security/                           # Image security, secrets, and RBAC
└── repository-management/              # Multi-repo strategies and templates
```

## Quick Start Guide

### Prerequisites
- **Local Environment**: Docker Desktop, Kind, DevBox installed
- **Cloud Access**: Azure CLI authenticated with active subscription
- **GitHub Access**: GitHub CLI authenticated (`gh auth status`)
- **Repository**: Your own GitHub repository created from this template

### 1. Environment Setup
```bash
# Clone your platform repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME

# Activate DevBox environment
devbox shell

# Set environment variables
export GITHUB_USERNAME="your-username"
export GITHUB_REPO_NAME="your-repo-name"
export PLATFORM_GITOPS_REPO_URL="https://github.com/$GITHUB_USERNAME/$GITHUB_REPO_NAME.git"
```

### 2. Local Cluster Setup
```bash
# Create local Kubernetes cluster
kind create cluster --config exercises/getting-started/cluster-setup/kind-cluster.yaml

# Verify cluster is ready
kubectl cluster-info
```

### 3. GitOps Bootstrap
Follow the instructions in [Getting Started](getting-started/) to:
- Install ArgoCD
- Set up Sealed Secrets
- Configure your platform repository

### 4. Progressive Learning Path
1. **[Getting Started](getting-started/)** - Local setup and GitOps foundation
2. **[GitOps Fundamentals](gitops-fundamentals/)** - ArgoCD integration and workflow patterns
3. **[Crossplane Fundamentals](crossplane-fundamentals/)** - Provider setup and first resources
4. **[Advanced Patterns](crossplane-advanced-patterns-01/)** - Compositions and environment management
5. **[Observability](observability/)** - Monitoring and alerting
6. **[Security](security/)** - Image security, secrets, and RBAC

## Usage Patterns

### GitOps Execution Model

All exercises follow a consistent GitOps execution model with two approaches:

#### Option A: Bootstrap (Initial Seeding)
For initial setup or one-time configurations:
```bash
# Copy files to target location
cp exercises/SECTION/AREA/file.yaml target/path/

# One-liner commit and push
git add target/path/file.yaml && \
git commit -m "chore(gitops): seed FILE (SECTION/AREA)" && \
git push -u origin HEAD
```

#### Option B: Feature Development (PR Workflow)
For ongoing changes requiring review:
```bash
# Copy files to target location
cp exercises/SECTION/AREA/file.yaml target/path/

# Create feature branch and PR
git switch -c feat/add-FEATURE
git add target/path/file.yaml
git commit -m "feat: add FEATURE (SECTION/AREA)"
git push -u origin $(git branch --show-current)
gh pr create --title "Add FEATURE" --body "Description of changes"
gh pr merge --auto --squash --delete-branch
```

### File Organization Standards

All exercise files follow consistent naming and structure:

```yaml
# File: exercises/section/functional-area/filename.yaml
# Description: Brief description of what this configuration does
# Tutorial Reference: docs/section/module.md
# Prerequisites: List any dependencies

apiVersion: ...
kind: ...
# ... configuration content
```

### Target Path Mapping

Files are organized to map to these GitOps target paths:

| Exercise Area | Target Path | Purpose |
|---------------|-------------|---------|
| `getting-started/gitops-bootstrap/` | `gitops-bootstrap/argocd/` | ArgoCD bootstrap configs |
| `getting-started/sealed-secrets/` | `platform-core/` | Platform applications |
| `gitops-fundamentals/rbac/` | `platform-core/` | RBAC configurations |
| `crossplane-fundamentals/azure-provider/` | `platform-core/providers/` | Provider installations |
| `crossplane-advanced-patterns-01/platform-core/` | `platform-core/` | XRDs and Compositions |
| `observability/monitoring/` | `platform-core/applications/` | Monitoring applications |
| `security/rbac/` | `platform-core/rbac/` | Security configurations |

## Learning Objectives

### By Section

#### Getting Started
- ✅ Local development environment setup
- ✅ GitOps workflow fundamentals
- ✅ Secure secret management with Sealed Secrets

#### GitOps Fundamentals
- ✅ ArgoCD configuration and integration
- ✅ Advanced workflow patterns
- ✅ Environment promotion strategies
- ✅ GitHub Actions automation

#### Crossplane Fundamentals
- ✅ Cloud provider setup and authentication
- ✅ Managed resource lifecycle
- ✅ GitOps-driven infrastructure provisioning

#### Advanced Patterns
- ✅ Composite Resource Definitions (XRDs)
- ✅ Composition development and testing
- ✅ Multi-environment management
- ✅ Environment-specific configurations

#### Observability
- ✅ Health monitoring and status reporting
- ✅ Metrics collection with Prometheus
- ✅ Alerting and incident response

#### Security
- ✅ Container image security and verification
- ✅ External secret management
- ✅ Role-based access control (RBAC)

### Overall Platform Capabilities
After completing all exercises, you will have built a production-ready Internal Developer Platform with:

- **GitOps-Native**: All infrastructure managed through Git
- **Multi-Environment**: Dev, staging, production environment patterns
- **Self-Service**: Developers can provision infrastructure via Composite Resources
- **Secure**: Image verification, encrypted secrets, proper RBAC
- **Observable**: Comprehensive monitoring, metrics, and alerting
- **Scalable**: Patterns that scale from startup to enterprise

## Troubleshooting

### Common Issues

1. **Repository URL not updated**
   ```bash
   # Update all placeholder URLs
   find platform-core -name "*.yaml" -exec sed -i 's/__YOUR_PLATFORM_GITOPS_REPO_URL__/'"$PLATFORM_GITOPS_REPO_URL"'/g' {} +
   ```

2. **ArgoCD applications not syncing**
   ```bash
   # Check application status
   kubectl get applications -n argocd
   
   # Force sync if needed
   argocd app sync APPLICATION_NAME
   ```

3. **Crossplane resources not becoming ready**
   ```bash
   # Check provider status
   kubectl get providers
   
   # Check provider logs
   kubectl logs -n crossplane-system deployment/PROVIDER_NAME
   ```

4. **GitHub Actions workflows failing**
   ```bash
   # Verify secrets are configured
   gh secret list
   
   # Check workflow status
   gh run list
   ```

### Getting Help

- **Tutorial Documentation**: Each exercise references specific tutorial sections
- **Exercise READMEs**: Detailed instructions in each section
- **Troubleshooting Guides**: Common issues and solutions in section READMEs
- **Community Resources**: Links to Crossplane and ArgoCD documentation

## Contributing to the Tutorial

If you find issues or have improvements:

1. **File Issues**: Report problems or suggest enhancements
2. **Submit PRs**: Contribute fixes or additional examples
3. **Share Feedback**: Help improve the learning experience

## Next Steps

Start with the [Getting Started](getting-started/) section to set up your local environment and GitOps foundation, then progress through the learning path at your own pace.

Each section builds upon the previous ones, creating a comprehensive understanding of modern platform engineering with GitOps, Crossplane, and cloud-native tools.
