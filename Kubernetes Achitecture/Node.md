#prova 
[Doc](https://kubernetes.io/docs/concepts/architecture/nodes/)
There are two main ways to have Nodes added to the [API server](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver):

1. The kubelet on a node self-registers to the control plane
2. You (or another human user) manually add a Node object

Kubernetes creates a Node object internally (the representation). Kubernetes checks that a kubelet has registered to the API server that matches the `metadata.name` field of the Node. If the node is healthy (i.e. all necessary services are running), then it is eligible to run a [Pod](Pod.md). Otherwise, that node is ignored for any cluster activity until it becomes healthy.

### **Node name uniqueness:**

The [name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#names) identifies a Node. Two Nodes cannot have the same name at the same time. Kubernetes also assumes that a resource with the same name is the same object.

### [Self-registration of Nodes:](https://kubernetes.io/docs/concepts/architecture/nodes/#self-registration-of-nodes)

When the kubelet flag `--register-node` is true (the default), the kubelet will attempt to register itself with the API server. This is the preferred pattern, used by most distros.

### [**Manual Node administration:**](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration)

You can create and modify Node objects using [kubectl](https://kubernetes.io/docs/reference/kubectl/).

### [Node status](https://kubernetes.io/docs/concepts/architecture/nodes/#node-status):

A Node's status contains the following information:

- [Addresses](https://kubernetes.io/docs/reference/node/node-status/#addresses)
- [Conditions](https://kubernetes.io/docs/reference/node/node-status/#condition)
- [Capacity and Allocatable](https://kubernetes.io/docs/reference/node/node-status/#capacity)
- [Info](https://kubernetes.io/docs/reference/node/node-status/#info)

You can use `kubectl` to view a Node's status and other details:

### [**Node heartbeats](https://kubernetes.io/docs/concepts/architecture/nodes/#node-heartbeats):**

Heartbeats, sent by Kubernetes nodes, help your cluster determine the availability of each node, and to take action when failures are detected.

For nodes there are two forms of heartbeats:

- Updates to the [`.status`](https://kubernetes.io/docs/reference/node/node-status/) of a Node;
- [Lease](https://kubernetes.io/docs/concepts/architecture/leases/) objects within the `kube-node-lease` [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces). Each Node has an associated Lease object.

### [Node controller:](https://kubernetes.io/docs/concepts/architecture/nodes/#node-controller)

The node [controller](https://kubernetes.io/docs/concepts/architecture/controller/) is a Kubernetes control plane component that manages various aspects of nodes.

The node controller has multiple roles in a node's life.

1. Assigning a CIDR block to the node when it is registered (if CIDR assignment is turned on);
2. Keeping the node controller's internal list of nodes up to date with the cloud provider's list of available machines;
3. Monitoring the nodes' health.

### [Communication between Nodes and the Control Plane](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/):

"hub-and-spoke" API pattern;

[**Node to Control Plane](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#node-to-control-plane):**

The API server is configured to listen for remote connections on a secure HTTPS port (typically 443) with one or more forms of client [authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) enabled. One or more forms of [authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) should be enabled, especially if [anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests) or [service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens) are allowed.

Nodes → public root [certificate](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) → Cluster → API server along credentials

See [kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) for automated provisioning of kubelet client certificates.

[**Control plane to node:**](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#control-plane-to-node)

- API server to the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) process; [link](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#api-server-to-kubelet)
- API server to any node, pod, or service through the API server's _proxy_ functionality. [link](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#api-server-to-nodes-pods-and-services)

[**SSH tunnels**](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#ssh-tunnels)

[**Konnectivity service**](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#konnectivity-service)