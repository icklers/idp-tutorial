# 03: Alerting on Infrastructure State

Dashboards are great for passive observation, but we need to be proactively notified when things go wrong. This is the job of Prometheus Alertmanager.

Alertmanager allows us to define alerting rules based on Prometheus queries. When an alert fires, it can send notifications to a variety of receivers, such as Slack, PagerDuty, or email.

## Defining Alerting Rules

Alerting rules are defined in a `PrometheusRule` CRD.

**Example `PrometheusRule` to detect failing reconciliations:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: crossplane-alerts
  namespace: crossplane-system
spec:
  groups:
  - name: crossplane.rules
    rules:
    - alert: CrossplaneReconciliationErrors
      expr: rate(crossplane_managed_resource_reconcile_errors_total[5m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Crossplane is experiencing reconciliation errors"
        description: "The rate of reconciliation errors for managed resources is greater than 0. This indicates a problem with one or more providers or managed resources."
```

This rule will:

1.  **`expr`**: Continuously evaluate the rate of reconciliation errors over a 5-minute window.
2.  **`for`**: If the expression is true for 5 consecutive minutes, the alert will enter the `Firing` state.
3.  **`labels`**: Attach a `severity` label, which can be used for routing the alert (e.g., `critical` alerts go to PagerDuty, `warning` alerts go to Slack).
4.  **`annotations`**: Provide a human-readable summary and description of the alert.

## Important Alerts to Configure

Here are some essential alerts you should configure for your Crossplane environment:

-   **`CrossplaneReconciliationErrors` (Critical):** As defined above. This is your most important alert.
-   **`CrossplaneProviderNotReady` (Warning):** Alert if a Crossplane `Provider` resource is not in the `Ready` state for more than 10 minutes.
    -   `expr: crossplane_provider_ready == 0`
-   **`ArgoCDAppNotHealthy` (Warning):** Alert if an ArgoCD Application has been in a `Degraded` state for more than 15 minutes.
    -   `expr: argocd_app_info{health_status!="Healthy"} == 1`
-   **`ClaimNotReady` (Warning):** Alert if a Crossplane `Claim` has not reached the `Ready` state within 30 minutes of its creation. This requires tracking the creation timestamp.

## Configuring Receivers

Once your rules are in place, you need to configure Alertmanager to send the notifications.

This is done in the `prometheus-operator-alertmanager` secret.

**Example: Configuring a Slack receiver.**

```yaml
# alertmanager.yaml
global:
  resolve_timeout: 5m
route:
  group_by: ['job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'null'
  routes:
  - receiver: 'slack-notifications'
    match:
      severity: critical
receivers:
- name: 'null'
- name: 'slack-notifications'
  slack_configs:
  - api_url: <YOUR_SLACK_WEBHOOK_URL>
    channel: '#alerts-critical'
```

This configuration tells Alertmanager to send any alert with `severity: critical` to the `#alerts-critical` Slack channel.

Like all our other configuration, these `PrometheusRule` and Alertmanager config files should be stored in your `platform` repository and managed via GitOps. This ensures that your alerting and monitoring strategy evolves along with your infrastructure.

**➡️ [Next Section: Advanced Topics](../advanced/index.md)**
