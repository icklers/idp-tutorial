# Hetzner Solutions

This file contains the solutions to the exercises in the Hetzner-related sections of the tutorial.

---

## Solution: Multi-Cloud Composition

**Source Exercise:** `docs/providers/hetzner/02-provisioning-servers.md`

### 1. `compositions/multi-cloud/webapp.yaml`

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: compositewebapps.app.example.org
spec:
  group: app.example.org
  names:
    kind: CompositeWebApp
    plural: compositewebapps
  claimNames:
    kind: WebApp
    plural: webapps
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
                hetznerLocation:
                  type: string
                serverType:
                  type: string
                azureRegion:
                  type: string
                dbSku:
                  type: string
                  default: "GP_Gen5_2"
              required:
                - hetznerLocation
                - serverType
                - azureRegion
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: hetzner-azure-webapp.v1alpha1.app.example.org
  labels:
    provider: multi-cloud
spec:
  compositeTypeRef:
    apiVersion: app.example.org/v1alpha1
    kind: CompositeWebApp
  resources:
    # Hetzner Server
    - name: webServer
      base:
        apiVersion: hcloud.crossplane.io/v1alpha1
        kind: Server
        spec:
          forProvider:
            location: "from-field-path(spec.hetznerLocation)"
            serverType: "from-field-path(spec.serverType)"
            image: "ubuntu-22.04"

    # Azure Database
    - name: azureResourceGroup
      base:
        apiVersion: azure.upbound.io/v1beta1
        kind: ResourceGroup
        spec:
          forProvider:
            location: "from-field-path(spec.azureRegion)"
    - name: azureMySQL
      base:
        apiVersion: dbformysql.azure.upbound.io/v1beta1
        kind: FlexibleServer
        spec:
          forProvider:
            resourceGroupNameSelector:
              matchControllerRef: true
            location: "from-field-path(spec.azureRegion)"
            skuName: "from-field-path(spec.dbSku)"
            administratorLogin: "mysqladmin"
            administratorPasswordSecretRef:
              namespace: crossplane-system
              name: mysql-password
              key: password
```

### 2. `WebApp` Claim

```yaml
apiVersion: app.example.org/v1alpha1
kind: WebApp
metadata:
  name: my-multi-cloud-app
spec:
  hetznerLocation: "fsn1"
  serverType: "cx11"
  azureRegion: "westeurope"
```

---

## Solution: Add an SSH key to the Hetzner server `Composition`

### 1. Create the SSH Key in Hetzner

First, add your public SSH key to your Hetzner project and give it a name (e.g., `my-ssh-key`).

### 2. Update the `Composition`

Modify the `hetzner-bare-metal` `Composition` to include the `sshKeys` field.

```yaml
# ... (Composition spec)
  resources:
    - name: hetznerServer
      base:
        apiVersion: hcloud.crossplane.io/v1alpha1
        kind: Server
        spec:
          forProvider:
            # ... other fields
            sshKeys:
              - "my-ssh-key" # Add the name of your key here
```

Now, any new `BareMetalServer` claimed from this `Composition` will automatically have your SSH key installed.
