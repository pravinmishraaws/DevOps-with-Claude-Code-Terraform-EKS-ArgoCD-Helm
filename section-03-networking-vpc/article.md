# Section 3 — Networking: VPC

## What You Build

A complete AWS VPC for the Petclinic platform:

| Resource | Details |
|----------|---------|
| VPC | CIDR `10.0.0.0/16`, DNS enabled |
| Public Subnet A | `10.0.1.0/24` — eu-central-1a |
| Public Subnet B | `10.0.2.0/24` — eu-central-1b |
| Internet Gateway | Attached to the VPC |
| Route Table | Public route `0.0.0.0/0` → IGW |
| Security Group: EKS nodes | Controls traffic between nodes and pods |
| Security Group: RDS | Port 3306, only from EKS node SG |
| Security Group: ALB | Port 80/443 from internet, to EKS nodes |
| Security Group: EKS cluster | Control plane to node communication |

**No NAT Gateway** — all-public subnet design. Saves ~$35/month.

---

## Why All-Public Subnets?

In production at scale, you'd use private subnets with NAT Gateways. But a NAT Gateway costs ~$35/month minimum — before data transfer. For a learning project, that's most of the AWS budget.

The trade-off: nodes and pods have public IPs, but security groups restrict all inbound traffic. No direct access unless you explicitly allow it.

---

## Key Terraform Concepts in This Section

**Module structure** — the VPC is a reusable module at `terraform/modules/vpc/`. Both dev and prod environments call the same module with different values.

```
terraform/
├── modules/
│   └── vpc/
│       ├── main.tf        # Resources
│       ├── variables.tf   # Inputs
│       ├── outputs.tf     # Outputs (vpc_id, subnet_ids, sg ids)
│       └── versions.tf    # Provider constraints
├── environments/
│   ├── dev/main.tf        # Calls vpc module with dev values
│   └── prod/main.tf       # Calls vpc module with prod values
```

**EKS subnet tags** — EKS requires specific tags on subnets so it can find them:
```
kubernetes.io/cluster/{cluster-name} = shared
kubernetes.io/role/elb                = 1
```

**Security group rules as separate resources** — avoids circular dependency issues between security groups that reference each other.

---

## Jira Stories (Epic E-2)

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform6 | VPC module — VPC, subnets, IGW, route table |
| PetclinicPlatform8 | Baseline security groups (4 groups) |
| PetclinicPlatform9 | Wire VPC module into dev environment |
| PetclinicPlatform10 | Wire VPC module into prod environment |

> Note: PetclinicPlatform7 (NAT Gateway) is intentionally skipped — no NAT in this design.

---

## After This Section

- VPC exists in AWS with two subnets across two AZs
- All 4 security groups created
- Both dev and prod environments pass `terraform validate`
- `terraform apply` succeeds — VPC is live in AWS

---

## External Links

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Terraform aws_vpc resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
- [Terraform aws_security_group resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)
- [EKS Network Requirements](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html)
- [EKS Subnet Tagging](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html#subnet-tagging-for-load-balancers)
