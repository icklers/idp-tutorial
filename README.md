# IDP Tutorial: Building a Production-Ready Platform with Crossplane and ArgoCD

Welcome to the Internal Developer Platform (IDP) tutorial. This repository contains a complete, hands-on guide to building a production-ready, multi-cloud platform using Crossplane v2 for infrastructure-as-code and ArgoCD for GitOps delivery.

This project is designed for Senior DevOps Engineers who are already comfortable with Kubernetes and want to master the art of platform engineering through practical, hands-on exercises.

## 🎯 What You'll Learn

- **Crossplane v2 Fundamentals**: Master infrastructure provisioning with compositions and managed resources
- **GitOps Workflows**: Implement enterprise-grade deployment patterns with ArgoCD
- **Platform Engineering**: Build self-service infrastructure capabilities for development teams
- **Multi-Cloud Strategy**: Work with Azure and other cloud providers using consistent patterns
- **Security Best Practices**: Implement image verification, secret management, and RBAC
- **Observability**: Set up comprehensive monitoring and alerting for your platform

## 📚 Documentation & Learning Path

### **Complete Tutorial Documentation**
The detailed, step-by-step documentation is available in the `/docs` directory:

**➡️ [Start Here: Getting Started Guide](./docs/getting-started/01-introduction.md)**

### **Hands-On Exercise Files**
Ready-to-use YAML configurations and examples in the `/exercises` directory:

**➡️ [Exercise Files Overview](./exercises/README.md)**

### **Learning Path Structure**
1. **[Getting Started](./docs/getting-started/)** - Environment setup and GitOps bootstrap
2. **[GitOps Fundamentals](./docs/gitops-fundamentals/)** - ArgoCD integration and workflow patterns
3. **[Crossplane Fundamentals](./docs/crossplane-fundamentals/)** - Infrastructure provisioning basics
4. **[Advanced Patterns](./docs/crossplane-advanced-patterns/)** - Compositions and multi-environment management
5. **[Observability](./docs/observability/)** - Monitoring and alerting setup
6. **[Security](./docs/security/)** - Image verification, secrets, and RBAC

## Prerequisites

Before you begin, ensure you have the following:

