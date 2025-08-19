# 01: Introduction to the GitOps Workflow

Welcome to the first module of the Crossplane GitOps tutorial. In this section, we will lay the foundation for everything that follows. We'll start with a high-level overview of the core technologies and the architectural philosophy behind this course.

## The Core Components

Our platform is built on three key pillars:

1.  **Crossplane:** The infrastructure-as-code engine. Crossplane extends Kubernetes with Custom Resource Definitions (CRDs) that allow us to model infrastructure from any cloud provider as native Kubernetes objects. Instead of writing HCL, Bicep, or CloudFormation, we write YAML.

2.  **ArgoCD:** The GitOps engine. ArgoCD continuously monitors a Git repository and applies the desired state (defined in YAML) to our Kubernetes cluster. This ensures that our cluster's state always matches what's in Git.

3.  **DevBox:** The environment management tool. DevBox ensures that every developer on the team has the exact same version of every tool, library, and dependency, providing a reproducible development environment that works consistently across different machines and CI/CD pipelines.

## The Architectural Vision: A True IDP

Our goal is not just to automate infrastructure provisioning. We are building an **Internal Developer Platform (IDP)**. What does this mean?

-   **Self-Service for Developers:** Developers can provision the infrastructure they need (e.g., a PostgreSQL database, a Redis cache, a Kubernetes cluster) by simply creating a YAML file and pushing it to Git. They don't need to be experts in Azure or GCP.
-   **Platform Team as Enablers:** The platform team (that's us!) defines the "blueprints" for this infrastructure using Crossplane Compositions. We control the security, networking, and configuration details, ensuring that all provisioned resources adhere to company standards.
-   **Git as the Single Source of Truth:** Every single piece of infrastructure, from a single S3 bucket to an entire Kubernetes cluster, is defined declaratively in a Git repository. This provides a complete audit trail and enables powerful automation.

## What We Will Build

By the end of this tutorial, you will have a fully functional, multi-cloud IDP running on a local KinD cluster. You will be able to:

-   Define a new type of "product" (e.g., a `CompositePostgresInstance`) that bundles together all the necessary cloud resources.
-   Provision that product on Azure or GCP by creating a simple one-line YAML "claim".
-   Automatically deploy applications to newly provisioned Kubernetes clusters.
-   Monitor the health and status of your infrastructure using Prometheus and Grafana.

This is a powerful paradigm shift for infrastructure management. Let's get started by setting up our development environment.

**➡️ [Next: Devbox Setup](./02-devbox-setup.md)**
