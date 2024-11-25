Manage security and best-practices with policies.

Kubernetes policies are configurations that manage other configurations or runtime behaviors. Kubernetes offers various forms of policies, described below:

## Apply policies using API objects[](https://kubernetes.io/docs/concepts/policy/#apply-policies-using-api-objects)

Some API objects act as policies. Here are some examples:

- [NetworkPolicy](NetworkPolicy.md) can be used to restrict ingress and egress traffic for a workload.
- [LimitRanges](https://kubernetes.io/docs/concepts/policy/limit-range/) manage resource allocation constraints across different object kinds.
- [ResourceQuotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) limit resource consumption for a [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces).

## Apply policies using admission controllers[](https://kubernetes.io/docs/concepts/policy/#apply-policies-using-admission-controllers)

An [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) runs in the API server and can validate or mutate API requests. Some admission controllers act to apply policies. For example, the [AlwaysPullImages](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) admission controller modifies a new Pod to set the image pull policy to `Always`.

Kubernetes has several built-in admission controllers that are configurable via the API server `--enable-admission-plugins` flag.

Details on admission controllers, with the complete list of available admission controllers, are documented in a dedicated section:

- [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

## Apply policies using ValidatingAdmissionPolicy[](https://kubernetes.io/docs/concepts/policy/#apply-policies-using-validatingadmissionpolicy)

Validating admission policies allow configurable validation checks to be executed in the API server using the Common Expression Language (CEL). For example, a `ValidatingAdmissionPolicy` can be used to disallow use of the `latest` image tag.

A `ValidatingAdmissionPolicy` operates on an API request and can be used to block, audit, and warn users about non-compliant configurations.

Details on the `ValidatingAdmissionPolicy` API, with examples, are documented in a dedicated section:

- [Validating Admission Policy](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)

## Apply policies using dynamic admission control[](https://kubernetes.io/docs/concepts/policy/#apply-policies-using-dynamic-admission-control)

Dynamic admission controllers (or admission webhooks) run outside the API server as separate applications that register to receive webhooks requests to perform validation or mutation of API requests.

Dynamic admission controllers can be used to apply policies on API requests and trigger other policy-based workflows. A dynamic admission controller can perform complex checks including those that require retrieval of other cluster resources and external data. For example, an image verification check can lookup data from OCI registries to validate the container image signatures and attestations.

Details on dynamic admission control are documented in a dedicated section:

- [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)

### Implementations[](https://kubernetes.io/docs/concepts/policy/#implementations-admission-control)

**Note:** This section links to third party projects that provide functionality required by Kubernetes. The Kubernetes project authors aren't responsible for these projects, which are listed alphabetically. To add a project to this list, read the [content guide](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content) before submitting a change. [More information.](https://kubernetes.io/docs/concepts/policy/#third-party-content-disclaimer)

Dynamic Admission Controllers that act as flexible policy engines are being developed in the Kubernetes ecosystem, such as:

- [Kubewarden](https://github.com/kubewarden)
- [Kyverno](https://kyverno.io/)
- [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)
- [Polaris](https://polaris.docs.fairwinds.com/admission-controller/)

## Apply policies using Kubelet configurations[](https://kubernetes.io/docs/concepts/policy/#apply-policies-using-kubelet-configurations)

Kubernetes allows configuring the Kubelet on each worker node. Some Kubelet configurations act as policies:

- [Process ID limits and reservations](https://kubernetes.io/docs/concepts/policy/pid-limiting/) are used to limit and reserve allocatable PIDs.
- [Node Resource Managers](https://kubernetes.io/docs/concepts/policy/node-resource-managers/) can manage compute, memory, and device resources for latency-critical and high-throughput workloads.

Items on this page refer to third party products or projects that provide functionality required by Kubernetes. The Kubernetes project authors aren't responsible for those third-party products or projects. See the [CNCF website guidelines](https://github.com/cncf/foundation/blob/master/website-guidelines.md) for more details.

You should read the [content guide](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content) before proposing a change that adds an extra third-party link.