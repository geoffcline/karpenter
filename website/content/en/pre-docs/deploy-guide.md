---
title: "Deployment Guide"
linkTitle: "Deployment Guide"
weight: 50
---

# Resources Karpenter Expects/Creates

- Cluster on AWS
    - min version 1.16? 1.18?
    - must be EKS?
- IAM Roles (link examples)
    - `KarpenterNode` -- run containers, configure networking
    - `Karpenter` -- provision new instances
    - IRSA vs kube2iam?
- Service Accounts (cluster permissions?)
    - `Karpenter` -- associated with `Karpenter` IAM Role
- Tagged Subnets? (sample script)
    - does this make sense for non-EKS clusters?
- EKS? (done by service accounts and IAM?)
    - configure cluster to accept new instances as nodes 
    - must use eksctl? find cfn?
        - if you have cluster, do you need eksctl?
        - assuming IRSA setup, no
- Instance profile, has at least `KarpenterNode` role
- Launch template
    - will provide one by default


# Deploy Karpenter

- Helm Chart
    - or manually pull chart
    - pulls image from ECR
    - creates deployment
    - config options of helm chart?
- Provisioner, can use default


# Karpenter Changes Bx of Kubernetes?
- `k delete node`
- unusually high ratio of pods to per node daemonsets
- provisioning
    - watches for event
    - preregisters node -- is this unusual?
    - preassigns pods to expected node -- unusual?
    - do other tools interact with pending pods?
- deprovisioning
    - pod disruption budget?
    - finalizer on nodes?
    - expiration of nodes?
    - deletes empty nodes?
