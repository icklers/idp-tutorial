# 02: Monitoring with Prometheus

Health checks give us a binary status (is it working or not?), but for true observability, we need metrics. We need to track trends, understand performance, and be alerted to potential problems before they cause an outage.

We will use [Prometheus](https://prometheus.io/), the de facto standard for metrics and monitoring in the Kubernetes ecosystem.

## Exposing Metrics from Crossplane

Crossplane and its providers expose a wealth of metrics in the Prometheus format. These metrics give us deep insights into the reconciliation process.

Key metrics include:

-   `crossplane_managed_resource_reconcile_total`: A counter of how many times a managed resource has been reconciled.
-   `crossplane_managed_resource_reconcile_errors_total`: A counter of how many reconciliation errors have occurred.
-   `crossplane_managed_resource_reconcile_duration_seconds`: A histogram of how long reconciliations are taking.

These metrics are available on the `/metrics` endpoint of the Crossplane and Provider pods.

## Setting up the Prometheus Stack

We will use the `kube-prometheus-stack` Helm chart, which provides a complete, pre-configured monitoring solution:

-   **Prometheus:** Scrapes and stores the metrics.
-   **Grafana:** For visualizing the metrics in dashboards.
-   **Alertmanager:** For sending alerts based on metric thresholds.

### Installation

```bash
# Add the Prometheus community repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Install the stack
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

### ServiceMonitors

How does Prometheus know which pods to scrape? We use a CRD called `ServiceMonitor`.

We need to create `ServiceMonitor` resources that tell Prometheus to scrape the metrics endpoints of the Crossplane and ArgoCD pods.

**Example `ServiceMonitor` for Crossplane:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: crossplane-metrics
  namespace: crossplane-system
spec:
  selector:
    matchLabels:
      app: crossplane
  endpoints:
  - port: http-metrics
    path: /metrics
```

This resource tells the Prometheus Operator to find any `Service` in the `crossplane-system` namespace with the label `app: crossplane` and scrape its `http-metrics` port.

We will add these `ServiceMonitor` manifests to our `platform` repository, so our monitoring configuration is also managed via GitOps.

## Building a Grafana Dashboard

With the metrics flowing into Prometheus, we can now build a Grafana dashboard to visualize the health of our Crossplane control plane.

**Key Panels to Include:**

-   **Reconciliation Rate:** A graph of the `rate(crossplane_managed_resource_reconcile_total[5m])`. This shows you how much work Crossplane is doing.
-   **Reconciliation Error Rate:** A graph of the `rate(crossplane_managed_resource_reconcile_errors_total[5m])`. This should be zero. If it's not, something is wrong.
-   **Reconciliation Latency (95th percentile):** A graph of `histogram_quantile(0.95, sum(rate(crossplane_managed_resource_reconcile_duration_seconds_bucket[5m])) by (le))`. This shows you the latency of your reconciliations.
-   **Total Managed Resources:** A stat panel showing the output of `count(crossplane_managed_resources)`. This gives you an at-a-glance view of the size of your infrastructure.

By building these dashboards, you create a single pane of glass for observing the health and performance of your entire infrastructure provisioning system.

**➡️ [Next: Alerting](./03-alerting.md)**
