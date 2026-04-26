# Section 3 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build the VPC Module

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform6: Create VPC module
- PetclinicPlatform8: Create baseline security groups
- PetclinicPlatform9: Wire VPC module into dev
- PetclinicPlatform10: Wire VPC module into prod

Read the technical spec at docs/technical-spec.md, specifically:
- VPC Network Design section
- Security Groups section
- Terraform Modules section

Build everything these stories require. Follow the acceptance criteria exactly.
After you are done, run terraform validate in both dev and prod.
```

---

## What Claude Code Will Generate

- `terraform/modules/vpc/main.tf` — VPC, subnets, IGW, route table, route table associations
- `terraform/modules/vpc/variables.tf` — vpc_cidr, subnet CIDRs, environment, tags
- `terraform/modules/vpc/outputs.tf` — vpc_id, subnet_ids, all 4 security group IDs
- `terraform/modules/vpc/versions.tf` — provider constraints
- 4 security groups with separate rule resources (EKS nodes, RDS, ALB, EKS cluster)
- Updates to `terraform/environments/dev/main.tf` — calls vpc module
- Updates to `terraform/environments/prod/main.tf` — calls vpc module

---

## After the Prompt

1. Review `terraform/modules/vpc/main.tf` — check CIDRs match the spec
2. Check security group rules — RDS only reachable from EKS node SG, ALB only on 80/443
3. Check EKS subnet tags are present
4. Run `terraform plan` in dev to see what will be created
5. Run `terraform apply` in dev — first real AWS infrastructure
6. Verify in AWS console: VPC, subnets, IGW, security groups
