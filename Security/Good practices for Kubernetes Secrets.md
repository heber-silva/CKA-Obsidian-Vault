[Docs](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)

Principles and practices for good Secret management for cluster administrators and application developers.

In Kubernetes, a Secret is an object that stores sensitive information, such as passwords, OAuth tokens, and SSH keys.

Secrets give you more control over how sensitive information is used and reduces the risk of accidental exposure. Secret values are encoded as base64 strings and are stored unencrypted by default, but can be configured to be [encrypted at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted).

A [[Pod]] can reference the Secret in a variety of ways, such as in a volume mount or as an environment variable. Secrets are designed for confidential data and [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) are designed for non-confidential data.

The following good practices are intended for both cluster administrators and application developers. Use these guidelines to improve the security of your sensitive information in Secret objects, as well as to more effectively manage your Secrets.

## Cluster administrators[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#cluster-administrators)

This section provides good practices that cluster administrators can use to improve the security of confidential information in the cluster.

### Configure encryption at rest[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#configure-encryption-at-rest)

By default, Secret objects are stored unencrypted in [etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/). You should configure encryption of your Secret data in `etcd`. For instructions, refer to [Encrypt Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).

### Configure least-privilege access to Secrets[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#least-privilege-secrets)

When planning your access control mechanism, such as Kubernetes [Role-based Access Control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) [(RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/), consider the following guidelines for access to `Secret` objects. You should also follow the other guidelines in [[Role Based Access Control Good Practices]].

- **Components**: Restrict `watch` or `list` access to only the most privileged, system-level components. Only grant `get` access for Secrets if the component's normal behavior requires it.
- **Humans**: Restrict `get`, `watch`, or `list` access to Secrets. Only allow cluster administrators to access `etcd`. This includes read-only access. For more complex access control, such as restricting access to Secrets with specific annotations, consider using third-party authorization mechanisms.

#### Caution:

Granting `list` access to Secrets implicitly lets the subject fetch the contents of the Secrets.

A user who can create a Pod that uses a Secret can also see the value of that Secret. Even if cluster policies do not allow a user to read the Secret directly, the same user could have access to run a Pod that then exposes the Secret. You can detect or limit the impact caused by Secret data being exposed, either intentionally or unintentionally, by a user with this access. Some recommendations include:

- Use short-lived Secrets
- Implement audit rules that alert on specific events, such as concurrent reading of multiple Secrets by a single user

#### Additional ServiceAccount annotations for Secret management[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#additional-serviceaccount-annotations-for-secret-management)

You can also use the `kubernetes.io/enforce-mountable-secrets` annotation on a ServiceAccount to enforce specific rules on how Secrets are used in a Pod. For more details, see the [documentation on this annotation](https://kubernetes.io/docs/reference/labels-annotations-taints/#enforce-mountable-secrets).

### Improve etcd management policies[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#improve-etcd-management-policies)

Consider wiping or shredding the durable storage used by `etcd` once it is no longer in use.

If there are multiple `etcd` instances, configure encrypted SSL/TLS communication between the instances to protect the Secret data in transit.

### Configure access to external Secrets[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#configure-access-to-external-secrets)

**Note:** This section links to third party projects that provide functionality required by Kubernetes. The Kubernetes project authors aren't responsible for these projects, which are listed alphabetically. To add a project to this list, read the [content guide](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content) before submitting a change. [More information.](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#third-party-content-disclaimer)

You can use third-party Secrets store providers to keep your confidential data outside your cluster and then configure Pods to access that information. The [Kubernetes Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/) is a DaemonSet that lets the kubelet retrieve Secrets from external stores, and mount the Secrets as a volume into specific Pods that you authorize to access the data.

For a list of supported providers, refer to [Providers for the Secret Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#provider-for-the-secrets-store-csi-driver).

## Developers[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#developers)

This section provides good practices for developers to use to improve the security of confidential data when building and deploying Kubernetes resources.

### Restrict Secret access to specific containers[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#restrict-secret-access-to-specific-containers)

If you are defining multiple containers in a Pod, and only one of those containers needs access to a Secret, define the volume mount or environment variable configuration so that the other containers do not have access to that Secret.

### Protect Secret data after reading[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#protect-secret-data-after-reading)

Applications still need to protect the value of confidential information after reading it from an environment variable or volume. For example, your application must avoid logging the secret data in the clear or transmitting it to an untrusted party.

### Avoid sharing Secret manifests[](https://kubernetes.io/docs/concepts/security/secrets-good-practices/#avoid-sharing-secret-manifests)

If you configure a Secret through a [manifest](https://kubernetes.io/docs/reference/glossary/?all=true#term-manifest), with the secret data encoded as base64, sharing this file or checking it in to a source repository means the secret is available to everyone who can read the manifest.

#### Caution:

Base64 encoding is _not_ an encryption method, it provides no additional confidentiality over plain text.

Items on this page refer to third party products or projects that provide functionality required by Kubernetes. The Kubernetes project authors aren't responsible for those third-party products or projects. See the [CNCF website guidelines](https://github.com/cncf/foundation/blob/master/website-guidelines.md) for more details.

You should read the [content guide](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content) before proposing a change that adds an extra third-party link.