The [CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) [Volume Cloning](https://kubernetes.io/docs/concepts/storage/volume-pvc-datasource/) feature adds support for specifying existing [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)s in the `dataSource` field to indicate a user would like to clone a [Volume](https://kubernetes.io/docs/concepts/storage/volumes/).

A Clone is defined as a duplicate of an existing Kubernetes Volume that can be consumed as any standard Volume would be. The only difference is that upon provisioning, rather than creating a "new" empty Volume, the back end device creates an exact duplicate of the specified Volume.

The implementation of cloning, from the perspective of the Kubernetes API, adds the ability to specify an existing PVC as a dataSource during new PVC creation. The source PVC must be bound and available (not in use).

Users need to be aware of the following when using this feature:

- Cloning support (`VolumePVCDataSource`) is only available for CSI drivers.
- Cloning support is only available for dynamic provisioners.
- CSI drivers may or may not have implemented the volume cloning functionality.
- You can only clone a PVC when it exists in the same namespace as the destination PVC (source and destination must be in the same namespace).
- Cloning is supported with a different Storage Class.
    - Destination volume can be the same or a different storage class as the source.
    - Default storage class can be used and storageClassName omitted in the spec.
- Cloning can only be performed between two volumes that use the same VolumeMode setting (if you request a block mode volume, the source MUST also be block mode)