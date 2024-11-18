#prova 
[Doc](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)

Each PV contains a spec and status, which is the specification and status of the volume. The name of a PersistentVolume object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

#### Note:

Helper programs relating to the volume type may be required for consumption of a PersistentVolume within a cluster. In this example, the PersistentVolume is of type NFS and the helper program /sbin/mount.nfs is required to support the mounting of NFS filesystems.

### Capacity[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#capacity)

Generally, a PV will have a specific storage capacity. This is set using the PV's `capacity` attribute which is a [Quantity](https://kubernetes.io/docs/reference/glossary/?all=true#term-quantity) value.

Currently, storage size is the only resource that can be set or requested. Future attributes may include IOPS, throughput, etc.

### Volume Mode[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-mode)

FEATURE STATE: `Kubernetes v1.18 [stable]`

Kubernetes supports two `volumeModes` of PersistentVolumes: `Filesystem` and `Block`.

`volumeMode` is an optional API parameter. `Filesystem` is the default mode used when `volumeMode` parameter is omitted.

A volume with `volumeMode: Filesystem` is _mounted_ into Pods into a directory. If the volume is backed by a block device and the device is empty, Kubernetes creates a filesystem on the device before mounting it for the first time.

You can set the value of `volumeMode` to `Block` to use a volume as a raw block device. Such volume is presented into a Pod as a block device, without any filesystem on it. This mode is useful to provide a Pod the fastest possible way to access a volume, without any filesystem layer between the Pod and the volume. On the other hand, the application running in the Pod must know how to handle a raw block device. See [Raw Block Volume Support](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#raw-block-volume-support) for an example on how to use a volume with `volumeMode: Block` in a Pod.

### Access Modes[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume. For example, NFS can support multiple read/write clients, but a specific NFS PV might be exported on the server as read-only. Each PV gets its own set of access modes describing that specific PV's capabilities.

The access modes are:

`ReadWriteOnce`

the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node. For single pod access, please see ReadWriteOncePod.

`ReadOnlyMany`

the volume can be mounted as read-only by many nodes.

`ReadWriteMany`

the volume can be mounted as read-write by many nodes.

`ReadWriteOncePod`

FEATURE STATE: `Kubernetes v1.29 [stable]`

the volume can be mounted as read-write by a single Pod. Use ReadWriteOncePod access mode if you want to ensure that only one pod across the whole cluster can read that PVC or write to it.

#### Note:

The `ReadWriteOncePod` access mode is only supported for [CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) volumes and Kubernetes version 1.22+. To use this feature you will need to update the following [CSI sidecars](https://kubernetes-csi.github.io/docs/sidecar-containers.html) to these versions or greater:

- [csi-provisioner:v3.0.0+](https://github.com/kubernetes-csi/external-provisioner/releases/tag/v3.0.0)
- [csi-attacher:v3.3.0+](https://github.com/kubernetes-csi/external-attacher/releases/tag/v3.3.0)
- [csi-resizer:v1.3.0+](https://github.com/kubernetes-csi/external-resizer/releases/tag/v1.3.0)

In the CLI, the access modes are abbreviated to:

- RWO - ReadWriteOnce
- ROX - ReadOnlyMany
- RWX - ReadWriteMany
- RWOP - ReadWriteOncePod

#### Note:

Kubernetes uses volume access modes to match PersistentVolumeClaims and PersistentVolumes. In some cases, the volume access modes also constrain where the PersistentVolume can be mounted. Volume access modes do **not** enforce write protection once the storage has been mounted. Even if the access modes are specified as ReadWriteOnce, ReadOnlyMany, or ReadWriteMany, they don't set any constraints on the volume. For example, even if a PersistentVolume is created as ReadOnlyMany, it is no guarantee that it will be read-only. If the access modes are specified as ReadWriteOncePod, the volume is constrained and can be mounted on only a single Pod.

> **Important!** A volume can only be mounted using one access mode at a time, even if it supports many.

|Volume Plugin|ReadWriteOnce|ReadOnlyMany|ReadWriteMany|ReadWriteOncePod|
|---|---|---|---|---|
|AzureFile|✓|✓|✓|-|
|CephFS|✓|✓|✓|-|
|CSI|depends on the driver|depends on the driver|depends on the driver|depends on the driver|
|FC|✓|✓|-|-|
|FlexVolume|✓|✓|depends on the driver|-|
|HostPath|✓|-|-|-|
|iSCSI|✓|✓|-|-|
|NFS|✓|✓|✓|-|
|RBD|✓|✓|-|-|
|VsphereVolume|✓|-|- (works when Pods are collocated)|-|
|PortworxVolume|✓|-|✓|-|

### Class[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class)

A PV can have a class, which is specified by setting the `storageClassName` attribute to the name of a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/). A PV of a particular class can only be bound to PVCs requesting that class. A PV with no `storageClassName` has no class and can only be bound to PVCs that request no particular class.

In the past, the annotation `volume.beta.kubernetes.io/storage-class` was used instead of the `storageClassName` attribute. This annotation is still working; however, it will become fully deprecated in a future Kubernetes release.

### Reclaim Policy[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy)

Current reclaim policies are:

- Retain -- manual reclamation
- Recycle -- basic scrub (`rm -rf /thevolume/*`)
- Delete -- delete the volume

For Kubernetes 1.31, only `nfs` and `hostPath` volume types support recycling.

### Mount Options[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#mount-options)

A Kubernetes administrator can specify additional mount options for when a Persistent Volume is mounted on a node.

#### Note:

Not all Persistent Volume types support mount options.

The following volume types support mount options:

- `azureFile`
- `cephfs` (**deprecated** in v1.28)
- `cinder` (**deprecated** in v1.18)
- `iscsi`
- `nfs`
- `rbd` (**deprecated** in v1.28)
- `vsphereVolume`

Mount options are not validated. If a mount option is invalid, the mount fails.

In the past, the annotation `volume.beta.kubernetes.io/mount-options` was used instead of the `mountOptions` attribute. This annotation is still working; however, it will become fully deprecated in a future Kubernetes release.

### Node Affinity[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#node-affinity)

#### Note:

For most volume types, you do not need to set this field. You need to explicitly set this for [local](https://kubernetes.io/docs/concepts/storage/volumes/#local) volumes.

A PV can specify node affinity to define constraints that limit what nodes this volume can be accessed from. Pods that use a PV will only be scheduled to nodes that are selected by the node affinity. To specify node affinity, set `nodeAffinity` in the `.spec` of a PV. The [PersistentVolume](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeSpec) API reference has more details on this field.

### Phase[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#phase)

A PersistentVolume will be in one of the following phases:

`Available`

a free resource that is not yet bound to a claim

`Bound`

the volume is bound to a claim

`Released`

the claim has been deleted, but the associated storage resource is not yet reclaimed by the cluster

`Failed`

the volume has failed its (automated) reclamation

You can see the name of the PVC bound to the PV using `kubectl describe persistentvolume <name>`.

#### Phase transition timestamp[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#phase-transition-timestamp)

FEATURE STATE: `Kubernetes v1.31 [stable]` (enabled by default: true)

The `.status` field for a PersistentVolume can include an alpha `lastPhaseTransitionTime` field. This field records the timestamp of when the volume last transitioned its phase. For newly created volumes the phase is set to `Pending` and `lastPhaseTransitionTime` is set to the current time.

## Types of Persistent Volumes[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)

PersistentVolume types are implemented as plugins. Kubernetes currently supports the following plugins:

- [`csi`](https://kubernetes.io/docs/concepts/storage/volumes/#csi) - Container Storage Interface (CSI)
- [`fc`](https://kubernetes.io/docs/concepts/storage/volumes/#fc) - Fibre Channel (FC) storage
- [`hostPath`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) - HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using `local` volume instead)
- [`iscsi`](https://kubernetes.io/docs/concepts/storage/volumes/#iscsi) - iSCSI (SCSI over IP) storage
- [`local`](https://kubernetes.io/docs/concepts/storage/volumes/#local) - local storage devices mounted on nodes.
- [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) - Network File System (NFS) storage

The following types of PersistentVolume are deprecated but still available. If you are using these volume types except for `flexVolume`, `cephfs` and `rbd`, please install corresponding CSI drivers.

- [`awsElasticBlockStore`](https://kubernetes.io/docs/concepts/storage/volumes/#awselasticblockstore) - AWS Elastic Block Store (EBS) (**migration on by default** starting v1.23)
- [`azureDisk`](https://kubernetes.io/docs/concepts/storage/volumes/#azuredisk) - Azure Disk (**migration on by default** starting v1.23)
- [`azureFile`](https://kubernetes.io/docs/concepts/storage/volumes/#azurefile) - Azure File (**migration on by default** starting v1.24)
- [`cinder`](https://kubernetes.io/docs/concepts/storage/volumes/#cinder) - Cinder (OpenStack block storage) (**migration on by default** starting v1.21)
- [`flexVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#flexvolume) - FlexVolume (**deprecated** starting v1.23, no migration plan and no plan to remove support)
- [`gcePersistentDisk`](https://kubernetes.io/docs/concepts/storage/volumes/#gcePersistentDisk) - GCE Persistent Disk (**migration on by default** starting v1.23)
- [`portworxVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#portworxvolume) - Portworx volume (**migration on by default** starting v1.31)
- [`vsphereVolume`](https://kubernetes.io/docs/concepts/storage/volumes/#vspherevolume) - vSphere VMDK volume (**migration on by default** starting v1.25)

Older versions of Kubernetes also supported the following in-tree PersistentVolume types:

- [`cephfs`](https://kubernetes.io/docs/concepts/storage/volumes/#cephfs) (**not available** starting v1.31)
- `flocker` - Flocker storage. (**not available** starting v1.25)
- `photonPersistentDisk` - Photon controller persistent disk. (**not available** starting v1.15)
- `quobyte` - Quobyte volume. (**not available** starting v1.25)
- [`rbd`](https://kubernetes.io/docs/concepts/storage/volumes/#rbd) - Rados Block Device (RBD) volume (**not available** starting v1.31)
- `scaleIO` - ScaleIO volume. (**not available** starting v1.21)
- `storageos` - StorageOS volume. (**not available** starting v1.25)