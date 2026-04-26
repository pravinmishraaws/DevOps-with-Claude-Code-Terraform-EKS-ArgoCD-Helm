# Section 5 — Container Registry (ECR)

## What You Build

8 private Amazon ECR repositories — one per microservice — plus a script to build and push ARM64 Docker images:

| Feature | Dev | Prod |
|---------|-----|------|
| Repositories | 8 (one per service) | 8 (one per service) |
| Tag immutability | Mutable (overwrite allowed) | Immutable (tags locked forever) |
| Scan on push | Enabled | Enabled |
| Lifecycle policy | Keep last 10 images | Keep last 10 images |

Repository naming: `petclinic-dev/{service-name}` and `petclinic-prod/{service-name}`

---

## The ARM64 Critical Detail

Your EKS nodes are `t4g.small` — ARM-based Graviton. Docker images must be built for `linux/arm64`.

If you build on a standard x86 laptop without specifying the platform, you get an `amd64` image. Push it to ECR, deploy to the cluster, and every pod crashes with `exec format error`. The image looks fine in ECR — the crash only happens when the pod starts.

The fix — always build with:
```bash
docker buildx build --platform linux/arm64 -t your-image:tag .
```

The base image `eclipse-temurin:17` supports multi-arch, so the same `Dockerfile` works for both architectures.

---

## Why Tag Immutability Matters

**Dev: mutable tags** — convenient during development. Push the same `latest` tag multiple times during iteration.

**Prod: immutable tags** — once you push `abc1234` (a commit SHA), that tag is locked. Nobody can overwrite it. If that tag is running in production, you know exactly which code is behind it. Essential for rollbacks — you can always go back to a specific tag with confidence.

---

## Jira Stories (Epic E-4)

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform18 | ECR module — 8 repos, scan-on-push, lifecycle policy |
| PetclinicPlatform19 | Tag immutability config (mutable dev, immutable prod) |
| PetclinicPlatform20 | Wire ECR module into dev and prod, deploy |
| PetclinicPlatform21 | ECR login helper script (`scripts/ecr-login.sh`) |
| PetclinicPlatform85 | Build all 8 ARM64 Docker images and push to ECR |

---

## After This Section

- 8 ECR repos exist in AWS
- All 8 ARM64 images pushed to `petclinic-dev/` repos
- ECR login script works: `./scripts/ecr-login.sh`
- Images can be pulled by EKS nodes (node IAM role has `AmazonEC2ContainerRegistryReadOnly`)

---

## External Links

- [Amazon ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [ECR Lifecycle Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
- [ECR Image Scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html)
- [Docker Buildx Multi-Platform](https://docs.docker.com/buildx/working-with-buildx/)
- [eclipse-temurin Docker Hub](https://hub.docker.com/_/eclipse-temurin)
- [Terraform aws_ecr_repository](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecr_repository)
