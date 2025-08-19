# 02: Repository Templates

To make our multi-repo strategy scalable, we need to make it easy for teams to create new infrastructure and application repositories that adhere to our standards. GitHub Templates are an excellent way to achieve this.

We have provided two templates in the `repository-templates/` directory:

1.  `template-infra-composition`: A template for creating new Crossplane Compositions.
2.  `template-microservice`: A template for a standard microservice, including its Kubernetes manifests.

## Template 1: `template-infra-composition`

This template provides the basic scaffolding for defining a new type of infrastructure "product" that your platform will offer.

**File:** `repository-templates/template-infra-composition/composition.yaml`

```yaml
# --- This file is a template. To use it, copy this directory and update the content. ---
apiVersion: apiextensions.crossplane.io/v2
kind: CompositeResourceDefinition
metadata:
  name: # e.g., xpostgresqlinstances.database.example.org
spec:
  group: # e.g., database.example.org
  names:
    kind: # e.g., XPostgreSQLInstance
    plural: # e.g., xpostgresqlinstances
  scope: Namespaced  # Default in v2
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
                parameters:
                  type: object
                  properties:
                    # Add your Composite Resource parameter fields here
                    # e.g., storageGB, region, version
                  required: []
                crossplane:
                  type: object
                  properties:
                    compositionRef:
                      type: object
                      properties:
                        name:
                          type: string
                      required:
                      - name
                  required:
                  - compositionRef
              required:
              - parameters
              - crossplane
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: # e.g., azure-postgresql-server.v1alpha1.database.example.org
  labels:
    provider: azure
    # Add other identifying labels
spec:
  compositeTypeRef:
    apiVersion: # e.g., database.example.org/v1alpha1
    kind: # e.g., XPostgreSQLInstance
  resources:
    # Define the cloud resources that make up this Composition
    # e.g., azure.dbformysql.flexible.FlexibleServer, azure.dbformysql.flexible.FirewallRule
    - name: my-resource
      base:
        apiVersion: #
        kind: #
        spec: #
```

### How to Use It

1.  **Create a New Repository:** A platform engineer who wants to define a new infrastructure type (e.g., a Redis cluster) would create a new repository from this template.
2.  **Define the XRD:** They would fill out the `CompositeResourceDefinition` (XRD). The XRD defines the API for the new resource type. It specifies the fields that a developer can set in their Composite Resource parameters (e.g., `version`, `size`).
3.  **Define the Composition:** They would then fill out the `Composition`. The Composition defines which actual cloud resources (e.g., `azurerm_redis_cache`, `google_redis_instance`) will be created to satisfy the Composite Resource.
4.  **Commit and Push:** They commit this file to their new repository.

## Template 2: `template-microservice`

This template provides a starting point for a new microservice.

**File:** `repository-templates/template-microservice/deployment.yaml`

```yaml
# --- This file is a template. To use it, copy this directory and update the content. ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # my-app
  namespace: # my-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: # my-app
  template:
    metadata:
      labels:
        app: # my-app
    spec:
      containers:
        - name: # my-app
          image: # my-registry/my-app:latest
          ports:
            - containerPort: 8080
```

### How to Use It

1.  **Create a New Repository:** A developer starting a new microservice would create a new repository from this template.
2.  **Add Source Code:** They would add their application source code (e.g., `main.go`, `app.py`).
3.  **Update Deployment Manifest:** They would update the `deployment.yaml` with the correct image name, container port, and other application-specific settings.
4.  **Commit and Push:** They commit their code and the manifest to their new repository.

By providing these templates, we lower the barrier to entry and ensure that all new components in our ecosystem follow a consistent structure.

**➡️ [Next: Cross-Repo Coordination](./03-cross-repo-coordination.md)**
