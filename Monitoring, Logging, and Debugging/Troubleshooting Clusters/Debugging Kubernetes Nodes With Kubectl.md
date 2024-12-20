[Docs](https://kubernetes.io/docs/tasks/debug/debug-cluster/kubectl-node-debug/)

This page shows how to debug a [node](https://kubernetes.io/docs/concepts/architecture/nodes/) running on the Kubernetes cluster using `kubectl debug` command.

## Before you begin[](https://kubernetes.io/docs/tasks/debug/debug-cluster/kubectl-node-debug/#before-you-begin)

You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) or you can use one of these Kubernetes playgrounds:

- [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

Your Kubernetes server must be at or later than version 1.2. To check the version, enter `kubectl version`.

You need to have permission to create Pods and to assign those new Pods to arbitrary nodes. You also need to be authorized to create Pods that access filesystems from the host.

## Debugging a Node using `kubectl debug node`[](https://kubernetes.io/docs/tasks/debug/debug-cluster/kubectl-node-debug/#debugging-a-node-using-kubectl-debug-node)

Use the `kubectl debug node` command to deploy a Pod to a Node that you want to troubleshoot. This command is helpful in scenarios where you can't access your Node by using an SSH connection. When the Pod is created, the Pod opens an interactive shell on the Node. To create an interactive shell on a Node named “mynode”, run:

```shell
kubectl debug node/mynode -it --image=ubuntu
```

```console
Creating debugging pod node-debugger-mynode-pdx84 with container debugger on node mynode.
If you don't see a command prompt, try pressing enter.
root@mynode:/#
```

The debug command helps to gather information and troubleshoot issues. Commands that you might use include `ip`, `ifconfig`, `nc`, `ping`, and `ps` and so on. You can also install other tools, such as `mtr`, `tcpdump`, and `curl`, from the respective package manager.

#### Note:

The debug commands may differ based on the image the debugging pod is using and these commands might need to be installed.

The debugging Pod can access the root filesystem of the Node, mounted at `/host` in the Pod. If you run your kubelet in a filesystem namespace, the debugging Pod sees the root for that namespace, not for the entire node. For a typical Linux node, you can look at the following paths to find relevant logs:

`/host/var/log/kubelet.log`

Logs from the `kubelet`, responsible for running containers on the node.

`/host/var/log/kube-proxy.log`

Logs from `kube-proxy`, which is responsible for directing traffic to Service endpoints.

`/host/var/log/containerd.log`

Logs from the `containerd` process running on the node.

`/host/var/log/syslog`

Shows general messages and information regarding the system.

`/host/var/log/kern.log`

Shows kernel logs.

When creating a debugging session on a Node, keep in mind that:

- `kubectl debug` automatically generates the name of the new pod, based on the name of the node.
- The root filesystem of the Node will be mounted at `/host`.
- Although the container runs in the host IPC, Network, and PID namespaces, the pod isn't privileged. This means that reading some process information might fail because access to that information is restricted to superusers. For example, `chroot /host` will fail. If you need a privileged pod, create it manually or use the `--profile=sysadmin` flag.
- By applying [](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#debugging-profiles), you can set specific properties such as [securityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) to a debugging Pod.

## Cleaning up[](https://kubernetes.io/docs/tasks/debug/debug-cluster/kubectl-node-debug/#cleaning-up)

When you finish using the debugging Pod, delete it:

```shell
kubectl get pods
```

```none
NAME                          READY   STATUS       RESTARTS   AGE
node-debugger-mynode-pdx84    0/1     Completed    0          8m1s
```

```shell
# Change the pod name accordingly
kubectl delete pod node-debugger-mynode-pdx84 --now
```

```none
pod "node-debugger-mynode-pdx84" deleted
```

#### Note:

The `kubectl debug node` command won't work if the Node is down (disconnected from the network, or kubelet dies and won't restart, etc.). Check [](https://kubernetes.io/docs/tasks/debug/debug-cluster/#example-debugging-a-down-unreachable-node) in that case.