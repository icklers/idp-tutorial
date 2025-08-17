# 01: Health Checks and Resource Status

In a declarative system, how do we know if the actual state of the world matches our desired state? This is the fundamental question of observability in a GitOps workflow.

Both Crossplane and ArgoCD provide rich status information that we can use to monitor the health of our system.

## ArgoCD Health Checks

As we discussed in the "GitOps Fundamentals" section, ArgoCD uses Lua scripts to determine the health of a resource. A resource can be in one of the following states:

-   **Healthy:** The resource is in its desired state.
-   **Progressing:** The resource is in the process of being created or updated.
-   **Degraded:** The resource has failed to reach its desired state.
-   **Suspended:** The resource is intentionally not being reconciled.
-   **Missing:** The resource is defined in Git but is not present in the cluster.
-   **Unknown:** The health of the resource could not be determined.

We have already configured custom health checks for our Crossplane resources, so the ArgoCD UI gives us a real-time, at-a-glance view of the health of our entire infrastructure stack.

## Crossplane Resource Status

Crossplane itself provides detailed status conditions on all its resources (`Claims`, `CompositeResources`, and `ManagedResources`).

You can inspect the status of any Crossplane resource using `kubectl`.

**Example: Checking the status of a `MySQLInstance` claim.**

```bash
kubectl get mysqlinstance my-app-db -o yaml
```

```yaml
apiVersion: database.example.org/v1alpha1
kind: MySQLInstance
metadata:
  name: my-app-db
# ...
status:
  conditions:
    - type: Ready
      status: "True"
      lastTransitionTime: "2023-10-27T18:30:00Z"
      reason: Available
      message: "The composite resource is available and ready"
    - type: Synced
      status: "True"
      lastTransitionTime: "2023-10-27T18:28:00Z"
      reason: ReconcileSuccess
      message: "Successfully reconciled composite resource"
  connectionDetails:
    lastPublishedTime: "2023-10-27T18:30:01Z"
  # ... other fields
```

Key fields to watch:

-   **`status.conditions`**: This array tells you the current state of the resource. The `Ready` condition with `status: "True"` is the most important indicator of health.
-   **`status.connectionDetails`**: For claims, this section will be populated with the connection information for the provisioned resource (e.g., database endpoint, username, password secret reference).

## The `crossplane` CLI

The `crossplane` command-line tool provides some helpful commands for observing the status of your resources.

**`crossplane trace`**

The `trace` command gives you a complete dependency tree for a given claim, showing you the `CompositeResource` and all the underlying `ManagedResources`.

```bash
crossplane trace mysqlinstance my-app-db
```

**Output:**

```
NAME                                            READY   STATUS    AGE
MySQLInstance/my-app-db                         True    Available 5m
└── CompositeMySQLInstance/my-app-db-xyz123     True    Available 5m
    ├── FlexibleServer/my-app-db-server         True    Available 4m
    └── FirewallRule/my-app-db-fw               True    Available 3m
```

This is an invaluable tool for debugging. If a claim is not becoming `Ready`, you can use `trace` to pinpoint exactly which underlying resource is failing.

By combining the ArgoCD UI with the detailed status information from Crossplane, we can build a comprehensive picture of our system's health.

**➡️ [Next: Monitoring with Prometheus](./02-monitoring-with-prometheus.md)**
