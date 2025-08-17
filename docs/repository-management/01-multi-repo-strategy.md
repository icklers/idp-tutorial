# 01: Multi-Repo Strategy for GitOps

As a GitOps environment grows, managing all your code in a single repository (a "mono-repo") can become cumbersome. We advocate for a **multi-repo strategy**, which provides better separation of concerns, more granular access control, and clearer ownership.

## Our Recommended Repository Structure

We propose a three-tiered repository structure:

1.  **Platform Repository (This one):** This repository contains the core platform configuration. It defines the "shape" of your IDP.
    -   ArgoCD bootstrap configuration (`app-of-apps`).
    -   Crossplane Provider configurations.
    -   Crossplane Compositions (the blueprints for your infrastructure).
    -   RBAC policies and security configurations.
    -   **Audience:** Platform Engineering team.

2.  **Infrastructure-as-Code (IaC) Repositories:** Each of these repositories defines a specific piece of infrastructure using the Compositions from the platform repo.
    -   Contains Crossplane Claims (e.g., `CompositePostgresInstance`, `CompositeAKSCluster`).
    -   One repository per environment (e.g., `infra-dev`, `infra-staging`, `infra-prod`).
    -   **Audience:** DevOps / SRE / Platform team.

3.  **Application Repositories:** These are the standard repositories containing your microservice code.
    -   Contains application source code (Go, Python, Java, etc.).
    -   Contains Kubernetes manifests for deploying the application (Deployments, Services, Ingress, etc.).
    -   May contain a Crossplane Claim if the application requires its own dedicated infrastructure (e.g., a specific database).
    -   **Audience:** Development teams.

## Why Multi-Repo?

| Benefit                 | Description                                                                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Clear Ownership**     | It's immediately clear which team owns which part of the system. Developers own their apps, and the platform team owns the platform.         |
| **Granular RBAC**       | You can set different permissions for each repository. For example, only the Platform team can approve PRs to the platform repo.            |
| **Reduced Blast Radius**  | A mistake in an application repository is unlikely to take down your entire platform. Changes are isolated to their respective domains. |
| **Scalability**         | As your organization grows, you can easily add new application and infrastructure repositories without creating conflicts in a single repo. |

## The Role of the Platform Repo

This repository, `crossplane-gitops-tutorial`, serves as the **Platform Repository**. It is the heart of our system. The Crossplane Compositions we define here are published to the Kubernetes API server as new CRDs (e.g., `CompositePostgresInstance.example.org`).

Other repositories can then create instances of these CRDs (Claims) without needing to know the complex implementation details. They are consuming the API provided by the platform.

In the next section, we will look at how to standardize the creation of these repositories using templates.

**➡️ [Next: Repository Templates](./02-repository-templates.md)**
