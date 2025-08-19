# Observability Exercises

This directory contains exercise files for the Observability tutorial section.

## Structure

### `health-checks/`
- **Purpose**: Health monitoring configurations for Crossplane and ArgoCD
- **Files**: (Coming from tutorial content - health check examples)

### `monitoring/`
- **Purpose**: Prometheus and Grafana configurations for metrics collection
- **Files**:
  - `prometheus-stack.yaml` - ArgoCD Application for kube-prometheus-stack
  - `crossplane-servicemonitor.yaml` - ServiceMonitor for Crossplane metrics
  - `argocd-servicemonitor.yaml` - ServiceMonitor for ArgoCD metrics

### `alerting/`
- **Purpose**: Prometheus alerting rules and configurations
- **Files**:
  - `crossplane-alerts.yaml` - Prometheus alerting rules for Crossplane
  - `argocd-alerts.yaml` - Prometheus alerting rules for ArgoCD

## Usage Instructions

### Prerequisites
- ArgoCD installed and configured
- Crossplane installed with providers
- Sufficient cluster resources for monitoring stack

### Prometheus Stack Installation

Deploy the complete monitoring stack:

```bash
# Copy Prometheus stack configuration
user@host idp-tutorial> $ cp exercises/observability/monitoring/prometheus-stack.yaml platform-core/applications/

# OPTION A — Bootstrap (initial seeding) with a lean one-liner
user@host idp-tutorial> $ git add platform-core/applications/prometheus-stack.yaml && git commit -m "chore(gitops): seed prometheus stack (observability/monitoring)" && git push -u origin HEAD

# OPTION B — Ongoing changes (PR required)
user@host idp-tutorial> $ git switch -c chore/add-prometheus
user@host idp-tutorial> $ git add platform-core/applications/prometheus-stack.yaml
user@host idp-tutorial> $ git commit -m "chore(gitops): add prometheus stack (observability/monitoring)"
user@host idp-tutorial> $ git push -u origin $(git branch --show-current)
user@host idp-tutorial> $ gh pr create --title "Add Prometheus monitoring stack" --body "Deploys kube-prometheus-stack for metrics collection"
user@host idp-tutorial> $ gh pr merge --auto --squash --delete-branch

# Deploy the application
user@host idp-tutorial> $ kubectl apply -f platform-core/applications/prometheus-stack.yaml
```

### ServiceMonitors for Metrics Collection

Configure metrics collection from Crossplane and ArgoCD:

```bash
# Create monitoring directory structure
user@host idp-tutorial> $ mkdir -p platform-core/monitoring

# Copy ServiceMonitor configurations
user@host idp-tutorial> $ cp exercises/observability/monitoring/crossplane-servicemonitor.yaml platform-core/monitoring/
user@host idp-tutorial> $ cp exercises/observability/monitoring/argocd-servicemonitor.yaml platform-core/monitoring/

# Create ArgoCD Application for monitoring configs
cat > platform-core/applications/monitoring-configs.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-configs
  namespace: argocd
spec:
  project: default
  source:
    repoURL: __YOUR_PLATFORM_GITOPS_REPO_URL__
    targetRevision: HEAD
    path: platform-core/monitoring
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/monitoring-configs.yaml

# Commit and deploy
user@host idp-tutorial> $ git add platform-core/monitoring/ platform-core/applications/monitoring-configs.yaml
user@host idp-tutorial> $ git commit -m "feat: add monitoring configs for Crossplane and ArgoCD"
user@host idp-tutorial> $ git push

user@host idp-tutorial> $ kubectl apply -f platform-core/applications/monitoring-configs.yaml
```

### Alerting Rules

Configure Prometheus alerting for Crossplane and ArgoCD:

```bash
# Create alerting directory structure
user@host idp-tutorial> $ mkdir -p platform-core/alerting

# Copy alerting rule configurations
user@host idp-tutorial> $ cp exercises/observability/alerting/crossplane-alerts.yaml platform-core/alerting/
user@host idp-tutorial> $ cp exercises/observability/alerting/argocd-alerts.yaml platform-core/alerting/

# Create ArgoCD Application for alerting rules
cat > platform-core/applications/alerting-rules.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: alerting-rules
  namespace: argocd
spec:
  project: default
  source:
    repoURL: __YOUR_PLATFORM_GITOPS_REPO_URL__
    targetRevision: HEAD
    path: platform-core/alerting
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Update repository URL
user@host idp-tutorial> $ sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" platform-core/applications/alerting-rules.yaml

# Commit and deploy
user@host idp-tutorial> $ git add platform-core/alerting/ platform-core/applications/alerting-rules.yaml
user@host idp-tutorial> $ git commit -m "feat: add alerting rules for Crossplane and ArgoCD"
user@host idp-tutorial> $ git push

user@host idp-tutorial> $ kubectl apply -f platform-core/applications/alerting-rules.yaml
```

### Accessing Monitoring Tools

After deployment, access the monitoring tools:

```bash
# Access Grafana (default admin/admin123)
kubectl port-forward -n monitoring svc/prometheus-stack-grafana 3000:80

# Access Prometheus
kubectl port-forward -n monitoring svc/prometheus-stack-kube-prom-prometheus 9090:9090

# Access AlertManager
kubectl port-forward -n monitoring svc/prometheus-stack-kube-prom-alertmanager 9093:9093
```

### Key Metrics to Monitor

**Crossplane Metrics:**
- `crossplane_managed_resource_reconcile_total` - Reconciliation count
- `crossplane_managed_resource_reconcile_errors_total` - Error count
- `crossplane_managed_resource_reconcile_duration_seconds` - Reconciliation latency
- `crossplane_provider_ready` - Provider health status

**ArgoCD Metrics:**
- `argocd_app_info` - Application status and health
- `argocd_app_sync_total` - Sync operations count
- `argocd_git_request_total` - Git operations
- `argocd_redis_request_total` - Redis operations

## Learning Objectives

After completing these exercises, you will understand:

1. **Metrics Collection**: How to configure Prometheus to scrape Crossplane and ArgoCD metrics
2. **Health Monitoring**: Understanding resource status and health conditions
3. **Alerting**: Setting up proactive alerts for infrastructure issues
4. **Observability**: Building comprehensive monitoring for GitOps infrastructure
5. **Troubleshooting**: Using metrics and alerts for problem diagnosis

## Troubleshooting

### Common Issues

1. **ServiceMonitor not discovering targets**
   - Check label selectors match service labels
   - Verify Prometheus has permissions to scrape namespaces
   - Check service endpoints: `kubectl get endpoints -n <namespace>`

2. **High resource usage from monitoring stack**
   - Adjust retention periods in Prometheus config
   - Reduce scrape intervals in ServiceMonitors
   - Limit metrics with relabeling rules

3. **Alerts not firing**
   - Verify PrometheusRule is loaded: `kubectl get prometheusrule`
   - Check alert evaluation in Prometheus UI
   - Verify AlertManager configuration

## Tutorial Navigation
- [01: Health Checks](../../docs/observability/01-health-checks.md)
- [02: Monitoring with Prometheus](../../docs/observability/02-monitoring-with-prometheus.md)
- [03: Alerting](../../docs/observability/03-alerting.md)
