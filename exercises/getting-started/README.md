# Getting Started Exercises

This directory contains exercise files for the Getting Started tutorial section.

## Structure

### `cluster-setup/`
- **Purpose**: Local development cluster configuration
- **Files**:
  - `kind-cluster.yaml` - Kind cluster configuration for local development

### `gitops-bootstrap/`
- **Purpose**: Initial GitOps setup and ArgoCD installation
- **Files**: 
  - References existing files in `/gitops-bootstrap/` directory
  - Platform core setup through ArgoCD

### `sealed-secrets/`
- **Purpose**: Secure secret management setup
- **Files**:
  - `sealed-secrets-controller.yaml` - ArgoCD application for Sealed Secrets controller

## Usage Instructions

### Prerequisites
- Devbox environment activated (`devbox shell`)
- Docker Desktop running
- GitHub CLI authenticated (`gh auth status`)

### Local Cluster Setup
```bash
# Create local Kubernetes cluster
user@host idp-tutorial> $ kind create cluster --config exercises/getting-started/cluster-setup/kind-cluster.yaml
```

### GitOps Bootstrap
Follow the instructions in `docs/getting-started/04-gitops-bootstrap.md` using the existing bootstrap files in `/gitops-bootstrap/`.

### Sealed Secrets Setup
```bash
# Copy sealed secrets controller to platform core
user@host idp-tutorial> $ cp exercises/getting-started/sealed-secrets/sealed-secrets-controller.yaml platform-core/

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add platform-core/sealed-secrets-controller.yaml && git commit -m "chore(gitops): seed sealed-secrets-controller (getting-started/sealed-secrets)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-sealed-secrets
user@host idp-tutorial> $ git add platform-core/sealed-secrets-controller.yaml
user@host idp-tutorial> $ git commit -m "chore(gitops): add sealed-secrets-controller (getting-started/sealed-secrets)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add Sealed Secrets Controller" --body "Add secure secret management via Sealed Secrets"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch

# Optional: wait for ArgoCD
user@host idp-tutorial> $ argocd app wait sealed-secrets-controller --timeout 300
```

## Tutorial Navigation
- [02: Devbox Setup](../../docs/getting-started/02-devbox-setup.md)
- [03: Local Cluster Setup](../../docs/getting-started/03-local-cluster-setup.md)
- [04: GitOps Bootstrap](../../docs/getting-started/04-gitops-bootstrap.md)
- [05: Sealed Secrets Setup](../../docs/getting-started/05-sealed-secrets-setup.md)
