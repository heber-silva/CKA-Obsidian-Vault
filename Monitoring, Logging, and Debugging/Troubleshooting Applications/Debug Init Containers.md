[Doc](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/)

This page shows how to investigate problems related to the execution of [Init Container](../../Workloads/Pods/Init%20Container.md)s. The example command lines below refer to the Pod as `<pod-name>` and the Init Containers as `<init-container-1>` and `<init-container-2>`.

## Before you begin[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#before-you-begin)

You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) or you can use one of these Kubernetes playgrounds:

- [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

To check the version, enter `kubectl version`.

- You should be familiar with the basics of [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/).
- You should have [](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container).

## Checking the status of Init Containers[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#checking-the-status-of-init-containers)

Display the status of your pod:

```shell
kubectl get pod <pod-name>
```

For example, a status of `Init:1/2` indicates that one of two Init Containers has completed successfully:

```
NAME         READY     STATUS     RESTARTS   AGE
<pod-name>   0/1       Init:1/2   0          7s
```

See [](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#understanding-pod-status) for more examples of status values and their meanings.

## Getting details about Init Containers[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#getting-details-about-init-containers)

View more detailed information about Init Container execution:

```shell
kubectl describe pod <pod-name>
```

For example, a Pod with two Init Containers might show the following:

```
Init Containers:
  <init-container-1>:
    Container ID:    ...
    ...
    State:           Terminated
      Reason:        Completed
      Exit Code:     0
      Started:       ...
      Finished:      ...
    Ready:           True
    Restart Count:   0
    ...
  <init-container-2>:
    Container ID:    ...
    ...
    State:           Waiting
      Reason:        CrashLoopBackOff
    Last State:      Terminated
      Reason:        Error
      Exit Code:     1
      Started:       ...
      Finished:      ...
    Ready:           False
    Restart Count:   3
    ...
```

You can also access the Init Container statuses programmatically by reading the `status.initContainerStatuses` field on the Pod Spec:

```shell
kubectl get pod nginx --template '{{.status.initContainerStatuses}}'
```

This command will return the same information as above in raw JSON.

## Accessing logs from Init Containers[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#accessing-logs-from-init-containers)

Pass the Init Container name along with the Pod name to access its logs.

```shell
kubectl logs <pod-name> -c <init-container-2>
```

Init Containers that run a shell script print commands as they're executed. For example, you can do this in Bash by running `set -x` at the beginning of the script.

## Understanding Pod status[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/#understanding-pod-status)

A Pod status beginning with `Init:` summarizes the status of Init Container execution. The table below describes some example status values that you might see while debugging Init Containers.

|Status|Meaning|
|---|---|
|`Init:N/M`|The Pod has `M` Init Containers, and `N` have completed so far.|
|`Init:Error`|An Init Container has failed to execute.|
|`Init:CrashLoopBackOff`|An Init Container has failed repeatedly.|
|`Pending`|The Pod has not yet begun executing Init Containers.|
|`PodInitializing` or `Running`|The Pod has already finished executing Init Containers.|