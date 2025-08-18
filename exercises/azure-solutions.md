# Azure Solutions

This file contains the solutions to the exercises in the Azure-related sections of the tutorial.

---

## Solution: Composition for a Production-Ready Storage Account

**Source Exercise:** `docs/providers/azure/02-provisioning-aks.md`

### 1. `composition-azure-storage/composition.yaml`

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: compositestorageaccounts.storage.example.org
spec:
  group: storage.example.org
  names:
    kind: CompositeStorageAccount
    plural: compositestorageaccounts
  claimNames:
    kind: StorageAccount
    plural: storageaccounts
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                region:
                  type: string
                storageAccountName:
                  type: string
                redundancy:
                  type: string
                  enum: ["LRS", "GRS"]
                  default: "LRS"
              required:
                - region
                - storageAccountName
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: azure-storage-account.v1alpha1.storage.example.org
  labels:
    provider: azure
spec:
  compositeTypeRef:
    apiVersion: storage.example.org/v1alpha1
    kind: CompositeStorageAccount
  resources:
    - name: resourceGroup
      base:
        apiVersion: azure.upbound.io/v1beta1
        kind: ResourceGroup
        spec:
          forProvider:
            location: "from-field-path(spec.region)"
    - name: storageAccount
      base:
        apiVersion: storage.azure.upbound.io/v1beta1
        kind: Account
        spec:
          forProvider:
            resourceGroupNameSelector:
              matchControllerRef: true
            location: "from-field-path(spec.region)"
            accountTier: "Standard"
            accountReplicationType: "from-field-path(spec.redundancy)"
```

### 2. ArgoCD Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: composition-azure-storage
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/composition-azure-storage.git
    targetRevision: HEAD
    path: './'
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 3. `StorageAccount` Claim

```yaml
apiVersion: storage.example.org/v1alpha1
kind: StorageAccount
metadata:
  name: my-production-storage
spec:
  region: "westeurope"
  storageAccountName: "myprodsaxyz123"
  redundancy: "GRS"
```

---

## Solution: Enforcing Policy with Kyverno

**Source Exercise:** `docs/security/01-chainguard-images.md`

### `gitops-bootstrap/apps/platform/kyverno-policy.yaml`

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-provider-image-source
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: check-crossplane-provider-signatures
      match:
        any:
        - resources:
            kinds:
              - Pod
            namespaces:
              - crossplane-system
      verifyImages:
      - imageReferences:
        - "cgr.dev/chainguard/provider-*"
        attestors:
        - count: 1
          entries:
          - keyless:
              subject: "https://github.com/chainguard-images/images/.github/workflows/release.yaml@refs/heads/main"
              issuer: "https://token.actions.githubusercontent.com"
```
