# Observability

Repo paths:

```text
helm-charts/external-charts/prometheus/
helm-charts/external-charts/grafana/
helm-charts/external-charts/loki/
helm-charts/external-charts/alloy/
helm-charts/external-charts/prometheus-blackbox-exporter/
modules/aks-cluster/main.tf
```

## Purpose

Observability gives the AKS platform metrics, logs, dashboards, endpoint checks, and cluster diagnostics.

## Components

| Component | Purpose |
| --- | --- |
| Prometheus | Metrics collection. |
| Grafana | Dashboards and visualization. |
| Loki | Log storage and querying. |
| Alloy | Telemetry collection and forwarding. |
| Blackbox exporter | External endpoint health checks. |
| Log Analytics | Azure-side diagnostics such as cluster autoscaler logs. |

## Sample Prometheus Values

```yaml
server:
  retention: 15d
  persistentVolume:
    enabled: true
    size: 50Gi

alertmanager:
  enabled: true

nodeExporter:
  enabled: true
```

## Sample Grafana Values

```yaml
admin:
  existingSecret: grafana-admin

persistence:
  enabled: true
  size: 10Gi

datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus-server.monitoring.svc.cluster.local
      - name: Loki
        type: loki
        url: http://loki.monitoring.svc.cluster.local:3100
```

## Sample AKS Diagnostic Setting

```hcl
resource "azurerm_monitor_diagnostic_setting" "cluster_autoscaler" {
  name                       = "monitor-k8s-clusterautoscaler-logs"
  target_resource_id         = azurerm_kubernetes_cluster.aks.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.aks.id

  enabled_log {
    category = "cluster-autoscaler"
  }
}
```

## Notes

The AKS module includes Azure diagnostics, while Helm charts provide Kubernetes-level telemetry.
