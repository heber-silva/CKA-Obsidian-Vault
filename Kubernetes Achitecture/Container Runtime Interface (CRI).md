The CRI is a plugin interface which enables the kubelet to use a wide variety of container runtimes, without having a need to recompile the cluster components.

You need a working [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes) on each Node in your cluster, so that the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) can launch [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and their containers.

The Container Runtime Interface (CRI) is the main protocol for the communication between the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) and Container Runtime.

The Kubernetes Container Runtime Interface (CRI) defines the main [gRPC](https://grpc.io/) protocol for the communication between the [](https://kubernetes.io/docs/concepts/architecture/#node-components) [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) and [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes).

## The API[](https://kubernetes.io/docs/concepts/architecture/cri/#api)

FEATURE STATE: `Kubernetes v1.23 [stable]`

The kubelet acts as a client when connecting to the container runtime via gRPC. The runtime and image service endpoints have to be available in the container runtime, which can be configured separately within the kubelet by using the `--image-service-endpoint` [command line flags](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/).

For Kubernetes v1.31, the kubelet prefers to use CRI `v1`. If a container runtime does not support `v1` of the CRI, then the kubelet tries to negotiate any older supported version. The v1.31 kubelet can also negotiate CRI `v1alpha2`, but this version is considered as deprecated. If the kubelet cannot negotiate a supported CRI version, the kubelet gives up and doesn't register as a node.

## Upgrading[](https://kubernetes.io/docs/concepts/architecture/cri/#upgrading)

When upgrading Kubernetes, the kubelet tries to automatically select the latest CRI version on restart of the component. If that fails, then the fallback will take place as mentioned above. If a gRPC re-dial was required because the container runtime has been upgraded, then the container runtime must also support the initially selected version or the redial is expected to fail. This requires a restart of the kubelet.

## What's next [](https://kubernetes.io/docs/concepts/architecture/cri/#what-s-next)

- Learn more about the CRI [protocol definition](https://github.com/kubernetes/cri-api/blob/c75ef5b/pkg/apis/runtime/v1/api.proto)