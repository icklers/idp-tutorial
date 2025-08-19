# 01: ArgoCD Setup for Crossplane

We have already installed ArgoCD, but to make it work effectively with Crossplane, we need to understand a few key configuration details.

## Health Checks

ArgoCD determines the health of a resource by running a series of Lua scripts. By default, it doesn't know how to interpret the status of Crossplane's custom resources. A `Composition` might be creating resources, but ArgoCD will show it as `Progressing` indefinitely.

We need to provide custom health checks for Crossplane resources.

**Example Health Check for a Crossplane `Composition`:**

```lua
-- health.lua
hs = {}
if obj.status ~= nil then
  if obj.status.conditions ~= nil and #obj.status.conditions > 0 then
    for i, condition in ipairs(obj.status.conditions) do
      if condition.type == "Ready" then
        if condition.status == "True" then
          hs.status = "Healthy"
          hs.message = condition.message or "Resource is ready"
        else
          hs.status = "Progressing"
          hs.message = condition.message or "Resource is not ready"
        end
        return hs
      end
    end
  end
end
hs.status = "Progressing"
hs.message = "No Ready condition found"
return hs
```

This script tells ArgoCD to look at the `status.conditions` array of a Composition resource. It safely iterates through the conditions to find the one with `type: Ready`. If that condition has a `status: "True"`, then the resource is considered `Healthy`. The script handles edge cases where conditions might not exist.

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

> 📁 **Exercise Files**: Available at [`exercises/gitops-fundamentals/argocd/argocd-cm-crossplane-health.yaml`](../../exercises/gitops-fundamentals/argocd/argocd-cm-crossplane-health.yaml)
>
> ```bash
> user@host idp-tutorial> $ cp exercises/gitops-fundamentals/argocd/argocd-cm-crossplane-health.yaml gitops-bootstrap/argocd/
> 
> # OPTION A — Bootstrap (one-liner commit+push)
> user@host idp-tutorial> $ git add gitops-bootstrap/argocd/argocd-cm-crossplane-health.yaml && git commit -m "gitops: add crossplane health checks to ArgoCD" && git push -u origin HEAD
>
> # OPTION B — PR flow for reviewable changes
> user@host idp-tutorial> $ git switch -c chore/argocd-health
> user@host idp-tutorial> $ git add gitops-bootstrap/argocd/argocd-cm-crossplane-health.yaml
> user@host idp-tutorial> $ git commit -m "gitops: add crossplane health checks to ArgoCD"
> user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
> user@host idp-tutorial> $ gh pr create --title "Add ArgoCD Crossplane health checks" --body "Adds health.lua customizations via argocd-cm"
> user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
> ```

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

> 📁 **Exercise Files**: Available at [`exercises/gitops-fundamentals/rbac/argocd-crossplane-clusterrole.yaml`](../../exercises/gitops-fundamentals/rbac/argocd-crossplane-clusterrole.yaml)
>
> ```bash
> user@host idp-tutorial> $ cp exercises/gitops-fundamentals/rbac/argocd-crossplane-clusterrole.yaml platform-core/
> 
> # OPTION A — Bootstrap (one-liner commit+push)
> user@host idp-tutorial> $ git add platform-core/argocd-crossplane-clusterrole.yaml && git commit -m "gitops: grant ArgoCD controller permissions for Crossplane" && git push -u origin HEAD
>
> # OPTION B — PR flow for reviewable changes
> user@host idp-tutorial> $ git switch -c chore/argocd-rbac
> user@host idp-tutorial> $ git add platform-core/argocd-crossplane-clusterrole.yaml
> user@host idp-tutorial> $ git commit -m "gitops: grant ArgoCD controller permissions for Crossplane"
> user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
> user@host idp-tutorial> $ gh pr create --title "Grant ArgoCD controller Crossplane RBAC" --body "Adds ClusterRole for compositions and XRDs"
> user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
> ```

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
