# GitOps Fundamentals Exercises

This directory contains exercise files for the GitOps Fundamentals tutorial section.

## Structure

### `argocd/`
- **Purpose**: ArgoCD configuration for Crossplane integration
- **Files**:
  - `argocd-cm-crossplane-health.yaml` - ConfigMap with Crossplane health checks

### `rbac/`
- **Purpose**: Role-based access control for ArgoCD and Crossplane
- **Files**:
  - `argocd-crossplane-clusterrole.yaml` - ClusterRole and ClusterRoleBinding for Crossplane permissions

### `crossplane-integration/`
- **Purpose**: Example Composite Resources demonstrating ArgoCD + Crossplane integration
- **Files**:
  - `xpostgresqlinstance-dev.yaml` - Example PostgreSQL Composite Resource

### `workflow-patterns/`
- **Purpose**: Advanced GitOps workflow patterns and automation
- **Files**:
  - `argocd-app-infrastructure-dev.yaml` - ArgoCD Application for dev environment
  - `applicationset-application-infrastructure.yaml` - ApplicationSet for multi-environment deployment
  - `github-actions/` - GitHub Actions workflows for GitOps automation
  - `examples/` - Example infrastructure configurations

## Usage Instructions

### Prerequisites
- ArgoCD installed and configured
- Crossplane installed with providers
- GitHub CLI authenticated (`gh auth status`)
- Proper RBAC configured for cluster access

### ArgoCD Health Checks
```bash
# Copy ArgoCD health checks configuration
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/argocd/argocd-cm-crossplane-health.yaml gitops-bootstrap/argocd/

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add gitops-bootstrap/argocd/argocd-cm-crossplane-health.yaml && git commit -m "chore(gitops): seed argocd health checks (gitops-fundamentals/argocd)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-argocd-health
user@host idp-tutorial> $ git add gitops-bootstrap/argocd/argocd-cm-crossplane-health.yaml
user@host idp-tutorial> $ git commit -m "chore(gitops): add argocd health checks (gitops-fundamentals/argocd)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add ArgoCD Crossplane health checks" --body "Adds health.lua customizations via argocd-cm"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
```

### RBAC Configuration
```bash
# Copy RBAC configuration to platform core
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/rbac/argocd-crossplane-clusterrole.yaml platform-core/

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add platform-core/argocd-crossplane-clusterrole.yaml && git commit -m "chore(gitops): seed argocd rbac (gitops-fundamentals/rbac)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-argocd-rbac
user@host idp-tutorial> $ git add platform-core/argocd-crossplane-clusterrole.yaml
user@host idp-tutorial> $ git commit -m "chore(gitops): add argocd rbac (gitops-fundamentals/rbac)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Grant ArgoCD controller Crossplane RBAC" --body "Adds ClusterRole for compositions and XRDs"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
```

### Crossplane Integration Example
```bash
# Copy example Composite Resource to dev environment
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/crossplane-integration/xpostgresqlinstance-dev.yaml exercises/crossplane-advanced-patterns-01/environments/dev/infrastructure/

# Commit on a feature branch and open a PR
user@host idp-tutorial> $ git switch -c feat/dev-db-example
user@host idp-tutorial> $ git add exercises/crossplane-advanced-patterns-01/environments/dev/infrastructure/xpostgresqlinstance-dev.yaml
user@host idp-tutorial> $ git commit -m "gitops: add xpostgresqlinstance for dev (integration example)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add dev XPostgreSQLInstance example" --body "Adds integration example CR for dev"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
```

### Workflow Patterns
```bash
# Copy ArgoCD Applications for environment management
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/workflow-patterns/argocd-app-infrastructure-dev.yaml gitops-bootstrap/argocd/
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/workflow-patterns/applicationset-application-infrastructure.yaml gitops-bootstrap/argocd/

# Copy GitHub Actions workflows to your repository
user@host idp-tutorial> $ mkdir -p .github/workflows
user@host idp-tutorial> $ cp exercises/gitops-fundamentals/workflow-patterns/github-actions/*.yml .github/workflows/

# Update repository URLs in the files
user@host idp-tutorial> $ sed -i 's/your-org/your-github-username/g' gitops-bootstrap/argocd/*.yaml
user@host idp-tutorial> $ sed -i 's/your-org/your-github-username/g' .github/workflows/*.yml

# Commit workflow patterns
user@host idp-tutorial> $ git add gitops-bootstrap/argocd/*.yaml .github/workflows/*.yml
user@host idp-tutorial> $ git commit -m "gitops: add workflow patterns and automation"
user@host idp-tutorial> $ git push
```

## Learning Objectives

After completing these exercises, you will understand:

1. **ArgoCD + Crossplane Integration**: How to configure ArgoCD to properly understand and monitor Crossplane resources
2. **RBAC Configuration**: How to grant ArgoCD the necessary permissions to manage Crossplane resources
3. **GitOps Reconciliation Loop**: The end-to-end flow from Git commit to infrastructure provisioning
4. **Workflow Patterns**: Advanced GitOps patterns for multi-environment management
5. **Automation**: GitHub Actions integration for CI/CD and drift detection

## Troubleshooting

### Common Issues

1. **ArgoCD shows Crossplane resources as "Progressing"**
   - Ensure health checks are applied via `argocd-cm-crossplane-health.yaml`
   - Check ArgoCD ConfigMap: `kubectl get cm argocd-cm -n argocd -o yaml`

2. **ArgoCD cannot create Crossplane resources**
   - Verify RBAC is applied: `kubectl get clusterrole argocd-crossplane-manager-role`
   - Check ArgoCD controller permissions: `kubectl auth can-i create compositions --as=system:serviceaccount:argocd:argocd-application-controller`

3. **GitHub Actions workflows failing**
   - Verify secrets are configured: `KUBECONFIG`, `ARGOCD_SERVER`, etc.
   - Check cluster access and kubectl configuration

## Tutorial Navigation
- [01: ArgoCD Setup](../../docs/gitops-fundamentals/01-argocd-setup.md)
- [02: Crossplane Integration](../../docs/gitops-fundamentals/02-crossplane-integration.md)
- [03: Workflow Patterns](../../docs/gitops-fundamentals/03-workflow-patterns.md)
