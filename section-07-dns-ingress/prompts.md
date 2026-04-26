# Section 7 — Prompts

Run from inside `petclinic-platform/` in Claude Code.

---

## Prompt — Build DNS & Ingress

```
Read the following stories from docs/jira-backlog.md:
- PetclinicPlatform28: Create DNS module — Route 53 hosted zone
- PetclinicPlatform29: Install AWS Load Balancer Controller on EKS
- PetclinicPlatform30: Create Ingress manifest for API Gateway
- PetclinicPlatform31: Create DNS record pointing to ALB
- PetclinicPlatform32: Wire DNS module into dev environment

Read the technical spec at docs/technical-spec.md, specifically:
- DNS and Ingress section
- Security Groups section (ALB security group)
- IRSA Roles section (LB controller role)
- Terraform Modules section (DNS module)

Build everything these stories require. Follow the acceptance criteria exactly.
This section produces both Terraform resources AND Kubernetes resources — make sure you generate both.
The AWS Load Balancer Controller should be installed via Helm chart (aws-load-balancer-controller from eks.amazonaws.com/charts). Generate a shell script at scripts/install-lb-controller.sh that handles the CRDs, Helm repo add, and helm install with the correct service account and IRSA role ARN. Make sure the script is executable (chmod +x). Important: when fetching CRDs from GitHub, use the controller application version tag (e.g. v2.8.1), not the Helm chart version (e.g. 1.8.1) — they use different numbering schemes and the wrong version will return a 404.
For the Route 53 hosted zone — use a data source to look up the existing zone, not a resource block. Route 53 automatically creates a hosted zone when you register a domain. Creating a second one with Terraform will put the ACM validation records in the wrong zone and the certificate will never be issued.
After you are done, run terraform validate in both dev and prod.
Then use the terraform-reviewer agent to review the Terraform module for security and best practices.
Then use the k8s-validator agent to validate the Kubernetes manifests against project standards.
Fix any issues either reviewer finds, then validate again.
```

---

## What Claude Code Will Generate

- `terraform/modules/dns/main.tf` — Route 53 zone data source, ACM cert, DNS validation record, A record
- `terraform/modules/dns/variables.tf` — domain_name, alb_hostname, environment
- `terraform/modules/dns/outputs.tf` — certificate_arn, zone_id
- IRSA role for the Load Balancer Controller (in EKS module or new IAM resources)
- `scripts/install-lb-controller.sh` — CRDs + Helm install for AWS LBC
- `k8s/base/ingress/ingress.yaml` — Ingress resource for API Gateway

---

## After the Prompt

1. Check the Route 53 zone — confirm it's a `data` source, not a `resource`
2. Check the ACM cert — domain matches your registered domain
3. Run `terraform apply` in dev
4. Install the Load Balancer Controller:
   ```bash
   chmod +x scripts/install-lb-controller.sh
   ./scripts/install-lb-controller.sh
   ```
5. Apply the Ingress manifest:
   ```bash
   kubectl apply -f k8s/base/ingress/ingress.yaml
   ```
6. Wait for the ALB to provision (~2 min):
   ```bash
   kubectl get ingress -n petclinic-dev
   ```
7. Test: `curl https://yourdomain.com`
