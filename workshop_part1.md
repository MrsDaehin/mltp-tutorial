# PromQL Basic Workshop – Part 1

Welcome to the PromQL Basic Workshop! This guide will help you get started with Prometheus Query Language (PromQL) and understand the monitoring stack defined in your `docker-compose-otel.yml`.

---

## 1. Stack Overview

Your environment is composed of several services, including:

- **Prometheus**: Scrapes and stores metrics.
- **Alertmanager**: Handles alerts sent by Prometheus.
- **Grafana**: Visualizes metrics, logs, and traces.
- **Tempo**: Stores and queries traces.
- **Loki**: Stores and queries logs.
- **Pyroscope**: Handles continuous profiling.
- **RabbitMQ**: Message queue for microservices.
- **Postgres**: Database for the API server.
- **Beyla**: Auto-instrumentation for services.
- **k6**: Load testing tool.

---

## 2. Key Prometheus & Alertmanager Configuration

### Prometheus Service

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    - ./prometheus/alert.rules.yml:/etc/prometheus/alert.rules.yml
  ports:
    - "9090:9090"
  command:
    - "--config.file=/etc/prometheus/prometheus.yml"
```

- **Config file**: `./prometheus/prometheus.yml`
- **Alert rules**: `./prometheus/alert.rules.yml`
- **Web UI**: [http://localhost:9090](http://localhost:9090)

### Alertmanager Service

```yaml
alertmanager:
  image: prom/alertmanager:latest
  ports:
    - "9093:9093"
  volumes:
    - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
  command:
    - "--config.file=/etc/alertmanager/alertmanager.yml"
```

- **Config file**: `./alertmanager/alertmanager.yml`
- **Web UI**: [http://localhost:9093](http://localhost:9093)

---
## 3. Useful Links

- [Prometheus Querying Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)

---


[Continue to Part 2: Advanced PromQL Workshop →](./workshop_part2.md)