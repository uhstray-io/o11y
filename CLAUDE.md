# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack observability deployment repository for Uhstray.io using Docker Compose to orchestrate a comprehensive monitoring, logging, and tracing stack. The architecture implements the Grafana observability ecosystem with Prometheus for metrics, Loki for logs, Tempo for tracing, and Mimir for long-term metrics storage.

## Common Commands

### Deployment Commands
```bash
# Start the full observability stack
docker compose up -d

# View logs from all services
docker compose logs

# View logs from a specific service
docker compose logs grafana
docker compose logs -f prometheus  # Follow logs live

# Stop the deployment
docker compose down

# Stop and remove all volumes (full cleanup)
docker compose down -v

# Stop and remove all images and volumes
docker compose down --rmi="all" -v
```

### Testing Commands
```bash
# k6 load testing is included for trace generation
# The k6-tracing service automatically generates test traffic
```

## Architecture Overview

The system is built using a microservices approach with the following key components:

### Data Collection Layer
- **Alloy**: Unified telemetry collector for logs and traces (ports 4317, 4318)
- **Prometheus**: Local time-series database for metrics collection (port 9090)
- **Node Exporter**: System metrics collection (port 9100)
- **cAdvisor**: Container metrics collection (port 8080)
- **Promtail**: Log collection and forwarding agent

### Data Processing and Storage
- **Mimir**: Distributed metrics storage with 3-replica cluster behind nginx load balancer (port 9009)
  - Uses S3-compatible MinIO for object storage
  - Configured for high availability with memberlist gossip protocol
- **Loki**: Log aggregation system (port 3100)
- **Tempo**: Distributed tracing backend (port 3200)
  - Configured with metrics generator for service graphs and span metrics
- **MinIO**: S3-compatible object storage for Mimir and Tempo (port 9000)

### Visualization and Analysis
- **Grafana**: Primary visualization platform (port 3000, admin/admin)
  - Pre-configured dashboards for Mimir monitoring
  - Datasources for Prometheus, Loki, Tempo, and Pyroscope
- **Pyroscope**: Continuous profiling platform (port 4040)

## Key Configuration Files

### Component Configurations
- `compose.yml`: Main orchestration file including all services
- `mimir/mimir.yml`: Mimir configuration with S3 backend and clustering
- `alloy/config.alloy`: Alloy collector configuration for OTLP ingestion
- `tempo/config.yml`: Tempo distributed tracing configuration
- `grafana/provisioning/`: Pre-configured dashboards and datasources

### Service Architecture Patterns
- **Modular Compose Structure**: Each service has its own compose.yml in subdirectories
- **Shared Networks**: Services communicate via `front-tier` and `back-tier` networks
- **Volume Management**: Persistent storage for Grafana data and component configurations
- **S3 Integration**: Mimir and Tempo use MinIO for scalable object storage

## Development Workflow

1. **Local Development**: Use `docker compose up -d` for full stack deployment
2. **Component Testing**: Individual services can be started by including specific compose files
3. **Monitoring**: Access Grafana at localhost:3000 for comprehensive observability
4. **Troubleshooting**: Use `docker compose logs <service>` for debugging

## Service Dependencies

- Grafana depends on Mimir cluster (mimir-1, mimir-2, mimir-3)
- Mimir services require MinIO for object storage
- Tempo uses MinIO for trace storage and connects to Prometheus for metrics
- Alloy forwards telemetry to Loki and Tempo
- k6-tracing generates test traffic for the distributor

## Security Considerations

- Default credentials are used for development (should be changed for production)
- Services communicate over insecure connections within Docker networks
- MinIO uses development credentials (minio/supersecret)