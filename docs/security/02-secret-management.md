# 02: Secret Management in a GitOps World

One of the biggest challenges in a GitOps workflow is managing secrets. Git is a public, version-controlled system, which is the worst possible place to store sensitive information like API keys, database passwords, or TLS certificates.

We need a robust solution for managing secrets that integrates with our GitOps workflow without compromising security.

## The Problem with Secrets in Git

-   **Exposure:** Committing a secret to a Git repository, even a private one, is a huge risk. Once it's in the Git history, it's very difficult to truly purge.
-   **Manual Workflows:** If secrets aren't in Git, how do you get them to the cluster? Manual `kubectl create secret` commands are error-prone, not auditable, and don't scale.

## Solution: External Secrets Operator

We will use the [External Secrets Operator (ESO)](https://external-secrets.io/). ESO extends Kubernetes with a set of CRDs that allow you to fetch secrets from an external secret management system (like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault) and automatically sync them as native Kubernetes `Secret` objects.

### The Workflow

1.  **Store the Secret:** A human or a CI/CD pipeline stores a secret in your chosen secret management system (e.g., Azure Key Vault).

2.  **Create a `SecretStore`:** In your Git repository, you create a `SecretStore` or `ClusterSecretStore` resource. This tells ESO how to connect to your external secret manager.

    ```yaml
    # platform/secret-stores/azure-key-vault.yaml
    apiVersion: external-secrets.io/v1beta1
    kind: ClusterSecretStore
    metadata:
      name: azure-key-vault
    spec:
      provider:
        azure:
          vaultUrl: "https://my-key-vault.vault.azure.net/"
          authType: "WorkloadIdentity"
    ```

3.  **Create an `ExternalSecret`:** In your application or infrastructure repository, you create an `ExternalSecret` resource. This tells ESO *which* secret to fetch and *what* to name the resulting Kubernetes `Secret`.

    ```yaml
    # infra-dev/my-app-db-secret.yaml
    apiVersion: external-secrets.io/v1beta1
    kind: ExternalSecret
    metadata:
      name: my-app-db-credentials
      namespace: my-app
    spec:
      refreshInterval: "1h"
      secretStoreRef:
        name: azure-key-vault
        kind: ClusterSecretStore
      target:
        name: my-app-db-credentials # Name of the k8s Secret to create
      data:
      - secretKey: "password"
        remoteRef:
          key: "my-app-db-password" # Name of the secret in Azure Key Vault
    ```

4.  **ArgoCD Syncs:** ArgoCD syncs the `ExternalSecret` manifest from Git to the cluster.

5.  **ESO Syncs:** The External Secrets Operator sees the new `ExternalSecret`. It connects to Azure Key Vault, fetches the value of `my-app-db-password`, and creates a new Kubernetes `Secret` named `my-app-db-credentials` in the `my-app` namespace with the fetched value.

6.  **Pod Consumes the Secret:** Your application pod can now mount the `my-app-db-credentials` secret just like any other Kubernetes secret.

## Benefits of this Approach

-   **No Secrets in Git:** The actual secret value never touches the Git repository.
-   **GitOps Native:** The *request* for a secret is managed via GitOps. We have a full audit trail of who requested which secret and for what purpose.
-   **Rotation:** ESO can be configured to automatically re-fetch secrets on a schedule, enabling automated secret rotation.
-   **Separation of Concerns:** The Platform Team manages the `SecretStores`, and the Application Teams manage the `ExternalSecrets` for their own applications.

## Crossplane and Secret Management

Crossplane also needs to store secrets. When Crossplane provisions a database, it generates a password. This password needs to be stored somewhere so that applications can consume it.

Crossplane is configured to automatically write these generated secrets to a Kubernetes `Secret` object. We can then use ESO to **push** these secrets from the Kubernetes `Secret` into a central secret manager like Azure Key Vault.

This creates a closed-loop system:

1.  Crossplane creates a database and a Kubernetes `Secret` with the password.
2.  An `ExternalSecret` is configured to read from this Kubernetes `Secret`.
3.  ESO pushes the password into Azure Key Vault.
4.  Another application can then use a different `ExternalSecret` to read the password from Azure Key Vault.

**➡️ [Next: RBAC](./03-rbac.md)**
