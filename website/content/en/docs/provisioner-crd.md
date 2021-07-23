---
title: "Provisioner CRD"
linkTitle: "Provisioner CRD"
weight: 40
date: 2017-01-05
---

## Example Provisioner Resource

```bash
apiVersion: karpenter.sh/v1alpha3
kind: Provisioner
metadata:
  name: default
spec:
  # Provisioned nodes connect to this cluster
  cluster:
    name: "${CLUSTER_NAME}"
    caBundle: "${CLUSTER_CA_BUNDLE}"
    endpoint: "${CLUSTER_ENDPOINT}"

  # If nil, the feature is disabled, nodes will never expire
  ttlSecondsUntilExpired: 2592000 # 30 Days = 60 * 60 * 24 * 30 Seconds;

  # If nil, the feature is disabled, nodes will never scale down due to low utilization
  ttlSecondsAfterEmpty: 30

  # Provisioned nodes will have these taints
  taints:
    - key: example.com/special-taint
      effect: NoSchedule

  # Provisioned nodes will have these labels
  labels:
    ##### AWS Specific #####
    # Constrain node launch template, default="bottlerocket"
    node.k8s.aws/launch-template-id: "bottlerocket-qwertyuiop"
    # Constrain node launch template, default="$LATEST"
    node.k8s.aws/launch-template-version: "my-special-version"
    # Constrain node capacity type, default="on-demand"
    node.k8s.aws/capacity-type: "spot"
```
## Alternative Provisioners

Karpenter supports multiple provisioners with different defaults. By default, pods will provision using the default provisioner. However, you can select an alternative provisioner using a node selector. This enables you to setup multiple provisioning profiles in a cluster.

*provisioner.yaml*

apiVersion: karpenter.sh/v1alpha3
kind: Provisioner
metadata:
  name: alternative
spec:
  cluster:
    endpoint: ...
    name: ...

*pod.yaml*

spec:
  template:
    spec:
      nodeSelector:
        karpenter.sh/provisioner-name: alternative

