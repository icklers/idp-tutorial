# 02: Crossplane Integration with ArgoCD

Now that ArgoCD is configured to understand Crossplane, let's look at how they work together to create our GitOps-powered IDP.

## The Core Concept: Separation of Duties

The power of this integration comes from a clear separation of duties:

-   **Crossplane** is the **engine**. It knows *how* to create infrastructure. It contains the logic for talking to cloud provider APIs. It defines the `Compositions` (the blueprints).
-   **ArgoCD** is the **delivery mechanism**. It knows *what* to create. It watches our Git repositories and applies the desired state (the `Claims`) to the cluster.

This separation is crucial. The Platform Team owns the Crossplane side of the house, and the Development Teams own the ArgoCD side (by creating claims in their repositories).

## The Reconciliation Loop

Let's trace the end-to-end reconciliation loop:

1.  **A developer pushes a Claim** to an `infra-` repository.
    ```yaml
    # infra-dev/my-app-db.yaml
    apiVersion: database.example.org/v1alpha1
    kind: MySQLInstance
    metadata:
      name: my-app-db
    spec:
      storageGB: 20
    ```

2.  **ArgoCD syncs the repository.** It sees the new `MySQLInstance` claim and applies it to the Kubernetes cluster using `kubectl apply`.

3.  **Crossplane's `Claim Controller` wakes up.** It sees a new `MySQLInstance` claim that has no corresponding `CompositeResource` (CR).

4.  **Crossplane selects a `Composition`.** It looks at the `claim` and, based on labels or other selectors, chooses a `Composition` that can satisfy it. For example, it might select `azure-mysql-server.v1alpha1.database.example.org`.

5.  **Crossplane creates a `CompositeResource` (CR).** It uses the selected `Composition` as a template to create a new `CompositeMySQLInstance` resource.

6.  **Crossplane's `CompositeResource Controller` wakes up.** It sees the new `CompositeMySQLInstance` and, based on the `Composition`, starts creating the actual managed resources (e.g., an `azure.dbformysql.flexible.FlexibleServer` and a `FirewallRule`).

7.  **Crossplane Provider Controllers take over.** The `provider-azure` controller sees the new Azure-specific resources and makes the necessary API calls to Azure to provision the database.

8.  **Status is reported back up the chain.** As the resources are created in Azure, the status is updated on the `FlexibleServer` resource, which updates the `CompositeMySQLInstance`, which in turn updates the `MySQLInstance` claim. The developer can run `kubectl get mysqlinstance my-app-db -o yaml` and see the connection details and status.

## Viewing the Sync in ArgoCD

This entire workflow is visible in the ArgoCD UI. You can click on the `infra-dev` Application and see:

-   The `MySQLInstance` claim that was synced from Git.
-   The `CompositeMySQLInstance` that was created by Crossplane.
-   The underlying `FlexibleServer` and `FirewallRule` resources.

ArgoCD provides a complete, real-time dependency graph of your entire infrastructure, from the Git commit all the way down to the individual cloud resources.

This tight integration provides unparalleled visibility and control over your infrastructure.

**➡️ [Next: Workflow Patterns](./03-workflow-patterns.md)**
