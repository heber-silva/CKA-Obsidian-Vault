[This page](https://kubernetes.io/docs/concepts/storage/storage-limits/) describes the maximum number of volumes that can be attached to a [Node](../Kubernetes%20Achitecture/Node.md) for various cloud providers.

Cloud providers like Google, Amazon, and Microsoft typically have a limit on how many volumes can be attached to a Node. It is important for Kubernetes to respect those limits. Otherwise, Pods scheduled on a Node could get stuck waiting for volumes to attach.

## Kubernetes default limits[](https://kubernetes.io/docs/concepts/storage/storage-limits/#kubernetes-default-limits)

The Kubernetes scheduler has default limits on the number of volumes that can be attached to a Node:

|Cloud service|Maximum volumes per Node|
|---|---|
|[Amazon Elastic Block Store (EBS)](EBS))|39|
|[Google Persistent Disk](https://cloud.google.com/persistent-disk/)|16|
|[Microsoft Azure Disk Storage](https://azure.microsoft.com/en-us/services/storage/main-disks/)|16|

## Custom limits[](https://kubernetes.io/docs/concepts/storage/storage-limits/#custom-limits)

You can change these limits by setting the value of the `KUBE_MAX_PD_VOLS` environment variable, and then starting the scheduler. CSI drivers might have a different procedure, see their documentation on how to customize their limits.

Use caution if you set a limit that is higher than the default limit. Consult the cloud provider's documentation to make sure that Nodes can actually support the limit you set.

The limit applies to the entire cluster, so it affects all Nodes.

## Dynamic volume limits[](https://kubernetes.io/docs/concepts/storage/storage-limits/#dynamic-volume-limits)

FEATURE STATE: `Kubernetes v1.17 [stable]`

Dynamic volume limits are supported for following volume types.

- Amazon EBS
- Google Persistent Disk
- Azure Disk
- CSI

For volumes managed by in-tree volume plugins, Kubernetes automatically determines the Node type and enforces the appropriate maximum number of volumes for the node. For example:

- On [Google Compute Engine](https://cloud.google.com/compute/), up to 127 volumes can be attached to a node, [](https://cloud.google.com/compute/docs/disks/#pdnumberlimits).
    
- For Amazon EBS disks on M5,C5,R5,T3 and Z1D instance types, Kubernetes allows only 25 volumes to be attached to a Node. For other instance types on [Amazon Elastic Compute Cloud (EC2)](EC2)), Kubernetes allows 39 volumes to be attached to a Node.
    
- On Azure, up to 64 disks can be attached to a node, depending on the node type. For more details, refer to [Sizes for virtual machines in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes).
    
- If a CSI storage driver advertises a maximum number of volumes for a Node (using `NodeGetInfo`), the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) honors that limit. Refer to the [](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodegetinfo) for details.
    
- For volumes managed by in-tree plugins that have been migrated to a CSI driver, the maximum number of volumes will be the one reported by the CSI driver.