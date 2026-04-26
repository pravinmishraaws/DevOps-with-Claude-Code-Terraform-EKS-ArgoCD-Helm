# Section 7 — DNS & Ingress

## What You Build

Public HTTPS access to the application via a real domain name:

| Resource | Details |
|----------|---------|
| Route 53 Hosted Zone | Looked up via data source (not created — already exists after domain purchase) |
| ACM Certificate | TLS certificate for your domain, DNS validated |
| AWS Load Balancer Controller | Helm chart on EKS — watches Ingress resources, creates ALBs |
| Ingress Manifest | Routes `yourdomain.com` → API Gateway service on port 8080 |
| DNS Record | `A` record pointing to ALB hostname |
| IRSA Role | Allows LB Controller to create/manage ALBs in EC2 |

---

## You Need a Domain Name

You need a registered domain to complete this section. Buy one in Route 53 — $12/year for a `.com`.

**Route 53 automatically creates a hosted zone when you register a domain.** Do not create a second hosted zone in Terraform — if you do, the ACM DNS validation records go into the wrong zone and the certificate never validates.

The Terraform code uses a **data source** to look up the existing zone:
```hcl
data "aws_route53_zone" "main" {
  name = var.domain_name
}
```

---

## How the Traffic Flows

```
Internet
  ↓
Route 53 (yourdomain.com → ALB IP)
  ↓
ALB (created by Load Balancer Controller)
  ↓ HTTPS terminated here
Ingress resource (Kubernetes)
  ↓
API Gateway Service (port 8080)
  ↓
API Gateway Pod
```

The AWS Load Balancer Controller watches for Ingress resources with the annotation `kubernetes.io/ingress.class: alb`. When it sees one, it creates an Application Load Balancer in AWS automatically.

---

## AWS Load Balancer Controller — Version Gotcha

The controller has two version numbers that use different schemes:
- **Application version**: `v2.8.1` — used in the CRD download URL
- **Helm chart version**: `1.8.1` — used in `helm install`

If you use the Helm chart version in the CRD URL, GitHub returns a 404. The prompt handles this explicitly.

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform28 | DNS module — Route 53 hosted zone data source, ACM cert |
| PetclinicPlatform29 | AWS Load Balancer Controller Helm install script |
| PetclinicPlatform30 | Ingress manifest for API Gateway |
| PetclinicPlatform31 | DNS A record pointing to ALB |
| PetclinicPlatform32 | Wire DNS module into dev environment |

---

## After This Section

- `https://yourdomain.com` loads the Petclinic app
- TLS certificate is issued and valid
- ALB created and healthy

---

## External Links

- [Buy a domain in Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html)
- [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [AWS Load Balancer Controller — Install Guide](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Terraform aws_route53_zone data source](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/route53_zone)
