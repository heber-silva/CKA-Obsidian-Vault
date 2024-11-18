#prova 
[Doc](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
## Before you begin[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)

- A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
- 2 GB or more of RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more for control plane machines.
- Full network connectivity between all machines in the cluster (public or private network is fine).
- Unique hostname, MAC address, and product_uuid for every node. See [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address) for more details.
- Certain ports are open on your machines. See [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports) for more details.
## Verify the MAC address and product_uuid are unique for every node[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address)

- You can get the MAC address of the network interfaces using the command `ip link` or `ifconfig -a`
- The product_uuid can be checked by using the command `sudo cat /sys/class/dmi/id/product_uuid`

It is very likely that hardware devices will have unique addresses, although some virtual machines may have identical values. Kubernetes uses these values to uniquely identify the nodes in the cluster. If these values are not unique to each node, the installation process may [fail](https://github.com/kubernetes/kubeadm/issues/31).

## Check network adapters[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-network-adapters)

If you have more than one network adapter, and your Kubernetes components are not reachable on the default route, we recommend you add IP route(s) so Kubernetes cluster addresses go via the appropriate adapter.

## Check required ports[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

These [required ports](https://kubernetes.io/docs/reference/networking/ports-and-protocols/) need to be open in order for Kubernetes components to communicate with each other. You can use tools like [netcat](https://netcat.sourceforge.net/) to check if a port is open. For example:

```shell
nc 127.0.0.1 6443 -v
```

The pod network plugin you use may also require certain ports to be open. Since this differs with each pod network plugin, please see the documentation for the plugins about what port(s) those need.

## Swap configuration[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#swap-configuration)

The default behavior of a kubelet is to fail to start if swap memory is detected on a node. This means that swap should either be disabled or tolerated by kubelet.

- To tolerate swap, add `failSwapOn: false` to kubelet configuration or as a command line argument. Note: even if `failSwapOn: false` is provided, workloads wouldn't have swap access by default. This can be changed by setting a `swapBehavior`, again in the kubelet configuration file. To use swap, set a `swapBehavior` other than the default `NoSwap` setting. See [Swap memory management](https://kubernetes.io/docs/concepts/architecture/nodes/#swap-memory) for more details.
- To disable swap, `sudo swapoff -a` can be used to disable swapping temporarily. To make this change persistent across reboots, make sure swap is disabled in config files like `/etc/fstab`, `systemd.swap`, depending how it was configured on your system.

## Installing a container runtime[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)

To run containers in Pods, Kubernetes uses a [[container runtime]].

By default, Kubernetes uses the [Container Runtime Interface](https://kubernetes.io/docs/concepts/architecture/#container-runtime) (CRI) to interface with your chosen container runtime.

If you don't specify a runtime, kubeadm automatically tries to detect an installed container runtime by scanning through a list of known endpoints.

If multiple or no container runtimes are detected kubeadm will throw an error and will request that you specify which one you want to use.

See [container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) for more information.

#### Note:

Docker Engine does not implement the [CRI](https://kubernetes.io/docs/concepts/architecture/cri/) which is a requirement for a container runtime to work with Kubernetes. For that reason, an additional service [cri-dockerd](https://mirantis.github.io/cri-dockerd/) has to be installed. cri-dockerd is a project based on the legacy built-in Docker Engine support that was [removed](https://kubernetes.io/dockershim) from the kubelet in version 1.24.

The tables below include the known endpoints for supported operating systems:

**Linux:**

| Runtime                           | Path to Unix domain socket                   |
| --------------------------------- | -------------------------------------------- |
| containerd                        | `unix:///var/run/containerd/containerd.sock` |
| CRI-O                             | `unix:///var/run/crio/crio.sock`             |
| Docker Engine (using cri-dockerd) | `unix:///var/run/cri-dockerd.sock`           |
**Windows:**

| Runtime                           | Path to Windows named pipe               |
| --------------------------------- | ---------------------------------------- |
| containerd                        | `npipe:////./pipe/containerd-containerd` |
| Docker Engine (using cri-dockerd) | `npipe:////./pipe/cri-dockerd`           |
## Installing kubeadm, kubelet and kubectl[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)

You will install these packages on all of your machines:

- `kubeadm`: the command to bootstrap the cluster.
    
- `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
    
- `kubectl`: the command line util to talk to your cluster.
    

kubeadm **will not** install or manage `kubelet` or `kubectl` for you, so you will need to ensure they match the version of the Kubernetes control plane you want kubeadm to install for you. If you do not, there is a risk of a version skew occurring that can lead to unexpected, buggy behaviour. However, _one_ minor version skew between the kubelet and the control plane is supported, but the kubelet version may never exceed the API server version. For example, the kubelet running 1.7.0 should be fully compatible with a 1.8.0 API server, but not vice versa.

For information about installing `kubectl`, see [Install and set up kubectl](https://kubernetes.io/docs/tasks/tools/).

#### Warning:

These instructions exclude all Kubernetes packages from any system upgrades. This is because kubeadm and Kubernetes require [special attention to upgrade](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).

For more information on version skews, see:

- Kubernetes [version and version-skew policy](https://kubernetes.io/docs/setup/release/version-skew-policy/)
- Kubeadm-specific [version skew policy](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

---
**Note:** The legacy package repositories (`apt.kubernetes.io` and `yum.kubernetes.io`) have been [deprecated and frozen starting from September 13, 2023](https://kubernetes.io/blog/2023/08/31/legacy-package-repository-deprecation/). **Using the [new package repositories hosted at `pkgs.k8s.io`](https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/) is strongly recommended and required in order to install Kubernetes versions released after September 13, 2023.** The deprecated legacy repositories, and their contents, might be removed at any time in the future and without a further notice period. The new package repositories provide downloads for Kubernetes versions starting with v1.24.0.

---
#### Note:

There's a dedicated package repository for each Kubernetes minor version. If you want to install a minor version other than v1.31, please see the installation guide for your desired minor version.

---
**These instructions are for Kubernetes v1.31.**

1. Update the `apt` package index and install packages needed to use the Kubernetes `apt` repository:
    
    ```shell
    sudo apt-get update
    # apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    ```
    
2. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:
    
    ```shell
    # If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
    # sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```
    

#### Note:

In releases older than Debian 12 and Ubuntu 22.04, directory `/etc/apt/keyrings` does not exist by default, and it should be created before the curl command.

3. Add the appropriate Kubernetes `apt` repository. Please note that this repository have packages only for Kubernetes 1.31; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).
    
    ```shell
    # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```
    
4. Update the `apt` package index, install kubelet, kubeadm and kubectl, and pin their version:
    
    ```shell
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
    
5. (Optional) Enable the kubelet service before running kubeadm:
    
    ```shell
    sudo systemctl enable --now kubelet
    ```
    

The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do.

## Configuring a cgroup driver
Both the container runtime and the kubelet have a property called ["cgroup driver"](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers), which is important for the management of cgroups on Linux machines.

#### Warning:

Matching the container runtime and kubelet cgroup drivers is required or otherwise the kubelet process will fail.

See [[Configuring a cgroup driver]] for more details.

## Troubleshooting[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#troubleshooting)

If you are running into difficulties with kubeadm, please consult [[Troubleshooting kubeadm]].

## Creating a Cluster with Kubeadm

Using `kubeadm`, you can create a minimum viable Kubernetes cluster that conforms to best practices. In fact, you can use `kubeadm` to set up a cluster that will pass the [Kubernetes Conformance tests](https://kubernetes.io/blog/2017/10/software-conformance-certification/). `kubeadm` also supports other cluster lifecycle functions, such as [bootstrap tokens](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/) and cluster upgrades.

The `kubeadm` tool is good if you need:

- A simple way for you to try out Kubernetes, possibly for the first time.
- A way for existing users to automate setting up a cluster and test their application.
- A building block in other ecosystem and/or installer tools with a larger scope.

You can install and use `kubeadm` on various machines: your laptop, a set of cloud servers, a Raspberry Pi, and more. Whether you're deploying into the cloud or on-premises, you can integrate `kubeadm` into provisioning systems such as Ansible or Terraform.

## Before you begin[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin)

To follow this guide, you need:

- One or more machines running a deb/rpm-compatible Linux OS; for example: Ubuntu or CentOS.
- 2 GiB or more of RAM per machine--any less leaves little room for your apps.
- At least 2 CPUs on the machine that you use as a control-plane node.
- Full network connectivity among all machines in the cluster. You can use either a public or a private network.

You also need to use a version of `kubeadm` that can deploy the version of Kubernetes that you want to use in your new cluster.

[Kubernetes' version and version skew support policy](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-versions) applies to `kubeadm` as well as to Kubernetes overall. Check that policy to learn about what versions of Kubernetes and `kubeadm` are supported. This page is written for Kubernetes v1.31.

The `kubeadm` tool's overall feature state is General Availability (GA). Some sub-features are still under active development. The implementation of creating the cluster may change slightly as the tool evolves, but the overall implementation should be pretty stable.

#### Note:

Any commands under `kubeadm alpha` are, by definition, supported on an alpha level.

## Objectives[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#objectives)

- Install a single control-plane Kubernetes cluster
- Install a Pod network on the cluster so that your Pods can talk to each other

## Instructions[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#instructions)

### Preparing the hosts[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#preparing-the-hosts)

#### Component installation[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#component-installation)

Install a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes) and kubeadm on all the hosts. For detailed instructions and other prerequisites, see [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).

#### Note:

If you have already installed kubeadm, see the first two steps of the [Upgrading Linux nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/) document for instructions on how to upgrade kubeadm.

When you upgrade, the kubelet restarts every few seconds as it waits in a crashloop for kubeadm to tell it what to do. This crashloop is expected and normal. After you initialize your control-plane, the kubelet runs normally.

#### Network setup[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#network-setup)

kubeadm similarly to other Kubernetes components tries to find a usable IP on the network interfaces associated with a default gateway on a host. Such an IP is then used for the advertising and/or listening performed by a component.

To find out what this IP is on a Linux host you can use:

```shell
ip route show # Look for a line starting with "default via"
```

#### Note:

If two or more default gateways are present on the host, a Kubernetes component will try to use the first one it encounters that has a suitable global unicast IP address. While making this choice, the exact ordering of gateways might vary between different operating systems and kernel versions.

Kubernetes components do not accept custom network interface as an option, therefore a custom IP address must be passed as a flag to all components instances that need such a custom configuration.

#### Note:

If the host does not have a default gateway and if a custom IP address is not passed to a Kubernetes component, the component may exit with an error.

To configure the API server advertise address for control plane nodes created with both `init` and `join`, the flag `--apiserver-advertise-address` can be used. Preferably, this option can be set in the [kubeadm API](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta4/) as `InitConfiguration.localAPIEndpoint` and `JoinConfiguration.controlPlane.localAPIEndpoint`.

For kubelets on all nodes, the `--node-ip` option can be passed in `.nodeRegistration.kubeletExtraArgs` inside a kubeadm configuration file (`InitConfiguration` or `JoinConfiguration`).

For dual-stack see [Dual-stack support with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/).

The IP addresses that you assign to control plane components become part of their X.509 certificates' subject alternative name fields. Changing these IP addresses would require signing new certificates and restarting the affected components, so that the change in certificate files is reflected. See [Manual certificate renewal](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#manual-certificate-renewal) for more details on this topic.

#### Warning:

The Kubernetes project recommends against this approach (configuring all component instances with custom IP addresses). Instead, the Kubernetes maintainers recommend to setup the host network, so that the default gateway IP is the one that Kubernetes components auto-detect and use. On Linux nodes, you can use commands such as `ip route` to configure networking; your operating system might also provide higher level network management tools. If your node's default gateway is a public IP address, you should configure packet filtering or other security measures that protect the nodes and your cluster.

### Preparing the required container images[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#preparing-the-required-container-images)

This step is optional and only applies in case you wish `kubeadm init` and `kubeadm join` to not download the default container images which are hosted at `registry.k8s.io`.

Kubeadm has commands that can help you pre-pull the required images when creating a cluster without an internet connection on its nodes. See [Running kubeadm without an internet connection](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#without-internet-connection) for more details.

Kubeadm allows you to use a custom image repository for the required images. See [Using custom images](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#custom-images) for more details.

### Initializing your control-plane node[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node)

The control-plane node is the machine where the control plane components run, including [etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/) (the cluster database) and the [API Server](https://kubernetes.io/docs/concepts/architecture/#kube-apiserver) (which the [kubectl](https://kubernetes.io/docs/reference/kubectl/) command line tool communicates with).

1. (Recommended) If you have plans to upgrade this single control-plane `kubeadm` cluster to [high availability](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) you should specify the `--control-plane-endpoint` to set the shared endpoint for all control-plane nodes. Such an endpoint can be either a DNS name or an IP address of a load-balancer.
2. Choose a Pod network add-on, and verify whether it requires any arguments to be passed to `kubeadm init`. Depending on which third-party provider you choose, you might need to set the `--pod-network-cidr` to a provider-specific value. See [Installing a Pod network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network).
3. (Optional) `kubeadm` tries to detect the container runtime by using a list of well known endpoints. To use different container runtime or if there are more than one installed on the provisioned node, specify the `--cri-socket` argument to `kubeadm`. See [Installing a runtime](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime).

To initialize the control-plane node run:

```bash
kubeadm init <args>
```

### Considerations about apiserver-advertise-address and ControlPlaneEndpoint[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#considerations-about-apiserver-advertise-address-and-controlplaneendpoint)

While `--apiserver-advertise-address` can be used to set the advertised address for this particular control-plane node's API server, `--control-plane-endpoint` can be used to set the shared endpoint for all control-plane nodes.

`--control-plane-endpoint` allows both IP addresses and DNS names that can map to IP addresses. Please contact your network administrator to evaluate possible solutions with respect to such mapping.

Here is an example mapping:

```
192.168.0.102 cluster-endpoint
```

Where `192.168.0.102` is the IP address of this node and `cluster-endpoint` is a custom DNS name that maps to this IP. This will allow you to pass `--control-plane-endpoint=cluster-endpoint` to `kubeadm init` and pass the same DNS name to `kubeadm join`. Later you can modify `cluster-endpoint` to point to the address of your load-balancer in a high availability scenario.

Turning a single control plane cluster created without `--control-plane-endpoint` into a highly available cluster is not supported by kubeadm.

### More information[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#more-information)

For more information about `kubeadm init` arguments, see the [kubeadm reference guide](https://kubernetes.io/docs/reference/setup-tools/kubeadm/).

To configure `kubeadm init` with a configuration file see [Using kubeadm init with a configuration file](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file).

To customize control plane components, including optional IPv6 assignment to liveness probe for control plane components and etcd server, provide extra arguments to each component as documented in [custom arguments](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/control-plane-flags/).

To reconfigure a cluster that has already been created see [Reconfiguring a kubeadm cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-reconfigure/).

To run `kubeadm init` again, you must first [tear down the cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down).

If you join a node with a different architecture to your cluster, make sure that your deployed DaemonSets have container image support for this architecture.

`kubeadm init` first runs a series of prechecks to ensure that the machine is ready to run Kubernetes. These prechecks expose warnings and exit on errors. `kubeadm init` then downloads and installs the cluster control plane components. This may take several minutes. After it finishes you should see:

```none
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

To make kubectl work for your non-root user, run these commands, which are also part of the `kubeadm init` output:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Alternatively, if you are the `root` user, you can run:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### Warning:

The kubeconfig file `admin.conf` that `kubeadm init` generates contains a certificate with `Subject: O = kubeadm:cluster-admins, CN = kubernetes-admin`. The group `kubeadm:cluster-admins` is bound to the built-in `cluster-admin` ClusterRole. Do not share the `admin.conf` file with anyone.

`kubeadm init` generates another kubeconfig file `super-admin.conf` that contains a certificate with `Subject: O = system:masters, CN = kubernetes-super-admin`. `system:masters` is a break-glass, super user group that bypasses the authorization layer (for example RBAC). Do not share the `super-admin.conf` file with anyone. It is recommended to move the file to a safe location.

See [Generating kubeconfig files for additional users](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubeconfig-additional-users) on how to use `kubeadm kubeconfig user` to generate kubeconfig files for additional users.

Make a record of the `kubeadm join` command that `kubeadm init` outputs. You need this command to [join nodes to your cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes).

The token is used for mutual authentication between the control-plane node and the joining nodes. The token included here is secret. Keep it safe, because anyone with this token can add authenticated nodes to your cluster. These tokens can be listed, created, and deleted with the `kubeadm token` command. See the [kubeadm reference guide](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/).

### Installing a Pod network add-on[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)

#### Caution:

This section contains important information about networking setup and deployment order. Read all of this advice carefully before proceeding.

**You must deploy a [Container Network Interface](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) (CNI) based Pod network add-on so that your Pods can communicate with each other. Cluster DNS (CoreDNS) will not start up before a network is installed.**

- Take care that your Pod network must not overlap with any of the host networks: you are likely to see problems if there is any overlap. (If you find a collision between your network plugin's preferred Pod network and some of your host networks, you should think of a suitable CIDR block to use instead, then use that during `kubeadm init` with `--pod-network-cidr` and as a replacement in your network plugin's YAML).
    
- By default, `kubeadm` sets up your cluster to use and enforce use of [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (role based access control). Make sure that your Pod network plugin supports RBAC, and so do any manifests that you use to deploy it.
    
- If you want to use IPv6--either dual-stack, or single-stack IPv6 only networking--for your cluster, make sure that your Pod network plugin supports IPv6. IPv6 support was added to CNI in [v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0).
    

#### Note:

Kubeadm should be CNI agnostic and the validation of CNI providers is out of the scope of our current e2e testing. If you find an issue related to a CNI plugin you should log a ticket in its respective issue tracker instead of the kubeadm or kubernetes issue trackers.

Several external projects provide Kubernetes Pod networks using CNI, some of which also support [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

See a list of add-ons that implement the [Kubernetes networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model).

Please refer to the [Installing Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy) page for a non-exhaustive list of networking addons supported by Kubernetes. You can install a Pod network add-on with the following command on the control-plane node or a node that has the kubeconfig credentials:

```bash
kubectl apply -f <add-on.yaml>
```

#### Note:

Only a few CNI plugins support Windows. More details and setup instructions can be found in [Adding Windows worker nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/#network-config).

You can install only one Pod network per cluster.

Once a Pod network has been installed, you can confirm that it is working by checking that the CoreDNS Pod is `Running` in the output of `kubectl get pods --all-namespaces`. And once the CoreDNS Pod is up and running, you can continue by joining your nodes.

If your network is not working or CoreDNS is not in the `Running` state, check out the [troubleshooting guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/) for `kubeadm`.

### Managed node labels[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#managed-node-labels)

By default, kubeadm enables the [NodeRestriction](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction) admission controller that restricts what labels can be self-applied by kubelets on node registration. The admission controller documentation covers what labels are permitted to be used with the kubelet `--node-labels` option. The `node-role.kubernetes.io/control-plane` label is such a restricted label and kubeadm manually applies it using a privileged client after a node has been created. To do that manually you can do the same by using `kubectl label` and ensure it is using a privileged kubeconfig such as the kubeadm managed `/etc/kubernetes/admin.conf`.

### Control plane node isolation[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation)

By default, your cluster will not schedule Pods on the control plane nodes for security reasons. If you want to be able to schedule Pods on the control plane nodes, for example for a single machine Kubernetes cluster, run:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

The output will look something like:

```
node "test-01" untainted
...
```

This will remove the `node-role.kubernetes.io/control-plane:NoSchedule` taint from any nodes that have it, including the control plane nodes, meaning that the scheduler will then be able to schedule Pods everywhere.

Additionally, you can execute the following command to remove the [`node.kubernetes.io/exclude-from-external-load-balancers`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-exclude-from-external-load-balancers) label from the control plane node, which excludes it from the list of backend servers:

```bash
kubectl label nodes --all node.kubernetes.io/exclude-from-external-load-balancers-
```

### Adding more control plane nodes[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#adding-more-control-plane-nodes)

See [Creating Highly Available Clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) for steps on creating a high availability kubeadm cluster by adding more control plane nodes.

### Adding worker nodes[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes)

The worker nodes are where your workloads run.

The following pages show how to add Linux and Windows worker nodes to the cluster by using the `kubeadm join` command:

- [Adding Linux worker nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-linux-nodes/)
- [Adding Windows worker nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/)

### (Optional) Controlling your cluster from machines other than the control-plane node[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#optional-controlling-your-cluster-from-machines-other-than-the-control-plane-node)

In order to get a kubectl on some other computer (e.g. laptop) to talk to your cluster, you need to copy the administrator kubeconfig file from your control-plane node to your workstation like this:

```bash
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

#### Note:

The example above assumes SSH access is enabled for root. If that is not the case, you can copy the `admin.conf` file to be accessible by some other user and `scp` using that other user instead.

The `admin.conf` file gives the user _superuser_ privileges over the cluster. This file should be used sparingly. For normal users, it's recommended to generate an unique credential to which you grant privileges. You can do this with the `kubeadm kubeconfig user --client-name <CN>` command. That command will print out a KubeConfig file to STDOUT which you should save to a file and distribute to your user. After that, grant privileges by using `kubectl create (cluster)rolebinding`.

### (Optional) Proxying API Server to localhost[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#optional-proxying-api-server-to-localhost)

If you want to connect to the API Server from outside the cluster, you can use `kubectl proxy`:

```bash
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

You can now access the API Server locally at `http://localhost:8001/api/v1`

## Clean up[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down)

If you used disposable servers for your cluster, for testing, you can switch those off and do no further clean up. You can use `kubectl config delete-cluster` to delete your local references to the cluster.

However, if you want to deprovision your cluster more cleanly, you should first [drain the node](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#drain) and make sure that the node is empty, then deconfigure the node.

### Remove the node[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#remove-the-node)

Talking to the control-plane node with the appropriate credentials, run:

```bash
kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets
```

Before removing the node, reset the state installed by `kubeadm`:

```bash
kubeadm reset
```

The reset process does not reset or clean up iptables rules or IPVS tables. If you wish to reset iptables, you must do so manually:

```bash
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

If you want to reset the IPVS tables, you must run the following command:

```bash
ipvsadm -C
```

Now remove the node:

```bash
kubectl delete node <node name>
```

If you wish to start over, run `kubeadm init` or `kubeadm join` with the appropriate arguments.

### Clean up the control plane[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#clean-up-the-control-plane)

You can use `kubeadm reset` on the control plane host to trigger a best-effort clean up.

See the [`kubeadm reset`](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-reset/) reference documentation for more information about this subcommand and its options.

## Version skew policy[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

While kubeadm allows version skew against some components that it manages, it is recommended that you match the kubeadm version with the versions of the control plane components, kube-proxy and kubelet.

### kubeadm's skew against the Kubernetes version[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#kubeadm-s-skew-against-the-kubernetes-version)

kubeadm can be used with Kubernetes components that are the same version as kubeadm or one version older. The Kubernetes version can be specified to kubeadm by using the `--kubernetes-version` flag of `kubeadm init` or the [`ClusterConfiguration.kubernetesVersion`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta4/) field when using `--config`. This option will control the versions of kube-apiserver, kube-controller-manager, kube-scheduler and kube-proxy.

Example:

- kubeadm is at 1.31
- `kubernetesVersion` must be at 1.31 or 1.30

### kubeadm's skew against the kubelet[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#kubeadm-s-skew-against-the-kubelet)

Similarly to the Kubernetes version, kubeadm can be used with a kubelet version that is the same version as kubeadm or three versions older.

Example:

- kubeadm is at 1.31
- kubelet on the host must be at 1.31, 1.30, 1.29 or 1.28

### kubeadm's skew against kubeadm[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#kubeadm-s-skew-against-kubeadm)

There are certain limitations on how kubeadm commands can operate on existing nodes or whole clusters managed by kubeadm.

If new nodes are joined to the cluster, the kubeadm binary used for `kubeadm join` must match the last version of kubeadm used to either create the cluster with `kubeadm init` or to upgrade the same node with `kubeadm upgrade`. Similar rules apply to the rest of the kubeadm commands with the exception of `kubeadm upgrade`.

Example for `kubeadm join`:

- kubeadm version 1.31 was used to create a cluster with `kubeadm init`
- Joining nodes must use a kubeadm binary that is at version 1.31

Nodes that are being upgraded must use a version of kubeadm that is the same MINOR version or one MINOR version newer than the version of kubeadm used for managing the node.

Example for `kubeadm upgrade`:

- kubeadm version 1.30 was used to create or upgrade the node
- The version of kubeadm used for upgrading the node must be at 1.30 or 1.31

To learn more about the version skew between the different Kubernetes component see the [Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/).

## Limitations[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#limitations)

### Cluster resilience[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#resilience)

The cluster created here has a single control-plane node, with a single etcd database running on it. This means that if the control-plane node fails, your cluster may lose data and may need to be recreated from scratch.

Workarounds:

- Regularly [back up etcd](https://etcd.io/docs/v3.5/op-guide/recovery/). The etcd data directory configured by kubeadm is at `/var/lib/etcd` on the control-plane node.
    
- Use multiple control-plane nodes. You can read [Options for Highly Available topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/) to pick a cluster topology that provides [high-availability](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/).
    

### Platform compatibility[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#multi-platform)

kubeadm deb/rpm packages and binaries are built for amd64, arm (32-bit), arm64, ppc64le, and s390x following the [multi-platform proposal](https://git.k8s.io/design-proposals-archive/multi-platform.md).

Multiplatform container images for the control plane and addons are also supported since v1.12.

Only some of the network providers offer solutions for all platforms. Please consult the list of network providers above or the documentation from each provider to figure out whether the provider supports your chosen platform.

## Troubleshooting[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#troubleshooting)

If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/).

## What's next[](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#what-s-next)

- Verify that your cluster is running properly with [Sonobuoy](https://github.com/heptio/sonobuoy)
- See [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) for details about upgrading your cluster using `kubeadm`.
- Learn about advanced `kubeadm` usage in the [kubeadm reference documentation](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
- Learn more about Kubernetes [concepts](https://kubernetes.io/docs/concepts/) and [`kubectl`](https://kubernetes.io/docs/reference/kubectl/).
- See the [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) page for a bigger list of Pod network add-ons.
- See the [list of add-ons](https://kubernetes.io/docs/concepts/cluster-administration/addons/) to explore other add-ons, including tools for logging, monitoring, network policy, visualization & control of your Kubernetes cluster.
- Configure how your cluster handles logs for cluster events and from applications running in Pods. See [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) for an overview of what is involved.