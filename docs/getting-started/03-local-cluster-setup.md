# 03: Local Cluster Setup

With our Devbox environment activated, we have all the necessary command-line tools. Now, we need to create the local Kubernetes cluster that will serve as the foundation for our platform.

## 1. Create the Local Kubernetes Cluster

We have provided a `kind-cluster.yaml` file inside the `gitops-bootstrap/kind-cluster/` directory. Take a moment to inspect its contents:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
```

Now, from within your `devbox shell`, run:

```bash
# Use the provided gitops-bootstrap configuration (recommended)
kind create cluster --config gitops-bootstrap/kind-cluster/kind-cluster.yaml
```

> **Note:** There's also an exercise file at `exercises/getting-started/cluster-setup/kind-cluster.yaml` with the same configuration for practice, but we recommend using the gitops-bootstrap version for consistency with the tutorial structure.

This will take a few minutes to provision a new, single-node Kubernetes cluster. Once it's complete, your `kubectl` context will automatically be configured to point to the new `kind-idp-tutorial` cluster.

Verify the cluster is running:

```bash
kubectl cluster-info
```

With our cluster running, we are ready to bootstrap our GitOps engine, ArgoCD.

**➡️ [Next: GitOps Bootstrap](./04-gitops-bootstrap.md)**