# Section 4 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build the EKS Module

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform12: Create EKS module — cluster and IAM roles
- PetclinicPlatform13: Add managed node group
- PetclinicPlatform14: Create kubectl access configuration
- PetclinicPlatform15: Wire EKS module into dev
- PetclinicPlatform16: Deploy and verify dev EKS cluster
- PetclinicPlatform17: Wire EKS module into prod
- PetclinicPlatform84: Manage EKS add-ons via Terraform

Read the technical spec at docs/technical-spec.md, specifically:
- EKS Cluster section
- IRSA Roles section

Build everything these stories require. Follow the acceptance criteria exactly.
After you are done, run terraform validate in both dev and prod.
Then use the terraform-reviewer agent to review the module for security and best practices.
Fix any issues the reviewer finds, then validate again.
```

---

## What Claude Code Will Generate

- `terraform/modules/eks/main.tf` — cluster, IAM roles, OIDC provider, node group, add-ons
- `terraform/modules/eks/variables.tf` — vpc_id, subnet_ids, security group IDs, cluster name, node config
- `terraform/modules/eks/outputs.tf` — cluster_name, cluster_endpoint, oidc_provider_arn, node_role_arn
- `terraform/modules/eks/versions.tf` — provider constraints
- Updates to both environment `main.tf` files — wires EKS module with VPC outputs

---

## After the Prompt

1. Review the IAM roles — check the policies match the spec
2. Review the node group — instance type `t4g.small`, min/max counts
3. Run `terraform plan` in dev
4. Run `terraform apply` in dev — cluster will take ~12 minutes to create
5. Configure kubectl:
   ```bash
   aws eks update-kubeconfig --region eu-central-1 --name petclinic-dev
   ```
6. Verify nodes are ready:
   ```bash
   kubectl get nodes
   ```
