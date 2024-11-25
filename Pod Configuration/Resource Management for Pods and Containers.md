When you specify a [Pod](Pod.md), you can optionally specify how much of each resource a [Container](Container.md) needs. The most common resources to specify are CPU and memory (RAM); there are others.

When you specify the resource _request_ for containers in a Pod, the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) uses this information to decide which node to place the Pod on. When you specify a resource _limit_ for a container, the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) enforces those limits so that the running container is not allowed to use more of that resource than the limit you set. The kubelet also reserves at least the _request_ amount of that system resource specifically for that container to use.

## Requests and limits[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits)

If the node where a Pod is running has enough of a resource available, it's possible (and allowed) for a container to use more resource than its `request` for that resource specifies. However, a container is not allowed to use more than its resource `limit`.

For example, if you set a `memory` request of 256 MiB for a container, and that container is in a Pod scheduled to a Node with 8GiB of memory and no other Pods, then the container can try to use more RAM.

If you set a `memory` limit of 4GiB for that container, the kubelet (and [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes)) enforce the limit. The runtime prevents the container from using more than the configured resource limit. For example: when a process in the container tries to consume more than the allowed amount of memory, the system kernel terminates the process that attempted the allocation, with an out of memory (OOM) error.

Limits can be implemented either reactively (the system intervenes once it sees a violation) or by enforcement (the system prevents the container from ever exceeding the limit). Different runtimes can have different ways to implement the same restrictions.

#### Note:

If you specify a limit for a resource, but do not specify any request, and no admission-time mechanism has applied a default request for that resource, then Kubernetes copies the limit you specified and uses it as the requested value for the resource.

## Resource types[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-types)

_CPU_ and _memory_ are each a _resource type_. A resource type has a base unit. CPU represents compute processing and is specified in units of [Kubernetes CPUs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu). Memory is specified in units of bytes. For Linux workloads, you can specify _huge page_ resources. Huge pages are a Linux-specific feature where the node kernel allocates blocks of memory that are much larger than the default page size.

For example, on a system where the default page size is 4KiB, you could specify a limit, `hugepages-2Mi: 80Mi`. If the container tries allocating over 40 2MiB huge pages (a total of 80 MiB), that allocation fails.

#### Note:

You cannot overcommit `hugepages-*` resources. This is different from the `memory` and `cpu` resources.

