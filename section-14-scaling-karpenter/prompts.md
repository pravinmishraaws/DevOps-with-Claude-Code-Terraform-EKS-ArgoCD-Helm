# Section 14 — Prompts

Run both prompts from inside `petclinic-platform/` in Claude Code.

> **Note:** Two prompts — Terraform first, then Kubernetes resources.

---

## Prompt 1 — Karpenter Terraform Module & AWS Budget

```
Read the following stories from docs/jira-backlog.md:
- PETPLAT-73: Install Karpenter on EKS (IAM role, SQS, EventBridge rules, instance profile)
- PETPLAT-75: Create CloudWatch budget alerts ($100/env, 50%/80%/100% thresholds, email)

Read the technical spec at docs/technical-spec.md, specifically:
- Karpenter Node Autoscaling section (IAM permissions, SQS queue, EventBridge rules, instance profile)
- Scaling and Cost section (budget alert thresholds, email notification variable)
- IRSA Roles section (petclinic-{env}-karpenter-role)
- Module: karpenter section (terraform/modules/karpenter/ inputs and outputs)

Build the following:

1. terraform/modules/karpenter/
   - main.tf: IAM role for Karpenter controller (IRSA with OIDC trust policy), IAM policy with EC2/EKS/IAM/SQS/pricing permissions, IAM instance profile for Karpenter-launched nodes, SQS interruption queue (20-minute visibility timeout), EventBridge rules for 4 events: spot interruption, rebalance recommendation, instance state change, scheduled change
   - variables.tf: cluster_name, oidc_provider_arn, oidc_provider_url, node_role_arn, environment, project, tags
   - outputs.tf: karpenter_role_arn, karpenter_queue_name, karpenter_instance_profile_name

2. Wire the karpenter module into terraform/environments/dev/main.tf and terraform/environments/prod/main.tf (same inputs as eks module outputs)

3. Add AWS Budget resource to terraform/environments/dev/main.tf and terraform/environments/prod/main.tf:
   - Monthly budget: $100
   - Alert thresholds: 50%, 80%, 100% of actual spend
   - Email notification to var.budget_alert_email
   - Add budget_alert_email variable to variables.tf in both environments

Enforcement rules:
- Karpenter IRSA trust policy MUST use the OIDC provider from the EKS module output — not a hardcoded value. Use var.oidc_provider_arn and var.oidc_provider_url.
- The SQS queue MUST have a resource policy allowing EventBridge to publish to it. Without this policy, the interruption events arrive at EventBridge but never reach Karpenter — spot terminations are not handled gracefully.
- IAM policy for Karpenter MUST include iam:PassRole scoped to the node instance profile ARN only — not iam:PassRole on "*". Scoping prevents Karpenter from being used to escalate privileges.
- The karpenter instance profile name MUST follow the pattern petclinic-{env}-karpenter-node-profile — this exact name is referenced in the EC2NodeClass CRD we build next.
- Budget resource uses aws_budgets_budget, not aws_ce_cost_category. Notification type must be ACTUAL (not FORECASTED) so alerts fire on real spend.

Then run terraform validate on both environments.
```

---

## Prompt 2 — Karpenter Kubernetes Resources & Metrics Server

```
Read the following stories from docs/jira-backlog.md:
- PETPLAT-72: Install Metrics Server
- PETPLAT-73: Install Karpenter on EKS (Helm install, NodePool, EC2NodeClass)
- PETPLAT-74: Configure Karpenter NodePool for Spot in dev

Read the technical spec at docs/technical-spec.md, specifically:
- Karpenter Node Autoscaling section (NodePool YAML, EC2NodeClass YAML, Helm chart details)
- Scaling and Cost section (instance types, capacity type, consolidation policy)

Build the following:

1. k8s/base/karpenter/metrics-server.yaml
   - Metrics Server deployment from official manifest: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   - This is a reference manifest — download and save the contents to this file

2. k8s/base/karpenter/karpenter-install.yaml
   - Namespace: kube-system
   - Karpenter Helm install via Helm command (not a raw manifest) — document the command:
     helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
       --version 1.0.0 \
       --namespace kube-system \
       --set settings.clusterName=$(terraform output -raw cluster_name) \
       --set settings.interruptionQueue=$(terraform output -raw karpenter_queue_name) \
       --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=$(terraform output -raw karpenter_role_arn)
   - Save the full install command in the file as a comment block with the actual values from terraform outputs

3. k8s/base/karpenter/nodepool.yaml
   - NodePool CRD (karpenter.sh/v1) with:
     - ARM64 requirement (kubernetes.io/arch: arm64)
     - Instance families: t4g.small, t4g.medium
     - Capacity type: on-demand (not spot — Graviton free trial active, note in comment)
     - Resource limits: CPU 8, memory 32Gi
     - Consolidation policy: WhenUnderutilized, consolidateAfter 30s
   - EC2NodeClass CRD (karpenter.k8s.aws/v1) with:
     - AMI family: AL2023
     - Subnet selector: karpenter.sh/discovery: petclinic-{env}
     - Security group selector: karpenter.sh/discovery: petclinic-{env}
     - Instance profile: petclinic-{env}-karpenter-node-profile

4. k8s/base/karpenter/nodepool-spot-dev.yaml
   - A SEPARATE NodePool override file for dev spot configuration (PETPLAT-74)
   - Same as nodepool.yaml but capacity type includes: ["spot", "on-demand"]
   - Comment at top: "Apply this file AFTER the Graviton free trial expires to enable Spot savings"

Enforcement rules:
- The NodePool resource limits (CPU 8, memory 32Gi) are REQUIRED. Without limits, Karpenter has no upper bound and could provision dozens of nodes on a single large workload — unbounded cost.
- EC2NodeClass subnet and security group selectors MUST use the karpenter.sh/discovery tag. These tags must be set on VPC subnets and security groups in Terraform — if they are missing, Karpenter cannot find the network resources and node provisioning fails silently.
- The instance profile name in EC2NodeClass MUST match exactly: petclinic-{env}-karpenter-node-profile. This name is created by the Terraform karpenter module. A mismatch means the node launches but cannot join the cluster — the kubelet cannot authenticate.
- nodepool-spot-dev.yaml is separate from nodepool.yaml intentionally. Students apply the spot file explicitly when they are ready. Do not merge them.

Then run kubectl apply --dry-run=client on metrics-server.yaml and nodepool.yaml.
```

---

## After the Prompts

1. Apply Terraform (Prompt 1):
   ```bash
   cd terraform/environments/dev && terraform apply
   ```
2. Install Metrics Server (Prompt 2):
   ```bash
   kubectl apply -f k8s/base/karpenter/metrics-server.yaml
   ```
3. Install Karpenter (use the command from karpenter-install.yaml):
   ```bash
   helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
     --version 1.0.0 --namespace kube-system \
     --set settings.clusterName=$(cd terraform/environments/dev && terraform output -raw cluster_name) \
     ...
   ```
4. Apply the NodePool:
   ```bash
   kubectl apply -f k8s/base/karpenter/nodepool.yaml
   kubectl get nodepools
   ```
5. Test autoscaling:
   ```bash
   kubectl scale deployment api-gateway --replicas=10 -n petclinic-dev
   kubectl get nodes -w
   ```
