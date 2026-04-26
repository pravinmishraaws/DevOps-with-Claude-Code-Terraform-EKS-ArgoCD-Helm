# Section 13 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build the Observability Stack

```
Read the following stories from docs/jira-backlog.md:
- PETPLAT-55: Prometheus setup (deployment, scrape config, PV, retention)
- PETPLAT-56: Deploy Grafana (Prometheus + Loki datasources, PV, credentials)
- PETPLAT-57: Per-service Grafana dashboards
- PETPLAT-58: Alerting rules (PrometheusRule CRDs, 5 rules)
- PETPLAT-59: Loki and FluentBit (Loki deployment, FluentBit DaemonSet, Loki alert rules)
- PETPLAT-60: Zipkin tracing (deployment, service, tracing namespace)
- PETPLAT-103: Alertmanager (deployment, Prometheus integration, notification channel)

Read the technical spec at docs/technical-spec.md, specifically:
- Observability section (Prometheus, Grafana, Loki, FluentBit, Zipkin, Alertmanager, alert rules)

Build the Kubernetes manifests at k8s/base/observability/ with:
- prometheus.yaml: Deployment, Service, ConfigMap (scrape config for 5 application services only — see enforcement rule below), PersistentVolumeClaim
- grafana.yaml: Deployment, Service, ConfigMaps (Prometheus datasource + Loki datasource + dashboard provisioning)
- loki.yaml: Deployment, Service, ConfigMap, PersistentVolumeClaim
- fluentbit.yaml: DaemonSet, ConfigMap (input/filter/output pointing to http://loki.monitoring:3100), ServiceAccount
- alertmanager.yaml: Deployment, Service, ConfigMap (routing config, at least one receiver)
- zipkin.yaml: Deployment, Service in tracing namespace
- alerting-rules.yaml: PrometheusRule CRDs (all 5 metric alert rules) + Loki alert rules (error spike, OOM)

Enforcement rules:
- Prometheus scrape config MUST target exactly these 5 services: api-gateway, customers-service, visits-service, vets-service, genai-service. These are the only services with micrometer-registry-prometheus in their pom.xml. DO NOT add config-server, discovery-server, or admin-server — they do not have the prometheus dependency and will return 404. Each target must scrape /actuator/prometheus on the correct port. Scrape interval: 15s.
- Grafana MUST have two datasources auto-configured in a provisioning ConfigMap — Prometheus at http://prometheus:9090 and Loki at http://loki:3100. Each datasource MUST have an explicit uid field — uid: prometheus and uid: loki. Without pinned UIDs, Grafana auto-generates random UIDs that do not match what the dashboard JSON expects, and every panel shows No Data.
- Every dashboard panel target MUST include "refId": "A" (increment to "B", "C" for multiple targets in the same panel) and "datasource": {"type": "prometheus", "uid": "prometheus"}. Grafana 10+ requires both fields — missing either causes "Failed to upgrade legacy queries" on every dashboard.
- FluentBit output MUST point to http://loki.monitoring:3100 — not CloudWatch, not stdout. No ServiceAccount annotation for IRSA is needed — Loki is in-cluster.
- Alertmanager MUST be connected to Prometheus via the alertmanager_config in the Prometheus ConfigMap. Without this connection, Prometheus alert rules fire but no notification is sent.
- Zipkin MUST deploy to the tracing namespace — not monitoring. The application services send traces to http://zipkin.tracing:9411. Wrong namespace means no traces arrive.
- The 5 application service ConfigMaps (api-gateway, customers-service, visits-service, vets-service, genai-service) MUST include two tracing env vars: MANAGEMENT_ZIPKIN_TRACING_ENDPOINT=http://zipkin.tracing:9411/api/v2/spans and MANAGEMENT_TRACING_SAMPLING_PROBABILITY=1.0. Without the endpoint, services have the tracing library but no destination. Without sampling=1.0, only 10% of requests generate traces — invisible in a demo.
- All 5 PrometheusRule alert conditions must match exactly what is in the technical spec — including the PromQL expressions, the for durations, and the severity labels.

Install the PrometheusRule CRD before running dry-run — kubectl needs the type registered to validate the alerting-rules.yaml PrometheusRule resource, even in dry-run mode:

kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml

Then run kubectl apply --dry-run=client on all K8s manifests.
```

---

## What Claude Code Will Generate

- `k8s/base/observability/prometheus.yaml`
- `k8s/base/observability/grafana.yaml`
- `k8s/base/observability/loki.yaml`
- `k8s/base/observability/fluentbit.yaml`
- `k8s/base/observability/alertmanager.yaml`
- `k8s/base/observability/alerting-rules.yaml`
- `k8s/base/observability/zipkin.yaml` (in `tracing` namespace)
- Updates to service ConfigMaps — adds Zipkin tracing env vars

---

## After the Prompt

1. Check Prometheus scrape config — only 5 services listed, correct ports
2. Check Grafana datasource ConfigMap — both datasources have explicit `uid` field
3. Check Zipkin is in `tracing` namespace, not `monitoring`
4. Apply to cluster:
   ```bash
   kubectl apply -f k8s/base/observability/
   ```
5. Access Prometheus:
   ```bash
   kubectl port-forward svc/prometheus 9090:9090 -n monitoring
   # Check Targets — 5 services should show UP
   ```
6. Access Grafana:
   ```bash
   kubectl port-forward svc/grafana 3000:3000 -n monitoring
   # Open http://localhost:3000, login admin/admin
   ```
7. Access Zipkin:
   ```bash
   kubectl port-forward svc/zipkin 9411:9411 -n tracing
   # Make a few requests to the app, then check traces
   ```
