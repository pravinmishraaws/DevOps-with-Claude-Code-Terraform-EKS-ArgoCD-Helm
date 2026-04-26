# Section 13 — Observability

## What You Build

A complete observability stack running on EKS:

| Component | Port | Role |
|-----------|------|------|
| Prometheus | 9090 | Scrapes metrics from app services every 15s |
| Grafana | 3000 | Dashboards for all metrics |
| Loki | 3100 | Log aggregation |
| FluentBit | DaemonSet | Collects logs from all pods, sends to Loki |
| Alertmanager | 9093 | Routes Prometheus alerts to notification channels |
| Zipkin | 9411 | Distributed tracing |

---

## Which Services Export Metrics

Only 5 of the 8 services have `micrometer-registry-prometheus` in their `pom.xml`:
- api-gateway
- customers-service
- visits-service
- vets-service
- genai-service

**Do not scrape config-server, discovery-server, or admin-server.** They expose `/actuator` endpoints but don't have the Prometheus dependency — Prometheus would get 404 errors from them.

---

## Grafana Datasource UIDs — Critical Detail

Grafana 10+ requires both a `type` and a `uid` field on every dashboard panel datasource. Without pinned UIDs in the provisioning ConfigMap, Grafana auto-generates random UIDs on startup. The dashboard JSON references specific UIDs — if they don't match, every panel shows "No Data."

Always provision datasources with explicit UIDs:
```yaml
datasources:
  - name: Prometheus
    type: prometheus
    uid: prometheus          # must match what's in dashboard JSON
    url: http://prometheus:9090
  - name: Loki
    type: loki
    uid: loki
    url: http://loki:3100
```

---

## Distributed Tracing (Zipkin)

Zipkin must run in the `tracing` namespace (not `monitoring`). The application services send traces to `http://zipkin.tracing:9411/api/v2/spans`. Wrong namespace = traces go nowhere.

Two environment variables enable tracing in the 5 instrumented services:
```
MANAGEMENT_ZIPKIN_TRACING_ENDPOINT=http://zipkin.tracing:9411/api/v2/spans
MANAGEMENT_TRACING_SAMPLING_PROBABILITY=1.0
```

Without `SAMPLING_PROBABILITY=1.0`, only 10% of requests generate traces — nearly invisible in a demo environment.

---

## Alert Rules (5 rules)

| Rule | Condition |
|------|-----------|
| High error rate | HTTP 5xx rate > threshold for 5 min |
| Pod restart loop | Pod restarts > 5 in 15 min |
| High memory usage | Memory usage > 80% of limit |
| Service down | No metrics for 2 min |
| Slow response time | P99 latency > 2s for 5 min |

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PETPLAT-55 | Prometheus (Deployment, ConfigMap, PVC) |
| PETPLAT-56 | Grafana (Deployment, datasource/dashboard provisioning) |
| PETPLAT-57 | Per-service Grafana dashboards |
| PETPLAT-58 | Alerting rules (PrometheusRule CRDs) |
| PETPLAT-59 | Loki + FluentBit |
| PETPLAT-60 | Zipkin (in tracing namespace) |
| PETPLAT-103 | Alertmanager |

---

## After This Section

- All observability components running in `monitoring` namespace
- Zipkin in `tracing` namespace
- `kubectl port-forward svc/grafana 3000:3000 -n monitoring` → dashboard shows live data
- Prometheus targets show 5 services as UP
- FluentBit sending pod logs to Loki

---

## External Links

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Prometheus Operator PrometheusRule CRD](https://prometheus-operator.dev/docs/user-guides/alerting/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Grafana Datasource Provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/#data-sources)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [FluentBit Documentation](https://docs.fluentbit.io/manual)
- [Zipkin Documentation](https://zipkin.io/)
- [Spring Boot Micrometer](https://micrometer.io/docs/registry/prometheus)
- [Spring Boot Zipkin/Tracing](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.micrometer-tracing)
