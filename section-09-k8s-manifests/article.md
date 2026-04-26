# Section 9 — Kubernetes Manifests

## What You Build

Base Kubernetes manifests for all 8 services, plus dev and prod overlays:

```
k8s/
├── base/
│   ├── namespaces.yaml
│   ├── config-server/        deployment.yaml, service.yaml
│   ├── discovery-server/     deployment.yaml, service.yaml
│   ├── api-gateway/          deployment.yaml, service.yaml
│   ├── customers-service/    deployment.yaml, service.yaml
│   ├── visits-service/       deployment.yaml, service.yaml
│   ├── vets-service/         deployment.yaml, service.yaml
│   ├── genai-service/        deployment.yaml, service.yaml
│   └── admin-server/         deployment.yaml, service.yaml
└── overlays/
    ├── dev/                  1 replica, smaller resource limits
    └── prod/                 2+ replicas, HPA, larger resources
```

---

## Startup Order in Kubernetes

In Docker Compose, `depends_on` enforced startup order. Kubernetes has no equivalent.

The solution: **init containers** using `busybox`. Each service that depends on Config Server and Discovery Server gets init containers that poll the health endpoints in a loop before the main container starts:

```yaml
initContainers:
  - name: wait-for-config-server
    image: busybox:1.36
    command: ['sh', '-c', 'until wget -q http://config-server:8888/actuator/health; do sleep 2; done']
  - name: wait-for-discovery-server
    image: busybox:1.36
    command: ['sh', '-c', 'until wget -q http://discovery-server:8761/actuator/health; do sleep 2; done']
```

---

## Three Probes on Every Deployment

All deployments get three probes — these are required, not optional:

| Probe | Endpoint | Purpose |
|-------|----------|---------|
| `startupProbe` | `/actuator/health` | Gives Spring Boot time to start before liveness kicks in |
| `readinessProbe` | `/actuator/health/readiness` | Pod only gets traffic when truly ready |
| `livenessProbe` | `/actuator/health/liveness` | Restarts the pod if it hangs |

**Config Server exception:** uses `/actuator/health` for all three probes (it doesn't expose readiness/liveness separately).

Without a `startupProbe`, the `livenessProbe` fires during Spring Boot's slow startup and kills the pod in a restart loop.

---

## Security Context on Every Deployment

All deployments include a hardened security context:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault
  capabilities:
    drop: ["ALL"]
```

---

## Dev vs Prod Overlays

| Setting | Dev | Prod |
|---------|-----|------|
| Replicas | 1 | 2+ |
| CPU request | 100m | 250m |
| Memory request | 256Mi | 512Mi |
| HPA | No | Yes (for API Gateway, customers, visits, vets) |
| PDB | No | Yes |

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform38 | Namespaces manifest |
| PetclinicPlatform39 | Config Server manifests |
| PetclinicPlatform40 | Discovery Server manifests |
| PetclinicPlatform41 | Domain services manifests (customers, visits, vets) |
| PetclinicPlatform42 | GenAI Service manifests |
| PetclinicPlatform43 | API Gateway manifests |
| PetclinicPlatform44 | Admin Server manifests |
| PetclinicPlatform45 | Dev overlay patches |
| PetclinicPlatform46 | Prod overlay patches |
| PetclinicPlatform47 | HPA for prod |

---

## External Links

- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Kustomize Overlays](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/)
- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)
- [Spring Boot Actuator Probes](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.kubernetes-probes)
