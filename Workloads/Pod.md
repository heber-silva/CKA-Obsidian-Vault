#prova
_Pods_ are the smallest deployable units of computing that you can create and manage in Kubernetes.


A _Pod_ (as in a pod of whales or pea pod) is a group of one or more [Container](../Container/Container.md)s, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain [Init Container](Pods/Init%20Container.md)s that run during Pod startup. You can also inject [Ephemeral Container](Pods/Ephemeral%20Container.md)s for debugging a running Pod.

---
Kubernetes provides several built-in workload resources:

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) and [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) (replacing the legacy resource [](https://kubernetes.io/docs/reference/glossary/?all=true#term-replication-controller)). Deployment is a good fit for managing a stateless application workload on your cluster, where any Pod in the Deployment is interchangeable and can be replaced if needed.
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) lets you run one or more related Pods that do track state somehow. For example, if your workload records data persistently, you can run a StatefulSet that matches each Pod with a [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). Your code, running in the Pods for that StatefulSet, can replicate data to other Pods in the same StatefulSet to improve overall resilience.
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) defines Pods that provide facilities that are local to nodes. Every time you add a node to your cluster that matches the specification in a DaemonSet, the control plane schedules a Pod for that DaemonSet onto the new node. Each pod in a DaemonSet performs a job similar to a system daemon on a classic Unix / POSIX server. A DaemonSet might be fundamental to the operation of your cluster, such as a plugin to run [](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model), it might help you to manage the node, or it could provide optional behavior that enhances the container platform you are running.
- [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) provide different ways to define tasks that run to completion and then stop. You can use a [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) to define a task that runs to completion, just once. You can use a [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) to run the same Job multiple times according a schedule.
---

**Note:**
You need to install a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) into each node in the cluster so that Pods can run there.

The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a [Container](../Container/Container.md). Within a Pod's context, the individual applications may have further sub-isolations applied.

A Pod is similar to a set of containers with shared namespaces and shared filesystem volumes.

Pods in a Kubernetes cluster are used in two main ways:

- **Pods that run a single container**. The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.
    
- **Pods that run multiple containers that need to work together**. A Pod can encapsulate an application composed of [](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers) that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit.
    
    Grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled.
    
    You don't need to run multiple containers to provide replication (for resilience or capacity); if you need multiple replicas, see [Workload management](https://kubernetes.io/docs/concepts/workloads/controllers/).

## Using Pods[](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

The following is an example of a Pod which consists of a container running the image `nginx:1.14.2`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

To create the Pod shown above, run the following command:

