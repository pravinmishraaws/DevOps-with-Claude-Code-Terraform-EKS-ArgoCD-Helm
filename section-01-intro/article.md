# Section 1 — Introduction

## What This Course Covers

You are a DevOps/Cloud Engineer hired to take the Spring Petclinic Microservices application — 8 Spring Boot services, containerized, developed over 2 years — to **production on AWS**. A Project Manager creates Jira tasks with priorities. You pick tasks and deliver using Claude Code as your primary tool.

**You write zero code.** Claude Code writes all the Terraform, Helm charts, Kubernetes manifests, and CI/CD pipelines. Your job is to:
1. **Architect** — decide what to build, in what order, and why
2. **Prompt** — give Claude Code clear, specific instructions
3. **Review** — validate every output before it touches real infrastructure

---

## The Application

Spring Petclinic — 8 microservices:

| Service | Port | Role |
|---------|------|------|
| Config Server | 8888 | Pulls config from Git. Must start first. |
| Discovery Server | 8761 | Eureka service registry. Must start second. |
| API Gateway | 8080 | Routes all external traffic. Serves frontend. |
| Customers Service | 8081 | Owners and pets data (MySQL) |
| Visits Service | 8082 | Visit records (MySQL) |
| Vets Service | 8083 | Veterinarians data (MySQL) |
| GenAI Service | 8084 | AI chatbot via Spring AI (needs OPENAI_API_KEY) |
| Admin Server | 9090 | Spring Boot Admin monitoring |

GitHub: https://github.com/spring-petclinic/spring-petclinic-microservices

---

## The Infrastructure Stack

| Layer | Tool |
|-------|------|
| Cloud | AWS eu-central-1 |
| IaC | Terraform >= 1.6 |
| State | S3 + DynamoDB |
| Compute | Amazon EKS (t4g.small Graviton nodes) |
| Registry | Amazon ECR (8 repos) |
| Database | Amazon RDS MySQL (db.t4g.micro free tier) |
| DNS | Route 53 + ACM |
| Secrets | AWS Secrets Manager + External Secrets Operator |
| Ingress | AWS ALB Ingress Controller |
| Packaging | Helm |
| GitOps | ArgoCD |
| CI | GitHub Actions |
| Observability | Prometheus, Grafana, Loki, FluentBit, Zipkin |
| Autoscaling | Karpenter |

---

## Two Repos

```
~/spring-petclinic/
├── spring-petclinic-microservices/   # Application — read-only reference
└── petclinic-platform/               # Infrastructure — where you build everything
```

All infrastructure code goes into `petclinic-platform/`. You never modify the application.

---

## Cost

**Target: under $10 for the entire course.**

- EKS control plane: $0.10/hr — the only unavoidable cost. Run `terraform destroy` after each session.
- EKS nodes: t4g.small — free trial until Dec 2026
- RDS: db.t4g.micro — free tier 12 months
- No NAT Gateway — saves ~$35/month

---

## Setup Checklist

- [ ] AWS account created
- [ ] AWS CLI installed and configured (`aws configure`)
- [ ] Docker Desktop installed
- [ ] Terraform >= 1.6 installed
- [ ] kubectl installed
- [ ] Claude Code CLI installed
- [ ] `spring-petclinic-microservices` repo cloned (fork it first)
- [ ] `petclinic-platform` repo cloned

---

## External Links

- [Install Claude Code](https://claude.ai/download)
- [Install Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Install Terraform](https://developer.hashicorp.com/terraform/install)
- [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [spring-petclinic-microservices on GitHub](https://github.com/spring-petclinic/spring-petclinic-microservices)
