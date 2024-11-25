A [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) runs a group of [Pod](../Pod.md)s, and maintains a sticky identity for each of those Pods. This is useful for managing applications that need persistent storage or a stable, unique network identity.

StatefulSet is the workload API object used to manage stateful applications.

Manages the [Deployment](Deployment.md) and scaling of a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), _and provides guarantees about the ordering and uniqueness_ of these Pods.

Like a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.

## Using StatefulSets[](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#using-statefulsets)

StatefulSets are valuable for applications that require one or more of the following.

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

In the above, stable is synonymous with persistence across Pod (re)scheduling. If an application doesn't require any stable identifiers or ordered deployment, deletion, or scaling, you should deploy your application using a workload object that provides a set of stateless replicas. [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) may be better suited to your stateless needs.

## Limitations[](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#limitations)

- The storage for a given Pod must either be provisioned by a [PersistentVolume Provisioner](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) ([examples here](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md)) based on the requested _storage class_, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will _not_ delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
- StatefulSets currently require a [](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
- When using [](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#rolling-updates) with the default [](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies) (`OrderedReady`), it's possible to get into a broken state that requires [](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#forced-rollback).