# Security Exercises

This directory contains exercise files for the Security tutorial section.

## Structure

### `chainguard-images/`
- **Purpose**: Image security and signature verification
- **Files**:
  - `kyverno-installation.yaml` - ArgoCD Application for Kyverno policy engine
  - `chainguard-image-policy.yaml` - Kyverno policy for Chainguard image verification
  - `crossplane-provider-policy.yaml` - Policy to enforce Chainguard images for Crossplane providers

### `secret-management/`
- **Purpose**: External secret management with Azure Key Vault
- **Files**:
  - `external-secrets-installation.yaml` - ArgoCD Application for External Secrets Operator
  - `azure-key-vault-secretstore.yaml` - ClusterSecretStore for Azure Key Vault
  - `external-secret-example.yaml` - Example ExternalSecret for database credentials

### `rbac/`
- **Purpose**: Role-based access control configurations
- **Files**:
  - `argocd-rbac-config.yaml` - ArgoCD RBAC configuration
  - `crossplane-dev-team-rbac.yaml` - Kubernetes RBAC for development team
  - `platform-team-rbac.yaml` - Kubernetes RBAC for platform team

## Usage Instructions

### Prerequisites
- ArgoCD installed and configured
- Crossplane installed with providers
- Azure Key Vault configured (for secret management)
- User and group management system (OIDC, LDAP, etc.)

### Image Security with Kyverno

Deploy Kyverno and configure image verification policies:

```bash
# Copy Kyverno installation
user@host idp-tutorial> $ cp exercises/security/chainguard-images/kyverno-installation.yaml platform-core/applications/

# Create security policies directory
user@host idp-tutorial> $ mkdir -p platform-core/security-policies

# Copy security policies
user@host idp-tutorial> $ cp exercises/security/chainguard-images/chainguard-image-policy.yaml platform-core/security-policies/
user@host idp-tutorial> $ cp exercises/security/chainguard-images/crossplane-provider-policy.yaml platform-core/security-policies/

# Create ArgoCD Application for security policies
cat > platform-core/applications/security-policies.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: security-policies
  namespace: argocd
spec:
  project: default
  source:
    repoURL: __YOUR_PLATFORM_GITOPS_REPO_URL__
    targetRevision: HEAD
    path: platform-core/security-policies
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/security-policies.yaml

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add platform-core/applications/kyverno-installation.yaml platform-core/applications/security-policies.yaml platform-core/security-policies/ && git commit -m "chore(gitops): seed kyverno and security policies (security/chainguard-images)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-security-policies
user@host idp-tutorial> $ git add platform-core/applications/kyverno-installation.yaml platform-core/applications/security-policies.yaml platform-core/security-policies/
user@host idp-tutorial> $ git commit -m "chore(gitops): add kyverno and security policies (security/chainguard-images)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add Kyverno and Security Policies" --body "Deploys image verification and security policies"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch

# Deploy the applications
user@host idp-tutorial> $ kubectl apply -f platform-core/applications/kyverno-installation.yaml
user@host idp-tutorial> $ kubectl apply -f platform-core/applications/security-policies.yaml
```

### External Secret Management

Set up External Secrets Operator with Azure Key Vault:

```bash
# Copy External Secrets installation
user@host idp-tutorial> $ cp exercises/security/secret-management/external-secrets-installation.yaml platform-core/applications/

# Create secret management directory
user@host idp-tutorial> $ mkdir -p platform-core/secret-management

# Copy secret management configurations
user@host idp-tutorial> $ cp exercises/security/secret-management/azure-key-vault-secretstore.yaml platform-core/secret-management/
user@host idp-tutorial> $ cp exercises/security/secret-management/external-secret-example.yaml platform-core/secret-management/

# Update Azure Key Vault URL and tenant ID
user@host idp-tutorial> $ sed -i 's/your-key-vault/your-actual-key-vault-name/g' platform-core/secret-management/azure-key-vault-secretstore.yaml
user@host idp-tutorial> $ sed -i 's/your-tenant-id/your-actual-tenant-id/g' platform-core/secret-management/azure-key-vault-secretstore.yaml

# Create ArgoCD Application for secret management
cat > platform-core/applications/secret-management.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: secret-management
  namespace: argocd
spec:
  project: default
  source:
    repoURL: __YOUR_PLATFORM_GITOPS_REPO_URL__
    targetRevision: HEAD
    path: platform-core/secret-management
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/secret-management.yaml

# Commit and deploy
user@host idp-tutorial> $ git add platform-core/applications/external-secrets-installation.yaml platform-core/applications/secret-management.yaml platform-core/secret-management/
user@host idp-tutorial> $ git commit -m "feat: add external secrets management with Azure Key Vault"
user@host idp-tutorial> $ git push

user@host idp-tutorial> $ kubectl apply -f platform-core/applications/external-secrets-installation.yaml
user@host idp-tutorial> $ kubectl apply -f platform-core/applications/secret-management.yaml
```

