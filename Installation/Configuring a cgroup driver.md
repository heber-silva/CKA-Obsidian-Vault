This page explains how to configure the kubelet's cgroup driver to match the container runtime cgroup driver for kubeadm clusters.

## Before you begin[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#before-you-begin)

You should be familiar with the Kubernetes [container runtime requirements](https://kubernetes.io/docs/setup/production-environment/container-runtimes/).

## Configuring the container runtime cgroup driver[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-container-runtime-cgroup-driver)

The [Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) page explains that the `systemd` driver is recommended for kubeadm based setups instead of the kubelet's [default](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/) `cgroupfs` driver, because kubeadm manages the kubelet as a [systemd service](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/).

The page also provides details on how to set up a number of different container runtimes with the `systemd` driver by default.

## Configuring the kubelet cgroup driver[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver)

kubeadm allows you to pass a `KubeletConfiguration` structure during `kubeadm init`. This `KubeletConfiguration` can include the `cgroupDriver` field which controls the cgroup driver of the kubelet.

#### Note:

In v1.22 and later, if the user does not set the `cgroupDriver` field under `KubeletConfiguration`, kubeadm defaults it to `systemd`.

In Kubernetes v1.28, you can enable automatic detection of the cgroup driver as an alpha feature. See [systemd cgroup driver](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#systemd-cgroup-driver) for more details.

A minimal example of configuring the field explicitly:

```yaml
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta4
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

Such a configuration file can then be passed to the kubeadm command:

```shell
kubeadm init --config kubeadm-config.yaml
```

#### Note:

Kubeadm uses the same `KubeletConfiguration` for all nodes in the cluster. The `KubeletConfiguration` is stored in a [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) object under the `kube-system` namespace.

Executing the sub commands `init`, `join` and `upgrade` would result in kubeadm writing the `KubeletConfiguration` as a file under `/var/lib/kubelet/config.yaml` and passing it to the local node kubelet.

## Using the `cgroupfs` driver[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#using-the-cgroupfs-driver)

To use `cgroupfs` and to prevent `kubeadm upgrade` from modifying the `KubeletConfiguration` cgroup driver on existing setups, you must be explicit about its value. This applies to a case where you do not wish future versions of kubeadm to apply the `systemd` driver by default.

See the below section on "[Modify the kubelet ConfigMap](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#modify-the-kubelet-configmap)" for details on how to be explicit about the value.

If you wish to configure a container runtime to use the `cgroupfs` driver, you must refer to the documentation of the container runtime of your choice.

## Migrating to the `systemd` driver[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#migrating-to-the-systemd-driver)

To change the cgroup driver of an existing kubeadm cluster from `cgroupfs` to `systemd` in-place, a similar procedure to a kubelet upgrade is required. This must include both steps outlined below.

#### Note:

Alternatively, it is possible to replace the old nodes in the cluster with new ones that use the `systemd` driver. This requires executing only the first step below before joining the new nodes and ensuring the workloads can safely move to the new nodes before deleting the old nodes.

### Modify the kubelet ConfigMap[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#modify-the-kubelet-configmap)

- Call `kubectl edit cm kubelet-config -n kube-system`.
    
- Either modify the existing `cgroupDriver` value or add a new field that looks like this:
    
    ```yaml
    cgroupDriver: systemd
    ```
    
    This field must be present under the `kubelet:` section of the ConfigMap.
    

### Update the cgroup driver on all nodes[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#update-the-cgroup-driver-on-all-nodes)

For each node in the cluster:

- [Drain the node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) using `kubectl drain <node-name> --ignore-daemonsets`
- Stop the kubelet using `systemctl stop kubelet`
- Stop the container runtime
- Modify the container runtime cgroup driver to `systemd`
- Set `cgroupDriver: systemd` in `/var/lib/kubelet/config.yaml`
- Start the container runtime
- Start the kubelet using `systemctl start kubelet`
- [Uncordon the node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) using `kubectl uncordon <node-name>`

Execute these steps on nodes one at a time to ensure workloads have sufficient time to schedule on different nodes.

Once the process is complete ensure that all nodes and workloads are healthy.