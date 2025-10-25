# Grafana Cheatsheet

## Overview
Grafana is an open-source platform for observability, enabling dashboards, alerts, and analytics across multiple datasources.

## Installation

### Docker
```bash
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise
# Login at http://localhost:3000 (default admin/admin)
```

### Kubernetes (Helm)
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana --set adminPassword=admin
```

## Core Concepts
- Organization, Users, Teams, Folders
- Dashboards: collection of panels
- Panels: visualizations (time series, gauge, stat, table, logs)
- Datasources: Prometheus, Loki, Elasticsearch, Graphite, InfluxDB, CloudWatch, etc.
- Variables: dynamic dashboard parameters
- Alerting: Unified alerting (contact points, notification policies)

## Datasources

### Add Prometheus
1. Configuration > Data sources > Add data source
2. Select Prometheus
3. URL: http://prometheus:9090
4. Save & Test

### Add Loki (Logs)
- URL: http://loki:3100

### Add Elasticsearch
- URL: http://elasticsearch:9200
- Index pattern: logs-*
- Time field: @timestamp

## Panels & Visualizations

### Common Panels
- Time series: metrics over time
- Stat: single value KPIs
- Gauge/Bar gauge: thresholds visualization
- Table: tabular data
- Logs: Loki/Elasticsearch logs

### Query Examples

#### Prometheus
- CPU usage: `100 - (avg by (instance)(irate(node_cpu_seconds_total{mode="idle"}[5m]))*100)`
- Memory usage: `(1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100`
- HTTP 5xx rate: `sum(rate(http_requests_total{status=~"5.."}[5m]))`

#### Loki
- Error logs: `{app="api"} |= "error"`
- Count by level: `sum by (level) (count_over_time({app="api"}[5m]))`

#### Elasticsearch (Lucene/KQL)
- KQL: `kubernetes.namespace: prod and level: error`

## Dashboard Variables

### Types
- Query, Constant, Custom, Datasource, Interval

### Prometheus Variable Example
- Name: instance
- Query: `label_values(up, instance)`
- Multi-value: On
- Include All: On

Use in queries: `{instance=~"$instance"}`

## Thresholds & Transformations
- Thresholds: set per panel (e.g., 70 warning, 90 critical)
- Transformations: add fields, rename, merge, aggregate, calculate field

## Alerting (Unified Alerting)

### Create Alert Rule
1. Alerting > Alert rules > New alert rule
2. Query: Prometheus expression
3. Condition: e.g., WHEN avg() OF query(A,5m,now) IS ABOVE 80
4. Configure contact points and routes

### Contact Points
- Email, Slack, Teams, PagerDuty, Webhook, Opsgenie

### Simple Prometheus Alert Query
```
sum by (instance) (rate(http_requests_total[5m])) > 100
```

## Dashboard JSON Provisioning

### Provision Datasource
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

### Provision Dashboard
```yaml
apiVersion: 1
providers:
  - name: default
    orgId: 1
    folder: "General"
    type: file
    disableDeletion: false
    updateIntervalSeconds: 30
    options:
      path: /var/lib/grafana/dashboards
```

## Templating & Annotations
- Templating: use variables with `$var` in titles/queries
- Annotations: add event markers (manual or from datasource)

## Best Practices
- Use folders and consistent naming
- Parameterize dashboards with variables
- Use repeat panels/rows for multi-instance views
- Limit query time ranges and resolution for performance
- Reuse community dashboards (Grafana.com)

## Useful Links
- Grafana docs: https://grafana.com/docs/
- Community dashboards: https://grafana.com/grafana/dashboards/
- Alerting docs: https://grafana.com/docs/grafana/latest/alerting/