### RBAC Configuration

Configure role-based access control for teams:

```bash
# Create RBAC directory
user@host idp-tutorial> $ mkdir -p platform-core/rbac

# Copy RBAC configurations
user@host idp-tutorial> $ cp exercises/security/rbac/argocd-rbac-config.yaml platform-core/rbac/
user@host idp-tutorial> $ cp exercises/security/rbac/crossplane-dev-team-rbac.yaml platform-core/rbac/
user@host idp-tutorial> $ cp exercises/security/rbac/platform-team-rbac.yaml platform-core/rbac/

# Update user/group names in RBAC files
user@host idp-tutorial> $ sed -i 's/alice@company.com/alice@yourcompany.com/g' platform-core/rbac/crossplane-dev-team-rbac.yaml
user@host idp-tutorial> $ sed -i 's/bob@company.com/bob@yourcompany.com/g' platform-core/rbac/crossplane-dev-team-rbac.yaml
user@host idp-tutorial> $ sed -i 's/platform-admin@company.com/platform-admin@yourcompany.com/g' platform-core/rbac/platform-team-rbac.yaml

# Create ArgoCD Application for RBAC
cat > platform-core/applications/rbac-configs.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rbac-configs
  namespace: argocd
spec:
  project: default
  source:
    repoURL: __YOUR_PLATFORM_GITOPS_REPO_URL__
    targetRevision: HEAD
    path: platform-core/rbac
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/rbac-configs.yaml

# Commit and deploy
user@host idp-tutorial> $ git add platform-core/rbac/ platform-core/applications/rbac-configs.yaml
user@host idp-tutorial> $ git commit -m "feat: add RBAC configurations for team access control"
user@host idp-tutorial> $ git push

user@host idp-tutorial> $ kubectl apply -f platform-core/applications/rbac-configs.yaml
```

## Security Verification

### Verify Kyverno Policies

Test that image verification policies are working:

```bash
# Check Kyverno policies are active
kubectl get clusterpolicy

# Test policy enforcement (should be rejected)
kubectl run test-pod --image=nginx:latest --rm -it --restart=Never

# Test with Chainguard image (should be allowed)
kubectl run test-pod --image=cgr.dev/chainguard/nginx:latest --rm -it --restart=Never
```

### Verify External Secrets

Test secret synchronization:

```bash
# Check External Secrets Operator is running
kubectl get pods -n external-secrets-system

# Check ClusterSecretStore status
kubectl get clustersecretstore azure-key-vault

# Check ExternalSecret status
kubectl get externalsecret database-credentials
kubectl describe externalsecret database-credentials
```

### Verify RBAC

Test role-based access control:

```bash
# Check ArgoCD RBAC configuration
kubectl get configmap argocd-rbac-cm -n argocd -o yaml

# Test user permissions (requires proper authentication setup)
kubectl auth can-i create mysqlinstances --as=alice@yourcompany.com
kubectl auth can-i create compositions --as=alice@yourcompany.com  # Should be denied
```

## Learning Objectives

After completing these exercises, you will understand:

1. **Image Security**: Using Kyverno to enforce signed container images
2. **Secret Management**: External secret synchronization with Azure Key Vault
3. **RBAC**: Configuring role-based access control for teams
4. **Security Policies**: Implementing security controls via GitOps
5. **Least Privilege**: Applying principle of least privilege access

## Troubleshooting

### Common Issues

1. **Kyverno policies blocking legitimate pods**
   - Check policy syntax and image references
   - Verify Chainguard image signatures are valid
   - Use `kubectl describe pod` to see admission errors

2. **External Secrets not syncing**
   - Verify Azure Key Vault permissions
   - Check service principal credentials
   - Review External Secrets Operator logs

3. **RBAC denying expected access**
   - Verify user/group names match identity provider
   - Check ClusterRole and Role permissions
   - Test with `kubectl auth can-i` commands

## Tutorial Navigation
- [01: Chainguard Images](../../docs/security/01-chainguard-images.md)
- [02: Secret Management](../../docs/security/02-secret-management.md)
- [03: RBAC](../../docs/security/03-rbac.md)
