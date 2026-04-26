# Section 6 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build the RDS Module

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform22: Create RDS module
- PetclinicPlatform23: Create database credentials in Secrets Manager
- PetclinicPlatform24: Create database initialization strategy
- PetclinicPlatform25: Wire RDS module into dev environment
- PetclinicPlatform26: Deploy and verify dev RDS
- PetclinicPlatform27: Wire RDS module into prod environment

Read the technical spec at docs/technical-spec.md, specifically:
- RDS Database section
- Secrets Management section
- Security Groups section (RDS security group)
- Terraform Modules section (RDS module)

Build everything these stories require. Follow the acceptance criteria exactly.
After you are done, run terraform validate in both dev and prod.
Then use the terraform-reviewer agent to review the module for security and best practices.
Fix any issues the reviewer finds, then validate again.
```

---

## What Claude Code Will Generate

- `terraform/modules/rds/main.tf` — RDS instance, subnet group, parameter group, Secrets Manager secret
- `terraform/modules/rds/variables.tf` — instance class, storage, db name, credentials config
- `terraform/modules/rds/outputs.tf` — db_endpoint, db_port, secret_arn
- `terraform/modules/rds/versions.tf`
- Updates to both environment `main.tf` files — wires RDS module with VPC outputs (subnet IDs, RDS SG)

---

## After the Prompt

1. Review the RDS module — check instance class is `db.t4g.micro`, storage 20 GB, single-AZ
2. Check the Secrets Manager secret format — should include host, port, username, password, dbname
3. Run `terraform plan` in dev — RDS creation takes ~5 minutes
4. Run `terraform apply` in dev
5. Verify in AWS console: RDS → Databases — instance shows `Available`
6. Check Secrets Manager — secret `petclinic/dev/rds` exists with credentials
