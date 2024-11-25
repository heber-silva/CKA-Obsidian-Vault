FEATURE STATE: `Kubernetes v1.18 [stable]`

The following volume plugins support raw block volumes, including dynamic provisioning where applicable:

- CSI
- FC (Fibre Channel)
- iSCSI
- Local volume
- OpenStack Cinder
- RBD (deprecated)
- RBD (Ceph Block Device; deprecated)
- VsphereVolume

### [PersistentVolume](PersistentVolume.md) using a Raw Block Volume[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volume-using-a-raw-block-volume)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```

### [PersistentVolumeClaim](PersistentVolumeClaim.md) requesting a Raw Block Volume[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volume-claim-requesting-a-raw-block-volume)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```

### Pod specification adding Raw Block Device path in container[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#pod-specification-adding-raw-block-device-path-in-container)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```

#### Note:

When adding a raw block device for a Pod, you specify the device path in the container instead of a mount path.

### Binding Block Volumes[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#binding-block-volumes)

If a user requests a raw block volume by indicating this using the `volumeMode` field in the PersistentVolumeClaim spec, the binding rules differ slightly from previous releases that didn't consider this mode as part of the spec. Listed is a table of possible combinations the user and admin might specify for requesting a raw block device. The table indicates if the volume will be bound or not given the combinations: Volume binding matrix for statically provisioned volumes:

|PV volumeMode|PVC volumeMode|Result|
|---|---|---|
|unspecified|unspecified|BIND|
|unspecified|Block|NO BIND|
|unspecified|Filesystem|BIND|
|Block|unspecified|NO BIND|
|Block|Block|BIND|
|Block|Filesystem|NO BIND|
|Filesystem|Filesystem|BIND|
|Filesystem|Block|NO BIND|
|Filesystem|unspecified|BIND|

#### Note:

Only statically provisioned volumes are supported for alpha release. Administrators should take care to consider these values when working with raw block devices.