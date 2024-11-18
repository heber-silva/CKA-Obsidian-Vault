#prova 
In Kubernetes, [scheduling](https://kubernetes.io/docs/concepts/scheduling-eviction/) refers to making sure that [[Pod]]s are matched to [[Node]]s so that the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) can run them. Preemption is the process of terminating Pods with lower [Priority](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#pod-priority) so that Pods with higher Priority can schedule on Nodes. Eviction is the process of terminating one or more Pods on Nodes.

## Scheduling[](https://kubernetes.io/docs/concepts/scheduling-eviction/#scheduling)

- [[Kubernetes Scheduler]]
- [[Assigning Pods to Nodes]]
- [[Pod Overhead]]
- [[Pod Topology Spread Constraints]]
- [[Taints and Tolerations]]
- [[Scheduling Framework]]
- [[Dynamic Resource Allocation]]
- [[Scheduler Performance Tuning]]
- [[Resource Bin Packing]]
- [[Pod Scheduling Readiness]]
- [Descheduler](https://github.com/kubernetes-sigs/descheduler#descheduler-for-kubernetes)

## Pod Disruption[](https://kubernetes.io/docs/concepts/scheduling-eviction/#pod-disruption)

[Pod disruption](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) is the process by which Pods on Nodes are terminated either voluntarily or involuntarily.

Voluntary disruptions are started intentionally by application owners or cluster administrators. Involuntary disruptions are unintentional and can be triggered by unavoidable issues like Nodes running out of resources, or by accidental deletions.

- [[Pod Priority and Preemption]]
- [[Node-pressure Eviction]]
- [[API-initiated Eviction]]