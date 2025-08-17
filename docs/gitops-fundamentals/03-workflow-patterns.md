# 03: GitOps Workflow Patterns

With our tooling in place, we can now implement powerful GitOps workflows. Let's explore a few common patterns for managing infrastructure and applications.

## Pattern 1: Infrastructure Promotion (Git-based)

How do you promote infrastructure changes from a `dev` environment to a `staging` and `prod` environment?

We will use a Git-based promotion strategy. Each environment has its own infrastructure repository (`infra-dev`, `infra-staging`, `infra-prod`).

1.  **Initial Deployment to Dev:** A new database claim is first committed to the `main` branch of the `infra-dev` repository. ArgoCD deploys it to the `dev` cluster.

2.  **Promotion to Staging:** Once the change is validated in `dev`, we promote it to `staging`. Instead of copying files, we will create a **pull request** from the `infra-dev` repository to the `infra-staging` repository.
    -   This PR contains the exact commit with the new database claim.
    -   The Platform Team can review the PR, ensuring that the correct changes are being promoted.
    -   Once the PR is merged, ArgoCD deploys the claim to the `staging` cluster.

3.  **Promotion to Production:** The same process is repeated for production. A PR is opened from `infra-staging` to `infra-prod`.

This Git-based workflow provides a full audit trail for every change in every environment. We can see exactly who promoted what, and when.

## Pattern 2: Application and Infrastructure Together

Sometimes, an application has a dedicated piece of infrastructure that isn't shared. In this case, it makes sense to manage the application and its infrastructure in the same repository.

Consider a microservice that needs a dedicated Redis cache.

**Repository:** `my-microservice`

```
my-microservice/
├── app/
│   └── main.go
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
└── infrastructure/
    └── redis-claim.yaml
```

We can create an ArgoCD `ApplicationSet` to manage this.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices
spec:
  generators:
    - git:
        repoURL: https://github.com/your-org/my-microservice.git
        revision: HEAD
        directories:
          - path: "*"
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/your-org/my-microservice.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
```

This `ApplicationSet` will:

1.  Scan the `my-microservice` repository for directories.
2.  For each directory (`kubernetes`, `infrastructure`), it will create a new ArgoCD `Application`.
3.  The `kubernetes` app will deploy the manifests to the `kubernetes` namespace.
4.  The `infrastructure` app will deploy the Redis claim to the `infrastructure` namespace.

This pattern keeps related resources together and allows developers to manage their app's entire lifecycle from a single repository.

## Pattern 3: Ephemeral Environments for Pull Requests

For true end-to-end testing, we want to spin up an entire environment for each pull request.

1.  A developer opens a PR in an application repository.
2.  A GitHub Action is triggered.
3.  The action creates a new infrastructure claim for a temporary database and a new Kubernetes `Deployment` manifest, both with the PR number in their name (e.g., `my-app-pr-123`).
4.  These temporary manifests are pushed to a dedicated `infra-pr` repository.
5.  ArgoCD deploys the temporary infrastructure and application.
6.  The GitHub Action runs integration tests against the temporary environment.
7.  When the PR is merged or closed, another GitHub Action runs to delete the temporary manifests from the `infra-pr` repo, and ArgoCD automatically tears down the environment.

This powerful pattern allows for complete, isolated testing of every change before it hits your main environments.

**➡️ [Next Section: Security](../security/01-chainguard-images.md)**
