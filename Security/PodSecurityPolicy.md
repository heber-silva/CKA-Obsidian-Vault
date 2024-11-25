[Doc](https://kubernetes.io/docs/concepts/security/pod-security-policy/)
#### Removed feature

PodSecurityPolicy was [deprecated](https://kubernetes.io/blog/2021/04/08/kubernetes-1-21-release-announcement/#podsecuritypolicy-deprecation) in Kubernetes v1.21, and removed from Kubernetes in v1.25.

Instead of using PodSecurityPolicy, you can enforce similar restrictions on Pods using either or both:

- [Pod Security Admission](Pod%20Security%20Admission.md)
- a 3rd party admission plugin, that you deploy and configure yourself

For a migration guide, see [Migrate from PodSecurityPolicy to the Built-In PodSecurity Admission Controller](https://kubernetes.io/docs/tasks/configure-pod-container/migrate-from-psp/). For more information on the removal of this API, see [PodSecurityPolicy Deprecation: Past, Present, and Future](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/).

If you are not running Kubernetes v1.31, check the documentation for your version of Kubernetes.