#prova 
[Docs](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

This page explains how to upgrade a Kubernetes cluster created with kubeadm from version 1.30.x to version 1.31.x, and from version 1.31.x to 1.31.y (where `y > x`). Skipping MINOR versions when upgrading is unsupported. For more details, please visit [Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/).

To see information about upgrading clusters created using older versions of kubeadm, please refer to following pages instead:

- [Upgrading a kubeadm cluster from 1.29 to 1.30](https://v1-30.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Upgrading a kubeadm cluster from 1.28 to 1.29](https://v1-29.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Upgrading a kubeadm cluster from 1.27 to 1.28](https://v1-28.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Upgrading a kubeadm cluster from 1.26 to 1.27](https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

The Kubernetes project recommends upgrading to the latest patch releases promptly, and to ensure that you are running a supported minor release of Kubernetes. Following this recommendation helps you to to stay secure.

The upgrade workflow at high level is the following:

1. Upgrade a primary control plane node.
2. Upgrade additional control plane nodes.
3. Upgrade worker nodes.

## Before you begin[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#before-you-begin)

- Make sure you read the [release notes](https://git.k8s.io/kubernetes/CHANGELOG) carefully.
- The cluster should use a static control plane and etcd pods or external etcd.
- Make sure to back up any important components, such as app-level state stored in a database. `kubeadm upgrade` does not touch your workloads, only components internal to Kubernetes, but backups are always a best practice.
- [Swap must be disabled](https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux).

### Additional information[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#additional-information)

- The instructions below outline when to drain each node during the upgrade process. If you are performing a **minor** version upgrade for any kubelet, you **must** first drain the node (or nodes) that you are upgrading. In the case of control plane nodes, they could be running CoreDNS Pods or other critical workloads. For more information see [Draining nodes](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/).
- The Kubernetes project recommends that you match your kubelet and kubeadm versions. You can instead use a version of kubelet that is older than kubeadm, provided it is within the range of supported versions. For more details, please visit [](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#kubeadm-s-skew-against-the-kubelet).
- All containers are restarted after upgrade, because the container spec hash value is changed.
- To verify that the kubelet service has successfully restarted after the kubelet has been upgraded, you can execute `systemctl status kubelet` or view the service logs with `journalctl -xeu kubelet`.
- `kubeadm upgrade` supports `--config` with a [`UpgradeConfiguration` API type](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta4/) which can be used to configure the upgrade process.
- `kubeadm upgrade` does not support reconfiguration of an existing cluster. Follow the steps in [Reconfiguring a kubeadm cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-reconfigure/) instead.

### Considerations when upgrading etcd[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#considerations-when-upgrading-etcd)

Because the `kube-apiserver` static pod is running at all times (even if you have drained the node), when you perform a kubeadm upgrade which includes an etcd upgrade, in-flight requests to the server will stall while the new etcd static pod is restarting. As a workaround, it is possible to actively stop the `kube-apiserver` process a few seconds before starting the `kubeadm upgrade apply` command. This permits to complete in-flight requests and close existing connections, and minimizes the consequence of the etcd downtime. This can be done as follows on control plane nodes:

```shell
killall -s SIGTERM kube-apiserver # trigger a graceful kube-apiserver shutdown
sleep 20 # wait a little bit to permit completing in-flight requests
kubeadm upgrade ... # execute a kubeadm upgrade command
```

## Changing the package repository[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#changing-the-package-repository)

If you're using the community-owned package repositories (`pkgs.k8s.io`), you need to enable the package repository for the desired Kubernetes minor release. This is explained in [Changing the Kubernetes package repository](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/) document.

**Note:** The legacy package repositories (`apt.kubernetes.io` and `yum.kubernetes.io`) have been [deprecated and frozen starting from September 13, 2023](https://kubernetes.io/blog/2023/08/31/legacy-package-repository-deprecation/). **Using the [new package repositories hosted at `pkgs.k8s.io`](https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/) is strongly recommended and required in order to install Kubernetes versions released after September 13, 2023.** The deprecated legacy repositories, and their contents, might be removed at any time in the future and without a further notice period. The new package repositories provide downloads for Kubernetes versions starting with v1.24.0.

## Determine which version to upgrade to[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#determine-which-version-to-upgrade-to)

Find the latest patch release for Kubernetes 1.31 using the OS package manager:

- [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-versions-0)
- [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-versions-1)

```shell
# Find the latest 1.31 version in the list.
# It should look like 1.31.x-*, where x is the latest patch.
sudo apt update
sudo apt-cache madison kubeadm
```

## Upgrading control plane nodes[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrading-control-plane-nodes)

The upgrade procedure on control plane nodes should be executed one node at a time. Pick a control plane node that you wish to upgrade first. It must have the `/etc/kubernetes/admin.conf` file.

### Call "kubeadm upgrade"[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#call-kubeadm-upgrade)

**For the first control plane node**

1. Upgrade kubeadm:
    
    - [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-kubeadm-first-cp-0)
    - [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-kubeadm-first-cp-1)
    
    ```shell
    # replace x in 1.31.x-* with the latest patch version
    sudo apt-mark unhold kubeadm && \
    sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
    sudo apt-mark hold kubeadm
    ```
    
2. Verify that the download works and has the expected version:
    
    ```shell
    kubeadm version
    ```
    
3. Verify the upgrade plan:
    
    ```shell
    sudo kubeadm upgrade plan
    ```
    
    This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.
    
    #### Note:
    
    `kubeadm upgrade` also automatically renews the certificates that it manages on this node. To opt-out of certificate renewal the flag `--certificate-renewal=false` can be used. For more information see the [certificate management guide](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/).
    
4. Choose a version to upgrade to, and run the appropriate command. For example:
    
    ```shell
    # replace x with the patch version you picked for this upgrade
    sudo kubeadm upgrade apply v1.31.x
    ```
    
    Once the command finishes you should see:
    
    ```
    [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.31.x". Enjoy!
    
    [upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
    ```
    
    #### Note:
    
    For versions earlier than v1.28, kubeadm defaulted to a mode that upgrades the addons (including CoreDNS and kube-proxy) immediately during `kubeadm upgrade apply`, regardless of whether there are other control plane instances that have not been upgraded. This may cause compatibility problems. Since v1.28, kubeadm defaults to a mode that checks whether all the control plane instances have been upgraded before starting to upgrade the addons. You must perform control plane instances upgrade sequentially or at least ensure that the last control plane instance upgrade is not started until all the other control plane instances have been upgraded completely, and the addons upgrade will be performed after the last control plane instance is upgraded.
    
5. Manually upgrade your CNI provider plugin.
    
    Your Container Network Interface (CNI) provider may have its own upgrade instructions to follow. Check the [addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/) page to find your CNI provider and see whether additional upgrade steps are required.
    
    This step is not required on additional control plane nodes if the CNI provider runs as a DaemonSet.
    

**For the other control plane nodes**

Same as the first control plane node but use:

```shell
sudo kubeadm upgrade node
```

instead of:

```shell
sudo kubeadm upgrade apply
```

Also calling `kubeadm upgrade plan` and upgrading the CNI provider plugin is no longer needed.

### Drain the node[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#drain-the-node)

Prepare the node for maintenance by marking it unschedulable and evicting the workloads:

```shell
# replace <node-to-drain> with the name of your node you are draining
kubectl drain <node-to-drain> --ignore-daemonsets
```

### Upgrade kubelet and kubectl[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-kubelet-and-kubectl)

1. Upgrade the kubelet and kubectl:
    
    - [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-kubelet-0)
    - [](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#k8s-install-kubelet-1)
    
    ```shell
    # replace x in 1.31.x-* with the latest patch version
    sudo apt-mark unhold kubelet kubectl && \
    sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
    sudo apt-mark hold kubelet kubectl
    ```
    
2. Restart the kubelet:
    
    ```shell
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet
    ```
    

### Uncordon the node[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#uncordon-the-node)

Bring the node back online by marking it schedulable:

```shell
# replace <node-to-uncordon> with the name of your node
kubectl uncordon <node-to-uncordon>
```

## Upgrade worker nodes[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-worker-nodes)

The upgrade procedure on worker nodes should be executed one node at a time or few nodes at a time, without compromising the minimum required capacity for running your workloads.

The following pages show how to upgrade Linux and Windows worker nodes:

- [Upgrade Linux nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/)
- [Upgrade Windows nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-windows-nodes/)

## Verify the status of the cluster[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#verify-the-status-of-the-cluster)

After the kubelet is upgraded on all nodes verify that all nodes are available again by running the following command from anywhere kubectl can access the cluster:

```shell
kubectl get nodes
```

The `STATUS` column should show `Ready` for all your nodes, and the version number should be updated.

## Recovering from a failure state[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#recovering-from-a-failure-state)

If `kubeadm upgrade` fails and does not roll back, for example because of an unexpected shutdown during execution, you can run `kubeadm upgrade` again. This command is idempotent and eventually makes sure that the actual state is the desired state you declare.

To recover from a bad state, you can also run `sudo kubeadm upgrade apply --force` without changing the version that your cluster is running.

During upgrade kubeadm writes the following backup folders under `/etc/kubernetes/tmp`:

- `kubeadm-backup-etcd-<date>-<time>`
- `kubeadm-backup-manifests-<date>-<time>`

`kubeadm-backup-etcd` contains a backup of the local etcd member data for this control plane Node. In case of an etcd upgrade failure and if the automatic rollback does not work, the contents of this folder can be manually restored in `/var/lib/etcd`. In case external etcd is used this backup folder will be empty.

`kubeadm-backup-manifests` contains a backup of the static Pod manifest files for this control plane Node. In case of a upgrade failure and if the automatic rollback does not work, the contents of this folder can be manually restored in `/etc/kubernetes/manifests`. If for some reason there is no difference between a pre-upgrade and post-upgrade manifest file for a certain component, a backup file for it will not be written.

#### Note:

After the cluster upgrade using kubeadm, the backup directory `/etc/kubernetes/tmp` will remain and these backup files will need to be cleared manually.

## How it works[](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#how-it-works)

`kubeadm upgrade apply` does the following:

- Checks that your cluster is in an upgradeable state:
    - The API server is reachable
    - All nodes are in the `Ready` state
    - The control plane is healthy
- Enforces the version skew policies.
- Makes sure the control plane images are available or available to pull to the machine.
- Generates replacements and/or uses user supplied overwrites if component configs require version upgrades.
- Upgrades the control plane components or rollbacks if any of them fails to come up.
- Applies the new `CoreDNS` and `kube-proxy` manifests and makes sure that all necessary RBAC rules are created.
- Creates new certificate and key files of the API server and backs up old files if they're about to expire in 180 days.

`kubeadm upgrade node` does the following on additional control plane nodes:

- Fetches the kubeadm `ClusterConfiguration` from the cluster.
- Optionally backups the kube-apiserver certificate.
- Upgrades the static Pod manifests for the control plane components.
- Upgrades the kubelet configuration for this node.

`kubeadm upgrade node` does the following on worker nodes:

- Fetches the kubeadm `ClusterConfiguration` from the cluster.
- Upgrades the kubelet configuration for this node.