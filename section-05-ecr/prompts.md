# Section 5 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build the ECR Module

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform18: Create ECR module
- PetclinicPlatform19: Add lifecycle policy and tag immutability configuration
- PetclinicPlatform20: Wire ECR module into dev environment and deploy
- PetclinicPlatform21: Create ECR login helper script
- PetclinicPlatform85: Build and push Docker images to ECR (initial)

Read the technical spec at docs/technical-spec.md, specifically:
- ECR Container Registry section
- Docker Build section
- Terraform Modules section (ECR module)

Build everything these stories require. Follow the acceptance criteria exactly.
The build-push script must use docker buildx with --platform linux/arm64 for Graviton nodes. Do not rely on Maven's buildDocker profile for image creation — use Maven to build the JARs, then docker buildx to build ARM64 images.
After you are done, run terraform validate in both dev and prod.
Then use the terraform-reviewer agent to review the module for security and best practices.
Fix any issues the reviewer finds, then validate again.
```

---

## What Claude Code Will Generate

- `terraform/modules/ecr/main.tf` — 8 ECR repos with lifecycle policies and scan-on-push
- `terraform/modules/ecr/variables.tf` — environment, repository names, tag mutability
- `terraform/modules/ecr/outputs.tf` — repository URLs for all 8 services
- `terraform/modules/ecr/versions.tf`
- `scripts/ecr-login.sh` — authenticates Docker to the private registry
- `scripts/build-push.sh` — builds ARM64 images from `../spring-petclinic-microservices/` and pushes to ECR
- Updates to both environment `main.tf` files

---

## After the Prompt

1. Check `terraform/modules/ecr/main.tf` — confirm tag mutability is `MUTABLE` for dev, `IMMUTABLE` for prod
2. Run `terraform apply` in dev
3. Authenticate Docker to ECR:
   ```bash
   chmod +x scripts/ecr-login.sh
   ./scripts/ecr-login.sh
   ```
4. Build and push images (takes several minutes — first build compiles Java):
   ```bash
   chmod +x scripts/build-push.sh
   ./scripts/build-push.sh
   ```
5. Verify in AWS console: ECR → Repositories — 8 repos with images
