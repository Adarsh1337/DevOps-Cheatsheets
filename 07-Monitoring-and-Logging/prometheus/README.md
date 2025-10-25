# Prometheus Cheatsheet

## Overview
Prometheus is an open-source systems monitoring and alerting toolkit designed for reliability and scalability.

## Installation & Setup

### Binary Installation
```bash
# Download and extract
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64/

# Start Prometheus
./prometheus --config.file=prometheus.yml
```

### Docker Installation
```bash
docker run -d -p 9090:9090 prom/prometheus
```

### Kubernetes Installation (Helm)
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

## Configuration

### Basic prometheus.yml
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "first_rules.yml"
  - "second_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

## PromQL (Prometheus Query Language)

### Basic Selectors
```promql
# Select all time series with metric name
http_requests_total

# Filter by label
http_requests_total{job="api-server"}

# Multiple label filters
http_requests_total{job="api-server",method="GET"}

# Regular expression matching
http_requests_total{status=~"4.."}

# Negative matching
http_requests_total{status!="404"}
```

### Range Vectors
```promql
# Last 5 minutes
http_requests_total[5m]

# Last 1 hour
cpu_usage[1h]

# Time units: s, m, h, d, w, y
```

### Functions
```promql
# Rate (per-second average rate)
rate(http_requests_total[5m])

# Increase (total increase)
increase(http_requests_total[5m])

# Sum
sum(http_requests_total)

# Sum by label
sum by (job) (http_requests_total)

# Average
avg(cpu_usage)

# Maximum
max(memory_usage)

# Minimum
min(disk_free)

# Count
count(up == 1)
```

### Aggregation Operators
```promql
# Group by specific labels
sum by (instance, job) (http_requests_total)

# Group by all labels except specified
sum without (instance) (http_requests_total)

# Aggregation operators:
# sum, min, max, avg, count, count_values, 
# bottomk, topk, quantile
```

### Mathematical Operations
```promql
# Addition
metric1 + metric2

# Percentage calculation
(used_memory / total_memory) * 100

# Rate calculation
rate(http_requests_total[5m]) * 60
```

### Useful Queries
```promql
# CPU usage percentage
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)

# HTTP request rate
sum(rate(http_requests_total[5m])) by (instance)

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# 95th percentile response time
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

## Exporters

### Node Exporter (System Metrics)
```bash
# Download and run
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.0.linux-amd64.tar.gz
cd node_exporter-1.6.0.linux-amd64/
./node_exporter

# Docker
docker run -d -p 9100:9100 prom/node-exporter
```

### Common Exporters
- **Node Exporter**: System metrics (CPU, memory, disk, network)
- **Blackbox Exporter**: HTTP, TCP, DNS probing
- **MySQL Exporter**: MySQL database metrics
- **Redis Exporter**: Redis metrics
- **Nginx Exporter**: Nginx web server metrics
- **PostgreSQL Exporter**: PostgreSQL database metrics

## Alerting Rules

### Alert Rules File (alerts.yml)
```yaml
groups:
  - name: example
    rules:
    - alert: HighRequestLatency
      expr: http_request_duration_seconds > 0.5
      for: 10m
      labels:
        severity: page
      annotations:
        summary: High request latency
        description: "Request latency is above 0.5s for more than 10 minutes."
    
    - alert: InstanceDown
      expr: up == 0
      for: 5m
      labels:
        severity: page
      annotations:
        summary: "Instance {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
    
    - alert: HighMemoryUsage
      expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High memory usage
        description: "Memory usage is above 80% for more than 5 minutes."
    
    - alert: DiskSpaceLow
      expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Low disk space
        description: "Disk space is below 10% on {{ $labels.instance }}"
```

## Service Discovery

### File-based Service Discovery
```yaml
scrape_configs:
  - job_name: 'file_sd'
    file_sd_configs:
      - files:
          - '/path/to/targets.json'
        refresh_interval: 10s
```

### Kubernetes Service Discovery
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

## Recording Rules

### Recording Rules File
```yaml
groups:
  - name: cpu_rules
    interval: 10s
    rules:
    - record: instance:cpu_usage:rate5m
      expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    
    - record: job:http_requests:rate5m
      expr: sum by (job) (rate(http_requests_total[5m]))
```

## Federation

### Federated Setup
```yaml
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job=~"mysql.*"}'  # Federate MySQL metrics
        - '{__name__=~"job:.*"}' # Federate recording rules
    static_configs:
      - targets:
        - 'prometheus-1:9090'
        - 'prometheus-2:9090'
```

## API Usage

### Query API
```bash
# Instant query
curl 'http://localhost:9090/api/v1/query?query=up'

# Range query
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2023-01-01T00:00:00Z&end=2023-01-01T01:00:00Z&step=15s'

# Get all metric names
curl 'http://localhost:9090/api/v1/label/__name__/values'

# Get label values
curl 'http://localhost:9090/api/v1/label/job/values'
```

## Performance Tuning

### Configuration Optimization
```yaml
global:
  scrape_interval: 15s     # Balance between data freshness and load
  scrape_timeout: 10s      # Prevent hanging scrapes
  evaluation_interval: 15s # How often to evaluate rules

# Storage settings
storage:
  tsdb:
    retention.time: 15d    # How long to keep data
    retention.size: 10GB   # Maximum storage size
    min-block-duration: 2h # Minimum block duration
    max-block-duration: 25h # Maximum block duration
```

### Command Line Options
```bash
./prometheus \
  --config.file=prometheus.yml \
  --storage.tsdb.path=/data \
  --storage.tsdb.retention.time=15d \
  --storage.tsdb.retention.size=10GB \
  --web.console.libraries=/usr/share/prometheus/console_libraries \
  --web.console.templates=/usr/share/prometheus/consoles \
  --web.enable-lifecycle \
  --log.level=info
```

## Best Practices

### Naming Conventions
- Use snake_case for metric names
- Start with application name: `myapp_http_requests_total`
- Use base units (seconds, bytes, not milliseconds, KB)
- Add `_total` suffix for counters
- Add `_seconds` suffix for durations

### Label Best Practices
- Keep cardinality low (avoid high-cardinality labels like user IDs)
- Use consistent label names across metrics
- Avoid labels that change frequently
- Use descriptive but concise label names

### Query Optimization
- Use recording rules for expensive queries
- Avoid high-cardinality label operations
- Use appropriate time ranges
- Filter early in queries using label selectors

## Troubleshooting

### Common Issues
```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Check configuration
./promtool check config prometheus.yml

# Check rules
./promtool check rules alerts.yml

# Query troubleshooting
# Use /graph interface for testing queries
# Check for high cardinality metrics
# Monitor Prometheus self-metrics
```

### Monitoring Prometheus Itself
```promql
# Prometheus configuration reload
prometheus_config_last_reload_successful

# Number of samples ingested
prometheus_tsdb_samples_appended_total

# Query duration
prometheus_engine_query_duration_seconds

# Storage size
prometheus_tsdb_size_bytes
```

## Related Tools
- **Grafana**: Visualization and dashboards
- **Alertmanager**: Alert routing and notification
- **Pushgateway**: For short-lived jobs
- **Blackbox Exporter**: Probing endpoints
- **Thanos**: Long-term storage and global view
