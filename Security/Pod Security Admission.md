[Docs](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
An overview of the Pod Security Admission Controller, which can enforce the Pod Security Standards.

FEATURE STATE: `Kubernetes v1.25 [stable]`

The Kubernetes [Pod Security Standards](Pod%20Security%20Standards.md) define different isolation levels for Pods. These standards let you define how you want to restrict the behavior of pods in a clear, consistent fashion.

Kubernetes offers a built-in _Pod Security_ [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) to enforce the Pod Security Standards. Pod security restrictions are applied at the [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces) level when pods are created.

### Built-in Pod Security admission enforcement[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#built-in-pod-security-admission-enforcement)

This page is part of the documentation for Kubernetes v1.31. If you are running a different version of Kubernetes, consult the documentation for that release.

## Pod Security levels[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#pod-security-levels)

Pod Security admission places requirements on a Pod's [Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) and other related fields according to the three levels defined by the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/): `privileged`, `baseline`, and `restricted`. Refer to the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) page for an in-depth look at those requirements.

## Pod Security Admission labels for namespaces[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#pod-security-admission-labels-for-namespaces)

Once the feature is enabled or the webhook is installed, you can configure namespaces to define the admission control mode you want to use for pod security in each namespace. Kubernetes defines a set of [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) that you can set to define which of the predefined Pod Security Standard levels you want to use for a namespace. The label you select defines what action the [](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) takes if a potential violation is detected:

|Mode|Description|
|---|---|
|**enforce**|Policy violations will cause the pod to be rejected.|
|**audit**|Policy violations will trigger the addition of an audit annotation to the event recorded in the [audit log](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/), but are otherwise allowed.|
|**warn**|Policy violations will trigger a user-facing warning, but are otherwise allowed.|

A namespace can configure any or all modes, or even set a different level for different modes.

For each mode, there are two labels that determine the policy used:

```yaml
# The per-mode level label indicates which policy level to apply for the mode.
#
# MODE must be one of `enforce`, `audit`, or `warn`.
# LEVEL must be one of `privileged`, `baseline`, or `restricted`.
pod-security.kubernetes.io/<MODE>: <LEVEL>

# Optional: per-mode version label that can be used to pin the policy to the
# version that shipped with a given Kubernetes minor version (for example v1.31).
#
# MODE must be one of `enforce`, `audit`, or `warn`.
# VERSION must be a valid Kubernetes minor version, or `latest`.
pod-security.kubernetes.io/<MODE>-version: <VERSION>
```

Check out [Enforce Pod Security Standards with Namespace Labels](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/) to see example usage.

## Workload resources and Pod templates[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#workload-resources-and-pod-templates)

Pods are often created indirectly, by creating a [workload object](https://kubernetes.io/docs/concepts/workloads/controllers/) such as a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/). The workload object defines a _Pod template_ and a [controller](https://kubernetes.io/docs/concepts/architecture/controller/) for the workload resource creates Pods based on that template. To help catch violations early, both the audit and warning modes are applied to the workload resources. However, enforce mode is **not** applied to workload resources, only to the resulting pod objects.

## Exemptions[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#exemptions)

You can define _exemptions_ from pod security enforcement in order to allow the creation of pods that would have otherwise been prohibited due to the policy associated with a given namespace. Exemptions can be statically configured in the [](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/#configure-the-admission-controller).

Exemptions must be explicitly enumerated. Requests meeting exemption criteria are _ignored_ by the Admission Controller (all `enforce`, `audit` and `warn` behaviors are skipped). Exemption dimensions include:

- **Usernames:** requests from users with an exempt authenticated (or impersonated) username are ignored.
- **RuntimeClassNames:** pods and [](https://kubernetes.io/docs/concepts/security/pod-security-admission/#workload-resources-and-pod-templates) specifying an exempt runtime class name are ignored.
- **Namespaces:** pods and [](https://kubernetes.io/docs/concepts/security/pod-security-admission/#workload-resources-and-pod-templates) in an exempt namespace are ignored.

#### Caution:

Most pods are created by a controller in response to a [](https://kubernetes.io/docs/concepts/security/pod-security-admission/#workload-resources-and-pod-templates), meaning that exempting an end user will only exempt them from enforcement when creating pods directly, but not when creating a workload resource. Controller service accounts (such as `system:serviceaccount:kube-system:replicaset-controller`) should generally not be exempted, as doing so would implicitly exempt any user that can create the corresponding workload resource.

Updates to the following pod fields are exempt from policy checks, meaning that if a pod update request only changes these fields, it will not be denied even if the pod is in violation of the current policy level:

- Any metadata updates **except** changes to the seccomp or AppArmor annotations:
    - `seccomp.security.alpha.kubernetes.io/pod` (deprecated)
    - `container.seccomp.security.alpha.kubernetes.io/*` (deprecated)
    - `container.apparmor.security.beta.kubernetes.io/*` (deprecated)
- Valid updates to `.spec.activeDeadlineSeconds`
- Valid updates to `.spec.tolerations`

## Metrics[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#metrics)

Here are the Prometheus metrics exposed by kube-apiserver:

- `pod_security_errors_total`: This metric indicates the number of errors preventing normal evaluation. Non-fatal errors may result in the latest restricted profile being used for enforcement.
- `pod_security_evaluations_total`: This metric indicates the number of policy evaluations that have occurred, not counting ignored or exempt requests during exporting.
- `pod_security_exemptions_total`: This metric indicates the number of exempt requests, not counting ignored or out of scope requests.

## What's next[](https://kubernetes.io/docs/concepts/security/pod-security-admission/#what-s-next)

- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Enforcing Pod Security Standards](https://kubernetes.io/docs/setup/best-practices/enforcing-pod-security-standards/)
- [Enforce Pod Security Standards by Configuring the Built-in Admission Controller](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/)
- [Enforce Pod Security Standards with Namespace Labels](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/)

If you are running an older version of Kubernetes and want to upgrade to a version of Kubernetes that does not include PodSecurityPolicies, read [migrate from PodSecurityPolicy to the Built-In PodSecurity Admission Controller](https://kubernetes.io/docs/tasks/configure-pod-container/migrate-from-psp/).