```shell
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

Pods are generally not created directly and are created using workload resources. See [](https://kubernetes.io/docs/concepts/workloads/pods/#working-with-pods) for more information on how Pods are used with workload resources.

### Workload resources for managing pods[](https://kubernetes.io/docs/concepts/workloads/pods/#workload-resources-for-managing-pods)

Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/). If your Pods need to track state, consider the [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) resource.

Each Pod is meant to run a single instance of a given application. If you want to scale your application horizontally (to provide more overall resources by running more instances), you should use multiple Pods, one for each instance. In Kubernetes, this is typically referred to as _replication_. Replicated Pods are usually created and managed as a group by a workload resource and its [controller](https://kubernetes.io/docs/concepts/architecture/controller/).

See [](https://kubernetes.io/docs/concepts/workloads/pods/#pods-and-controllers) for more information on how Kubernetes uses workload resources, and their controllers, to implement application scaling and auto-healing.

Pods natively provide two kinds of shared resources for their constituent containers: [](https://kubernetes.io/docs/concepts/workloads/pods/#pod-networking) and [](https://kubernetes.io/docs/concepts/workloads/pods/#pod-storage).

## Working with Pods[](https://kubernetes.io/docs/concepts/workloads/pods/#working-with-pods)

You'll rarely create individual Pods directly in Kubernetes—even singleton Pods. This is because Pods are designed as relatively ephemeral, disposable entities. When a Pod gets created (directly by you, or indirectly by a [controller](https://kubernetes.io/docs/concepts/architecture/controller/)), the new Pod is scheduled to run on a [Node](https://kubernetes.io/docs/concepts/architecture/nodes/) in your cluster. The Pod remains on that node until the Pod finishes execution, the Pod object is deleted, the Pod is _evicted_ for lack of resources, or the node fails.

#### Note:

Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.

The name of a Pod must be a valid [](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostname. For best compatibility, the name should follow the more restrictive rules for a [](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names).

### Pod OS[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-os)

FEATURE STATE: `Kubernetes v1.25 [stable]`

You should set the `.spec.os.name` field to either `windows` or `linux` to indicate the OS on which you want the pod to run. These two are the only operating systems supported for now by Kubernetes. In the future, this list may be expanded.

In Kubernetes v1.31, the value of `.spec.os.name` does not affect how the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) picks a node for the Pod to run on. In any cluster where there is more than one operating system for running nodes, you should set the [](https://kubernetes.io/docs/reference/labels-annotations-taints/#kubernetes-io-os) label correctly on each node, and define pods with a `nodeSelector` based on the operating system label. The kube-scheduler assigns your pod to a node based on other criteria and may or may not succeed in picking a suitable node placement where the node OS is right for the containers in that Pod. The [Pod security standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) also use this field to avoid enforcing policies that aren't relevant to the operating system.

### Pods and controllers[](https://kubernetes.io/docs/concepts/workloads/pods/#pods-and-controllers)

You can use workload resources to create and manage multiple Pods for you. A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.

Here are some examples of workload resources that manage one or more Pods:

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset)

### Pod templates[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates)

Controllers for [workload](https://kubernetes.io/docs/concepts/workloads/) resources create Pods from a _pod template_ and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/), and [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

Each controller for a workload resource uses the `PodTemplate` inside the workload object to make actual Pods. The `PodTemplate` is part of the desired state of whatever workload resource you used to run your app.

When you create a Pod, you can include [environment variables](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) in the Pod template for the containers that run in the Pod.

The sample below is a manifest for a simple Job with a `template` that starts one container. The container in that Pod prints a message then pauses.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

Modifying the pod template or switching to a new pod template has no direct effect on the Pods that already exist. If you change the pod template for a workload resource, that resource needs to create replacement Pods that use the updated template.

For example, the StatefulSet controller ensures that the running Pods match the current pod template for each StatefulSet object. If you edit the StatefulSet to change its pod template, the StatefulSet starts to create new Pods based on the updated template. Eventually, all of the old Pods are replaced with new Pods, and the update is complete.

Each workload resource implements its own rules for handling changes to the Pod template. If you want to read more about StatefulSet specifically, read [](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets) in the StatefulSet Basics tutorial.

On Nodes, the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) does not directly observe or manage any of the details around pod templates and updates; those details are abstracted away. That abstraction and separation of concerns simplifies system semantics, and makes it feasible to extend the cluster's behavior without changing existing code.

## Pod update and replacement[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-update-and-replacement)

As mentioned in the previous section, when the Pod template for a workload resource is changed, the controller creates new Pods based on the updated template instead of updating or patching the existing Pods.

Kubernetes doesn't prevent you from managing Pods directly. It is possible to update some fields of a running Pod, in place. However, Pod update operations like [](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.31/#patch-pod-v1-core), and [](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.31/#replace-pod-v1-core) have some limitations:

- Most of the metadata about a Pod is immutable. For example, you cannot change the `namespace`, `name`, `uid`, or `creationTimestamp` fields; the `generation` field is unique. It only accepts updates that increment the field's current value.
    
- If the `metadata.deletionTimestamp` is set, no new entry can be added to the `metadata.finalizers` list.
    
- Pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations`. For `spec.tolerations`, you can only add new entries.
    
- When updating the `spec.activeDeadlineSeconds` field, two types of updates are allowed:
    
    1. setting the unassigned field to a positive number;
    2. updating the field from a positive number to a smaller, non-negative number.

## Resource sharing and communication[](https://kubernetes.io/docs/concepts/workloads/pods/#resource-sharing-and-communication)

Pods enable data sharing and communication among their constituent containers.

### Storage in Pods[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-storage)

A Pod can specify a set of shared storage [volumes](https://kubernetes.io/docs/concepts/storage/volumes/). All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted. See [Storage](https://kubernetes.io/docs/concepts/storage/) for more information on how Kubernetes implements shared storage and makes it available to Pods.

### Pod networking[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-networking)

Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and **only** then), the containers that belong to the Pod can communicate with one another using `localhost`. When containers in a Pod communicate with entities _outside the Pod_, they must coordinate how they use the shared network resources (such as ports). Within a Pod, containers share an IP address and port space, and can find each other via `localhost`. The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. Containers in different Pods have distinct IP addresses and can not communicate by OS-level IPC without special configuration. Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

Containers within the Pod see the system hostname as being the same as the configured `name` for the Pod. There's more about this in the [networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) section.

## Pod security settings[](https://kubernetes.io/docs/concepts/workloads/pods/#pod-security)

To set security constraints on Pods and containers, you use the `securityContext` field in the Pod specification. This field gives you granular control over what a Pod or individual containers can do. For example:

- Drop specific Linux capabilities to avoid the impact of a CVE.
- Force all processes in the Pod to run as a non-root user or as a specific user or group ID.
- Set a specific seccomp profile.
- Set Windows security options, such as whether containers run as HostProcess.

