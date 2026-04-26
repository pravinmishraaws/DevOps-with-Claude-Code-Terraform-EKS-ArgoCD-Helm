# Section 10 — Helm Charts

## What You Build

A generic Helm chart that can deploy any of the 8 Petclinic microservices, plus per-service and per-environment values files:

```
helm/
└── petclinic-service/           # Generic chart (one chart, 8 services)
    ├── Chart.yaml
    ├── values.yaml              # Defaults
    └── templates/
        ├── deployment.yaml
        ├── service.yaml
        ├── configmap.yaml
        ├── serviceaccount.yaml
        ├── hpa.yaml             # Conditional — only renders when enabled
        └── pdb.yaml             # Conditional — only renders when enabled

helm-values/
├── config-server.yaml           # Per-service values
├── discovery-server.yaml
├── api-gateway.yaml
├── customers-service.yaml
├── visits-service.yaml
├── vets-service.yaml
├── genai-service.yaml
├── admin-server.yaml
├── dev.yaml                     # Environment overrides
└── prod.yaml
```

---

## Why Helm Instead of Raw Manifests?

The raw manifests from Section 9 work, but they create a maintenance problem: 8 services × 2 environments = 16 sets of manifests. Change one label or probe — update 16 files.

Helm solves this with a **generic template + values separation**:
- The chart template handles the structure (same for all services)
- Per-service values files provide the service-specific config (ports, image, env vars)
- Per-environment values files override replicas, resource limits, HPA settings

Deploy any service to any environment with one command:
```bash
helm upgrade --install customers-service helm/petclinic-service \
  -f helm-values/customers-service.yaml \
  -f helm-values/dev.yaml \
  -n petclinic-dev
```

---

## Values Hierarchy

When you pass multiple `-f` flags, later files override earlier ones:

```
helm/petclinic-service/values.yaml   ← defaults
  + helm-values/customers-service.yaml  ← service-specific (ports, image, env vars)
  + helm-values/dev.yaml                ← environment (replicas, resources)
```

---

## Conditional Templates

Not every service gets an HPA or PDB. The chart uses conditional rendering:

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
...
{{- end }}
```

This prevents empty `HorizontalPodAutoscaler: {}` resources from being rendered for services that don't need autoscaling.

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform107 | Generic Helm chart |
| PetclinicPlatform108 | Per-service values files (8 files) |
| PetclinicPlatform109 | Per-environment values files (dev.yaml, prod.yaml) |
| PetclinicPlatform110 | Validation — helm lint + helm template + kubectl dry-run |
| PetclinicPlatform111 | Documentation of chart usage |

---

## After This Section

- `helm lint helm/petclinic-service` passes
- `helm template` renders correct manifests for each service
- ArgoCD (Section 12) will use this chart to deploy all 8 services

---

## External Links

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Chart Template Guide](https://helm.sh/docs/chart_template_guide/)
- [Helm Values Files](https://helm.sh/docs/chart_template_guide/values_files/)
- [Helm Lint](https://helm.sh/docs/helm/helm_lint/)
- [Helm Template](https://helm.sh/docs/helm/helm_template/)
