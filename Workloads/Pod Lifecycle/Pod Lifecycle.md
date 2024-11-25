Like individual application containers, [Pod](../Pod.md) are considered to be relatively ephemeral (rather than durable) entities. Pods are created, assigned a unique ID ([](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#uids)), and scheduled to run on nodes where they remain until termination (according to restart policy) or deletion. If a [Node](../../Kubernetes%20Achitecture/Node.md) dies, the Pods running on (or scheduled to run on) that node are [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection). The control plane marks the Pods for removal after a timeout period.

## Pod lifetime[](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-lifetime)

Whilst a [Pod](../Pod.md) is running, the kubelet is able to restart containers to handle some kind of faults. Within a Pod, Kubernetes tracks different container [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-states) and determines what action to take to make the Pod healthy again.

In the Kubernetes API, Pods have both a specification and an actual status. The status for a Pod object consists of a set of [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions). You can also inject [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate) into the condition data for a Pod, if that is useful to your application.

Pods are only [scheduled](https://kubernetes.io/docs/concepts/scheduling-eviction/) once in their lifetime; assigning a Pod to a specific node is called _binding_, and the process of selecting which node to use is called _scheduling_. Once a Pod has been scheduled and is bound to a node, Kubernetes tries to run that Pod on the node. The Pod runs on that node until it stops, or until the Pod is [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination); if Kubernetes isn't able start the Pod on the selected node (for example, if the node crashes before the Pod starts), then that particular Pod never starts.

You can use [Pod Scheduling Readiness](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/) to delay scheduling for a Pod until all its _scheduling gates_ are removed. For example, you might want to define a set of Pods but only trigger scheduling once all the Pods have been created.

### Pods and fault recovery[](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-fault-recovery)

If one of the containers in the Pod fails, then Kubernetes may try to restart that specific container. Read [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-restarts) to learn more.

Pods can however fail in a way that the cluster cannot recover from, and in that case Kubernetes does not attempt to heal the Pod further; instead, Kubernetes deletes the Pod and relies on other components to provide automatic healing.

If a Pod is scheduled to a [Node](../../Kubernetes%20Achitecture/Node.md) and that node then fails, the Pod is treated as unhealthy and Kubernetes eventually deletes the Pod. A Pod won't survive an [eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/) due to a lack of resources or Node maintenance.

Kubernetes uses a higher-level abstraction, called a [Controller](../../Kubernetes%20Achitecture/Controller.md), that handles the work of managing the relatively disposable Pod instances.

A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead, that Pod can be replaced by a new, near-identical Pod. If you make a replacement Pod, it can even have same name (as in `.metadata.name`) that the old Pod had, but the replacement would have a different `.metadata.uid` from the old Pod.

Kubernetes does not guarantee that a replacement for an existing Pod would be scheduled to the same node as the old Pod that was being replaced.

### Associated lifetimes[](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#associated-lifetimes)

When something is said to have the same lifetime as a Pod, such as a [volume](https://kubernetes.io/docs/concepts/storage/volumes/), that means that the thing exists as long as that specific Pod (with that exact UID) exists. If that Pod is deleted for any reason, and even if an identical replacement is created, the related thing (a volume, in this example) is also destroyed and created anew.

![c24fe148fb7c359eb7952b15c2729fdb_MD5](../../Images/c24fe148fb7c359eb7952b15c2729fdb_MD5.svg)

#### Figure 1.

A multi-container Pod that contains a file puller [sidecar](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) and a web server. The Pod uses an [](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) for shared storage between the containers.