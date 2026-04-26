# Section 9 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build Kubernetes Manifests

```
Read the following stories from docs/jira-backlog.md:

Base manifests (Epic E-8):
- PetclinicPlatform38: Create K8s namespaces manifest
- PetclinicPlatform39: Create Config Server K8s manifests
- PetclinicPlatform40: Create Discovery Server K8s manifests
- PetclinicPlatform41: Create domain services K8s manifests (customers, visits, vets)
- PetclinicPlatform42: Create GenAI Service K8s manifests
- PetclinicPlatform43: Create API Gateway K8s manifests
- PetclinicPlatform44: Create Admin Server K8s manifests

Environment overlays (Epic E-9):
- PetclinicPlatform45: Create dev overlay patches
- PetclinicPlatform46: Create prod overlay patches
- PetclinicPlatform47: Add Horizontal Pod Autoscaler for prod

Read the technical spec at docs/technical-spec.md, specifically:
- Kubernetes Manifests section (namespaces, labels, probes, resources, env vars, init containers, security context)
- Application Services section (ports, profiles, startup order)
- Kubernetes Overlays section (dev and prod settings)

Build everything these stories require. Follow the acceptance criteria exactly.
Use init containers with busybox:1.36 to enforce startup order — wget loops polling health endpoints.
All deployments must include a startupProbe, readinessProbe, and livenessProbe. The startupProbe uses /actuator/health on all services. The readinessProbe uses /actuator/health/readiness and the livenessProbe uses /actuator/health/liveness — except Config Server, which uses /actuator/health for all three probes. The startupProbe allows Spring Boot time to initialize before liveness kills the pod.
All deployments must include the full securityContext from the spec (runAsNonRoot, runAsUser, fsGroup, seccompProfile RuntimeDefault, drop ALL capabilities).
All deployments must include imagePullPolicy: Always.
After you are done, run kubectl apply --dry-run=client on all manifests to validate.
Then use the k8s-validator agent to review all manifests against project standards.
Fix any issues the reviewer finds, then run dry-run again.
```

---

## What Claude Code Will Generate

- `k8s/base/namespaces.yaml` — petclinic-dev and petclinic-prod namespaces
- `k8s/base/{service}/deployment.yaml` — for all 8 services (with init containers, probes, securityContext)
- `k8s/base/{service}/service.yaml` — ClusterIP services
- `k8s/overlays/dev/` — kustomization.yaml with replica and resource patches
- `k8s/overlays/prod/` — kustomization.yaml with replica, resource, HPA, PDB patches
- `k8s/base/ingress/` — Ingress resource (may already exist from Section 7)

---

## After the Prompt

1. Review one deployment file — check probes, init containers, security context, image pull policy
2. Check the dev overlay — 1 replica, smaller resources
3. Check the prod overlay — 2+ replicas, HPA targets correct services
4. Validate all manifests:
   ```bash
   kubectl apply --dry-run=client -f k8s/base/namespaces.yaml
   kubectl apply --dry-run=client -f k8s/base/config-server/
   # repeat for each service
   ```
5. Apply namespaces:
   ```bash
   kubectl apply -f k8s/base/namespaces.yaml
   ```
