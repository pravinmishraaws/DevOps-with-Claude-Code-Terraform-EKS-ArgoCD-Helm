# Section 11 — Prompts

Run both prompts from inside `petclinic-platform/` in Claude Code.

> **Note:** This section has two prompts. Run them in order.

---

## Prompt 1 — OIDC Federation & Image Tag Update Workflow

```
Read the following stories from docs/jira-backlog.md:
- PETPLAT-52: Set up OIDC federation for GitHub Actions
- PETPLAT-50: Update image tags workflow
- PETPLAT-87: Image tag update mechanism
- PETPLAT-54: Rollback strategy documentation

Read the technical spec at docs/technical-spec.md, specifically:
- CI/CD Pipeline section (workflow triggers, OIDC trust policy, image tag mechanism)

Read the existing Helm values files at helm-values/ — the image.tag field in each service file is what the update-image-tags workflow will modify.

Create the OIDC federation Terraform resources in terraform/modules/github-oidc/ — the IAM OIDC provider and the GitHub Actions IAM role with ECR push permissions.
Create the update-image-tags workflow at .github/workflows/update-image-tags.yml.
Document the rollback process in docs/rollback-runbook.md.

Enforcement rules:
- Run `git -C ../spring-petclinic-microservices remote get-url origin` to get the app repo remote URL. Extract the GitHub username from it. Use that username in the OIDC trust policy subject — do not hardcode a placeholder.
- The OIDC trust policy MUST restrict token.actions.githubusercontent.com:sub to the application repo fork and main branch — repo:{derived-username}/spring-petclinic-microservices:ref:refs/heads/main. Not the platform repo. Not *. The build workflow runs in the app repo context, so the trust policy must reference the app repo.
- The Action MUST be sts:AssumeRoleWithWebIdentity — not sts:AssumeRole.
- The IAM role MUST have ECR-only permissions — ecr:GetAuthorizationToken, ecr:BatchCheckLayerAvailability, ecr:PutImage, and layer upload actions. No ecr:*, no *.
- The update-image-tags workflow MUST be triggered by repository_dispatch event type app-image-built — not workflow_run.
- The update-image-tags workflow MUST use yq to update image.tag in helm-values/{service}.yaml for all 8 services using the SHA passed in the repository_dispatch payload.
- The update-image-tags workflow MUST commit and push the updated helm-values files to the platform repo.
```

---

## Prompt 2 — Build & Push Workflow (App Repo)

```
Read the following stories from docs/jira-backlog.md:
- PETPLAT-49: Build and push Docker images workflow
- PETPLAT-105: Trivy vulnerability scanning
- PETPLAT-53: Reusable workflow templates

Read the technical spec at docs/technical-spec.md, specifically:
- CI/CD Pipeline section (build steps, ARM64 cross-compilation, Trivy thresholds)

The application repo fork is at ../spring-petclinic-microservices/ — it sits next to this repo in the same parent folder.

Create the build-push workflow at ../spring-petclinic-microservices/.github/workflows/build-push.yml.
Extract reusable steps into ../spring-petclinic-microservices/.github/workflows/reusable/ if appropriate.

Enforcement rules:
- The build-push workflow MUST trigger on push to main — on: push: branches: [main].
- The workflow MUST use dorny/paths-filter to detect which service directories changed. Each service maps to its folder — spring-petclinic-customers-service/**, spring-petclinic-vets-service/**, and so on for all 8 services. Only services with changes get built.
- The build job MUST use a matrix strategy driven by the paths-filter output — only matrix entries where the service changed set to true are included.
- The build MUST produce linux/arm64 images using Docker Buildx and QEMU — the --platform linux/arm64 flag is required.
- The workflow MUST authenticate to AWS using OIDC — aws-actions/configure-aws-credentials with role-to-assume. No hardcoded access keys.
- Trivy MUST scan every image BEFORE push to ECR — the scan gates the push. If CRITICAL vulnerabilities are found, the pipeline MUST fail.
- Image tags MUST use the 7-character commit SHA — github.sha[:7] — not latest, not a version number.
- After all changed services are pushed, the workflow MUST fire a repository_dispatch event to the platform repo using the PLATFORM_REPO_TOKEN secret. Event type: app-image-built. Payload must include the SHA and the list of services that were built — not all 8, only the ones that changed.
- The PLATFORM_REPO_TOKEN secret is a GitHub PAT with write access to the platform repo — students create this manually before running the pipeline.
```

---

## What Claude Code Will Generate

**From Prompt 1:**
- `terraform/modules/github-oidc/main.tf` — OIDC provider, IAM role with ECR permissions
- `.github/workflows/update-image-tags.yml` — updates helm-values on repository_dispatch
- `docs/rollback-runbook.md`

**From Prompt 2:**
- `../spring-petclinic-microservices/.github/workflows/build-push.yml`
- Optional reusable workflow files

---

## After the Prompts

1. Apply the OIDC Terraform module:
   ```bash
   cd terraform/environments/dev && terraform apply
   ```
2. Add `PLATFORM_REPO_TOKEN` secret to the app repo on GitHub
3. Push a small change to the app repo main branch
4. Watch the build-push workflow trigger in GitHub Actions
5. Verify the image appears in ECR
6. Watch update-image-tags trigger in the platform repo
7. Check `helm-values/{service}.yaml` — image.tag updated to the new SHA
