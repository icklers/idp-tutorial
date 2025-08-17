# 03: Cross-Repo Coordination with ArgoCD

Our multi-repo strategy is powerful, but it introduces a new challenge: how do we make all these repositories work together? How does ArgoCD discover and deploy all the Compositions, Claims, and Applications defined across our entire organization?

The answer lies back in the **Platform Core ApplicationSet** pattern.

## Revisiting the `platform-core.yaml`

Our bootstrap `platform-core.yaml` manifest deploys other ArgoCD `Application` resources. Let's look at the `gitops-bootstrap/apps` directory that it points to.

Imagine this structure:

`gitops-bootstrap/apps/`
├── `platform.yaml`
├── `infra-dev.yaml`
└── `infra-prod.yaml`

### `platform.yaml`

This file tells ArgoCD to monitor all repositories that contain our core platform definitions (like Compositions).

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-compositions
  namespace: argocd
spec:
  project: default
  source:
    # We use a repo glob to discover all repos matching a pattern
    repoURL: https://github.com/your-org/composition-*
    targetRevision: HEAD
    path: './' # Look at the root of each discovered repo
  destination:
    server: https://kubernetes.default.svc
    # Compositions are cluster-scoped, so no namespace is needed
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Key Idea:** We can use wildcards (`*`) in the `repoURL`. This single `Application` manifest tells ArgoCD to find every repository in your GitHub organization that starts with `composition-`, and deploy any `.yaml` or `.json` files it finds in them. This is how our platform automatically discovers and installs new infrastructure blueprints.

### `infra-dev.yaml`

This file tells ArgoCD to deploy the infrastructure claims for the `dev` environment.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infra-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/infra-dev.git
    targetRevision: HEAD
    path: './'
  destination:
    server: https://kubernetes.default.svc
    namespace: crossplane-claims # We deploy claims to a dedicated namespace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

This is a more traditional ArgoCD Application, pointing to a single repository that contains all the infrastructure claims for the development environment.

## The Complete Workflow

Now, let's put it all together.

1.  **A Platform Engineer wants to offer a new database type.**
    -   They create a new repository `composition-postgres-ha` from the `template-infra-composition`.
    -   They define the XRD and Composition for a highly-available PostgreSQL cluster.
    -   They push the repository to the `your-org` GitHub organization.
    -   ArgoCD's `platform-compositions` app automatically discovers the new repo and installs the Composition into the cluster.
    -   The new CRD, `CompositePostgresHA`, is now available in the Kubernetes API.

2.  **A Developer needs a new HA PostgreSQL database for their project.**
    -   They open a pull request against the `infra-dev` repository.
    -   They add a new file, `my-project-db.yaml`, containing the claim:
        ```yaml
        apiVersion: database.example.org/v1alpha1
        kind: PostgresHA
        metadata:
          name: my-project-db
        spec:
          storageGB: 50
          region: us-east-1
        ```
    -   The pull request is reviewed and merged by the DevOps team.
    -   ArgoCD's `infra-dev` app sees the new file and applies it to the cluster.
    -   Crossplane sees the new `PostgresHA` claim.
    -   It finds the `composition-postgres-ha` Composition and starts provisioning all the necessary cloud resources (e.g., a primary database, a read replica, firewall rules, etc.).

This elegant workflow allows for seamless coordination across multiple teams and repositories, all orchestrated through Git and ArgoCD.

**➡️ [Next Section: GitOps Fundamentals](../gitops-fundamentals/01-argocd-setup.md)**