CPU and memory are collectively referred to as _compute resources_, or _resources_. Compute resources are measurable quantities that can be requested, allocated, and consumed. They are distinct from [API resources](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). API resources, such as Pods and [Services](https://kubernetes.io/docs/concepts/services-networking/service/) are objects that can be read and modified through the Kubernetes API server.

## Resource requests and limits of Pod and container[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container)

For each container, you can specify resource limits and requests, including the following:

- `spec.containers[].resources.limits.cpu`
- `spec.containers[].resources.limits.memory`
- `spec.containers[].resources.limits.hugepages-<size>`
- `spec.containers[].resources.requests.cpu`
- `spec.containers[].resources.requests.memory`
- `spec.containers[].resources.requests.hugepages-<size>`

Although you can only specify requests and limits for individual containers, it is also useful to think about the overall resource requests and limits for a Pod. For a particular resource, a _Pod resource request/limit_ is the sum of the resource requests/limits of that type for each container in the Pod.

## Resource units in Kubernetes[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes)

### CPU resource units[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu)

Limits and requests for CPU resources are measured in _cpu_ units. In Kubernetes, 1 CPU unit is equivalent to **1 physical CPU core**, or **1 virtual core**, depending on whether the node is a physical host or a virtual machine running inside a physical machine.

Fractional requests are allowed. When you define a container with `spec.containers[].resources.requests.cpu` set to `0.5`, you are requesting half as much CPU time compared to if you asked for `1.0` CPU. For CPU resource units, the [quantity](https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/quantity/) expression `0.1` is equivalent to the expression `100m`, which can be read as "one hundred millicpu". Some people say "one hundred millicores", and this is understood to mean the same thing.

CPU resource is always specified as an absolute amount of resource, never as a relative amount. For example, `500m` CPU represents the roughly same amount of computing power whether that container runs on a single-core, dual-core, or 48-core machine.

#### Note:

Kubernetes doesn't allow you to specify CPU resources with a precision finer than `1m` or `0.001` CPU. To avoid accidentally using an invalid CPU quantity, it's useful to specify CPU units using the milliCPU form instead of the decimal form when using less than 1 CPU unit.

For example, you have a Pod that uses `5m` or `0.005` CPU and would like to decrease its CPU resources. By using the decimal form, it's harder to spot that `0.0005` CPU is an invalid value, while by using the milliCPU form, it's easier to spot that `0.5m` is an invalid value.

### Memory resource units[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

Limits and requests for `memory` are measured in bytes. You can express memory as a plain integer or as a fixed-point number using one of these [quantity](https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/quantity/) suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:

```shell
128974848, 129e6, 129M,  128974848000m, 123Mi
```

Pay attention to the case of the suffixes. If you request `400m` of memory, this is a request for 0.4 bytes. Someone who types that probably meant to ask for 400 mebibytes (`400Mi`) or 400 megabytes (`400M`).

## Container resources example[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1)

The following Pod has two containers. Both containers are defined with a request for 0.25 CPU and 64MiB (226 bytes) of memory. Each container has a limit of 0.5 CPU and 128MiB of memory. You can say the Pod has a request of 0.5 CPU and 128 MiB of memory, and a limit of 1 CPU and 256MiB of memory.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## How Pods with resource requests are scheduled[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#how-pods-with-resource-requests-are-scheduled)

When you create a Pod, the Kubernetes scheduler selects a node for the Pod to run on. Each node has a maximum capacity for each of the resource types: the amount of CPU and memory it can provide for Pods. The scheduler ensures that, for each resource type, the sum of the resource requests of the scheduled containers is less than the capacity of the node. Note that although actual memory or CPU resource usage on nodes is very low, the scheduler still refuses to place a Pod on a node if the capacity check fails. This protects against a resource shortage on a node when resource usage later increases, for example, during a daily peak in request rate.

## How Kubernetes applies resource requests and limits[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#how-pods-with-resource-limits-are-run)

When the kubelet starts a container as part of a Pod, the kubelet passes that container's requests and limits for memory and CPU to the container runtime.

On Linux, the container runtime typically configures kernel [cgroups](https://kubernetes.io/docs/reference/glossary/?all=true#term-cgroup) that apply and enforce the limits you defined.

- The CPU limit defines a hard ceiling on how much CPU time that the container can use. During each scheduling interval (time slice), the Linux kernel checks to see if this limit is exceeded; if so, the kernel waits before allowing that cgroup to resume execution.
- The CPU request typically defines a weighting. If several different containers (cgroups) want to run on a contended system, workloads with larger CPU requests are allocated more CPU time than workloads with small requests.
- The memory request is mainly used during (Kubernetes) Pod scheduling. On a node that uses cgroups v2, the container runtime might use the memory request as a hint to set `memory.min` and `memory.low`.
- The memory limit defines a memory limit for that cgroup. If the container tries to allocate more memory than this limit, the Linux kernel out-of-memory subsystem activates and, typically, intervenes by stopping one of the processes in the container that tried to allocate memory. If that process is the container's PID 1, and the container is marked as restartable, Kubernetes restarts the container.
- The memory limit for the Pod or container can also apply to pages in memory backed volumes, such as an `emptyDir`. The kubelet tracks `tmpfs` emptyDir volumes as container memory use, rather than as local ephemeral storage.　When using memory backed `emptyDir`, be sure to check the notes [below](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#memory-backed-emptydir).

If a container exceeds its memory request and the node that it runs on becomes short of memory overall, it is likely that the Pod the container belongs to will be [evicted](https://kubernetes.io/docs/concepts/scheduling-eviction/).

A container might or might not be allowed to exceed its CPU limit for extended periods of time. However, container runtimes don't terminate Pods or containers for excessive CPU usage.

To determine whether a container cannot be scheduled or is being killed due to resource limits, see the [Troubleshooting](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#troubleshooting) section.

### Monitoring compute & memory resource usage[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#monitoring-compute-memory-resource-usage)

The kubelet reports the resource usage of a Pod as part of the Pod [`status`](https://kubernetes.io/docs/concepts/overview/working-with-objects/#object-spec-and-status).

If optional [tools for monitoring](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/) are available in your cluster, then Pod resource usage can be retrieved either from the [Metrics API](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/#metrics-api) directly or from your monitoring tools.

### Considerations for memory backed `emptyDir` volumes[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#memory-backed-emptydir)

#### Caution:

If you do not specify a `sizeLimit` for an `emptyDir` volume, that volume may consume up to that pod's memory limit (`Pod.spec.containers[].resources.limits.memory`). If you do not set a memory limit, the pod has no upper bound on memory consumption, and can consume all available memory on the node. Kubernetes schedules pods based on resource requests (`Pod.spec.containers[].resources.requests`) and will not consider memory usage above the request when deciding if another pod can fit on a given node. This can result in a denial of service and cause the OS to do out-of-memory (OOM) handling. It is possible to create any number of `emptyDir`s that could potentially consume all available memory on the node, making OOM more likely.

From the perspective of memory management, there are some similarities between when a process uses memory as a work area and when using memory-backed `emptyDir`. But when using memory as a volume like memory-backed `emptyDir`, there are additional points below that you should be careful of.

- Files stored on a memory-backed volume are almost entirely managed by the user application. Unlike when used as a work area for a process, you can not rely on things like language-level garbage collection.
- The purpose of writing files to a volume is to save data or pass it between applications. Neither Kubernetes nor the OS may automatically delete files from a volume, so memory used by those files can not be reclaimed when the system or the pod are under memory pressure.
- A memory-backed `emptyDir` is useful because of its performance, but memory is generally much smaller in size and much higher in cost than other storage media, such as disks or SSDs. Using large amounts of memory for `emptyDir` volumes may affect the normal operation of your pod or of the whole node, so should be used carefully.

If you are administering a cluster or namespace, you can also set [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/) that limits memory use; you may also want to define a [LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/) for additional enforcement. If you specify a `spec.containers[].resources.limits.memory` for each Pod, then the maximum size of an `emptyDir` volume will be the pod's memory limit.

As an alternative, a cluster administrator can enforce size limits for `emptyDir` volumes in new Pods using a policy mechanism such as [ValidationAdmissionPolicy](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/).

## Local ephemeral storage[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage)

FEATURE STATE: `Kubernetes v1.25 [stable]`

Nodes have local ephemeral storage, backed by locally-attached writeable devices or, sometimes, by RAM. "Ephemeral" means that there is no long-term guarantee about durability.

Pods use ephemeral local storage for scratch space, caching, and for logs. The kubelet can provide scratch space to Pods using local ephemeral storage to mount [`emptyDir`](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) [volumes](https://kubernetes.io/docs/concepts/storage/volumes/) into containers.

The kubelet also uses this kind of storage to hold [node-level container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/#logging-at-the-node-level), container images, and the writable layers of running containers.

#### Caution:

If a node fails, the data in its ephemeral storage can be lost. Your applications cannot expect any performance SLAs (disk IOPS for example) from local ephemeral storage.

#### Note:

To make the resource quota work on ephemeral-storage, two things need to be done:

- An admin sets the resource quota for ephemeral-storage in a namespace.
- A user needs to specify limits for the ephemeral-storage resource in the Pod spec.

If the user doesn't specify the ephemeral-storage resource limit in the Pod spec, the resource quota is not enforced on ephemeral-storage.

Kubernetes lets you track, reserve and limit the amount of ephemeral local storage a Pod can consume.

### Configurations for local ephemeral storage[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#configurations-for-local-ephemeral-storage)

Kubernetes supports two ways to configure local ephemeral storage on a node:

- [Single filesystem](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#local-storage-configurations-0)
- [Two filesystems](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#local-storage-configurations-1)

In this configuration, you place all different kinds of ephemeral local data (`emptyDir` volumes, writeable layers, container images, logs) into one filesystem. The most effective way to configure the kubelet means dedicating this filesystem to Kubernetes (kubelet) data.

The kubelet also writes [node-level container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/#logging-at-the-node-level) and treats these similarly to ephemeral local storage.

The kubelet writes logs to files inside its configured log directory (`/var/log` by default); and has a base directory for other locally stored data (`/var/lib/kubelet` by default).

Typically, both `/var/lib/kubelet` and `/var/log` are on the system root filesystem, and the kubelet is designed with that layout in mind.

Your node can have as many other filesystems, not used for Kubernetes, as you like.

The kubelet can measure how much local storage it is using. It does this provided that you have set up the node using one of the supported configurations for local ephemeral storage.

If you have a different configuration, then the kubelet does not apply resource limits for ephemeral local storage.

#### Note:

The kubelet tracks `tmpfs` emptyDir volumes as container memory use, rather than as local ephemeral storage.

#### Note:

The kubelet will only track the root filesystem for ephemeral storage. OS layouts that mount a separate disk to `/var/lib/kubelet` or `/var/lib/containers` will not report ephemeral storage correctly.

### Setting requests and limits for local ephemeral storage[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage)

You can specify `ephemeral-storage` for managing local ephemeral storage. Each container of a Pod can specify either or both of the following:

- `spec.containers[].resources.limits.ephemeral-storage`
- `spec.containers[].resources.requests.ephemeral-storage`

Limits and requests for `ephemeral-storage` are measured in byte quantities. You can express storage as a plain integer or as a fixed-point number using one of these suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following quantities all represent roughly the same value:

- `128974848`
- `129e6`
- `129M`
- `123Mi`

Pay attention to the case of the suffixes. If you request `400m` of ephemeral-storage, this is a request for 0.4 bytes. Someone who types that probably meant to ask for 400 mebibytes (`400Mi`) or 400 megabytes (`400M`).

In the following example, the Pod has two containers. Each container has a request of 2GiB of local ephemeral storage. Each container has a limit of 4GiB of local ephemeral storage. Therefore, the Pod has a request of 4GiB of local ephemeral storage, and a limit of 8GiB of local ephemeral storage. 500Mi of that limit could be consumed by the `emptyDir` volume.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
    volumeMounts:
    - name: ephemeral
      mountPath: "/tmp"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
    volumeMounts:
    - name: ephemeral
      mountPath: "/tmp"
  volumes:
    - name: ephemeral
      emptyDir:
        sizeLimit: 500Mi
```

### How Pods with ephemeral-storage requests are scheduled[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#how-pods-with-ephemeral-storage-requests-are-scheduled)

When you create a Pod, the Kubernetes scheduler selects a node for the Pod to run on. Each node has a maximum amount of local ephemeral storage it can provide for Pods. For more information, see [Node Allocatable](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable).

The scheduler ensures that the sum of the resource requests of the scheduled containers is less than the capacity of the node.

### Ephemeral storage consumption management[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-emphemeralstorage-consumption)

If the kubelet is managing local ephemeral storage as a resource, then the kubelet measures storage use in:

- `emptyDir` volumes, except _tmpfs_ `emptyDir` volumes
- directories holding node-level logs
- writeable container layers

If a Pod is using more ephemeral storage than you allow it to, the kubelet sets an eviction signal that triggers Pod eviction.

For container-level isolation, if a container's writable layer and log usage exceeds its storage limit, the kubelet marks the Pod for eviction.

For pod-level isolation the kubelet works out an overall Pod storage limit by summing the limits for the containers in that Pod. In this case, if the sum of the local ephemeral storage usage from all containers and also the Pod's `emptyDir` volumes exceeds the overall Pod storage limit, then the kubelet also marks the Pod for eviction.

#### Caution:

If the kubelet is not measuring local ephemeral storage, then a Pod that exceeds its local storage limit will not be evicted for breaching local storage resource limits.

However, if the filesystem space for writeable container layers, node-level logs, or `emptyDir` volumes falls low, the node [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) itself as short on local storage and this taint triggers eviction for any Pods that don't specifically tolerate the taint.

See the supported [configurations](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#configurations-for-local-ephemeral-storage) for ephemeral local storage.

The kubelet supports different ways to measure Pod storage use:

- [Periodic scanning](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-emphemeralstorage-measurement-0)
- [Filesystem project quota](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-emphemeralstorage-measurement-1)

The kubelet performs regular, scheduled checks that scan each `emptyDir` volume, container log directory, and writeable container layer.

The scan measures how much space is used.

#### Note:

In this mode, the kubelet does not track open file descriptors for deleted files.

If you (or a container) create a file inside an `emptyDir` volume, something then opens that file, and you delete the file while it is still open, then the inode for the deleted file stays until you close that file but the kubelet does not categorize the space as in use.

## Extended resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#extended-resources)

Extended resources are fully-qualified resource names outside the `kubernetes.io` domain. They allow cluster operators to advertise and users to consume the non-Kubernetes-built-in resources.

There are two steps required to use Extended Resources. First, the cluster operator must advertise an Extended Resource. Second, users must request the Extended Resource in Pods.

### Managing extended resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#managing-extended-resources)

#### Node-level extended resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#node-level-extended-resources)

Node-level extended resources are tied to nodes.

##### Device plugin managed resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#device-plugin-managed-resources)

See [Device Plugin](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/) for how to advertise device plugin managed resources on each node.

##### Other resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#other-resources)

To advertise a new node-level extended resource, the cluster operator can submit a `PATCH` HTTP request to the API server to specify the available quantity in the `status.capacity` for a node in the cluster. After this operation, the node's `status.capacity` will include a new resource. The `status.allocatable` field is updated automatically with the new resource asynchronously by the kubelet.

Because the scheduler uses the node's `status.allocatable` value when evaluating Pod fitness, the scheduler only takes account of the new value after that asynchronous update. There may be a short delay between patching the node capacity with a new resource and the time when the first Pod that requests the resource can be scheduled on that node.

**Example:**

Here is an example showing how to use `curl` to form an HTTP request that advertises five "example.com/foo" resources on node `k8s-node-1` whose master is `k8s-master`.

```shell
curl --header "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path": "/status/capacity/example.com~1foo", "value": "5"}]' \
http://k8s-master:8080/api/v1/nodes/k8s-node-1/status
```

#### Note:

In the preceding request, `~1` is the encoding for the character `/` in the patch path. The operation path value in JSON-Patch is interpreted as a JSON-Pointer. For more details, see [IETF RFC 6901, section 3](https://tools.ietf.org/html/rfc6901#section-3).

#### Cluster-level extended resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#cluster-level-extended-resources)

Cluster-level extended resources are not tied to nodes. They are usually managed by scheduler extenders, which handle the resource consumption and resource quota.

You can specify the extended resources that are handled by scheduler extenders in [scheduler configuration](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/)

**Example:**

The following configuration for a scheduler policy indicates that the cluster-level extended resource "example.com/foo" is handled by the scheduler extender.

- The scheduler sends a Pod to the scheduler extender only if the Pod requests "example.com/foo".
- The `ignoredByScheduler` field specifies that the scheduler does not check the "example.com/foo" resource in its `PodFitsResources` predicate.

```json
{
  "kind": "Policy",
  "apiVersion": "v1",
  "extenders": [
    {
      "urlPrefix":"<extender-endpoint>",
      "bindVerb": "bind",
      "managedResources": [
        {
          "name": "example.com/foo",
          "ignoredByScheduler": true
        }
      ]
    }
  ]
}
```

### Consuming extended resources[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#consuming-extended-resources)

Users can consume extended resources in Pod specs like CPU and memory. The scheduler takes care of the resource accounting so that no more than the available amount is simultaneously allocated to Pods.

The API server restricts quantities of extended resources to whole numbers. Examples of _valid_ quantities are `3`, `3000m` and `3Ki`. Examples of _invalid_ quantities are `0.5` and `1500m` (because `1500m` would result in `1.5`).

#### Note:

Extended resources replace Opaque Integer Resources. Users can use any domain name prefix other than `kubernetes.io` which is reserved.

To consume an extended resource in a Pod, include the resource name as a key in the `spec.containers[].resources.limits` map in the container spec.

#### Note:

Extended resources cannot be overcommitted, so request and limit must be equal if both are present in a container spec.

A Pod is scheduled only if all of the resource requests are satisfied, including CPU, memory and any extended resources. The Pod remains in the `PENDING` state as long as the resource request cannot be satisfied.

**Example:**

The Pod below requests 2 CPUs and 1 "example.com/foo" (an extended resource).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: myimage
    resources:
      requests:
        cpu: 2
        example.com/foo: 1
      limits:
        example.com/foo: 1
```

## PID limiting[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#pid-limiting)

Process ID (PID) limits allow for the configuration of a kubelet to limit the number of PIDs that a given Pod can consume. See [PID Limiting](https://kubernetes.io/docs/concepts/policy/pid-limiting/) for information.

## Troubleshooting[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#troubleshooting)

### My Pods are pending with event message `FailedScheduling`[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#my-pods-are-pending-with-event-message-failedscheduling)

If the scheduler cannot find any node where a Pod can fit, the Pod remains unscheduled until a place can be found. An [Event](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/event-v1/) is produced each time the scheduler fails to find a place for the Pod. You can use `kubectl` to view the events for a Pod; for example:

```shell
kubectl describe pod frontend | grep -A 9999999999 Events
```

```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  23s   default-scheduler  0/42 nodes available: insufficient cpu
```

In the preceding example, the Pod named "frontend" fails to be scheduled due to insufficient CPU resource on any node. Similar error messages can also suggest failure due to insufficient memory (PodExceedsFreeMemory). In general, if a Pod is pending with a message of this type, there are several things to try:

- Add more nodes to the cluster.
- Terminate unneeded Pods to make room for pending Pods.
- Check that the Pod is not larger than all the nodes. For example, if all the nodes have a capacity of `cpu: 1`, then a Pod with a request of `cpu: 1.1` will never be scheduled.
- Check for node taints. If most of your nodes are tainted, and the new Pod does not tolerate that taint, the scheduler only considers placements onto the remaining nodes that don't have that taint.

You can check node capacities and amounts allocated with the `kubectl describe nodes` command. For example:

```shell
kubectl describe nodes e2e-test-node-pool-4lw4
```

```
Name:            e2e-test-node-pool-4lw4
[ ... lines removed for clarity ...]
Capacity:
 cpu:                               2
 memory:                            7679792Ki
 pods:                              110
Allocatable:
 cpu:                               1800m
 memory:                            7474992Ki
 pods:                              110
[ ... lines removed for clarity ...]
Non-terminated Pods:        (5 in total)
  Namespace    Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------    ----                                  ------------  ----------  ---------------  -------------
  kube-system  fluentd-gcp-v1.38-28bv1               100m (5%)     0 (0%)      200Mi (2%)       200Mi (2%)
  kube-system  kube-dns-3297075139-61lj3             260m (13%)    0 (0%)      100Mi (1%)       170Mi (2%)
  kube-system  kube-proxy-e2e-test-...               100m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system  monitoring-influxdb-grafana-v4-z1m12  200m (10%)    200m (10%)  600Mi (8%)       600Mi (8%)
  kube-system  node-problem-detector-v0.1-fj7m3      20m (1%)      200m (10%)  20Mi (0%)        100Mi (1%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests    CPU Limits    Memory Requests    Memory Limits
  ------------    ----------    ---------------    -------------
  680m (34%)      400m (20%)    920Mi (11%)        1070Mi (13%)
```

In the preceding output, you can see that if a Pod requests more than 1.120 CPUs or more than 6.23Gi of memory, that Pod will not fit on the node.

By looking at the “Pods” section, you can see which Pods are taking up space on the node.

The amount of resources available to Pods is less than the node capacity because system daemons use a portion of the available resources. Within the Kubernetes API, each Node has a `.status.allocatable` field (see [NodeStatus](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/node-v1/#NodeStatus) for details).

The `.status.allocatable` field describes the amount of resources that are available to Pods on that node (for example: 15 virtual CPUs and 7538 MiB of memory). For more information on node allocatable resources in Kubernetes, see [Reserve Compute Resources for System Daemons](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/).

You can configure [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) to limit the total amount of resources that a namespace can consume. Kubernetes enforces quotas for objects in particular namespace when there is a ResourceQuota in that namespace. For example, if you assign specific namespaces to different teams, you can add ResourceQuotas into those namespaces. Setting resource quotas helps to prevent one team from using so much of any resource that this over-use affects other teams.

You should also consider what access you grant to that namespace: **full** write access to a namespace allows someone with that access to remove any resource, including a configured ResourceQuota.

### My container is terminated[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#my-container-is-terminated)

Your container might get terminated because it is resource-starved. To check whether a container is being killed because it is hitting a resource limit, call `kubectl describe pod` on the Pod of interest:

```shell
kubectl describe pod simmemleak-hra99
```

The output is similar to:

```
Name:                           simmemleak-hra99
Namespace:                      default
Image(s):                       saadali/simmemleak
Node:                           kubernetes-node-tf0f/10.240.216.66
Labels:                         name=simmemleak
Status:                         Running
Reason:
Message:
IP:                             10.244.2.75
Containers:
  simmemleak:
    Image:  saadali/simmemleak:latest
    Limits:
      cpu:          100m
      memory:       50Mi
    State:          Running
      Started:      Tue, 07 Jul 2019 12:54:41 -0700
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Fri, 07 Jul 2019 12:54:30 -0700
      Finished:     Fri, 07 Jul 2019 12:54:33 -0700
    Ready:          False
    Restart Count:  5
Conditions:
  Type      Status
  Ready     False
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  42s   default-scheduler  Successfully assigned simmemleak-hra99 to kubernetes-node-tf0f
  Normal  Pulled     41s   kubelet            Container image "saadali/simmemleak:latest" already present on machine
  Normal  Created    41s   kubelet            Created container simmemleak
  Normal  Started    40s   kubelet            Started container simmemleak
  Normal  Killing    32s   kubelet            Killing container with id ead3fb35-5cf5-44ed-9ae1-488115be66c6: Need to kill Pod
```

In the preceding example, the `Restart Count: 5` indicates that the `simmemleak` container in the Pod was terminated and restarted five times (so far). The `OOMKilled` reason shows that the container tried to use more memory than its limit.

Your next step might be to check the application code for a memory leak. If you find that the application is behaving how you expect, consider setting a higher memory limit (and possibly request) for that container.

## What's next[](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#what-s-next)

- Get hands-on experience [assigning Memory resources to containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/).
- Get hands-on experience [assigning CPU resources to containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/).
- Read how the API reference defines a [container](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#Container) and its [resource requirements](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#resources)
- Read about [project quotas](https://www.linux.org/docs/man8/xfs_quota.html) in XFS
- Read more about the [kube-scheduler configuration reference (v1)](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/)
- Read more about [Quality of Service classes for Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/)