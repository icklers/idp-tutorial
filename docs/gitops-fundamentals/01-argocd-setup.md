# 01: ArgoCD Setup for Crossplane

We have already installed ArgoCD, but to make it work effectively with Crossplane, we need to understand a few key configuration details.

## Health Checks

ArgoCD determines the health of a resource by running a series of Lua scripts. By default, it doesn't know how to interpret the status of Crossplane's custom resources. A `Composition` might be creating resources, but ArgoCD will show it as `Progressing` indefinitely.

We need to provide custom health checks for Crossplane resources.

**Example Health Check for a Crossplane `Composition`:**

```lua
-- health.lua
hs = {}
hs.status = "Healthy"
hs.message = obj.status.conditions[1].message
if obj.status.conditions[1].type ~= "Ready" or obj.status.conditions[1].status ~= "True" then
  hs.status = "Progressing"
end
return hs
```

This script tells ArgoCD to look at the `status.conditions` array of a Composition resource. If the condition with `type: Ready` has a `status: "True"`, then the resource is considered `Healthy`.

### How to Apply Health Checks

You can add these custom health checks to the `argocd-cm` ConfigMap in the `argocd` namespace.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations: |
    apiextensions.crossplane.io/Composition:
      health.lua: |
        -- ... (lua script from above) ...
```

We have already applied a set of standard Crossplane health checks as part of our bootstrap process.

## RBAC for Crossplane Resources

By default, the ArgoCD Application Controller does not have permission to create or manage cluster-scoped resources like `Compositions` or `CompositeResourceDefinitions`. We need to grant it these permissions.

This is done by creating a `ClusterRole` and a `ClusterRoleBinding`.

**Example `ClusterRole`:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-crossplane-manager-role
rules:
- apiGroups:
  - apiextensions.crossplane.io
  resources:
  - compositions
  - compositeresourcedefinitions
  verbs:
  - "*"
```

Then, you bind this role to the ArgoCD Application Controller's Service Account.

This is also handled automatically by our bootstrap configuration.

## Sync Options

When working with CRDs, it's often necessary to tell ArgoCD to manage them in a specific way. The most common issue is that a CRD is applied, and then a Custom Resource (CR) that uses that CRD is applied immediately after. This can lead to a race condition where the CR is created before the CRD is fully recognized by the Kubernetes API server.

To solve this, we use a Sync Hook:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  annotations:
    argocd.argoproj.io/sync-wave: "1"
# ...
---
apiVersion: my.crd.io/v1alpha1
kind: MyCustomResource
metadata:
  name: my-cr
  annotations:
    argocd.argoproj.io/sync-wave: "2"
# ...
```

By adding `sync-wave` annotations, we can instruct ArgoCD to apply resources in a specific order. Resources with a lower sync-wave number are applied first, and ArgoCD waits for them to become healthy before proceeding to the next wave.

We use this pattern extensively in our bootstrap process to ensure that Crossplane Providers are healthy before we try to create any Compositions.

**➡️ [Next: Crossplane Integration](./02-crossplane-integration.md)**
