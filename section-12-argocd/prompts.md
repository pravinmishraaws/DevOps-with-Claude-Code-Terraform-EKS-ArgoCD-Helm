# Section 12 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build ArgoCD GitOps Setup

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform112: ArgoCD installation manifests
- PetclinicPlatform113: Dev Application CRDs with auto-sync
- PetclinicPlatform114: Prod Application CRDs with manual sync
- PetclinicPlatform115: ArgoCD RBAC and access control
- PetclinicPlatform116: Test GitOps loop

Read the technical spec at docs/technical-spec.md, specifically:
- GitOps with ArgoCD section (Application CRD structure, sync policies, RBAC)

Read the existing Helm chart at helm/petclinic-service/ and the values files at helm-values/ — use them to determine the correct path and valueFiles for each Application CRD.

Run `git remote get-url origin` to get the platform repo remote URL. Use that URL as the repoURL in all Application CRDs. Do not hardcode a URL or use a placeholder.

For the ArgoCD installation manifest, download the latest stable install.yaml using:
curl -sL https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -o k8s/argocd/install/install.yaml
Save it exactly as downloaded. Do NOT modify or regenerate its contents — the file contains namespace: argocd on every resource and must be preserved exactly as-is. Do not add comments, do not rewrite the file.

Create the ArgoCD installation manifests at k8s/argocd/install/:
- namespace.yaml — creates the argocd namespace
- install.yaml — full ArgoCD installation at pinned version (downloaded above)

Create dev Application CRDs at k8s/argocd/applications/dev/ — one per service (config-server, discovery-server, api-gateway, customers-service, visits-service, vets-service, genai-service, admin-server). Each Application MUST have:
- metadata.name: {service}-dev (e.g., customers-service-dev, genai-service-dev)
- metadata.namespace: argocd
- spec.project: default
- spec.source.repoURL: (from git remote get-url origin)
- spec.source.targetRevision: main
- spec.source.path: helm/petclinic-service
- spec.source.helm.releaseName: {service} (e.g., config-server, customers-service — without the -dev suffix)
- spec.source.helm.valueFiles: [../../helm-values/{service}.yaml, ../../helm-values/dev.yaml]
- spec.destination.server: https://kubernetes.default.svc
- spec.destination.namespace: petclinic-dev
- spec.syncPolicy.automated.prune: true
- spec.syncPolicy.automated.selfHeal: true
- spec.syncPolicy.syncOptions: [CreateNamespace=true, PruneLast=true, ApplyOutOfSyncOnly=true]

Create prod Application CRDs at k8s/argocd/applications/prod/ — same 8 services, same structure but:
- metadata.name: {service}-prod (e.g., customers-service-prod)
- spec.source.helm.valueFiles: [../../helm-values/{service}.yaml, ../../helm-values/prod.yaml]
- spec.destination.namespace: petclinic-prod
- NO syncPolicy.automated block at all — prod is manual sync only

Create the RBAC ConfigMap at k8s/argocd/argocd-rbac-cm.yaml — admin role (full access to all apps and settings) and developer role (view all apps, sync dev environment only, no prod sync).

After you are done, run kubectl apply --dry-run=client on all generated manifests to validate YAML structure.
```

---

## What Claude Code Will Generate

- `k8s/argocd/install/namespace.yaml`
- `k8s/argocd/install/install.yaml` — downloaded from GitHub
- `k8s/argocd/applications/dev/{service}-dev.yaml` — 8 files
- `k8s/argocd/applications/prod/{service}-prod.yaml` — 8 files
- `k8s/argocd/argocd-rbac-cm.yaml`

---

## After the Prompt

1. Check one Application CRD — confirm `repoURL` is your actual GitHub URL, not a placeholder
2. Check dev apps have `syncPolicy.automated`, prod apps do NOT
3. Install ArgoCD:
   ```bash
   kubectl apply -f k8s/argocd/install/namespace.yaml
   kubectl apply -f k8s/argocd/install/install.yaml
   kubectl wait --for=condition=available deployment -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
   ```
4. Apply Application CRDs:
   ```bash
   kubectl apply -f k8s/argocd/applications/dev/
   ```
5. Access the UI:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   # Open https://localhost:8080
   # Get initial admin password:
   kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
   ```
6. Watch all 8 apps sync and turn green
