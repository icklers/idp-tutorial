# 02: Provisioning AKS Clusters

With the Azure provider configured, we can now define our first infrastructure "product": a production-ready Azure Kubernetes Service (AKS) cluster.

We will create a `Composition` that bundles together all the necessary Azure resources to create a secure, reliable AKS cluster.

## Defining the `CompositeAKSCluster` XRD

First, we define the API for our new resource. This is the `CompositeResourceDefinition` (XRD).

**File: `compositions/azure/aks.yaml`**

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: compositeaksclusters.cluster.example.org
spec:
  group: cluster.example.org
  names:
    kind: CompositeAKSCluster
    plural: compositeaksclusters
  claimNames:
    kind: AKSCluster
    plural: aksclusters
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              region:
                type: string
                description: "The Azure region to deploy the cluster in."
              nodeSize:
                type: string
                description: "The VM size for the worker nodes."
                default: "Standard_D2s_v3"
              nodeCount:
                type: integer
                description: "The number of worker nodes."
                default: 3
            required:
              - region
```

This XRD defines a new Kubernetes resource called `AKSCluster` with a `spec` that allows developers to specify the `region`, `nodeSize`, and `nodeCount`.

## Composing the AKS Cluster

Next, we define the `Composition`. This is where we specify which Azure resources will be created to satisfy an `AKSCluster` claim.

**File: `compositions/azure/aks.yaml` (continued)**

```yaml
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: aks-cluster.v1alpha1.cluster.example.org
  labels:
    provider: azure
spec:
  compositeTypeRef:
    apiVersion: cluster.example.org/v1alpha1
    kind: CompositeAKSCluster
  resources:
    - name: resourceGroup
      base:
        apiVersion: azure.upbound.io/v1beta1
        kind: ResourceGroup
        spec:
          forProvider:
            location: "from-field-path(spec.region)"
    - name: virtualNetwork
      base:
        apiVersion: network.azure.upbound.io/v1beta1
        kind: VirtualNetwork
        spec:
          forProvider:
            resourceGroupNameSelector:
              matchControllerRef: true
            location: "from-field-path(spec.region)"
            addressSpace: ["10.0.0.0/16"]
    - name: subnet
      base:
        apiVersion: network.azure.upbound.io/v1beta1
        kind: Subnet
        spec:
          forProvider:
            resourceGroupNameSelector:
              matchControllerRef: true
            virtualNetworkNameSelector:
              matchControllerRef: true
            addressPrefixes: ["10.0.1.0/24"]
    - name: aksCluster
      base:
        apiVersion: containerservice.azure.upbound.io/v1beta1
        kind: KubernetesCluster
        spec:
          forProvider:
            resourceGroupNameSelector:
              matchControllerRef: true
            location: "from-field-path(spec.region)"
            dnsPrefix: "from-field-path(metadata.name)"
            defaultNodePool:
              - name: default
                vmSize: "from-field-path(spec.nodeSize)"
                nodeCount: "from-field-path(spec.nodeCount)"
                vnetSubnetIdSelector:
                  matchControllerRef: true
            identity:
              type: "SystemAssigned"
      patches:
        # Patch the connection secret to the claim
        - fromFieldPath: "status.atProvider.kubeConfigRaw"
          toFieldPath: "status.connectionDetails.kubeconfig"
```

### Key Concepts in this Composition

-   **`resources` array:** We are composing four Azure resources: a `ResourceGroup`, a `VirtualNetwork`, a `Subnet`, and a `KubernetesCluster`.
-   **`from-field-path` patches:** We are patching values from the `AKSCluster` claim (e.g., `spec.region`) into the `spec` of the managed resources.
-   **`matchControllerRef` selectors:** Instead of hardcoding names, we use selectors to link resources together. For example, the `VirtualNetwork` will automatically be created in the `ResourceGroup` that was created as part of this same `Composition`.
-   **Connection Secret Patching:** We are patching the `kubeConfigRaw` from the `KubernetesCluster` resource into the connection details of the `AKSCluster` claim. This allows developers to get the kubeconfig for their new cluster directly from the claim.

## Exercise: Claiming an AKS Cluster

**Objective:** Provision your first AKS cluster using the new `Composition`.

**Tasks:**

1.  **Create a Claim:** In your `infra-dev` repository, create a new file named `my-aks-cluster.yaml`.
2.  **Define the Claim:** In the file, define an `AKSCluster` claim. Choose a name and an Azure region.
    ```yaml
    apiVersion: cluster.example.org/v1alpha1
    kind: AKSCluster
    metadata:
      name: my-first-aks-cluster
    spec:
      region: "westeurope"
      nodeCount: 2
    ```
3.  **Commit and Push:** Commit the new file to the `main` branch of your `infra-dev` repository.
4.  **Observe in ArgoCD:** Watch in the ArgoCD UI as the new `AKSCluster` claim is synced.
5.  **Observe in Crossplane:** Use `crossplane trace akscluster my-first-aks-cluster` to see the underlying Azure resources being provisioned.
6.  **Get the Kubeconfig:** Once the cluster is ready, get the kubeconfig from the claim's secret and connect to your new AKS cluster.

**Success Criteria:**

-   A new AKS cluster is successfully provisioned in your Azure subscription.
-   You can connect to the new cluster using the kubeconfig from the claim's secret.

**➡️ [Next Section: Hetzner Provider](../hetzner/01-setup.md)**
