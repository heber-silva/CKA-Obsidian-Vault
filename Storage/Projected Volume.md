A [projected](https://kubernetes.io/docs/concepts/storage/projected-volumes/) volume maps several existing volume sources into the same directory.

Currently, the following types of volume sources can be projected:

- [Secret](Secret.md)
- [`downwardAPI`](https://kubernetes.io/docs/concepts/storage/volumes/#downwardapi)
- [ConfigMap](ConfigMap.md)
- [`serviceAccountToken`](https://kubernetes.io/docs/concepts/storage/projected-volumes/#serviceaccounttoken)
- [`clusterTrustBundle`](https://kubernetes.io/docs/concepts/storage/projected-volumes/#clustertrustbundle)

All sources are required to be in the same namespace as the Pod. For more details, see the [all-in-one volume](https://git.k8s.io/design-proposals-archive/node/all-in-one-volume.md) design document.