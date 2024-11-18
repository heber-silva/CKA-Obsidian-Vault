The cloud-controller-manager is a Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster

## [**cgroup v2**](https://kubernetes.io/docs/concepts/architecture/cgroups/)

cgroup v2 is the next version of the Linux `cgroup` API. cgroup v2 provides a unified control system with enhanced resource management capabilities.

## [**Container Runtime Interface (CRI)**](https://kubernetes.io/docs/concepts/architecture/cri/)

The CRI is a plugin interface which enables the kubelet to use a wide variety of container runtimes, without having a need to recompile the cluster components.

You need a working [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes) on each Node in your cluster, so that the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) can launch [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and their containers.

The Container Runtime Interface (CRI) is the main protocol for the communication between the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) and Container Runtime.

## [**Garbage Collection**](https://kubernetes.io/docs/concepts/architecture/garbage-collection/)

Garbage collection is a collective term for the various mechanisms Kubernetes uses to clean up cluster resources. This allows the clean up of resources like the following:

- [Terminated pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection)
- [Completed Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/)
- [Objects without owner references](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#owners-dependents)
- [Unused containers and container images](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#containers-images)
- [Dynamically provisioned PersistentVolumes with a StorageClass reclaim policy of Delete](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#delete)
- [Stale or expired CertificateSigningRequests (CSRs)](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#request-signing-process)
- [Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) deleted in the following scenarios:
    - On a cloud when the cluster uses a [cloud controller manager](https://kubernetes.io/docs/concepts/architecture/cloud-controller/)
    - On-premises when the cluster uses an addon similar to a cloud controller manager
- [Node Lease objects](https://kubernetes.io/docs/concepts/architecture/nodes/#heartbeats)

## [**Mixed Version Proxy**](https://kubernetes.io/docs/concepts/architecture/mixed-version-proxy/)

This enables cluster administrators to configure highly available clusters that can be upgraded more safely, by directing resource requests (made during the upgrade) to the correct kube-apiserver. That proxying prevents users from seeing unexpected 404 Not Found errors that stem from the upgrade process.