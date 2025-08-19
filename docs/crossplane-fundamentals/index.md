# Crossplane Fundamentals

Welcome to the Crossplane v2.0 learning journey! In this section, you'll transform from a Crossplane beginner to someone who can provision real Azure infrastructure using GitOps workflows.

## What Makes Crossplane v2.0 Different

Crossplane v2.0 introduces a **namespace-first architecture** that makes building control planes more intuitive and powerful:

### 🏗️ **Namespaced Everything**
- **Composite Resources (XRs)** live in user namespaces, not cluster-wide
- **Managed Resources** are namespaced for better isolation and RBAC
- **No more Claims** - developers work directly with XRs

### 🔧 **Composition Functions Only**
- **Native patch & transform removed** - functions provide more flexibility
- **Pipeline-based compositions** with reusable function libraries
- **Any Kubernetes resource** can be composed, not just managed resources

### 🎯 **Cleaner Developer Experience**
- **spec.crossplane** groups all Crossplane machinery for clarity
- **Direct resource creation** without intermediate claim objects
- **Better namespace isolation** for teams and projects

## What You'll Build

By the end of these modules, you'll have:

- ✅ **Secure Azure Provider** configured with Sealed Secrets  
- ✅ **Real Cloud Infrastructure** provisioned via Git commits  
- ✅ **Production-Ready Security** following best practices  
- ✅ **GitOps Workflows** for infrastructure management  

## Learning Philosophy: "Build First, Understand Later"

We start with working examples that demonstrate immediate value, then dive deeper into concepts. Every exercise follows GitOps principles from day one and produces production-ready artifacts.

## Prerequisites

Before starting this section, ensure you have completed:

- ✅ [Getting Started: DevBox Setup](../getting-started/02-devbox-setup.md)  
- ✅ [Getting Started: Local Cluster Setup](../getting-started/03-local-cluster-setup.md) 
- ✅ [Getting Started: GitOps Bootstrap](../getting-started/04-gitops-bootstrap.md)  
- ✅ [Getting Started: Sealed Secrets Setup](../getting-started/05-sealed-secrets-setup.md)  

## Azure Account Requirements

You'll need an active Azure subscription with:

- ✅ **Subscription access** to create resources  
- ✅ **Permission** to create service principals  
- ✅ **Azure CLI** configured and authenticated  

## Namespace Strategy for v2.0

In Crossplane v2.0, namespaces provide natural boundaries for:

### **Resource Organization**
```
tutorial/              # Learning resources
├── ResourceGroups
├── VirtualNetworks    
└── XDevEnvironments

production/           # Production workloads
├── Databases
├── AKSClusters
└── XProdEnvironments
```

### **Team Isolation** 
- **`frontend-team`** namespace for UI-focused resources
- **`backend-team`** namespace for API and data resources  
- **`platform-team`** namespace for shared infrastructure

### **RBAC Benefits**
- Fine-grained permissions per namespace
- Team-specific resource access
- Natural audit boundaries

**For this tutorial**: We'll use the **`tutorial`** namespace to keep all learning resources organized and isolated from other workloads.

## Function-Based Composition in v2

Crossplane v2 moves from native patch-and-transform to **function pipelines**. Understanding functions is crucial for v2 success:

### **Core Functions You'll Use**
- **`function-patch-and-transform`**: YAML-based transformations (most common)
- **`function-python`**: Python scripting for complex logic  
- **`function-kcl`**: KCL configuration language for validation
- **`function-go-templating`**: Go templates for dynamic generation

### **Function Installation Pattern**
```yaml
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-patch-and-transform
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-patch-and-transform:v0.2.1
```

### **Pipeline Execution**
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
spec:
  mode: Pipeline  # v2 default
  pipeline:
  - step: patch-resources
    functionRef:
      name: function-patch-and-transform
    input:
      apiVersion: pt.fn.crossplane.io/v1beta1
      kind: Resources
      # Function-specific configuration
```

### **When You'll Learn Functions**
- **Module 2**: Basic function installation and usage
- **Advanced Patterns**: Complex function pipelines and custom logic
- **Troubleshooting**: Function debugging techniques

## Module Overview

### Module 1: Azure Provider Setup (15 minutes)

**Objective:** Configure Crossplane to securely manage Azure resources

**What you'll learn:**  
- How to install Azure provider in Crossplane  
- Secure credential management with Sealed Secrets  
- Provider configuration and verification  
- GitOps deployment of provider components  

**Deliverables:**  
- Azure provider installed and configured  
- Service principal credentials securely stored  
- Provider health verification  

### Module 2: Your First Managed Resource (15 minutes)

**Objective:** Deploy a real Azure resource using Crossplane

**What you'll learn:**  
- What are Managed Resources (MRs)  
- How to create Azure Resource Groups  
- GitOps workflow for resource deployment  
- Resource status monitoring and troubleshooting  

**Deliverables:**  
- Azure Resource Group deployed via Git  
- Understanding of resource lifecycle  
- ArgoCD application managing infrastructure  

### Module 3: Understanding What Happened (15 minutes)

**Objective:** Explore the Crossplane resource model and reconciliation

**What you'll learn:**  
- Crossplane architecture and components  
- Resource states and conditions  
- Troubleshooting resource issues  
- Best practices for resource management  

**Deliverables:**  
- Deep understanding of Crossplane concepts  
- Troubleshooting skills and techniques  
- Foundation for advanced patterns  

## Operations in Crossplane v2 (Alpha Feature)

Crossplane v2 introduces **Operations** - an alpha feature for operational workflows that run to completion like Kubernetes Jobs:

### **Operation Types**
- **`Operation`**: Run function pipeline once  
- **`CronOperation`**: Scheduled function execution  
- **`WatchOperation`**: Event-driven function triggers  

### **Use Cases**
- **Certificate renewal**: Automated cert lifecycle management
- **Database backups**: Scheduled backup operations  
- **Health monitoring**: Periodic health checks and reporting
- **Configuration drift**: Automated configuration validation

### **Example: Daily Database Backup**
```yaml
apiVersion: ops.crossplane.io/v1alpha1
kind: CronOperation
metadata:
  name: backup-postgres
  namespace: production
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  mode: Pipeline
  pipeline:
  - step: backup-database
    functionRef:
      name: function-python
```

### **Enabling Operations**
Operations must be explicitly enabled:
```bash
# Add to Crossplane deployment
--enable-operations=true
```

**Note:** Operations are alpha in v2 and may change. We'll cover them in advanced patterns once you master the core concepts.

## Success Criteria

After completing this section, you should be able to:

- Explain the relationship between Providers, ProviderConfigs, and Managed Resources  
- Create Azure resources using Crossplane and GitOps workflows  
- Troubleshoot common resource provisioning issues  
- Follow security best practices for cloud credentials  
- Use ArgoCD to deploy and manage infrastructure components  

## Time Investment

**Total time:** ~45 minutes of hands-on exercises
**Format:** Self-paced with immediate feedback
**Approach:** Practical exercises with real Azure resources

## Next Steps

Ready to get started? Let's begin with setting up the Azure provider:

**➡️ [Module 1: Azure Provider Setup](01-azure-provider-setup.md)**
