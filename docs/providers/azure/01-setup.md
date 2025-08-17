# 01: Setting up the Azure Provider

Now we will configure Crossplane to connect to a specific cloud provider: Microsoft Azure.

The Crossplane `provider-azure` package contains all the necessary controllers and `ManagedResource` definitions to interact with the Azure Resource Manager (ARM) API.

## 1. Installing the Provider

First, we need to install the Azure Provider into our Crossplane installation. This is done by creating a `Provider` resource.

**File: `gitops-bootstrap/platform/provider-azure.yaml`**

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-azure
spec:
  package: "xpkg.upbound.io/upbound/provider-azure:v1.2.0"
```

This manifest tells Crossplane to download and install the specified version of the Azure provider package. We have already included this in our bootstrap process, so the provider is already installed in your cluster.

You can verify this by running:

```bash
kubectl get providers
```

## 2. Configuring Authentication

Next, Crossplane needs credentials to authenticate with your Azure subscription. We will use a **Workload Identity** for this, which is the most secure method for applications running in Kubernetes to access Azure resources.

### High-Level Steps

1.  **Create an Azure AD Application and Service Principal:** This will be the identity that Crossplane uses.
2.  **Create a Federated Identity Credential:** This links the Azure AD Service Principal to a Kubernetes Service Account.
3.  **Grant Permissions:** Assign the necessary RBAC roles (e.g., `Contributor`) to the Service Principal on your Azure subscription.
4.  **Create a `ProviderConfig`:** This Kubernetes resource tells the Crossplane Azure Provider how to authenticate.

### Creating the `ProviderConfig`

Once you have set up the necessary resources in Azure, you will create a `ProviderConfig` manifest. **Remember to replace the placeholder values for `clientId`, `tenantId`, and `subscriptionID` with your actual Azure credentials.**

**File: `platform/provider-configs/azure.yaml`**

```yaml
apiVersion: azure.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: azure-default
spec:
  credentials:
    source: "WorkloadIdentity"
    workloadIdentity:
      clientId: "<AZURE_CLIENT_ID>" # The client ID of your AAD Application
      tenantId: "<AZURE_TENANT_ID>" # The tenant ID of your Azure subscription
  subscriptionID: "<AZURE_SUBSCRIPTION_ID>"
```

This manifest instructs the Azure provider to use the Workload Identity associated with the specified `clientId` and `tenantId`.

**Important:** The `ProviderConfig` is a cluster-scoped resource. The `name: default` is a convention. You can have multiple `ProviderConfig` resources if you need to connect to different Azure subscriptions or tenants.

### Applying the `ProviderConfig`

This `ProviderConfig` manifest should be stored in your `platform` repository and synced by ArgoCD. Once it is applied, the Azure provider will attempt to authenticate with Azure.

You can check the status of the provider:

```bash
kubectl get providerconfig
```

Look for the `Ready` status to be `True`.

## Conclusion

With the `Provider` installed and the `ProviderConfig` applied, Crossplane is now ready to start provisioning resources in your Azure subscription.

In the next section, we will create a `Composition` to define a production-ready Azure Kubernetes Service (AKS) cluster.

**➡️ [Next: Provisioning AKS](./02-provisioning-aks.md)**
