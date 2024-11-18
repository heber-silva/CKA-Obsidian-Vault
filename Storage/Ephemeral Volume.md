Some applications need additional storage but don't care whether that data is stored persistently across restarts. For example, caching services are often limited by memory size and can move infrequently used data into storage that is slower than memory with little impact on overall performance.

Other applications expect some read-only input data to be present in files, like configuration data or secret keys.

[_Ephemeral volumes_](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/) are designed for these use cases. Because volumes follow the Pod's lifetime and get created and deleted along with the Pod, Pods can be stopped and restarted without being limited to where some persistent volume is available.

Ephemeral volumes are specified _inline_ in the Pod spec, which simplifies application deployment and management.

### Types of ephemeral volumes[](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#types-of-ephemeral-volumes)

Kubernetes supports several different kinds of ephemeral volumes for different purposes:

- [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir): empty at Pod startup, with storage coming locally from the kubelet base directory (usually the root disk) or RAM
- [configMap](https://kubernetes.io/docs/concepts/storage/volumes/#configmap), [downwardAPI](https://kubernetes.io/docs/concepts/storage/volumes/#downwardapi), [secret](https://kubernetes.io/docs/concepts/storage/volumes/#secret): inject different kinds of Kubernetes data into a Pod
- [image](https://kubernetes.io/docs/concepts/storage/volumes/#image): allows mounting container image files or artifacts, directly to a Pod.
- [CSI ephemeral volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volumes): similar to the previous volume kinds, but provided by special [CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) drivers which specifically [support this feature](https://kubernetes-csi.github.io/docs/ephemeral-local-volumes.html)
- [generic ephemeral volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#generic-ephemeral-volumes), which can be provided by all storage drivers that also support persistent volumes

`emptyDir`, `configMap`, `downwardAPI`, `secret` are provided as [local ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage). They are managed by kubelet on each node.

CSI ephemeral volumes _must_ be provided by third-party CSI storage drivers.

Generic ephemeral volumes _can_ be provided by third-party CSI storage drivers, but also by any other storage driver that supports dynamic provisioning. Some CSI drivers are written specifically for CSI ephemeral volumes and do not support dynamic provisioning: those then cannot be used for generic ephemeral volumes.

The advantage of using third-party drivers is that they can offer functionality that Kubernetes itself does not support, for example storage with different performance characteristics than the disk that is managed by kubelet, or injecting different data.