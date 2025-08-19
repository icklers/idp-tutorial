# 03: GitOps Workflow Patterns

With our tooling in place, we can now implement powerful GitOps workflows. This section focuses on **practical patterns** that work with modern development practices like GitHub Flow and single infrastructure repositories.

## The Modern Challenge: Single Repo + Trunk-Based Development

Most teams today use:
- **GitHub Flow** (trunk-based development) with short-lived feature branches
- **Single infrastructure repository** for all environments 
- **Fast iteration cycles** with continuous deployment

The challenge: How do you maintain **environment promotion safety** while keeping the **developer experience smooth** and the **operational overhead minimal**?

## Pattern 1: Environment-Based Directory Structure (Recommended)

**Best for:** Teams using GitHub Flow with a single infrastructure repository

### Repository Structure
```
infrastructure-gitops/
├── platform-core/              # Platform components (XRDs, Compositions)  
│   ├── xrds/
│   └── compositions/
├── environments/
│   ├── dev/                     # Development environment
│   │   ├── applications/
│   │   └── infrastructure/
│   ├── staging/                 # Staging environment  
│   │   ├── applications/
│   │   └── infrastructure/
│   └── production/              # Production environment
│       ├── applications/
│       └── infrastructure/
└── shared/                      # Shared resources across environments
    ├── secrets/
    └── policies/
```

### Workflow: Safe Trunk-Based Infrastructure Changes

#### Step 1: Feature Development
```bash
# Developer creates feature branch for infrastructure change
git checkout -b feature/add-redis-cache
git push -u origin feature/add-redis-cache

# Add new infrastructure to dev first
cat > environments/dev/infrastructure/redis-cache.yaml <<EOF
apiVersion: platform.company.com/v1alpha1
kind: XRedisCache
metadata:
  name: user-session-cache
  namespace: dev
spec:
  parameters:
    size: small
    version: "7.0"
    backup: false
EOF

git add environments/dev/infrastructure/redis-cache.yaml
git commit -m "feat: add Redis cache for user sessions (dev)"
git push
```

#### Step 2: Automated PR Environment
**GitHub Actions** automatically:
1. **Creates PR environment** in `dev` namespace with unique suffix
2. **Runs integration tests** against the PR environment
3. **Comments on PR** with test results and environment URL

```yaml
# .github/workflows/pr-infrastructure.yml
name: Infrastructure PR Testing
on:
  pull_request:
    paths: ['environments/dev/**']

jobs:
  test-infrastructure:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Create PR Environment
      run: |
        # Create PR-specific resources with suffix
        export PR_SUFFIX="pr-${{ github.event.number }}"
        
        # Process templates and create PR environment
        envsubst < environments/dev/infrastructure/redis-cache.yaml \
          | sed "s/name: user-session-cache/name: user-session-cache-${PR_SUFFIX}/" \
          | kubectl apply -f -
        
        # Wait for resources to be ready
        kubectl wait --for=condition=Ready xrediscache/user-session-cache-${PR_SUFFIX} -n dev --timeout=300s
    
    - name: Run Integration Tests
      run: |
        # Run tests against PR environment
        export REDIS_URL="user-session-cache-pr-${{ github.event.number }}.dev.svc.cluster.local"
        npm test -- --environment=pr-${{ github.event.number }}
    
    - name: Cleanup on Failure
      if: failure()
      run: |
        export PR_SUFFIX="pr-${{ github.event.number }}"
        kubectl delete xrediscache/user-session-cache-${PR_SUFFIX} -n dev
```

#### Step 3: Merge to Main (Automatic Dev Deployment)
```bash
# After PR approval and merge to main
# ArgoCD automatically syncs changes to dev environment

# ArgoCD Application for dev
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/infrastructure-gitops.git
    targetRevision: HEAD
    path: environments/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> 📁 **Exercise Files**: Available at [`exercises/gitops-fundamentals/workflow-patterns/argocd-app-infrastructure-dev.yaml`](../../exercises/gitops-fundamentals/workflow-patterns/argocd-app-infrastructure-dev.yaml)
>
> ```bash
> # Copy into your GitOps repo/Application folder
> user@host idp-tutorial> $ cp exercises/gitops-fundamentals/workflow-patterns/argocd-app-infrastructure-dev.yaml gitops-bootstrap/argocd/
> 
> # Commit and push so ArgoCD reconciles
> user@host idp-tutorial> $ git add gitops-bootstrap/argocd/argocd-app-infrastructure-dev.yaml
> user@host idp-tutorial> $ git commit -m "gitops: add dev infra ArgoCD Application"
> user@host idp-tutorial> $ git push
> ```

#### Step 4: Environment Promotion via PR
**Controlled promotion** using internal PRs:

```bash
# Platform team promotes to staging
git checkout main
git pull origin main

