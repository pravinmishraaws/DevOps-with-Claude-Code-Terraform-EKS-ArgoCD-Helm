# Section 2 — Prompts

Run these prompts from inside the `petclinic-platform/` directory in Claude Code.

---

## Prompt 1 — Epic 1: Foundation & Remote State

Use this prompt to build the full Terraform project foundation: directory structure, S3 state backend, DynamoDB lock table, backend configs for dev and prod, and the AWS provider.

**Via Jira (if you set it up):**

```
Read and implement the following Jira stories from the PETPLAT project:
- PetclinicPlatform1: Create Terraform project directory structure
- PetclinicPlatform2: Create S3 bucket and DynamoDB table for Terraform state
- PetclinicPlatform3: Configure Terraform backend for dev environment
- PetclinicPlatform4: Configure Terraform backend for prod environment
- PetclinicPlatform5: Configure AWS provider, versions, and common tags

Read docs/technical-spec.md for the exact values (region, bucket name, table name, naming conventions).

Build everything these stories require. Follow the acceptance criteria exactly.
After you are done, run terraform init in both dev and prod environments.
Update the Jira stories to Done when each one is complete.
```

**Via local backlog (recommended):**

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform1: Create Terraform project directory structure
- PetclinicPlatform2: Create S3 bucket and DynamoDB table for Terraform state
- PetclinicPlatform3: Configure Terraform backend for dev environment
- PetclinicPlatform4: Configure Terraform backend for prod environment
- PetclinicPlatform5: Configure AWS provider, versions, and common tags

Read docs/technical-spec.md for the exact values (region, bucket name, table name, naming conventions).

Build everything these stories require. Follow the acceptance criteria exactly.
After you are done, run terraform init in both dev and prod environments.
```

---

## What Claude Code Will Generate

- `terraform/environments/dev/` — root module for dev (main.tf, variables.tf, outputs.tf, backend.tf, versions.tf)
- `terraform/environments/prod/` — root module for prod (same files)
- `terraform/modules/` — empty module directories ready for later sections
- `scripts/bootstrap-state.sh` — one-time script to create S3 bucket and DynamoDB table
- `.gitignore` — ignores `.terraform/`, `*.tfstate`, `*.tfvars`

---

## After the Prompt

1. Review every generated file — check naming, region, bucket names against `docs/technical-spec.md`
2. Run the bootstrap script to create the state backend in AWS:
   ```bash
   chmod +x scripts/bootstrap-state.sh
   ./scripts/bootstrap-state.sh
   ```
3. Then run `terraform init` in both environments:
   ```bash
   cd terraform/environments/dev && terraform init
   cd terraform/environments/prod && terraform init
   ```
