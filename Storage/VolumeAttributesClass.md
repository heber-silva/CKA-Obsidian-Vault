A [VolumeAttributesClass](https://kubernetes.io/docs/concepts/storage/volume-attributes-classes/) provides a way for administrators to describe the mutable "classes" of storage they offer. Different classes might map to different quality-of-service levels. Kubernetes itself is un-opinionated about what these classes represent.

This is a beta feature and disabled by default.

If you want to test the feature whilst it's beta, you need to enable the `VolumeAttributesClass` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) for the kube-controller-manager, kube-scheduler, and the kube-apiserver. You use the `--feature-gates` command line argument:

```
--feature-gates="...,VolumeAttributesClass=true"
```

You will also have to enable the `storage.k8s.io/v1beta1` API group through the `kube-apiserver` [runtime-config](https://kubernetes.io/docs/tasks/administer-cluster/enable-disable-api/). You use the following command line argument:

```
--runtime-config=storage.k8s.io/v1beta1=true
```

You can also only use VolumeAttributesClasses with storage backed by [](https://kubernetes.io/docs/concepts/storage/volumes/#csi), and only where the relevant CSI driver implements the `ModifyVolume` API.****