[API-initiated eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/api-eviction/) is the process by which you use the [Eviction API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.31/#create-eviction-pod-v1-core) to create an `Eviction` object that triggers graceful pod termination.  

You can request eviction by calling the Eviction API directly, or programmatically using a client of the [API server](https://kubernetes.io/docs/concepts/architecture/#kube-apiserver), like the `kubectl drain` command. This creates an `Eviction` object, which causes the API server to terminate the Pod.

API-initiated evictions respect your configured [`PodDisruptionBudgets`](https://kubernetes.io/docs/tasks/run-application/configure-pdb/) and [`terminationGracePeriodSeconds`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).

Using the API to create an Eviction object for a Pod is like performing a policy-controlled [`DELETE` operation](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#delete-delete-a-pod) on the Pod.