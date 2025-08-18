# 01: Setting up the GCP (GKE) Provider

To complete our multi-cloud showcase, we will now add Google Cloud Platform (GCP) to our control plane, focusing on provisioning Google Kubernetes Engine (GKE) clusters.

## 1. Installing the Provider

As with our other providers, we first ensure the GCP provider package is installed.

**File: `gitops-bootstrap/platform/provider-gcp.yaml`**

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp
spec:
  package: "xpkg.upbound.io/upbound/provider-gcp:v1.3.0"
```

This is part of our standard bootstrap process.

## 2. Configuring Authentication

GCP authentication is similar to Azure's. We will use Workload Identity to securely connect Crossplane to GCP.

### High-Level Steps

1.  **Create a GCP Service Account:** This is the identity Crossplane will use.
2.  **Grant IAM Roles:** Assign the necessary roles to the Service Account (e.g., `roles/container.admin` for managing GKE).
3.  **Create a Workload Identity Pool and Provider:** This allows Kubernetes Service Accounts to impersonate GCP Service Accounts.
4.  **Bind the KSA to the GSA:** Create an IAM policy binding that links the Crossplane provider's Kubernetes Service Account to the GCP Service Account.

### 3. Create the `ProviderConfig`

Once the GCP resources are configured, we create the `ProviderConfig`.

**File: `platform/provider-configs/gcp.yaml`**

```yaml
apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: gcp-default
spec:
  projectID: "<YOUR_GCP_PROJECT_ID>"
  credentials:
    source: "WorkloadIdentity"
    workloadIdentity:
      serviceAccountEmail: "<GCP_SERVICE_ACCOUNT_EMAIL>"
```

This `ProviderConfig` tells the GCP provider:

-   Which GCP `projectID` to operate in.
-   To use `WorkloadIdentity` for authentication.
-   The specific GCP Service Account to impersonate.

After syncing this manifest from your `platform` repository, the GCP provider will be ready.

Check its status:

```bash
kubectl get providerconfig gcp-default -o yaml
```

## Exercise: Create a GKE Composition

**Objective:** Create a `Composition` for provisioning a GKE cluster.

**Tasks:**

1.  **Define the XRD:** Create a `CompositeResourceDefinition` for a `CompositeGKECluster`. The claim, `GKECluster`, should have fields for `region`, `machineType`, and `nodeCount`.
2.  **Define the Composition:** The `Composition` should create a `gcp.container.Cluster` resource.
    -   Map the fields from the claim to the `Cluster` resource's spec.
    -   Patch the connection details (kubeconfig, endpoint, certificate) from the `Cluster` back to the `GKECluster` claim.
3.  **Commit and Push:** Add the new `Composition` to your `platform` repository.
4.  **Claim a GKE Cluster:** In your `infra-dev` repository, create a `GKECluster` claim.
5.  **Verify:** Use `crossplane trace` and the GCP Console to verify that the GKE cluster is being provisioned.

By completing this exercise, you will have a single, unified control plane capable of provisioning and managing Kubernetes clusters across Azure (AKS), GCP (GKE), and even on bare metal with a provider like Hetzner. This is the core value proposition of building an Internal Developer Platform with Crossplane.

**➡️ [Next Section: Advanced Topics](../../advanced/01-composition-functions.md)**
