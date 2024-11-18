#prova 
Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.

In Kubernetes, a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) is a method for exposing a network application that is running as one or more [[Pod]] in your cluster.

A key aim of Services in Kubernetes is that you don't need to modify your existing application to use an unfamiliar service discovery mechanism. You can run code in Pods, whether this is a code designed for a cloud-native world, or an older app you've containerized. You use a Service to make that set of Pods available on the network so that clients can interact with it.

If you use a [[Deployment]] to run your app, that Deployment can create and destroy Pods dynamically. From one moment to the next, you don't know how many of those Pods are working and healthy; you might not even know what those healthy Pods are named. Kubernetes [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are created and destroyed to match the desired state of your cluster. Pods are ephemeral resources (you should not expect that an individual Pod is reliable and durable).

Each Pod gets its own IP address (Kubernetes expects network plugins to ensure this). For a given Deployment in your cluster, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them "backends") provides functionality to other Pods (call them "frontends") inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?

Enter _Services_.

## Services in Kubernetes[](https://kubernetes.io/docs/concepts/services-networking/service/#services-in-kubernetes)

The Service API, part of Kubernetes, is an abstraction to help you expose groups of Pods over a network. Each Service object defines a logical set of endpoints (usually these endpoints are Pods) along with a policy about how to make those pods accessible.

For example, consider a stateless image-processing backend which is running with 3 replicas. Those replicas are fungible—frontends do not care which backend they use. While the actual Pods that compose the backend set may change, the frontend clients should not need to be aware of that, nor should they need to keep track of the set of backends themselves.

The Service abstraction enables this decoupling.

The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) that you define. To learn about other ways to define Service endpoints, see [Services _without_ selectors](https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors).

If your workload speaks HTTP, you might choose to use an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to control how web traffic reaches that workload. Ingress is not a Service type, but it acts as the entry point for your cluster. An Ingress lets you consolidate your routing rules into a single resource, so that you can expose multiple components of your workload, running separately in your cluster, behind a single listener.

The [Gateway](https://gateway-api.sigs.k8s.io/#what-is-the-gateway-api) API for Kubernetes provides extra capabilities beyond Ingress and Service. You can add Gateway to your cluster - it is a family of extension APIs, implemented using [CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) - and then use these to configure access to network services that are running in your cluster.

### Cloud-native service discovery[](https://kubernetes.io/docs/concepts/services-networking/service/#cloud-native-service-discovery)

If you're able to use Kubernetes APIs for service discovery in your application, you can query the [API server](https://kubernetes.io/docs/concepts/architecture/#kube-apiserver) for matching EndpointSlices. Kubernetes updates the EndpointSlices for a Service whenever the set of Pods in a Service changes.

For non-native applications, Kubernetes offers ways to place a network port or load balancer in between your application and the backend Pods.

Either way, your workload can use these [service discovery](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services) mechanisms to find the target it wants to connect to.

## Service type[](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, one that's accessible from outside of your cluster.

Kubernetes Service types allow you to specify what kind of Service you want.

The available `type` values and their behaviors are:

[`ClusterIP`](https://kubernetes.io/docs/concepts/services-networking/service/#type-clusterip)

Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a `type` for a Service. You can expose the Service to the public internet using an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) or a [Gateway](https://gateway-api.sigs.k8s.io/).

[`NodePort`](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)

Exposes the Service on each Node's IP at a static port (the `NodePort`). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of `type: ClusterIP`.

[`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)

Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.

[`ExternalName`](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)

Maps the Service to the contents of the `externalName` field (for example, to the hostname `api.foo.bar.example`). The mapping configures your cluster's DNS server to return a `CNAME` record with that external hostname value. No proxying of any kind is set up.

The `type` field in the Service API is designed as nested functionality - each level adds to the previous. However there is an exception to this nested design. You can define a `LoadBalancer` Service by [disabling the load balancer `NodePort` allocation](https://kubernetes.io/docs/concepts/services-networking/service/#load-balancer-nodeport-allocation).