- **GitHub Account**: For repository management and GitOps workflows
- **Cloud Access**: Azure account (optional for provider-specific exercises)
- **Development Environment**: [DevBox](https://www.jetpack.io/devbox/docs/installing-devbox/) installed (Linux or macOS)
- **Kubernetes Knowledge**: Familiarity with pods, services, deployments, and `kubectl`
- **Basic Git Experience**: Understanding of git workflows and command-line usage

**Time Commitment**: Plan for 4-6 hours to complete the full tutorial, or work through individual sections as needed.

For detailed setup instructions, see the **[Getting Started Guide](./docs/getting-started/01-introduction.md)**.

## VS Code Configuration (Important)

This repository contains configuration for the Gemini Code Assist extension in the `.vscode/mcp.json` file. This allows the assistant to have long-term memory about our project.

After you clone this repository, you **must** update the `mcp.json` file to point to the absolute path of the `.memory` directory on your local machine.

1.  Open `.vscode/mcp.json`.
2.  Find the line containing `/home/user/idp-tutorial/.memory:/data`.
3.  Replace `/home/user/idp-tutorial/.memory` with the absolute path to where you have cloned this repository.

**Example:** If you cloned the repository to `/home/user/projects/idp-tutorial`, the new line should be `"-v", "/home/user/projects/idp-tutorial/.memory:/data",`.

## Quick Start: Bootstrapping the Environment

These steps will get your local learning environment up and running quickly. For detailed explanations, follow the complete tutorial documentation.

### 1. Clone and Set Up Your Platform Repository

This tutorial provides the foundation for your Platform GitOps Repository:

```bash
# Clone this tutorial repository
gh repo clone icklers/idp-tutorial
cd idp-tutorial

# Create your platform GitOps repository (replace <your-repo-name>)
gh repo create <your-repo-name> --public --source . --remote origin --push
```

Your new repository will be your GitOps source of truth for all platform configurations.

### 2. Set Up Development Environment

Activate the DevBox environment and install required tools:

```bash
# Activate development environment with all required tools
devbox shell

# Install Crossplane CLI and MkDocs dependencies
devbox run setup-tools
```

### 3. Create Local Kubernetes Cluster

Create a local development cluster using Kind:

```bash
# Create KinD cluster for platform development
kind create cluster --config gitops-bootstrap/kind-cluster/kind-cluster.yaml
```

### 4. Bootstrap ArgoCD GitOps Engine

Install ArgoCD and configure it to manage your platform:

```bash
# Create ArgoCD namespace and install
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Configure your repository URL and start GitOps sync
export PLATFORM_GITOPS_REPO_URL="https://github.com/your-username/your-repo-name.git"
sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" gitops-bootstrap/argocd/platform-core.yaml
kubectl apply -f gitops-bootstrap/argocd/platform-core.yaml
```

### 5. Access Your Platform

View your platform deployment in the ArgoCD UI:

```bash
# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Login at `https://localhost:8080` with username `admin` and the retrieved password.

## 🚀 What You've Built

After completing the bootstrap, you'll have:

- **Local Kubernetes Cluster**: Development environment with Kind
- **GitOps Engine**: ArgoCD managing your platform from Git
- **Platform Foundation**: Ready for Crossplane and infrastructure components
- **Exercise Files**: Complete set of YAML configurations for hands-on learning
- **Documentation**: Step-by-step guides for building production-ready platforms

## 📁 Repository Structure

```
idp-tutorial/
├── docs/                    # Complete tutorial documentation
│   ├── getting-started/     # Environment setup and bootstrap
│   ├── gitops-fundamentals/ # ArgoCD and workflow patterns
│   ├── crossplane-fundamentals/ # Infrastructure provisioning
│   ├── observability/       # Monitoring and alerting
│   └── security/           # Image verification, secrets, RBAC
├── exercises/              # Hands-on exercise files
│   ├── getting-started/    # Bootstrap configurations
│   ├── gitops-fundamentals/ # GitOps workflow examples
│   ├── crossplane-fundamentals/ # Provider and resource examples
│   ├── observability/     # Monitoring configurations  
│   └── security/          # Security policy examples
├── gitops-bootstrap/       # Initial GitOps setup
└── platform-core/         # Core platform configurations
```

## 🎓 Learning Approach

This tutorial follows a **progressive learning model**:

1. **Bootstrap First**: Get a working environment quickly
2. **Understand Core Concepts**: Learn GitOps and Crossplane fundamentals  
3. **Hands-On Practice**: Use exercise files for real implementations
4. **Advanced Patterns**: Build production-ready compositions and workflows
5. **Enterprise Features**: Add monitoring, security, and governance

Each section includes:
- **Conceptual explanations** for understanding the "why"
- **Step-by-step instructions** for the "how"
- **Exercise files** for hands-on practice
- **Troubleshooting guides** for common issues
- **Best practices** for production deployments

## Next Steps

Your platform foundation is now ready! Continue with the full tutorial to build your Internal Developer Platform:

**➡️ [Begin Tutorial: Getting Started](./docs/getting-started/01-introduction.md)**

Or jump to specific topics:
- **[GitOps Workflows](./docs/gitops-fundamentals/)** - Learn ArgoCD integration patterns
- **[Infrastructure Provisioning](./docs/crossplane-fundamentals/)** - Master Crossplane basics
- **[Exercise Files](./exercises/)** - Browse all hands-on configurations

## 🤝 Contributing

Found an issue or want to improve the tutorial? Contributions are welcome! Please see our contribution guidelines and open an issue or pull request.

## 📄 License

This tutorial is provided under the MIT License. See LICENSE file for details.
