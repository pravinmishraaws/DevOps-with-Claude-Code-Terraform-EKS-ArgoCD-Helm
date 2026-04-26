# Section 8 — Secrets Management

## What You Build

A complete secrets pipeline from AWS Secrets Manager to Kubernetes pods:

| Component | Role |
|-----------|------|
| AWS Secrets Manager | Stores the actual secret values |
| External Secrets Operator | Runs on EKS, watches ExternalSecret CRs |
| ExternalSecret CR | Tells ESO which AWS secret to sync and where |
| Kubernetes Secret | Created automatically by ESO, mounted in pods |
| IRSA Role | Allows ESO pods to read from Secrets Manager |

---

## The Secrets Pipeline

```
AWS Secrets Manager
  petclinic/dev/rds          → { username, password, host, port }
  petclinic/dev/openai       → { api_key }

External Secrets Operator
  (runs as a pod on EKS, reads AWS secrets via IRSA)

ExternalSecret CRs
  (Kubernetes CRDs that declare what to sync)

Kubernetes Secrets
  petclinic-db-credentials   → used by customers, visits, vets services
  openai-api-key             → used by genai-service
```

---

## Why Not Just Use Kubernetes Secrets Directly?

Kubernetes Secrets are base64 encoded — not encrypted. Anyone with access to the cluster can read them. More importantly, if you store them in Git (which you'd need to for GitOps), they're effectively plaintext in your repository.

AWS Secrets Manager:
- Encrypts values at rest (KMS)
- Full audit log (CloudTrail)
- Supports rotation
- IAM-based access control

External Secrets Operator bridges the two worlds — credentials live in AWS, pods get them as native Kubernetes Secrets without anyone storing secrets in Git.

---

## The Three Secrets in This Course

| Secret Path | Contents | Used By |
|------------|----------|---------|
| `petclinic/{env}/rds` | username, password, host, port, dbname | customers, visits, vets services |
| `petclinic/{env}/openai` | OPENAI_API_KEY | genai-service |
| `petclinic/{env}/config-server` | Git credentials (if private repo) | config-server |

The RDS secret was created in Section 6. This section creates the remaining secrets and sets up ESO to sync all of them.

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform33 | Secrets Manager resources for non-RDS secrets |
| PetclinicPlatform34 | Install External Secrets Operator on EKS |
| PetclinicPlatform35 | ExternalSecret CR for RDS credentials |
| PetclinicPlatform36 | ExternalSecret CR for OpenAI API key |
| PetclinicPlatform37 | IRSA role for External Secrets Operator |

---

## After This Section

- ESO running as a deployment on EKS
- `kubectl get externalsecret -n petclinic-dev` shows secrets synced
- `kubectl get secret petclinic-db-credentials -n petclinic-dev` exists

---

## External Links

- [External Secrets Operator Documentation](https://external-secrets.io/latest/)
- [ESO — AWS Secrets Manager Provider](https://external-secrets.io/latest/provider/aws-secrets-manager/)
- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [ExternalSecret CRD Reference](https://external-secrets.io/latest/api/externalsecret/)
- [IRSA — IAM Roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
