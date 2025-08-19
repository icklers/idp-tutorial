# Crossplane Advanced Patterns - Module 01: Your First Composition

This directory contains all the YAML configuration files referenced in the tutorial module **"Your First Composition"**.

## Directory Structure

This follows the **GitOps Environment-Based Directory Structure** pattern established in the tutorial:

```
platform-core/                    # Platform components (shared, reusable)
├── xrds/
│   └── dev-environment-xrd.yaml  # XRD defining the XDevEnvironment API
└── compositions/
    └── dev-environment-composition.yaml  # Composition implementing the API

environments/                     # Environment-specific infrastructure instances
└── dev/
    └── infrastructure/
        └── alice-dev.yaml        # Example development environment instance

applications/                     # GitOps management layer
└── platform/
    ├── platform-apis.yaml       # ArgoCD App managing platform components
    └── environment-applicationset.yaml  # ArgoCD ApplicationSet managing environments

.github/workflows/                # GitHub Actions automation
└── crossplane-pr-testing.yml    # PR environment testing workflow
```

## Usage Instructions

### 1. Copy to Your Repository Root
```bash
# Copy the entire structure to your repository root
cp -r exercises/crossplane-advanced-patterns-01/* .
```

### 2. Update Repository URLs
```bash
# Replace placeholders with your actual repository URL
find . -name "*.yaml" -type f -exec sed -i 's|__YOUR_PLATFORM_GITOPS_REPO_URL__|https://github.com/your-username/your-repo.git|g' {} \;
find . -name "*.yaml" -type f -exec sed -i 's|https://github.com/your-username/your-tutorial-repo.git|https://github.com/your-username/your-repo.git|g' {} \;
```

### 3. Deploy Platform Components
```bash
# Deploy the platform (XRDs and Compositions)
kubectl apply -f applications/platform/platform-apis.yaml

# Deploy environment management
kubectl apply -f applications/platform/environment-applicationset.yaml
```

### 4. Create Development Environment
```bash
# The environment will be automatically deployed by ArgoCD
git add environments/dev/infrastructure/alice-dev.yaml
git commit -m "feat: create Alice's development environment"
git push
```

## File Descriptions

### Platform Components
- **`platform-core/xrds/dev-environment-xrd.yaml`**: Defines the XDevEnvironment API that developers use
- **`platform-core/compositions/dev-environment-composition.yaml`**: Implements the API using Azure Resource Group + Virtual Network

### GitOps Management
- **`applications/platform/platform-apis.yaml`**: ArgoCD Application that manages platform components
- **`applications/platform/environment-applicationset.yaml`**: ArgoCD ApplicationSet that automatically manages environments

### Environment Instances
- **`environments/dev/infrastructure/alice-dev.yaml`**: Example development environment using the XDevEnvironment API

### Automation
- **`.github/workflows/crossplane-pr-testing.yml`**: GitHub Actions workflow that creates PR environments for testing

## Integration with Tutorial

These files correspond to the YAML configurations shown in the tutorial:
- **Step 2**: `platform-core/xrds/dev-environment-xrd.yaml`
- **Step 2**: `platform-core/compositions/dev-environment-composition.yaml`  
- **Step 3**: `applications/platform/platform-apis.yaml`
- **Step 5**: `environments/dev/infrastructure/alice-dev.yaml`
- **Step 5**: `applications/platform/environment-applicationset.yaml`
- **Step 5**: `.github/workflows/crossplane-pr-testing.yml`

## Prerequisites

- Crossplane v1.14+ installed in your cluster
- ArgoCD installed and configured
- Azure Provider for Crossplane configured
- GitHub repository with Actions enabled

## Next Steps

After completing this module, proceed to:
- **Module 02**: Networking Composition (enhances these examples with networking)
- **Module 03**: Environment Patterns (adds staging and production environments)
