# Crossplane Fundamentals Exercises

This directory contains exercise files for the Crossplane Fundamentals tutorial section.

## Structure

### `azure-provider/`
- **Purpose**: Azure provider setup and configuration
- **Files**:
  - `azure-provider.yaml` - Azure provider installation
  - `azure-provider-config.yaml` - ProviderConfig for authentication
  - `azure-provider-applications.yaml` - ArgoCD Applications for deployment

### `managed-resources/`
- **Purpose**: Basic managed resource examples
- **Files**:
  - `resource-group.yaml` - Azure Resource Group managed resource
  - `resource-group-application.yaml` - ArgoCD Application for Resource Group

### `understanding/`
- **Purpose**: Core concept examples and demonstrations
- **Files**: (To be populated based on Module 3)

## Usage Instructions

### Prerequisites
- Crossplane installed and running
- ArgoCD installed and configured
- Azure CLI authenticated (`az account show`)
- Azure service principal created
- Sealed Secrets controller installed

### Azure Provider Setup

First, create Azure service principal credentials:

```bash
# Set environment variables for your subscription
export SUBSCRIPTION_ID=$(az account show --query id --output tsv)
export TENANT_ID=$(az account show --query tenantId --output tsv)

# Create service principal with Contributor role
SP_OUTPUT=$(az ad sp create-for-rbac \
  --name "crossplane-tutorial-sp" \
  --role "Contributor" \
  --scopes "/subscriptions/$SUBSCRIPTION_ID" \
  --output json)

export CLIENT_ID=$(echo $SP_OUTPUT | jq -r '.appId')
export CLIENT_SECRET=$(echo $SP_OUTPUT | jq -r '.password')
```

Create sealed secret for credentials:

```bash
# Create temporary secret file (DO NOT COMMIT THIS)
cat > azure-secret-temp.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: azure-secret
  namespace: crossplane-system
  labels:
    app: crossplane
    component: azure-credentials
type: Opaque
stringData:
  creds: |
    {
      "clientId": "$CLIENT_ID",
      "clientSecret": "$CLIENT_SECRET",
      "subscriptionId": "$SUBSCRIPTION_ID",
      "tenantId": "$TENANT_ID"
    }
EOF

# Encrypt with Sealed Secrets
kubeseal -o yaml < azure-secret-temp.yaml > platform-core/provider-configs/azure-credentials-sealed-secret.yaml

# Clean up temporary file immediately
rm azure-secret-temp.yaml
```

Copy provider files to platform core:

```bash
# Copy provider configuration
user@host idp-tutorial> $ cp exercises/crossplane-fundamentals/azure-provider/azure-provider.yaml platform-core/providers/
user@host idp-tutorial> $ cp exercises/crossplane-fundamentals/azure-provider/azure-provider-config.yaml platform-core/provider-configs/

# Copy ArgoCD applications
user@host idp-tutorial> $ cp exercises/crossplane-fundamentals/azure-provider/azure-provider-applications.yaml platform-core/applications/

# Update repository URL in applications
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/azure-provider-applications.yaml

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add platform-core/ && git commit -m "chore(gitops): seed azure provider (crossplane-fundamentals/azure-provider)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-azure-provider
user@host idp-tutorial> $ git add platform-core/
user@host idp-tutorial> $ git commit -m "chore(gitops): add azure provider (crossplane-fundamentals/azure-provider)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add Azure Provider for Crossplane" --body "Sets up Azure provider with secure credentials via Sealed Secrets"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch

# Deploy the ArgoCD applications
user@host idp-tutorial> $ kubectl apply -f platform-core/applications/azure-provider-applications.yaml
```

### First Managed Resource

After Azure provider is installed and ready:

```bash
# Create directory structure
user@host idp-tutorial> $ mkdir -p platform-core/azure/01-resource-group

# Copy managed resource files
user@host idp-tutorial> $ cp exercises/crossplane-fundamentals/managed-resources/resource-group.yaml platform-core/azure/01-resource-group/
user@host idp-tutorial> $ cp exercises/crossplane-fundamentals/managed-resources/resource-group-application.yaml platform-core/azure/01-resource-group/application.yaml

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/azure/01-resource-group/application.yaml

# Commit managed resource
user@host idp-tutorial> $ git add platform-core/azure/
user@host idp-tutorial> $ git commit -m "feat: add first managed resource - Azure Resource Group"
user@host idp-tutorial> $ git push

# Deploy the ArgoCD application
user@host idp-tutorial> $ kubectl apply -f platform-core/azure/01-resource-group/application.yaml
```

### Verification

Check provider installation:

```bash
# Verify provider is ready
kubectl get provider provider-family-azure
# Should show INSTALLED: True, HEALTHY: True

# Verify provider config exists
kubectl get providerconfig default
# Should show READY: True

# Check ArgoCD applications
kubectl get applications -n argocd
```

Check managed resource:

```bash
# Watch resource creation
kubectl get resourcegroup tutorial-rg-001 -n tutorial -w

# Verify in Azure
az group show --name tutorial-rg-001 --output table
```

## Learning Objectives

After completing these exercises, you will understand:

1. **Azure Provider Setup**: How to install and configure the Azure provider securely
2. **Credential Management**: Using Sealed Secrets for secure cloud credentials
3. **Managed Resources**: Creating cloud resources with Crossplane
4. **GitOps Integration**: Deploying infrastructure through ArgoCD
5. **Resource Lifecycle**: Creation, monitoring, and management of cloud resources

## Troubleshooting

### Common Issues

1. **Provider not installing**
   - Check ArgoCD application status: `kubectl describe application azure-provider -n argocd`
   - Verify crossplane-system namespace exists

2. **Authentication failures**
   - Verify sealed secret is decrypted: `kubectl get secret azure-secret -n crossplane-system`
   - Check service principal permissions: `az role assignment list --assignee $CLIENT_ID`

3. **Resource creation fails**
   - Check resource events: `kubectl describe resourcegroup tutorial-rg-001 -n tutorial`
   - View provider logs: `kubectl logs -n crossplane-system deployment/upbound-provider-azure`

4. **ArgoCD sync issues**
   - Force sync: `argocd app sync azure-provider`
   - Check for repository access issues

## Tutorial Navigation
- [01: Azure Provider Setup](../../docs/crossplane-fundamentals/01-azure-provider-setup.md)
- [02: First Managed Resource](../../docs/crossplane-fundamentals/02-first-managed-resource.md)
- [03: Understanding Crossplane](../../docs/crossplane-fundamentals/03-understanding-crossplane.md)