#### Caution:

You can also use the Pod securityContext to enable [](https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/#privileged-containers) in Linux containers. Privileged mode overrides many of the other security settings in the securityContext. Avoid using this setting unless you can't grant the equivalent permissions by using other fields in the securityContext. In Kubernetes 1.26 and later, you can run Windows containers in a similarly privileged mode by setting the `windowsOptions.hostProcess` flag on the security context of the Pod spec. For details and instructions, see [Create a Windows HostProcess Pod](https://kubernetes.io/docs/tasks/configure-pod-container/create-hostprocess-pod/).

- To learn about kernel-level security constraints that you can use, see [Linux kernel security constraints for Pods and containers](https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/).
- To learn more about the Pod security context, see [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

## Static Pods[](https://kubernetes.io/docs/concepts/workloads/pods/#static-pods)

_Static Pods_ are managed directly by the kubelet daemon on a specific node, without the [](https://kubernetes.io/docs/concepts/architecture/#kube-apiserver) observing them. Whereas most Pods are managed by the control plane (for example, a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)), for static Pods, the kubelet directly supervises each static Pod (and restarts it if it fails).

Static Pods are always bound to one [Kubelet](https://kubernetes.io/docs/reference/generated/kubelet) on a specific node. The main use for static Pods is to run a self-hosted control plane: in other words, using the kubelet to supervise the individual [](https://kubernetes.io/docs/concepts/architecture/#control-plane-components).

The kubelet automatically tries to create a [](https://kubernetes.io/docs/reference/glossary/?all=true#term-mirror-pod) on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there. See the guide [Create static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) for more information.

#### Note:

The `spec` of a static Pod cannot refer to other API objects (e.g., [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/), [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/), [Secret](https://kubernetes.io/docs/concepts/configuration/secret/), etc).

## Pods with multiple containers[](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)

Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

Pods in a Kubernetes cluster are used in two main ways:

- **Pods that run a single container**. The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.
- **Pods that run multiple containers that need to work together**. A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit of service—for example, one container serving data stored in a shared volume to the public, while a separate [sidecar container](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) refreshes or updates those files. The Pod wraps these containers, storage resources, and an ephemeral network identity together as a single unit.

For example, you might have a container that acts as a web server for files in a shared volume, and a separate [sidecar container](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) that updates those files from a remote source, as in the following diagram:

[Open: Pasted image 20241117155641.png](../Images/32993132cd12abab3dbf454d0c1df022_MD5.jpeg)
![32993132cd12abab3dbf454d0c1df022_MD5](../Images/32993132cd12abab3dbf454d0c1df022_MD5.jpeg)

Some Pods have [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) as well as [](https://kubernetes.io/docs/reference/glossary/?all=true#term-app-container). By default, init containers run and complete before the app containers are started.

You can also have [sidecar containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) that provide auxiliary services to the main application Pod (for example: a service mesh).

FEATURE STATE: `Kubernetes v1.29 [beta]`

Enabled by default, the `SidecarContainers` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) allows you to specify `restartPolicy: Always` for init containers. Setting the `Always` restart policy ensures that the containers where you set it are treated as _sidecars_ that are kept running during the entire lifetime of the Pod. Containers that you explicitly define as sidecar containers start up before the main application Pod and remain running until the Pod is shut down.

## Container probes[](https://kubernetes.io/docs/concepts/workloads/pods/#container-probes)

A _probe_ is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet can invoke different actions:

- `ExecAction` (performed with the help of the container runtime)
- `TCPSocketAction` (checked directly by the kubelet)
- `HTTPGetAction` (checked directly by the kubelet)

You can read more about [](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) in the Pod Lifecycle documentation.

## What's next[](https://kubernetes.io/docs/concepts/workloads/pods/#what-s-next)

- Learn about the [lifecycle of a Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/).
- Learn about [RuntimeClass](https://kubernetes.io/docs/concepts/containers/runtime-class/) and how you can use it to configure different Pods with different container runtime configurations.
- Read about [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) and how you can use it to manage application availability during disruptions.
- Pod is a top-level resource in the Kubernetes REST API. The [Pod](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/) object definition describes the object in detail.
- [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/) explains common layouts for Pods with more than one container.
- Read about [Pod topology spread constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)

To understand the context for why Kubernetes wraps a common Pod API in other resources (such as [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) or [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)), you can read about the prior art, including:

- [](https://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)
- [Borg](https://research.google.com/pubs/pub43438.html)
- [Marathon](https://github.com/d2iq-archive/marathon)
- [Omega](https://research.google/pubs/pub41684/)
- [Tupperware](https://engineering.fb.com/data-center-engineering/tupperware/).