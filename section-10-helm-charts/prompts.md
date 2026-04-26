# Section 10 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build Helm Charts

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform107: Create generic Helm chart for Petclinic services
- PetclinicPlatform108: Create per-service Helm values files
- PetclinicPlatform109: Create per-environment Helm values files
- PetclinicPlatform110: Test Helm template rendering and validate output
- PetclinicPlatform111: Document Helm chart usage and conventions

Read the technical spec at docs/technical-spec.md, specifically:
- Helm Charts section (chart structure, values hierarchy, per-service values, deploy command)

Read the existing K8s manifests at k8s/base/ and k8s/overlays/ — use them as the source of truth for ports, env vars, probes, resources, init containers, security context, replica counts, HPA targets, and PDB settings. The rendered Helm output must produce equivalent Kubernetes resources.

Build the generic Helm chart at helm/petclinic-service/ with templates for Deployment, Service, ConfigMap, ServiceAccount, HPA (conditional), and PDB (conditional).
HPA and PDB templates must only render when enabled in values — not every service gets them.
Prod replica counts, HPA targets, and PDB settings are per-service — match the existing overlays exactly, do not use blanket defaults.
Secret references in per-service values must use the exact secret names and keys from the existing k8s/base/ manifests.
Create per-service values files at helm-values/ for all 8 services.
Create dev.yaml and prod.yaml environment overrides.
After you are done, run helm lint and helm template for each service with both dev and prod values.
Then run kubectl apply --dry-run=client on the rendered output.
Create a validation script at scripts/validate-helm.sh.
Then use the k8s-validator agent to review the rendered output.
Fix any issues found, then re-validate.
```

---

## What Claude Code Will Generate

- `helm/petclinic-service/Chart.yaml`
- `helm/petclinic-service/values.yaml` — defaults
- `helm/petclinic-service/templates/deployment.yaml`
- `helm/petclinic-service/templates/service.yaml`
- `helm/petclinic-service/templates/configmap.yaml`
- `helm/petclinic-service/templates/serviceaccount.yaml`
- `helm/petclinic-service/templates/hpa.yaml` — conditional
- `helm/petclinic-service/templates/pdb.yaml` — conditional
- `helm-values/{service}.yaml` — for all 8 services
- `helm-values/dev.yaml` and `helm-values/prod.yaml`
- `scripts/validate-helm.sh` — runs lint + template + dry-run for all services

---

## After the Prompt

1. Check one per-service values file — ports, image name, env vars match the existing base manifests
2. Check the conditional HPA — only in prod values for the right services
3. Run the validation script:
   ```bash
   chmod +x scripts/validate-helm.sh
   ./scripts/validate-helm.sh
   ```
4. Test a manual render:
   ```bash
   helm template customers-service helm/petclinic-service \
     -f helm-values/customers-service.yaml \
     -f helm-values/dev.yaml
   ```
