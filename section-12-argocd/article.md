# Section 12 — GitOps with ArgoCD

## What You Build

ArgoCD running on EKS, managing all 8 services with GitOps:

```
k8s/argocd/
├── install/
│   ├── namespace.yaml          # argocd namespace
│   └── install.yaml            # Full ArgoCD installation
└── applications/
    ├── dev/
    │   ├── config-server-dev.yaml
    │   ├── discovery-server-dev.yaml
    │   ├── api-gateway-dev.yaml
    │   ├── customers-service-dev.yaml
    │   ├── visits-service-dev.yaml
    │   ├── vets-service-dev.yaml
    │   ├── genai-service-dev.yaml
    │   └── admin-server-dev.yaml
    └── prod/
        └── (same 8 services, -prod suffix)
```

---

## How GitOps Works

Traditional deploy: CI runs `kubectl apply` directly — CI has cluster access, secrets, and knowledge of where to deploy.

GitOps: **Git is the source of truth**. ArgoCD runs inside the cluster and continuously watches the Git repo. When it sees a change (like an updated image tag from the CI pipeline), it applies it automatically.

```
Git push (image tag updated by CI)
  ↓
ArgoCD detects diff between desired state (Git) and actual state (cluster)
  ↓
ArgoCD applies the change
  ↓
Pod rolls out new image
```

No CI credentials for the cluster. No `kubectl` in CI. The cluster pulls from Git — it's never pushed to.

---

## Dev vs Prod Sync Policy

| | Dev | Prod |
|-|-----|------|
| Sync | **Auto** — ArgoCD syncs immediately on any Git change | **Manual** — you trigger sync deliberately |
| Prune | Yes — removes resources deleted from Git | Yes |
| Self-heal | Yes — reverts manual changes to match Git | No |
| Use case | Fast iteration, PRs auto-deploy | Stability, you control when changes roll out |

---

## Application CRD Structure

Each service gets an `Application` CRD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: customers-service-dev
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/your-org/petclinic-platform
    path: helm/petclinic-service
    helm:
      releaseName: customers-service
      valueFiles:
        - ../../helm-values/customers-service.yaml
        - ../../helm-values/dev.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: petclinic-dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## RBAC

Two roles:
- **admin** — full access to all apps and ArgoCD settings
- **developer** — view all apps, sync dev environment only, no prod sync

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform112 | ArgoCD installation manifests |
| PetclinicPlatform113 | Dev Application CRDs (auto-sync) |
| PetclinicPlatform114 | Prod Application CRDs (manual sync) |
| PetclinicPlatform115 | ArgoCD RBAC |
| PetclinicPlatform116 | Test GitOps loop |

---

## After This Section

- ArgoCD UI accessible (port-forward or Ingress)
- All 8 services shown in ArgoCD dashboard as `Synced` and `Healthy`
- Push an image tag change to Git → ArgoCD deploys it within 3 minutes

---

## External Links

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [ArgoCD Application CRD](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml)
- [ArgoCD RBAC](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/)
- [ArgoCD Sync Policies](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/)
- [GitOps Principles](https://opengitops.dev/)
