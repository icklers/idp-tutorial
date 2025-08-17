# 03: RBAC and Least Privilege

Role-Based Access Control (RBAC) is a critical component of a secure Kubernetes environment. In our GitOps model, we need to manage RBAC for two distinct domains:

1.  **Who can do what in the Kubernetes cluster?** (Kubernetes RBAC)
2.  **Who can do what in our Git repositories?** (Git Provider RBAC)

## Kubernetes RBAC

Our goal is the **principle of least privilege**. Every user and every component should only have the permissions they absolutely need to perform their function.

### ArgoCD RBAC

We have already configured the RBAC for the ArgoCD controllers. But what about human users?

ArgoCD has its own RBAC system for controlling who can manage applications, projects, and repositories within the ArgoCD UI and API.

We can manage this declaratively in the `argocd-rbac-cm` ConfigMap.

**Example: Granting a `dev-team` group read-only access to their project.**

```yaml
# argocd-rbac-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    g, dev-team, role:readonly
    p, role:readonly, applications, get, dev-project/*, allow
```

This configuration, when applied by our bootstrap ArgoCD app, will:

-   Define a group called `dev-team`.
-   Assign it a `readonly` role.
-   The `readonly` role is allowed to `get` applications within the `dev-project`.

### Crossplane RBAC

Crossplane introduces a new layer of RBAC. Who is allowed to create a `Claim` for a new database? Who is allowed to create a `Composition`?

We manage this with standard Kubernetes `Roles` and `RoleBindings`.

**Example: Allowing the `dev-team` to create `MySQLInstance` claims.**

```yaml
# rbac/dev-team-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-team-claim-creator
  namespace: default # Or the namespace where claims are created
rules:
- apiGroups:
  - "database.example.org"
  resources:
  - "mysqlinstances"
  verbs:
  - "create"
  - "get"
  - "list"
  - "watch"
  - "delete"
---
# rbac/dev-team-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-claim-binding
  namespace: default
subjects:
- kind: Group
  name: dev-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-team-claim-creator
  apiGroup: rbac.authorization.k8s.io
```

These manifests are stored in our `platform` repository and synced by ArgoCD, ensuring that our RBAC policies are also managed via GitOps.

## Git Provider RBAC

Equally important is the RBAC in your Git provider (e.g., GitHub, GitLab).

### Branch Protection Rules

-   **`platform` repository:** The `main` branch should be heavily protected. Require multiple approvers from the Platform Team for all pull requests. Forbid force pushing.
-   **`infra-*` repositories:** The `main` branch should be protected. Require at least one approval from a senior member of the DevOps/SRE team.
-   **Application repositories:** The `main` branch should be protected. Require at least one approval from a peer developer.

### CODEOWNERS

Use the `CODEOWNERS` file in each repository to automatically request reviews from the responsible team.

**Example: `platform/.github/CODEOWNERS`**

```
# All changes to Compositions must be reviewed by the platform team
/compositions/  @my-org/platform-team

# Changes to RBAC policies require security review
/rbac/          @my-org/security-team
```

By codifying our RBAC policies in both Kubernetes and Git, we create a secure, auditable, and automated system for managing permissions.

**➡️ [Next Section: Observability](../observability/01-health-checks.md)**
