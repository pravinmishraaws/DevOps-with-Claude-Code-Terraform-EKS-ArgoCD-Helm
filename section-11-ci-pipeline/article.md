# Section 11 — CI Pipeline (GitHub Actions)

## What You Build

Two GitHub Actions workflows — one in the application repo, one in the platform repo — plus the AWS OIDC federation for keyless authentication:

### In `spring-petclinic-microservices/` (app repo)
- `.github/workflows/build-push.yml` — triggered on push to main, builds ARM64 Docker images, scans with Trivy, pushes to ECR, fires a dispatch event to the platform repo

### In `petclinic-platform/` (platform repo)
- `.github/workflows/update-image-tags.yml` — triggered by the dispatch event, updates `image.tag` in `helm-values/{service}.yaml` for the services that changed, commits back to Git
- `terraform/modules/github-oidc/` — OIDC provider and IAM role for GitHub Actions

---

## Why OIDC Instead of Access Keys?

Storing `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in GitHub Secrets works but creates long-lived credentials that can leak, need rotation, and are hard to audit.

OIDC (OpenID Connect) replaces this: GitHub Actions generates a short-lived JWT token for each workflow run. AWS exchanges that token for temporary STS credentials. No static keys stored anywhere.

The IAM role trust policy restricts access to a specific repo and branch:
```
token.actions.githubusercontent.com:sub = 
  repo:{your-username}/spring-petclinic-microservices:ref:refs/heads/main
```

---

## The Full CI/CD Loop

```
1. Developer pushes code to spring-petclinic-microservices main branch
2. build-push.yml triggers
3. Detects which services changed (paths-filter)
4. Builds ARM64 Docker images for changed services only
5. Trivy scans each image — fails if CRITICAL CVEs found
6. Pushes images to ECR with tag = {7-char commit SHA}
7. Fires repository_dispatch to petclinic-platform repo
8. update-image-tags.yml triggers
9. Updates helm-values/{service}.yaml image.tag for changed services
10. Commits and pushes to platform repo
11. ArgoCD detects the commit, syncs the cluster (Section 12)
```

---

## ARM64 Build Details

Building ARM64 images on GitHub Actions (which runs on x86 runners):

```yaml
- uses: docker/setup-qemu-action@v3    # QEMU for cross-compilation
- uses: docker/setup-buildx-action@v3  # Buildx for multi-platform builds
- run: docker buildx build --platform linux/arm64 ...
```

QEMU emulates the ARM64 CPU on the x86 runner. Slower than native — expect 5-10 min per service on first build.

---

## Trivy Scanning

Every image is scanned **before** push:
```yaml
- uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ env.ECR_REGISTRY }}/petclinic-dev/config-server:${{ env.SHA }}
    severity: CRITICAL
    exit-code: '1'    # Fail the pipeline if CRITICAL found
```

If a CRITICAL CVE is found, the push is blocked. The image never reaches ECR.

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PETPLAT-49 | Build and push workflow (app repo) |
| PETPLAT-50 | Update image tags workflow (platform repo) |
| PETPLAT-52 | OIDC federation for GitHub Actions |
| PETPLAT-53 | Reusable workflow templates |
| PETPLAT-54 | Rollback strategy documentation |
| PETPLAT-87 | Image tag update mechanism |
| PETPLAT-105 | Trivy vulnerability scanning |

---

## Prerequisites for This Section

1. Fork `spring-petclinic-microservices` to your GitHub account
2. Create a GitHub PAT with write access to `petclinic-platform`
3. Add the PAT as `PLATFORM_REPO_TOKEN` secret in the app repo

---

## External Links

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [OIDC with AWS](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [dorny/paths-filter](https://github.com/dorny/paths-filter)
- [docker/setup-qemu-action](https://github.com/docker/setup-qemu-action)
- [docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)
- [Trivy Action](https://github.com/aquasecurity/trivy-action)
- [GitHub repository_dispatch event](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [yq — YAML processor](https://github.com/mikefarah/yq)
