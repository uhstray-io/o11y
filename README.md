<!-- omit in toc -->
# o11y

Automated Full-Stack Observability deployment resources for Uhstray.io

<!-- omit in toc -->
## Table of Contents

- [Architecture](#architecture)
- [Overview](#overview)
- [Contributing Guidelines](#contributing-guidelines)
- [Component Overview](#component-overview)
  - [Metrics Collection](#metrics-collection)
  - [Logs Management](#logs-management)
  - [Tracing](#tracing)
  - [Visualization](#visualization)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Deployment](#deployment)
  - [Accessing Dashboards](#accessing-dashboards)
- [Testing and Developing](#testing-and-developing)
- [Troubleshooting](#troubleshooting)
  - [Common Issues](#common-issues)
  - [Viewing Component Logs](#viewing-component-logs)
- [Technology References](#technology-references)
  - [Grafana Ecosystem](#grafana-ecosystem)
  - [Prometheus Ecosystem](#prometheus-ecosystem)
  - [OpenTelemetry](#opentelemetry)
  - [General Todo](#general-todo)
  - [WisBot Todo](#wisbot-todo)
  - [Security Enhancements](#security-enhancements)
  - [Operational Improvements](#operational-improvements)
  - [Application Observability](#application-observability)
  - [Documentation](#documentation)

---

## Architecture

```mermaid
graph LR
    %% Add direction controls to subgraphs
    subgraph Observability Collection
        subgraph Data Sources
            apps[Applications]
            containers[Containers]
            servers[Servers]
            logs[Logs]
        end
        
        subgraph Instrumentation
            subgraph Exporters
                otel_agent[OpenTelemetry Agent - Ports :4317, :4318]
                nodeexp[Node Exporter - Ports :9090]
                cadvisor[cAdvisor - Ports :9090]
                promtail[Promtail - Ports :3100]
            end
            subgraph Local OTEL Collector
                otel_col[OpenTelemetry Collector - Ports :4317, :4318]
            end
            subgraph Local Time Series Database
                prom[Prometheus - Ports :9090]
            end
        end
    end
    
    subgraph Observability Pipeline

        subgraph Logging
            subgraph Logs Pipeline
                alloy_logs{"Alloy<br>:4318"}
            end
            loki[Loki - Ports :3100]
            
        end
        subgraph Tracing
            subgraph Tracing Pipeline
                alloy_trace{"Alloy<br>:12345/:4319"}
            end
            tempoDistributor["Tempo Distributor"]
            tempoIngesters["Tempo Ingesters"]
            tempoQuery["Tempo Query Frontend<br>:3200"]
            tempoQuerier["Tempo Querier"]
            tempoCompactor["Tempo Compactor"]
            tempoMetricsGen["Tempo Metrics Generator"]
        end
        
        
        subgraph Metrics Pipeline
            mimirLB{"mimir Load Balancer<br>:9009"}
            mimir1["mimir-1<br>:8080"]
            mimir2["mimir-2<br>:8080"]
            mimir3["mimir-3<br>:8080"]
        end
    end

    subgraph Observability Analytics

        subgraph Visualization and Analytics
            grafana[Grafana - Ports :3000]
        end

        subgraph Profiling
            pyroscope["Pyroscope<br>:4040"]
        end

    end

    subgraph Data Storage and Recovery
            subgraph Object Storage
            minio[MinIO S3 Object Storage - Ports :9000]
            end
            subgraph Relational Storage
                postgres[PostgreSQL - Ports :5432]
        end
    end
    
    %% Data flow connections
    apps --> otel_agent
    containers --> cadvisor
    servers --> nodeexp
    logs --> promtail
    
    otel_agent --> otel_col
    nodeexp --> prom
    cadvisor --> prom
    tempoMetricsGen --> mimirLB
    promtail --> alloy_logs

    otel_col --> alloy_trace
    prom --> mimirLB
    alloy_logs --> loki
    alloy_trace --> tempoDistributor
    alloy_trace --> pyroscope
    mimirLB --> mimir1
    mimirLB --> mimir2
    mimirLB --> mimir3
    mimir1 --> minio
    mimir2 --> minio
    mimir3 --> minio
    
    
    tempoDistributor --> tempoIngesters
    tempoQuery --> tempoQuerier
    tempoQuerier --> tempoIngesters
    tempoCompactor --> minio
    tempoIngesters --> minio

    grafana --> mimirLB
    grafana --> loki
    grafana --> tempoQuery
    grafana --> pyroscope
    grafana --> postgres
```

## Overview

This repository contains the deployment resources for our observability stack, including Grafana, Prometheus, Mimir, Tempo, Loki, and OpenTelemetry components. The stack provides comprehensive monitoring, logging, tracing, and metrics collection for Uhstray.io services.

## Contributing Guidelines

- [Review our Code of Conduct](https://www.uhstray.io/en/code-of-conduct)
- [Check our CONTRIBUTING.MD](./CONTRIBUTING.md)

---

## Component Overview

### Metrics Collection
- **Prometheus**: Time-series database for storing metrics
- **Node Exporter**: Hardware and OS metrics collection
- **cAdvisor**: Container metrics collection
- **Mimir**: Scalable, long-term metrics storage

### Logs Management
- **Loki**: Log aggregation system
- **Promtail**: Log collection agent

### Tracing
- **Tempo**: Distributed tracing backend
- **OpenTelemetry Collector**: Trace collection and processing
- **Alloy**: Unified telemetry collector

### Visualization
- **Grafana**: Unified visualization platform for metrics, logs, and traces
- **Pyroscope**: Continuous profiling platform

---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/engine/install/) and Docker Compose installed
- Minimum recommended resources: 8 CPU cores, 16GB RAM

### Deployment

Pull down this repository and navigate to the main `o11y` directory:

```bash
git clone https://github.com/uhstray-io/o11y.git
cd ./o11y
```

Run docker compose:

```bash
docker compose up -d
```

### Accessing Dashboards

Navigate to the following dashboards:

- Grafana Dashboard: [http://localhost:3000](http://localhost:3000) (default credentials: admin/admin)
- Prometheus Dashboard: [http://localhost:9090](http://localhost:9090)
- Mimir Dashboard: [http://localhost:9009/](http://localhost:9009/)
- cAdvisor Dashboard: [http://localhost:9092/](http://localhost:9092/)
- Tempo UI: [http://localhost:3200](http://localhost:3200)
- Loki UI: [http://localhost:3100](http://localhost:3100)

---

## Testing and Developing

Get the current logs from the deployment to triage:

```bash
docker compose logs
```

Spin the current deployment down:

```bash
docker compose down
```

Spin down the deployment and remove all volumes:

```bash
docker compose down -v
```

Spin down the deployment and remove all images+volumes:

```bash
docker compose down --rmi="all" -v
```

---

## Troubleshooting

### Common Issues

1. **Services fail to start**: Check for port conflicts with `docker ps -a` and stop any conflicting services
2. **Out of memory errors**: Increase Docker memory allocation in Docker Desktop settings
3. **Permission issues**: Ensure proper file permissions for volume mounts

### Viewing Component Logs

```bash
# View logs for a specific service
docker compose logs grafana

# Follow logs live
docker compose logs -f prometheus
```

---

## Technology References

### Grafana Ecosystem
- [Grafana](https://github.com/grafana/grafana) - Visualization platform
- [Grafana Mimir](https://grafana.com/docs/mimir/latest/references/architecture/deployment-modes/) - Scalable metrics storage
- [Grafana Alloy](https://github.com/grafana/alloy) - Unified telemetry collector
- [Grafana Beyla](https://github.com/grafana/beyla) - eBPF-based auto-instrumentation
- [Grafana Pyroscope](https://github.com/grafana/pyroscope) - Continuous profiling

### Prometheus Ecosystem
- [Prometheus](https://prometheus.io/docs/prometheus/latest/installation/) - Metrics collection and storage
- [Node Exporter](https://github.com/prometheus/node_exporter) - System metrics collection
- [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/) - Log collector
- [Windows Exporter](https://github.com/prometheus-community/windows_exporter) - Windows metrics collection
- [PostgreSQL Exporter](https://github.com/prometheus-community/postgres_exporter) - PostgreSQL metrics

### OpenTelemetry
- [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector) - Telemetry collection
- [OTEL Protocol](https://opentelemetry.io/docs/specs/otlp/) - Telemetry protocol specification
- [OTEL GO Instrumentation](https://opentelemetry.io/docs/languages/go/getting-started/) - Go instrumentation
- [OpenLLMetry](https://github.com/traceloop/openllmetry) - LLM observability

---

<!-- omit in toc -->
## TODO

### General Todo

- [x] Initial deployment with Grafana, Prometheus, and exporters
- [X] Upgrade Grafana to use Mimir Prometheus TSDB
- [X] Develop OpenTelemetry Collector Process for Wisbot
- [ ] Deploy OpenTelemetry o11y collector integrated with Grafana
- [ ] Upgrade Alert Manager Storage to use GitHub Actions driven Secrets

### WisBot Todo
- [X] Upgrade to Alloy Collector where necessary for production needs
- [ ] Migrate Mimir to Microservice Deployment Mode
- [ ] Determine Beyla eBPF Instrumentation Targets
- [X] Add Pyroscope for Wisbot Profiling
- [ ] Setup relabeling to streamline service discovery | https://grafana.com/docs/loki/latest/send-data/promtail/scraping/
- [ ] Implement high availability configuration for production
- [ ] Add custom dashboards for Wisbot service monitoring

### Security Enhancements
- [ ] Implement proper secrets management through environment variables
- [ ] Remove hardcoded credentials from configuration files
- [ ] Configure TLS for exposed services
- [ ] Review and secure default credentials

### Operational Improvements
- [ ] Enable AlertManager integration with proper configuration
- [ ] Develop comprehensive alerting rules beyond basic infrastructure monitoring
- [ ] Setup proper backup/restore procedures for persistent data
- [ ] Standardize configuration practices across components

### Application Observability
- [ ] Complete Wisbot instrumentation to enable service-level metrics
- [ ] Implement distributed tracing for application components
- [ ] Configure application profiling via Pyroscope

### Documentation
- [ ] Create operational procedures and runbooks
- [ ] Develop dashboard usage guidelines
- [ ] Define metrics dictionary for key indicators
- [ ] Document architectural decisions for component choices
  
```yaml
alertmanager_storage:
      backend: s3
      s3:
        access_key_id: {{ .Values.minio.rootUser }}
        bucket_name: {{ include "mimir.minioBucketPrefix" . }}-ruler
        endpoint: {{ template "minio.fullname" .Subcharts.minio }}.{{ .Release.Namespace }}.svc:{{ .Values.minio.service.port }}
        insecure: true
        secret_access_key: {{ .Values.minio.rootPassword }}
```
