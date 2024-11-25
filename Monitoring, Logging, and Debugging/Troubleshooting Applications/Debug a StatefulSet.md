[Doc](https://kubernetes.io/docs/tasks/debug/debug-application/debug-statefulset/)
This task shows you how to debug a [StatefulSet](StatefulSet.md).

## Before you begin[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-statefulset/#before-you-begin)

- You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster.
- You should have a StatefulSet running that you want to investigate.

## Debugging a StatefulSet[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-statefulset/#debugging-a-statefulset)

In order to list all the pods which belong to a StatefulSet, which have a label `app.kubernetes.io/name=MyApp` set on them, you can use the following:

```shell
kubectl get pods -l app.kubernetes.io/name=MyApp
```

If you find that any Pods listed are in `Unknown` or `Terminating` state for an extended period of time, refer to the [Deleting StatefulSet Pods](https://kubernetes.io/docs/tasks/run-application/delete-stateful-set/) task for instructions on how to deal with them. You can debug individual Pods in a StatefulSet using the [Debug Pods](Debug%20Pods.md) guide.

## What's next[](https://kubernetes.io/docs/tasks/debug/debug-application/debug-statefulset/#what-s-next)

Learn more about [debugging an init-container](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/).