---
title: "Amazon Web Services (AWS)"
linkTitle: "AWS"
weight: 10
---

## Impact of Labels on Provisioning on AWS 

Custom Labels

Karpenter enables customers to set custom labels nodes it launches. Additionally, pods may specify additional labels which will be respected and applied to nodes launched for those pods. 

### Well Known Label: Capacity Type

Karpenter supports specifying capacity type and defaults to on-demand.

*Set Default with provisioner.yaml*

spec:
  labels: 
    node.k8s.aws/capacity-type: spot

*Override with pod.yaml*

spec:
  template:
    spec:
      nodeSelector:
        node.k8s.aws/capacity-type: spot

### instance type

*Set Default with provisioner.yaml*

spec:
  instanceTypes:
    - m5.large

*Override with pod.yaml*

spec:
  template:
    spec:
      nodeSelector:
        node.k8s.aws/instance-type: m5.large

### GPU Example

Resources: Accelerators

Karpenter supports GPUs. To specify a specific GPU type, use the instance type well known label selector (see above)

*Override with pod.yaml*

spec:
  template:
    spec:
      containers:
      - resources:
          limits:
            nvidia.com/gpu: "1"


```go
const (
	NvidiaGPU = "nvidia.com/gpu"
	AMDGPU    = "amd.com/gpu"
	AWSNeuron = "aws.amazon.com/neuron"
)
```


### kubernetes.io/arch

Example: `kubernetes.io/arch=amd64`

```go
var (
	ArchitectureAmd64 = "amd64"
	ArchitectureArm64 = "arm64"
)
```

Used on: Node

The Kubelet populates this with `runtime.GOARCH` as defined by Go. This can be handy if you are mixing arm and x86 nodes.

ArchitectureAmd64 = "amd64"
	ArchitectureArm64 = "arm64"

kubernetes.io/os 
Example: kubernetes.io/os=linux

Used on: Node

The Kubelet populates this with runtime.GOOS as defined by Go. This can be handy if you are mixing operating systems in your cluster (for example: mixing Linux and Windows nodes).

## kubernetes.io/os

Example: `kubernetes.io/os=linux`

Used on: Node

```go
var (
	OperatingSystemLinux = "linux"
)
```

The Kubelet populates this with `runtime.GOOS` as defined by Go. This can be handy if you are mixing operating systems in your cluster (for example: mixing Linux and Windows nodes).

### topology.kubernetes.io/region {#topologykubernetesioregion}

Example:

`topology.kubernetes.io/region=us-east-1`

### topology.kubernetes.io/zone {#topologykubernetesiozone}

Example:

`topology.kubernetes.io/zone=us-east-1c`

Used on: Node

On Node: The `kubelet` or the external `cloud-controller-manager` populates this with the information as provided by the `cloudprovider`.  This will be set only if you are using a `cloudprovider`. However, you should consider setting this on nodes if it makes sense in your topology.

A zone represents a logical failure domain.  It is common for Kubernetes clusters to span multiple zones for increased availability.  While the exact definition of a zone is left to infrastructure implementations, common properties of a zone include very low network latency within a zone, no-cost network traffic within a zone, and failure independence from other zones.  For example, nodes within a zone might share a network switch, but nodes in different zones should not.

A region represents a larger domain, made up of one or more zones.  It is uncommon for Kubernetes clusters to span multiple regions,  While the exact definition of a zone or region is left to infrastructure implementations, common properties of a region include higher network latency between them than within them, non-zero cost for network traffic between them, and failure independence from other zones or regions.  For example, nodes within a region might share power infrastructure (e.g. a UPS or generator), but nodes in different regions typically would not.

Kubernetes makes a few assumptions about the structure of zones and regions:
1) regions and zones are hierarchical: zones are strict subsets of regions and no zone can be in 2 regions
2) zone names are unique across regions; for example region "africa-east-1" might be comprised of zones "africa-east-1a" and "africa-east-1b"

It should be safe to assume that topology labels do not change.  Even though labels are strictly mutable, consumers of them can assume that a given node is not going to be moved between zones without being destroyed and recreated.

Kubernetes can use this information in various ways.  For example, the scheduler automatically tries to spread the Pods in a ReplicaSet across nodes in a single-zone cluster (to reduce the impact of node failures, see [kubernetes.io/hostname](#kubernetesiohostname)). With multiple-zone clusters, this spreading behavior also applies to zones (to reduce the impact of zone failures). This is achieved via _SelectorSpreadPriority_.

_SelectorSpreadPriority_ is a best effort placement. If the zones in your cluster are heterogeneous (for example: different numbers of nodes, different types of nodes, or different pod resource requirements), this placement might prevent equal spreading of your Pods across zones. If desired, you can use homogenous zones (same number and types of nodes) to reduce the probability of unequal spreading.

The scheduler (through the _VolumeZonePredicate_ predicate) also will ensure that Pods, that claim a given volume, are only placed into the same zone as that volume. Volumes cannot be attached across zones. 


## How we work with EC2