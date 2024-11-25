#prova 
In Kubernetes, [scheduling](https://kubernetes.io/docs/concepts/scheduling-eviction/) refers to making sure that [Pod](../Workloads/Pod.md)s are matched to [Node](../Kubernetes%20Achitecture/Node.md)s so that the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) can run them. Preemption is the process of terminating Pods with lower [](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#pod-priority) so that Pods with higher Priority can schedule on Nodes. Eviction is the process of terminating one or more Pods on Nodes.

## Scheduling[](https://kubernetes.io/docs/concepts/scheduling-eviction/#scheduling)

- [Kubernetes Scheduler](Kubernetes%20Scheduler.md)
- [Assigning Pods to Nodes](Assigning%20Pods%20to%20Nodes.md)
- [Pod Overhead](Pod%20Overhead.md)
- [Pod Topology Spread Constraints](Pod%20Topology%20Spread%20Constraints.md)
- [Taints and Tolerations](Taints%20and%20Tolerations.md)
- [Scheduling Framework](Scheduling%20Framework.md)
- [Dynamic Resource Allocation](Dynamic%20Resource%20Allocation.md)
- [Scheduler Performance Tuning](Scheduler%20Performance%20Tuning.md)
- [Resource Bin Packing](Resource%20Bin%20Packing.md)
- [Pod Scheduling Readiness](Pod%20Scheduling%20Readiness.md)
- [](https://github.com/kubernetes-sigs/descheduler#descheduler-for-kubernetes)

## Pod Disruption[](https://kubernetes.io/docs/concepts/scheduling-eviction/#pod-disruption)

[Pod disruption](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) is the process by which Pods on Nodes are terminated either voluntarily or involuntarily.

Voluntary disruptions are started intentionally by application owners or cluster administrators. Involuntary disruptions are unintentional and can be triggered by unavoidable issues like Nodes running out of resources, or by accidental deletions.

- [Pod Priority and Preemption](Pod%20Priority%20and%20Preemption.md)
- [Node-pressure Eviction](Node-pressure%20Eviction.md)
- [API-initiated Eviction](API-initiated%20Eviction.md)