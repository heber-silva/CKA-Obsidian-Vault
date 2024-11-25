[Storage capacity](https://kubernetes.io/docs/concepts/storage/storage-capacity/) is limited and may vary depending on the node on which a pod runs: network-attached storage might not be accessible by all nodes, or storage is local to a node to begin with.

FEATURE STATE: `Kubernetes v1.24 [stable]`

This page describes how Kubernetes keeps track of storage capacity and how the scheduler uses that information to [schedule Pods](https://kubernetes.io/docs/concepts/scheduling-eviction/) onto nodes that have access to enough storage capacity for the remaining missing volumes. Without storage capacity tracking, the scheduler may choose a node that doesn't have enough capacity to provision a volume and multiple scheduling retries will be needed.

## Before you begin[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#before-you-begin)

Kubernetes v1.31 includes cluster-level API support for storage capacity tracking. To use this you must also be using a CSI driver that supports capacity tracking. Consult the documentation for the CSI drivers that you use to find out whether this support is available and, if so, how to use it. If you are not running Kubernetes v1.31, check the documentation for that version of Kubernetes.

## API[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#api)

There are two API extensions for this feature:

- [CSIStorageCapacity](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/csi-storage-capacity-v1/) objects: these get produced by a CSI driver in the namespace where the driver is installed. Each object contains capacity information for one storage class and defines which nodes have access to that storage.
- [The `CSIDriverSpec.StorageCapacity` field](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/csi-driver-v1/#CSIDriverSpec): when set to `true`, the Kubernetes scheduler will consider storage capacity for volumes that use the CSI driver.

## Scheduling[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#scheduling)

Storage capacity information is used by the Kubernetes scheduler if:

- a [Pod](Pod.md) uses a [Volume](volume.md) that has not been created yet,
- that volume uses a [StorageClass](StorageClass.md) which references a CSI driver and uses `WaitForFirstConsumer` [volume binding mode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode), and
- the `CSIDriver` object for the driver has `StorageCapacity` set to true.

In that case, the scheduler only considers nodes for the Pod which have enough storage available to them. This check is very simplistic and only compares the size of the volume against the capacity listed in `CSIStorageCapacity` objects with a topology that includes the node.

For volumes with `Immediate` volume binding mode, the storage driver decides where to create the volume, independently of Pods that will use the volume. The scheduler then schedules Pods onto nodes where the volume is available after the volume has been created.

For [CSI ephemeral volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volumes), scheduling always happens without considering storage capacity. This is based on the assumption that this volume type is only used by special CSI drivers which are local to a node and do not need significant resources there.

## Rescheduling[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#rescheduling)

When a node has been selected for a Pod with `WaitForFirstConsumer` volumes, that decision is still tentative. The next step is that the CSI storage driver gets asked to create the volume with a hint that the volume is supposed to be available on the selected node.

Because Kubernetes might have chosen a node based on out-dated capacity information, it is possible that the volume cannot really be created. The node selection is then reset and the Kubernetes scheduler tries again to find a node for the Pod.

## Limitations[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#limitations)

Storage capacity tracking increases the chance that scheduling works on the first try, but cannot guarantee this because the scheduler has to decide based on potentially out-dated information. Usually, the same retry mechanism as for scheduling without any storage capacity information handles scheduling failures.

One situation where scheduling can fail permanently is when a Pod uses multiple volumes: one volume might have been created already in a topology segment which then does not have enough capacity left for another volume. Manual intervention is necessary to recover from this, for example by increasing capacity or deleting the volume that was already created.

## What's next[](https://kubernetes.io/docs/concepts/storage/storage-capacity/#what-s-next)

- For more information on the design, see the [Storage Capacity Constraints for Pod Scheduling KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-storage/1472-storage-capacity-tracking/README.md).