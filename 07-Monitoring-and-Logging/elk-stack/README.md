# ELK Stack Cheatsheet (Elasticsearch, Logstash, Kibana)

## Overview
ELK is a popular stack for log aggregation, search, analysis, and visualization.

- Elasticsearch: distributed search and analytics engine
- Logstash: data processing pipeline (ingest, transform, ship)
- Kibana: visualization and exploration UI

## Install & Run (Docker Compose)
```yaml
version: "3.8"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: ["9200:9200", "9300:9300"]
  kibana:
    image: docker.elastic.co/kibana/kibana:8.14.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports: ["5601:5601"]
    depends_on: [elasticsearch]
  logstash:
    image: docker.elastic.co/logstash/logstash:8.14.0
    ports: ["5044:5044", "9600:9600"]
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on: [elasticsearch]
```

## Logstash Pipeline Example
```conf
input {
  beats { port => 5044 }
}
filter {
  grok { match => { "message" => "%{COMBINEDAPACHELOG}" } }
  date { match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"] }
  mutate { remove_field => ["agent","ident","auth"] }
}
output {
  elasticsearch { hosts => ["http://elasticsearch:9200"], index => "weblogs-%{+YYYY.MM.dd}" }
  stdout { codec => rubydebug }
}
```

## Beats
- Filebeat: ship logs
- Metricbeat: ship metrics
- Heartbeat: uptime monitoring
- Packetbeat: network data

### Filebeat to Logstash
```yaml
filebeat.inputs:
  - type: filestream
    paths:
      - /var/log/*.log
output.logstash:
  hosts: ["localhost:5044"]
```

## Elasticsearch Basics

### Index Management
```bash
# Create index
curl -XPUT http://localhost:9200/logs-2025.10.25

# List indices
curl http://localhost:9200/_cat/indices?v

# Delete index
curl -XDELETE http://localhost:9200/logs-2025.10.25
```

### Query DSL Examples
```json
GET /weblogs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "response": 500 } },
        { "range": { "@timestamp": { "gte": "now-15m" } } }
      ]
    }
  },
  "aggs": {
    "by_url": { "terms": { "field": "request.keyword", "size": 10 } }
  }
}
```

## Kibana
- Discover: search and filter logs
- Visualize/Library: build visualizations (bar, line, pie, TSVB, Lens)
- Dashboards: compose visualizations
- Dev Tools: Console for Elasticsearch DSL

### Saved Search and Dashboard Tips
- Use keyword fields (*.keyword) for aggregations
- Use filters instead of query strings where possible
- Set index pattern time field to @timestamp

## Security & Scaling
- Enable xpack.security for auth/roles
- Use ILM (Index Lifecycle Management) for rollover and retention
- Use hot-warm-cold node architecture
- Tune shards/replicas based on data size and query patterns

## Common Troubleshooting
```bash
# Check Elasticsearch health
curl http://localhost:9200/_cluster/health?pretty

# Check Logstash pipeline status
curl http://localhost:9600/_node/pipelines?pretty

# Kibana server logs
docker logs <kibana-container>
```

## Useful Links
- Elasticsearch docs: https://www.elastic.co/guide/index.html
- Logstash docs: https://www.elastic.co/guide/en/logstash/current/index.html
- Kibana docs: https://www.elastic.co/guide/en/kibana/current/index.html
