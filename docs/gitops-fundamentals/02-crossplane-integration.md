# 02: Crossplane Integration with ArgoCD

Now that ArgoCD is configured to understand Crossplane, let's look at how they work together to create our GitOps-powered IDP.

## The Core Concept: Separation of Duties

The power of this integration comes from a clear separation of duties:

-   **Crossplane** is the **engine**. It knows *how* to create infrastructure. It contains the logic for talking to cloud provider APIs. It defines the `Compositions` (the blueprints).
-   **ArgoCD** is the **delivery mechanism**. It knows *what* to create. It watches our Git repositories and applies the desired state (the `Composite Resources`) to the cluster.

This separation is crucial. The Platform Team owns the Crossplane side of the house, and the Development Teams own the ArgoCD side (by creating Composite Resources in their repositories).

## The Reconciliation Loop

Let's trace the end-to-end reconciliation loop:

1.  **A developer pushes a Composite Resource** to an `infra-` repository.
    ```yaml
    # infra-dev/my-app-db.yaml
    apiVersion: database.example.org/v1alpha1
    kind: XPostgreSQLInstance
    metadata:
      name: my-app-db
      namespace: default
    spec:
      parameters:
        storageGB: 20
      crossplane:
        compositionRef:
          name: azure-postgresql-server
    ```

  > 📁 **Exercise Files**: Available at [`exercises/gitops-fundamentals/crossplane-integration/xpostgresqlinstance-dev.yaml`](../../exercises/gitops-fundamentals/crossplane-integration/xpostgresqlinstance-dev.yaml)
  > 
  > ```bash
  > # Copy into the dev environment infrastructure path (GitOps-managed)
  > user@host idp-tutorial> $ cp exercises/gitops-fundamentals/crossplane-integration/xpostgresqlinstance-dev.yaml exercises/crossplane-advanced-patterns-01/environments/dev/infrastructure/
  > 
  > # Commit on a feature branch and open a PR
  > user@host idp-tutorial> $ git switch -c feat/dev-db-example
  > user@host idp-tutorial> $ git add exercises/crossplane-advanced-patterns-01/environments/dev/infrastructure/xpostgresqlinstance-dev.yaml
  > user@host idp-tutorial> $ git commit -m "gitops: add xpostgresqlinstance for dev (integration example)"
  > user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
  > user@host idp-tutorial> $ gh pr create --title "Add dev XPostgreSQLInstance example" --body "Adds integration example CR for dev"
  > user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch
  > ```

2.  **ArgoCD syncs the repository.** It sees the new `XPostgreSQLInstance` Composite Resource and applies it to the Kubernetes cluster using `kubectl apply`.

3.  **Crossplane's Composition Controller wakes up.** It sees a new `XPostgreSQLInstance` Composite Resource and identifies the associated Composition.

4.  **Crossplane processes the Composition.** It uses the Composition template to determine which managed resources need to be created. For example, it might use the `azure-postgresql-server` Composition.

5.  **Crossplane creates Managed Resources.** It creates the individual cloud resources (e.g., `FlexibleServer`, `FirewallRule`) as defined in the Composition.

6.  **Crossplane Provider Controllers take over.** The `provider-azure` controller sees the new Azure-specific resources and makes the necessary API calls to Azure to provision the database.

7.  **Status is reported back up the chain.** As the resources are created in Azure, the status is updated on the `FlexibleServer` resource, which updates the `XPostgreSQLInstance` Composite Resource. The developer can run `kubectl get xpostgresqlinstance my-app-db -o yaml` and see the connection details and status.

## Viewing the Sync in ArgoCD

This entire workflow is visible in the ArgoCD UI. You can click on the `infra-dev` Application and see:

-   The `XPostgreSQLInstance` Composite Resource that was synced from Git.
-   The underlying `FlexibleServer` and `FirewallRule` resources.

ArgoCD provides a complete, real-time dependency graph of your entire infrastructure, from the Git commit all the way down to the individual cloud resources.

This tight integration provides unparalleled visibility and control over your infrastructure.

**➡️ [Next: Workflow Patterns](./03-workflow-patterns.md)**
