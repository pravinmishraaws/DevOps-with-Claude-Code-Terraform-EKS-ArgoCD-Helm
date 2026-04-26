# Section 4 — EKS Cluster

## What You Build

A fully managed Kubernetes cluster on AWS:

| Resource | Details |
|----------|---------|
| EKS Cluster | Kubernetes, public endpoint, eu-central-1 |
| Cluster IAM Role | `AmazonEKSClusterPolicy` |
| Managed Node Group | 2x `t4g.small` (ARM/Graviton), min 2 / max 4 |
| Node IAM Role | `AmazonEKSWorkerNodePolicy`, `AmazonEKS_CNI_Policy`, `AmazonEC2ContainerRegistryReadOnly` |
| OIDC Provider | Enables IRSA (IAM Roles for Service Accounts) |
| Add-on: CoreDNS | In-cluster DNS |
| Add-on: kube-proxy | Node networking |
| Add-on: vpc-cni | Pod IPs from VPC subnet range |
| Add-on: EBS CSI Driver | Persistent volumes (needed for Prometheus/Grafana) |
| kubectl config | `aws eks update-kubeconfig` command |

---

## Key Concepts

### IRSA — IAM Roles for Service Accounts
Pods need AWS permissions (to pull images from ECR, read secrets from Secrets Manager) without hardcoded credentials. IRSA solves this:

1. EKS creates an OIDC endpoint
2. An IAM role trusts that OIDC endpoint
3. A Kubernetes ServiceAccount is annotated with the role ARN
4. Pods using that ServiceAccount automatically get AWS credentials via environment variables

The OIDC provider you create here is used in every section that needs AWS permissions from a pod.

### Graviton Nodes (ARM64)
`t4g.small` = 2 vCPU, 2 GiB RAM. ARM-based Graviton processors. On the free trial until Dec 2026.

**Important:** Any Docker image you deploy must be built for `linux/arm64`. Deploying an `amd64` image to a Graviton node causes `exec format error` — the pod starts but the process crashes immediately. You handle this in Section 5 (ECR) and Section 11 (CI pipeline).

### Add-ons with Pinned Versions
All 4 add-ons use pinned versions in Terraform (`addon_version = "v1.x.x-eksbuild.x"`). Not `latest`. Upgrades are deliberate.

The EBS CSI Driver is the only add-on that needs its own IRSA role — because it creates and attaches EBS volumes, which requires EC2 permissions.

---

## Jira Stories (Epic E-3)

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform12 | EKS cluster, cluster IAM role, OIDC provider |
| PetclinicPlatform13 | Managed node group, node IAM role |
| PetclinicPlatform14 | kubectl access configuration |
| PetclinicPlatform15 | Wire EKS module into dev |
| PetclinicPlatform16 | Deploy and verify dev EKS cluster |
| PetclinicPlatform17 | Wire EKS module into prod |
| PetclinicPlatform84 | Managed add-ons (CoreDNS, kube-proxy, vpc-cni, EBS CSI) |

---

## After This Section

- EKS cluster running in AWS
- Two Graviton worker nodes in the node group
- `kubectl get nodes` shows nodes in `Ready` state
- OIDC provider created — future sections use it for IRSA

---

## Cost Reminder

EKS control plane costs **$0.10/hr**. Run `terraform destroy` at the end of each session to avoid unexpected charges.

---

## External Links

- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [EKS Managed Node Groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
- [EKS Managed Add-ons](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html)
- [EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [Graviton Free Trial](https://aws.amazon.com/ec2/graviton/)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
