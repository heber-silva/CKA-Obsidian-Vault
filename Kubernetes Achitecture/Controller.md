Control loops that watch the state of your [cluster](https://kubernetes.io/docs/reference/glossary/?all=true#term-cluster), then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state

### [**Controller pattern**](https://kubernetes.io/docs/concepts/architecture/controller/#controller-pattern)

A controller tracks at least one Kubernetes resource type. These [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects) have a spec field that represents the desired state. The controller(s) for that resource are responsible for making the current state come closer to that desired state

[**Control via API server**](https://kubernetes.io/docs/concepts/architecture/controller/#control-via-api-server)

[**Direct control**](https://kubernetes.io/docs/concepts/architecture/controller/#direct-control)

### [**Running controllers**](https://kubernetes.io/docs/concepts/architecture/controller/#running-controllers)

Kubernetes comes with a set of built-in controllers that run inside the [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/). These built-in controllers provide important core behaviors.