# Resources & External Links

All tools, documentation, and references used throughout the course.

---

## Tools to Install

| Tool | Link | Notes |
|------|------|-------|
| Claude Code | https://claude.ai/download | CLI — install via npm or direct download |
| Docker Desktop | https://www.docker.com/products/docker-desktop/ | Mac / Windows / Linux |
| Terraform | https://developer.hashicorp.com/terraform/install | >= 1.6 required |
| AWS CLI v2 | https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html | Configure with `aws configure` |
| kubectl | https://kubernetes.io/docs/tasks/tools/ | Kubernetes CLI |
| Helm | https://helm.sh/docs/intro/install/ | >= 3.x |
| Git | https://git-scm.com/downloads | |
| VS Code | https://code.visualstudio.com/ | Recommended editor |

---

## AWS Documentation

### Core Services
- [VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [ECR User Guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [RDS MySQL Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
- [Secrets Manager Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [Route 53 Developer Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)

### EKS Add-ons & IAM
- [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [EKS Managed Add-ons](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html)
- [EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)

### Free Tier & Cost
- [AWS Free Tier](https://aws.amazon.com/free/)
- [EKS Pricing](https://aws.amazon.com/eks/pricing/) — $0.10/hr for control plane
- [Graviton Free Trial](https://aws.amazon.com/ec2/graviton/) — t4g.small free trial
- [RDS Free Tier](https://aws.amazon.com/rds/free/)

---

## Terraform

- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Language Reference](https://developer.hashicorp.com/terraform/language)
- [S3 Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/s3)
- [Terraform Modules](https://developer.hashicorp.com/terraform/language/modules)

---

## Kubernetes & Helm

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Helm Documentation](https://helm.sh/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/references/kustomize/)

---

## ArgoCD

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [ArgoCD Application CRD Reference](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml)
- [ArgoCD RBAC Configuration](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/)

---

## CI/CD — GitHub Actions

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [OIDC with AWS](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [Docker Buildx (multi-platform builds)](https://docs.docker.com/buildx/working-with-buildx/)
- [Trivy vulnerability scanner](https://trivy.dev/)

---

## Karpenter

- [Karpenter Documentation](https://karpenter.sh/docs/)
- [NodePool CRD Reference](https://karpenter.sh/docs/concepts/nodepools/)
- [EC2NodeClass Reference](https://karpenter.sh/docs/concepts/nodeclasses/)

---

## Observability

- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [FluentBit Documentation](https://docs.fluentbit.io/manual)
- [Zipkin Documentation](https://zipkin.io/)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Micrometer Prometheus](https://micrometer.io/docs/registry/prometheus)

---

## External Secrets Operator

- [External Secrets Operator Docs](https://external-secrets.io/latest/)
- [ExternalSecret CRD Reference](https://external-secrets.io/latest/api/externalsecret/)

---

## Application Repos

- [spring-petclinic-microservices](https://github.com/spring-petclinic/spring-petclinic-microservices) — Application code
- [spring-petclinic-microservices-config](https://github.com/spring-petclinic/spring-petclinic-microservices-config) — Config Server Git backend

---

## Claude Code

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Claude Code MCP Servers](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [Atlassian MCP (Jira)](https://developer.atlassian.com/cloud/jira/platform/mcp/)
