#prova 
Technology for packaging an application along with its runtime dependencies.

This page will discuss containers and container images, as well as their use in operations and solution development.

The word _container_ is an overloaded term. Whenever you use the word, check whether your audience uses the same definition.

Each container that you run is repeatable; the standardization from having dependencies included means that you get the same behavior wherever you run it.

Containers decouple applications from the underlying host infrastructure. This makes deployment easier in different cloud or OS environments.

Each [Node](../Kubernetes%20Achitecture/Node.md) in a Kubernetes cluster runs the containers that form the [Pod](../Workloads/Pod.md)s assigned to that node. Containers in a Pod are co-located and co-scheduled to run on the same node.

## Container images[](https://kubernetes.io/docs/concepts/containers/#container-images)

A container [Image](Image.md) is a ready-to-run software package containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings.

Containers are intended to be stateless and [immutable](https://glossary.cncf.io/immutable-infrastructure/): you should not change the code of a container that is already running. If you have a containerized application and want to make changes, the correct process is to build a new image that includes the change, then recreate the container to start from the updated image.

## Container runtimes[](https://kubernetes.io/docs/concepts/containers/#container-runtimes)

A fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.

Kubernetes supports container runtimes such as [containerd](https://containerd.io/docs/), [](https://cri-o.io/#what-is-cri-o), and any other implementation of the [Kubernetes CRI (Container Runtime Interface)](Container%20Runtime%20Interface)).

Usually, you can allow your cluster to pick the default container runtime for a Pod. If you need to use more than one container runtime in your cluster, you can specify the [RuntimeClass](https://kubernetes.io/docs/concepts/containers/runtime-class/) for a Pod to make sure that Kubernetes runs those containers using a particular container runtime.

You can also use RuntimeClass to run different Pods with the same container runtime but with different settings.

---

##### [Container Environment](Container%20Environment.md)

##### [Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)