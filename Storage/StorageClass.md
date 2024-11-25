#prova 
This document describes the concept of a StorageClass in Kubernetes. Familiarity with [Volume](Volume.md) and [PersistentVolume](PersistentVolume.md) is suggested.

A [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) provides a way for administrators to describe the _classes_ of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent.

The Kubernetes concept of a storage class is similar to “profiles” in some other storage system designs.

## StorageClass objects[](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageclass-objects)

Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned to satisfy a PersistentVolumeClaim (PVC).

The name of a StorageClass object is significant, and is how users can request a particular class. Administrators set the name and other parameters of a class when first creating StorageClass objects.

As an administrator, you can specify a default StorageClass that applies to any PVCs that don't request a specific class. For more details, see the [](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).