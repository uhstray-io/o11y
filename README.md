# o11y
Observability deployment resources for Uhstray.io

# Deploying Observability Assets using Github Actions

## Architecture

![Observability Architecture](observability.drawio.png)

---

## Prometheus

### Prometheus Database

https://prometheus.io/docs/prometheus/latest/installation/

Intial build/test for Prometheus

```bash
# Create persistent volume for your data
docker volume create prometheus-data
# Start Prometheus container
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v prometheus-data:/prometheus \
    prom/prometheus
```

Update the prometheus.yml file with the proper

```dockerfile
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```

Run the docker image with the following commands

```bash
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```

https://github.com/prometheus-community/ansible

### Prometheus Node Exporter

https://github.com/prometheus/node_exporter

```bash
docker compose -f node-exporter-compose.yml up
```

### Prometheus PostgreSQL Exporter

https://github.com/prometheus-community/postgres_exporter

---

## OpenTelemetry

### OpenTelemetry Collector

https://github.com/open-telemetry/opentelemetry-collector

https://opentelemetry.io/docs/specs/otlp/

https://opentelemetry.io/docs/collector/deployment/agent/

https://opentelemetry.io/docs/collector/installation/

```yaml
otel-collector:
  image: otel/opentelemetry-collector-contrib
  volumes:
    - ./otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml
  ports:
    - 1888:1888 # pprof extension
    - 8888:8888 # Prometheus metrics exposed by the Collector
    - 8889:8889 # Prometheus exporter metrics
    - 13133:13133 # health_check extension
    - 4317:4317 # OTLP gRPC receiver
    - 4318:4318 # OTLP http receiver
    - 55679:55679 # zpages extension
```

### OTEL GO Instrumentation

https://opentelemetry.io/docs/languages/go/getting-started/

https://opentelemetry.io/docs/languages/go/instrumentation/

https://opentelemetry.io/docs/languages/go/exporters/

### OpenLLMetry Instrumentation

https://github.com/traceloop/openllmetry

https://www.traceloop.com/docs/openllmetry/getting-started-python

OTEL Collector Configuration

```bash
TRACELOOP_BASE_URL=https://<opentelemetry-collector-hostname>:4318
```