# Copy validated changes from dev to staging
cp environments/dev/infrastructure/redis-cache.yaml \
   environments/staging/infrastructure/redis-cache.yaml

# Update for staging-specific configuration
sed -i 's/size: small/size: medium/' environments/staging/infrastructure/redis-cache.yaml
sed -i 's/backup: false/backup: true/' environments/staging/infrastructure/redis-cache.yaml

git add environments/staging/infrastructure/redis-cache.yaml
git commit -m "feat: promote Redis cache to staging

- Increase size to medium for staging load
- Enable backup for data protection
- Validated in dev environment"

# Create internal PR for staging promotion
gh pr create --title "Promote: Redis cache to staging" \
             --body "Promoting validated Redis cache from dev to staging with staging-specific configuration"
```

#### Step 5: Production Promotion (Strict Control)
**Production requires explicit approval workflow:**

```yaml
# .github/workflows/production-promotion.yml
name: Production Promotion
on:
  push:
    paths: ['environments/production/**']
    branches: [main]

jobs:
  production-gate:
    runs-on: ubuntu-latest
    environment: production  # Requires GitHub Environment protection rules
    steps:
    - uses: actions/checkout@v4
    
    - name: Validate Production Changes
      run: |
        # Validate that changes exist in staging first
        if ! diff -r environments/staging/infrastructure environments/production/infrastructure; then
          echo "✅ Production changes differ from staging - manual review required"
        fi
        
        # Run production-specific validations
        kubeval environments/production/infrastructure/*.yaml
        conftest verify environments/production/infrastructure/*.yaml
    
    - name: Manual Approval Gate
      uses: hmarr/auto-approve-action@v3
      # This step requires manual approval via GitHub UI
```

### Safety Mechanisms

#### 1. **Staged Rollout with Canary**
```yaml
# environments/production/infrastructure/redis-cache.yaml
apiVersion: platform.company.com/v1alpha1
kind: XRedisCache
metadata:
  name: user-session-cache
  namespace: production
spec:
  parameters:
    size: large
    version: "7.0"
    backup: true
    highAvailability: true
    rolloutStrategy:
      type: canary
      canarySteps:
      - weight: 10   # 10% of traffic
        duration: "5m"
      - weight: 50   # 50% of traffic  
        duration: "10m"
      - weight: 100  # Full rollout
```

#### 2. **Automatic Rollback on Failure**
```yaml
# ArgoCD Application with health checks
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure-production
spec:
  # ... other config
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 3
      backoff:
        duration: 5s
        maxDuration: 3m0s
    syncOptions:
    - CreateNamespace=true
    - RespectIgnoreDifferences=true
  
  # Health monitoring
  health:
    - group: platform.company.com
      kind: XRedisCache
      check: |
        health_status = {}
        if obj.status and obj.status.conditions then
          for _, condition in ipairs(obj.status.conditions) do
            if condition.type == "Ready" then
              if condition.status == "True" then
                health_status.status = "Healthy"
                health_status.message = "Redis cache is ready"
              else
                health_status.status = "Degraded" 
                health_status.message = condition.message
              end
            end
          end
        else
          health_status.status = "Progressing"
          health_status.message = "Waiting for Redis cache status"
        end
        return health_status
```

### Monitoring and Observability

#### 3. **Environment Drift Detection**
```yaml
# .github/workflows/drift-detection.yml
name: Environment Drift Detection
on:
  schedule:
    - cron: '0 8 * * *'  # Daily at 8 AM

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Check Dev Environment Drift
      run: |
        # Compare desired state vs actual state
        argocd app diff infrastructure-dev --refresh
        
        # Check for unexpected manual changes
        kubectl get all -n dev -o yaml | \
          grep -E "(kubectl.kubernetes.io/last-applied-configuration|argocd.argoproj.io/)" | \
          grep -v "argocd.argoproj.io/managed-by" && \
          echo "⚠️ Manual changes detected in dev environment"
    
    - name: Slack Alert on Drift
      if: failure()
      uses: 8398a7/action-slack@v3
      with:
        status: failure
        text: "🚨 Environment drift detected! Manual intervention may be needed."
```

## Benefits of This Pattern

### ✅ **Developer Experience**
- **Single repository** - no context switching
- **Fast feedback** - PR environments for testing
- **Familiar workflow** - standard GitHub Flow
- **Automated testing** - catch issues early

### ✅ **Operational Safety**  
- **Environment parity** - same code across environments
- **Controlled promotion** - explicit approval for production
- **Audit trail** - full Git history of changes
- **Automatic rollback** - failed deployments revert automatically

### ✅ **Platform Engineering**
- **Reusable components** - shared XRDs and Compositions
- **Environment isolation** - namespace-based separation
- **Policy enforcement** - consistent across environments
- **Monitoring integration** - drift detection and alerting

This pattern gives you **enterprise-grade safety** with **startup-level agility** - perfect for teams that need to move fast while maintaining reliability.

This pattern gives you **enterprise-grade safety** with **startup-level agility** - perfect for teams that need to move fast while maintaining reliability.

## Pattern 2: Application-Coupled Infrastructure

**Best for:** Microservices that own their dedicated infrastructure components

When applications have dedicated infrastructure that isn't shared (databases, caches, queues), you can manage them together while maintaining the single-repo structure.

### Repository Structure
```
infrastructure-gitops/
├── platform-core/              # Shared platform components
├── applications/
│   ├── user-service/           # Application-specific infrastructure  
│   │   ├── infrastructure/
│   │   │   ├── postgres.yaml   # Dedicated database
│   │   │   └── redis.yaml      # Dedicated cache
│   │   └── config/
│   │       └── secrets.yaml
│   └── payment-service/
│       ├── infrastructure/
│       │   └── postgres.yaml   # Isolated database
│       └── config/
└── environments/               # Environment-specific overrides
    ├── dev/applications/
    ├── staging/applications/
    └── production/applications/
```

### ArgoCD ApplicationSet Pattern
```yaml
# ArgoCD ApplicationSet for application infrastructure
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: application-infrastructure
  namespace: argocd
spec:
  generators:
  - matrix:
      generators:
      - git:
          repoURL: https://github.com/your-org/infrastructure-gitops.git
          revision: HEAD
          directories:
          - path: applications/*
      - list:
          elements:
          - env: dev
            cluster: https://kubernetes.default.svc
          - env: staging  
            cluster: https://staging-cluster-url
          - env: production
            cluster: https://production-cluster-url
  template:
    metadata:
      name: '{{path.basename}}-{{env}}'
      labels:
        app: '{{path.basename}}'
        env: '{{env}}'
    spec:
      project: default
      sources:
      # Base application infrastructure
      - repoURL: https://github.com/your-org/infrastructure-gitops.git
        targetRevision: HEAD
        path: '{{path}}'
      # Environment-specific overrides
      - repoURL: https://github.com/your-org/infrastructure-gitops.git
        targetRevision: HEAD
        path: 'environments/{{env}}/{{path}}'
      destination:
        server: '{{cluster}}'
        namespace: '{{path.basename}}-{{env}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
```

> 📁 **Exercise Files**: Available at [`exercises/gitops-fundamentals/workflow-patterns/applicationset-application-infrastructure.yaml`](../../exercises/gitops-fundamentals/workflow-patterns/applicationset-application-infrastructure.yaml)
>
> ```bash
> # Copy into your GitOps repo/ApplicationSets folder
> user@host idp-tutorial> $ cp exercises/gitops-fundamentals/workflow-patterns/applicationset-application-infrastructure.yaml gitops-bootstrap/argocd/
>
> # Commit and push so ArgoCD reconciles
> user@host idp-tutorial> $ git add gitops-bootstrap/argocd/applicationset-application-infrastructure.yaml
> user@host idp-tutorial> $ git commit -m "gitops: add application infrastructure ApplicationSet"
> user@host idp-tutorial> $ git push
> ```

### Application Infrastructure Example
```yaml
# applications/user-service/infrastructure/postgres.yaml
apiVersion: platform.company.com/v1alpha1  
kind: XPostgreSQL
metadata:
  name: user-database
spec:
  parameters:
    name: user-database
    databaseName: users
    # Environment-specific values will be overridden
    storage: "10Gi"    # Default for dev
    replicas: 1        # Default for dev
    backup: false      # Default for dev
  crossplane:
    compositionRef:
      name: postgresql-composition
```

```yaml
# environments/production/applications/user-service/infrastructure/postgres.yaml  
apiVersion: platform.company.com/v1alpha1
kind: XPostgreSQL
metadata:
  name: user-database
spec:
  parameters:
    name: user-database
    databaseName: users
    storage: "100Gi"       # Production storage
    replicas: 3            # High availability
    backup: true           # Production backups
    backupRetention: 30    # 30-day retention
    monitoring: enabled    # Production monitoring
  crossplane:
    compositionRef:
      name: postgresql-production-composition  # Production-specific composition
```

## Pattern 3: Emergency Workflows & Troubleshooting

**Real-world scenarios** require robust emergency procedures and troubleshooting workflows.

### Emergency Hotfix Workflow
```bash
# Critical production issue - bypass normal workflow
git checkout main
git pull origin main

# Create hotfix branch
git checkout -b hotfix/critical-redis-memory-fix

# Apply emergency fix directly to production
cat > environments/production/infrastructure/redis-cache-hotfix.yaml <<EOF
apiVersion: platform.company.com/v1alpha1
kind: XRedisCache
metadata:
  name: user-session-cache
spec:
  parameters:
    # Emergency memory increase
    memory: "8Gi"  # Was 4Gi
    maxMemoryPolicy: "allkeys-lru"  # Prevent OOM
EOF

# Commit and push - triggers immediate deployment
git add environments/production/infrastructure/redis-cache-hotfix.yaml
git commit -m "HOTFIX: Increase Redis memory to prevent OOM

- Critical production issue: Redis hitting memory limits
- Immediate deployment needed
- Will backport to staging/dev post-incident"

git push -u origin hotfix/critical-redis-memory-fix

# Create emergency PR with override labels
gh pr create --title "🚨 HOTFIX: Critical Redis memory fix" \
             --body "Emergency production fix - Redis OOM prevention" \
             --label "emergency,auto-merge"
```

### GitOps Troubleshooting Checklist

#### Issue: Application Not Syncing
```bash
# 1. Check ArgoCD Application status
kubectl get application infrastructure-production -n argocd -o yaml

# 2. Check ArgoCD logs  
kubectl logs -n argocd deployment/argocd-application-controller

# 3. Manual sync with detailed output
argocd app sync infrastructure-production --dry-run --output yaml

# 4. Check for resource conflicts
kubectl get events -n production --sort-by='.lastTimestamp'

# 5. Validate manifests locally
kubectl diff -f environments/production/infrastructure/
```

#### Issue: Crossplane Resource Stuck
```bash
# 1. Check Composite Resource status
kubectl describe xrediscache user-session-cache -n production

# 2. Check underlying Managed Resources
kubectl get managed -l crossplane.io/composite=user-session-cache -n production

# 3. Check Crossplane logs
kubectl logs -n crossplane-system deployment/crossplane

# 4. Force reconciliation
kubectl annotate xrediscache user-session-cache -n production \
  crossplane.io/reconcile=$(date +%Y%m%d%H%M%S)

# 5. Emergency deletion (if needed)
kubectl patch xrediscache user-session-cache -n production \
  --type='merge' -p='{"metadata":{"finalizers":[]}}'
```

#### Issue: Environment Drift
```bash
# 1. Compare ArgoCD desired state vs cluster state  
argocd app diff infrastructure-production

# 2. Find manual changes
kubectl get all -n production -o yaml | \
  grep -v "argocd.argoproj.io/managed-by" | \
  grep "kubectl.kubernetes.io/last-applied-configuration"

# 3. Restore GitOps control
argocd app sync infrastructure-production --prune --force

# 4. Prevent future drift
kubectl create rolebinding block-manual-changes \
  --clusterrole=view --user=developers -n production
```

### Monitoring & Alerting Integration

#### ArgoCD Health Monitoring
```yaml
# Prometheus AlertManager rules for GitOps health
groups:
- name: gitops-health
  rules:
  - alert: ArgocdAppOutOfSync
    expr: |
      argocd_app_info{sync_status!="Synced"} == 1
    for: 5m
    labels:
      severity: warning
      team: platform
    annotations:
      summary: "ArgoCD Application {{ $labels.name }} is out of sync"
      description: "Application {{ $labels.name }} in namespace {{ $labels.namespace }} has been out of sync for more than 5 minutes"
      runbook: "https://runbooks.company.com/gitops/out-of-sync"
  
  - alert: CrossplaneResourceFailed
    expr: |
      crossplane_resource_ready{condition="False"} == 1
    for: 10m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "Crossplane resource {{ $labels.name }} failed"
      description: "Crossplane resource {{ $labels.name }} of type {{ $labels.kind }} has been in failed state for more than 10 minutes"
      runbook: "https://runbooks.company.com/crossplane/resource-failed"
```

## Best Practices Summary

### ✅ **DO**
- **Use environment directories** in single repo for clear separation
- **Automate PR environments** for infrastructure testing  
- **Require reviews** for production changes
- **Monitor for drift** and alert on inconsistencies
- **Practice emergency procedures** regularly
- **Document troubleshooting steps** and runbooks

### ❌ **DON'T**
- **Make manual changes** directly to clusters
- **Skip testing** infrastructure changes
- **Bypass approval processes** except for true emergencies
- **Ignore ArgoCD health checks** and sync failures
- **Delete resources** without understanding dependencies
- **Mix application code** with infrastructure in same repo (unless tightly coupled)

### 🔧 **Tools Integration**
- **GitHub Actions** for CI/CD pipelines and testing
- **ArgoCD** for GitOps deployment and health monitoring
- **Crossplane** for infrastructure provisioning
- **Prometheus/AlertManager** for monitoring and alerting
- **Slack/Teams** for incident response and notifications

This workflow pattern provides the **reliability of enterprise GitOps** with the **agility of trunk-based development** - giving you the best of both worlds for modern infrastructure management.

**➡️ [Next Section: Security](../security/01-chainguard-images.md)**
