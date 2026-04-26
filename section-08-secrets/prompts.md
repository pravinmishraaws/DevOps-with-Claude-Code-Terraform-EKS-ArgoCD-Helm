# Section 8 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build Secrets Management

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform33: Secrets Manager Terraform resources (non-RDS secrets)
- PetclinicPlatform34: Install External Secrets Operator on EKS
- PetclinicPlatform35: ExternalSecret for RDS credentials
- PetclinicPlatform36: ExternalSecret for OpenAI API key
- PetclinicPlatform37: IRSA role for External Secrets Operator

Read the technical spec at docs/technical-spec.md, specifically:
- Secrets Management section
- IRSA Roles section (ESO role)
- Terraform Modules section (Secrets module)

Build everything these stories require. Follow the acceptance criteria exactly.
The OpenAI API key value should be a sensitive Terraform variable — never hardcoded.
After you are done, run terraform validate in both dev and prod.
Then use the terraform-reviewer agent to review the Terraform module for security and best practices.
Then use the k8s-validator agent to validate the Kubernetes manifests and CRDs against project standards.
Fix any issues either reviewer finds, then validate again.
```

---

## What Claude Code Will Generate

- `terraform/modules/secrets/main.tf` — Secrets Manager secrets for OpenAI, Config Server; IRSA role for ESO
- `terraform/modules/secrets/variables.tf` — openai_api_key (sensitive), environment
- `terraform/modules/secrets/outputs.tf` — secret ARNs, ESO role ARN
- `k8s/base/external-secrets/eso-install.yaml` or install script — ESO Helm install
- `k8s/base/external-secrets/secret-store.yaml` — SecretStore CR pointing to AWS Secrets Manager
- `k8s/base/external-secrets/rds-external-secret.yaml` — ExternalSecret for DB credentials
- `k8s/base/external-secrets/openai-external-secret.yaml` — ExternalSecret for OpenAI key

---

## After the Prompt

1. Set the OpenAI API key variable (in `terraform.tfvars` or as env var):
   ```bash
   export TF_VAR_openai_api_key="sk-..."
   ```
2. Run `terraform apply` in dev
3. Install ESO:
   ```bash
   kubectl apply -f k8s/base/external-secrets/eso-install.yaml
   # or run the install script
   ```
4. Apply ExternalSecret CRs:
   ```bash
   kubectl apply -f k8s/base/external-secrets/
   ```
5. Verify secrets are synced:
   ```bash
   kubectl get externalsecret -n petclinic-dev
   kubectl get secret petclinic-db-credentials -n petclinic-dev
   ```
