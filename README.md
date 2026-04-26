# DevOps with Claude Code — Terraform, EKS, ArgoCD & Helm

> **This is the companion repository for the Udemy course:**
> ### [DevOps with Claude Code — Terraform, EKS, ArgoCD & Helm](https://www.udemy.com/course/draft/7093081/?referralCode=1C5B734505D65A010FA3)
> Build a production-grade AWS infrastructure for 8 microservices using AI. Zero code written by you — Claude Code does it all.

---

Course companion repository. Every section has two files:
- `article.md` — what you're building, key concepts, and external links
- `prompts.md` — the exact Claude Code prompts to copy and paste

---

## What You'll Build

A production-grade AWS infrastructure for 8 Spring Boot microservices:

| Layer | Tools |
|-------|-------|
| Infrastructure | Terraform, VPC, IAM, Route 53 |
| Compute | Amazon EKS (Graviton t4g.small nodes) |
| Containers | Docker, Amazon ECR (8 private repos) |
| Packaging | Helm |
| GitOps / CD | ArgoCD |
| CI | GitHub Actions (OIDC, ARM64 builds) |
| Database | Amazon RDS MySQL (free tier) |
| Secrets | AWS Secrets Manager + External Secrets Operator |
| Observability | Prometheus, Grafana, Loki, FluentBit, Zipkin |
| Autoscaling | Karpenter, HPA |

**AWS target region:** `eu-central-1`  
**Target cost:** Under $10 for the entire course (EKS control plane hours only)

---

## The Application

Spring Petclinic Microservices — 8 services, Java 17, Spring Boot 4.x.

| Service | Port |
|---------|------|
| Config Server | 8888 |
| Discovery Server (Eureka) | 8761 |
| API Gateway | 8080 |
| Customers Service | 8081 |
| Visits Service | 8082 |
| Vets Service | 8083 |
| GenAI Service | 8084 |
| Admin Server | 9090 |

Application repo: [spring-petclinic-microservices](https://github.com/spring-petclinic/spring-petclinic-microservices)

---

## How to Use This Repo

1. Open the section folder for the lecture you're watching
2. Read `article.md` for context and links
3. Open `prompts.md`, copy the prompt, paste it into Claude Code
4. Review what Claude Code generates before running any `terraform apply`

---

## Sections

| # | Section | What You Build |
|---|---------|---------------|
| [01](./section-01-intro/) | Introduction | Course overview, tools, scenario |
| [02](./section-02-getting-started/) | Getting Started | Docker Compose, Claude Code setup, Foundation (S3, DynamoDB, Terraform backend) |
| [03](./section-03-networking-vpc/) | Networking — VPC | VPC, subnets, Internet Gateway, security groups |
| [04](./section-04-eks/) | EKS Cluster | EKS cluster, node groups, OIDC, IAM, add-ons |
| [05](./section-05-ecr/) | Container Registry (ECR) | 8 ECR repos, lifecycle policies, ARM64 image builds |
| [06](./section-06-rds/) | Database (RDS) | RDS MySQL, subnet group, Secrets Manager credentials |
| [07](./section-07-dns-ingress/) | DNS & Ingress | Route 53, ACM cert, ALB Ingress Controller |
| [08](./section-08-secrets/) | Secrets Management | External Secrets Operator, ExternalSecret CRs |
| [09](./section-09-k8s-manifests/) | Kubernetes Manifests | Base manifests + dev/prod overlays for all 8 services |
| [10](./section-10-helm-charts/) | Helm Charts | Generic Helm chart, per-service & per-env values |
| [11](./section-11-ci-pipeline/) | CI Pipeline | GitHub Actions, OIDC, ARM64 build, Trivy scan, image tag update |
| [12](./section-12-argocd/) | GitOps with ArgoCD | ArgoCD install, Application CRDs, auto-sync (dev) / manual (prod) |
| [13](./section-13-observability/) | Observability | Prometheus, Grafana, Loki, FluentBit, Zipkin, Alertmanager |
| [14](./section-14-scaling-karpenter/) | Scaling & Cost | Karpenter, NodePool, HPA, CloudWatch budget alerts |

---

## Two Repos You'll Work With

```
~/spring-petclinic/
├── spring-petclinic-microservices/   # Application code (read-only reference)
└── petclinic-platform/               # Infrastructure (where you build everything)
```

All Terraform, Helm charts, K8s manifests, and CI pipelines go into `petclinic-platform/`.  
You never modify the application code.

---

## Prerequisites

See [RESOURCES.md](./RESOURCES.md) for install links and documentation.

- AWS account with CLI configured (`aws configure`)
- Docker Desktop
- Terraform >= 1.6
- kubectl
- Claude Code CLI
- Git + GitHub account
