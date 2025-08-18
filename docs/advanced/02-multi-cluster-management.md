# 02: Multi-Cluster Management

We have successfully used Crossplane to provision Kubernetes clusters (AKS and GKE). This is known as **Cluster as a Service**. The next logical step is to manage the applications and configurations *on* those newly created clusters.

This is the domain of **Multi-Cluster Management (MCM)**.

## The Challenge

How do we use our GitOps workflow, which is centered around our main KinD cluster, to push manifests and configurations to the new AKS and GKE clusters we are creating?

## Solution: ArgoCD ApplicationSets + Crossplane

We can combine the power of Crossplane and ArgoCD ApplicationSets to create a fully automated, closed-loop multi-cluster management system.

### The `Cluster` Object

ArgoCD has a concept of a `Cluster`. You can tell ArgoCD about a new remote cluster by creating a Kubernetes `Secret` with the cluster's kubeconfig.

The `ApplicationSet` controller can then use this `Cluster` secret as a target for deploying applications.

### The Workflow

1.  **Crossplane Provisions a Cluster:** We start by claiming an `AKSCluster` or `GKECluster` as we did in the previous sections.

2.  **Crossplane Creates a Kubeconfig Secret:** Our `Composition` is configured to patch the `kubeconfig` from the provisioned cluster into the connection details of the claim. This creates a `Secret` in our main KinD cluster.

3.  **The `ApplicationSet` Generator:** We will create an `ApplicationSet` that uses a `Cluster` generator. This generator automatically discovers any `Secret` that is labeled as an ArgoCD cluster secret.

    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: ApplicationSet
    metadata:
      name: guestbook-to-all-clusters
    spec:
      generators:
      - clusters: {}
      template:
        metadata:
          name: '{{name}}-guestbook'
        spec:
          project: default
          source:
            repoURL: https://github.com/argoproj/argocd-example-apps.git
            targetRevision: HEAD
            path: guestbook
          destination:
            server: '{{server}}'
            namespace: guestbook
    ```

4.  **Labeling the Secret:** We need to tell the `ApplicationSet` generator that our Crossplane-generated kubeconfig secret represents a new cluster. We can do this with a Composition Function or a simple Kubernetes controller that watches for our `AKSCluster` claims and automatically adds the `argocd.argoproj.io/secret-type: cluster` label to the generated secret.

5.  **ArgoCD Deploys the App:**
    -   The `ApplicationSet` controller discovers the newly labeled secret.
    -   It adds the new AKS/GKE cluster to ArgoCD's list of managed clusters.
    -   It then uses the `template` to generate a new ArgoCD `Application`.
    -   This new `Application` points to the `guestbook` app, but its `destination.server` is set to the API server of the new remote cluster.
    -   ArgoCD syncs the guestbook manifests to the new AKS/GKE cluster.

## The Result: A Fully Automated Pipeline

With this pattern, the entire lifecycle is automated:

1.  A developer claims a new `AKSCluster` in a Git repository.
2.  Crossplane provisions the cluster.
3.  Crossplane creates a kubeconfig secret.
4.  A controller labels the secret for ArgoCD.
5.  The `ApplicationSet` detects the new cluster.
6.  ArgoCD deploys a standard set of baseline applications (e.g., an ingress controller, a monitoring agent, the guestbook app) to the new cluster without any human intervention.

This is the pinnacle of GitOps-driven platform engineering. We have used Git to orchestrate the creation of a new cluster and the bootstrapping of all its day-1 applications.

**➡️ [Next: Troubleshooting](../troubleshooting.md)**
