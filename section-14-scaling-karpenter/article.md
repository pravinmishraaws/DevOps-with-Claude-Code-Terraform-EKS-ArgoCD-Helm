# Section 14 — Scaling & Cost (Karpenter)

## What You Build

Karpenter node autoscaling, Metrics Server, and AWS Budget alerts:

| Component | What it does |
|-----------|-------------|
| Karpenter Terraform Module | IAM role, SQS interruption queue, EventBridge rules, instance profile |
| Karpenter Helm install | Karpenter controller running on EKS |
| NodePool CRD | Defines what nodes Karpenter can provision |
| EC2NodeClass CRD | Defines the EC2 configuration for those nodes |
| Metrics Server | Required for HPA (Horizontal Pod Autoscaler) |
| AWS Budget | $100/env budget with alerts at 50%, 80%, 100% |

---

## Karpenter vs Cluster Autoscaler

| | Cluster Autoscaler | Karpenter |
|-|--------------------|-----------|
| Scaling speed | ~3-5 min | ~30-60 sec |
| Node selection | Fixed node groups | Picks optimal instance from a pool |
| Spot support | Manual configuration | Built-in, with diversification |
| Consolidation | Limited | Automatic pod binpacking |

Karpenter watches for unschedulable pods and provisions exactly the right node size. It also consolidates — if pods can fit on fewer nodes, it removes the empty ones.

---

## Spot Instances

This course uses on-demand instances (the Graviton free trial covers them). After the free trial expires, switch to Spot to save ~70%:

```yaml
# nodepool-spot-dev.yaml — apply this after the free trial expires
spec:
  template:
    spec:
      requirements:
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
```

The repo includes a separate `nodepool-spot-dev.yaml` for this. Apply it deliberately — not automatically.

---

## Resource Limits on NodePool

The NodePool has hard limits to prevent runaway costs:

```yaml
limits:
  cpu: "8"
  memory: 32Gi
```

Without limits, Karpenter has no upper bound. A single misconfigured workload could trigger dozens of node provisions — unbounded cost. Always set limits in production.

---

## Karpenter Discovery Tags

Karpenter finds subnets and security groups using tags, not IDs. The VPC module (Section 3) must tag subnets and security groups with:

```
karpenter.sh/discovery = petclinic-{env}
```

Without these tags, Karpenter cannot find network resources and node provisioning fails silently.

---

## The Instance Profile Name Contract

The EC2NodeClass references the instance profile by name:
```yaml
spec:
  instanceProfile: petclinic-dev-karpenter-node-profile
```

This exact name is created by the Terraform karpenter module. If there's a mismatch, the node launches but cannot join the cluster — the kubelet cannot authenticate.

---

## Jira Stories

| Story | What it creates |
|-------|----------------|
| PETPLAT-72 | Metrics Server |
| PETPLAT-73 (Terraform) | Karpenter IAM, SQS, EventBridge, instance profile |
| PETPLAT-73 (K8s) | Karpenter Helm install, NodePool, EC2NodeClass |
| PETPLAT-74 | NodePool Spot override for dev (separate file) |
| PETPLAT-75 | AWS Budget alerts |

---

## After This Section

- Karpenter running in `kube-system`
- `kubectl get nodepools` shows the NodePool
- Scale a deployment to trigger node provisioning:
  ```bash
  kubectl scale deployment api-gateway --replicas=10 -n petclinic-dev
  kubectl get nodes -w   # watch a new node appear in ~60 sec
  ```
- AWS Budget alert configured — you'll get an email if spend hits 50%

---

## External Links

- [Karpenter Documentation](https://karpenter.sh/docs/)
- [Karpenter NodePool Reference](https://karpenter.sh/docs/concepts/nodepools/)
- [Karpenter EC2NodeClass Reference](https://karpenter.sh/docs/concepts/nodeclasses/)
- [Karpenter Best Practices](https://aws.github.io/aws-eks-best-practices/karpenter/)
- [Kubernetes HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Spot Instance Pricing](https://aws.amazon.com/ec2/spot/pricing/)
