[Docs](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
A detailed look at the different policy levels defined in the Pod Security Standards.

The Pod Security Standards define three different _policies_ to broadly cover the security spectrum. These policies are _cumulative_ and range from highly-permissive to highly-restrictive. This guide outlines the requirements of each policy.

|Profile|Description|
|---|---|
|**Privileged**|Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.|
|**Baseline**|Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.|
|**Restricted**|Heavily restricted policy, following current Pod hardening best practices.|

## Profile Details[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#profile-details)

### Privileged[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#privileged)

**The _Privileged_ policy is purposely-open, and entirely unrestricted.** This type of policy is typically aimed at system- and infrastructure-level workloads managed by privileged, trusted users.

The Privileged policy is defined by an absence of restrictions. If you define a Pod where the Privileged security policy applies, the Pod you define is able to bypass typical container isolation mechanisms. For example, you can define a Pod that has access to the node's host network.

### Baseline[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline)

**The _Baseline_ policy is aimed at ease of adoption for common containerized workloads while preventing known privilege escalations.** This policy is targeted at application operators and developers of non-critical applications. The following listed controls should be enforced/disallowed:

#### Note:

In this table, wildcards (`*`) indicate all elements in a list. For example, `spec.containers[*].securityContext` refers to the Security Context object for _all defined containers_. If any of the listed containers fails to meet the requirements, the entire pod will fail validation.

|Control|Policy|
|---|---|
|HostProcess|Windows Pods offer the ability to run [HostProcess containers](https://kubernetes.io/docs/tasks/configure-pod-container/create-hostprocess-pod) which enables privileged access to the Windows host machine. Privileged access to the host is disallowed in the Baseline policy.<br><br>FEATURE STATE: `Kubernetes v1.26 [stable]`<br><br>**Restricted Fields**<br><br>- `spec.securityContext.windowsOptions.hostProcess`<br>- `spec.containers[*].securityContext.windowsOptions.hostProcess`<br>- `spec.initContainers[*].securityContext.windowsOptions.hostProcess`<br>- `spec.ephemeralContainers[*].securityContext.windowsOptions.hostProcess`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `false`|
|Host Namespaces|Sharing the host namespaces must be disallowed.<br><br>**Restricted Fields**<br><br>- `spec.hostNetwork`<br>- `spec.hostPID`<br>- `spec.hostIPC`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `false`|
|Privileged Containers|Privileged Pods disable most security mechanisms and must be disallowed.<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.privileged`<br>- `spec.initContainers[*].securityContext.privileged`<br>- `spec.ephemeralContainers[*].securityContext.privileged`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `false`|
|Capabilities|Adding additional capabilities beyond those listed below must be disallowed.<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.capabilities.add`<br>- `spec.initContainers[*].securityContext.capabilities.add`<br>- `spec.ephemeralContainers[*].securityContext.capabilities.add`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `AUDIT_WRITE`<br>- `CHOWN`<br>- `DAC_OVERRIDE`<br>- `FOWNER`<br>- `FSETID`<br>- `KILL`<br>- `MKNOD`<br>- `NET_BIND_SERVICE`<br>- `SETFCAP`<br>- `SETGID`<br>- `SETPCAP`<br>- `SETUID`<br>- `SYS_CHROOT`|
|HostPath Volumes|HostPath volumes must be forbidden.<br><br>**Restricted Fields**<br><br>- `spec.volumes[*].hostPath`<br><br>**Allowed Values**<br><br>- Undefined/nil|
|Host Ports|HostPorts should be disallowed entirely (recommended) or restricted to a known list<br><br>**Restricted Fields**<br><br>- `spec.containers[*](not%20supported%20by%20the%20built-in [Pod%20Security%20Admission%20controller))<br>- `0`|
|AppArmor|On supported hosts, the `RuntimeDefault` AppArmor profile is applied by default. The baseline policy should prevent overriding or disabling the default AppArmor profile, or restrict overrides to an allowed set of profiles.<br><br>**Restricted Fields**<br><br>- `spec.securityContext.appArmorProfile.type`<br>- `spec.containers[*].securityContext.appArmorProfile.type`<br>- `spec.initContainers[*].securityContext.appArmorProfile.type`<br>- `spec.ephemeralContainers[*].securityContext.appArmorProfile.type`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `RuntimeDefault`<br>- `Localhost`<br><br>---<br><br>- `metadata.annotations["container.apparmor.security.beta.kubernetes.io/*"]`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `runtime/default`<br>- `localhost/*`|
|SELinux|Setting the SELinux type is restricted, and setting a custom SELinux user or role option is forbidden.<br><br>**Restricted Fields**<br><br>- `spec.securityContext.seLinuxOptions.type`<br>- `spec.containers[*].securityContext.seLinuxOptions.type`<br>- `spec.initContainers[*].securityContext.seLinuxOptions.type`<br>- `spec.ephemeralContainers[*].securityContext.seLinuxOptions.type`<br><br>**Allowed Values**<br><br>- Undefined/""<br>- `container_t`<br>- `container_init_t`<br>- `container_kvm_t`<br>- `container_engine_t` (since Kubernetes 1.31)<br><br>---<br><br>**Restricted Fields**<br><br>- `spec.securityContext.seLinuxOptions.user`<br>- `spec.containers[*].securityContext.seLinuxOptions.user`<br>- `spec.initContainers[*].securityContext.seLinuxOptions.user`<br>- `spec.ephemeralContainers[*].securityContext.seLinuxOptions.user`<br>- `spec.securityContext.seLinuxOptions.role`<br>- `spec.containers[*].securityContext.seLinuxOptions.role`<br>- `spec.initContainers[*].securityContext.seLinuxOptions.role`<br>- `spec.ephemeralContainers[*].securityContext.seLinuxOptions.role`<br><br>**Allowed Values**<br><br>- Undefined/""|
|`/proc` Mount Type|The default `/proc` masks are set up to reduce attack surface, and should be required.<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.procMount`<br>- `spec.initContainers[*].securityContext.procMount`<br>- `spec.ephemeralContainers[*].securityContext.procMount`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `Default`|
|Seccomp|Seccomp profile must not be explicitly set to `Unconfined`.<br><br>**Restricted Fields**<br><br>- `spec.securityContext.seccompProfile.type`<br>- `spec.containers[*].securityContext.seccompProfile.type`<br>- `spec.initContainers[*].securityContext.seccompProfile.type`<br>- `spec.ephemeralContainers[*].securityContext.seccompProfile.type`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `RuntimeDefault`<br>- `Localhost`|
|Sysctls|Sysctls can disable security mechanisms or affect all containers on a host, and should be disallowed except for an allowed "safe" subset. A sysctl is considered safe if it is namespaced in the container or the Pod, and it is isolated from other Pods or processes on the same Node.<br><br>**Restricted Fields**<br><br>- `spec.securityContext.sysctls[*].name`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `kernel.shm_rmid_forced`<br>- `net.ipv4.ip_local_port_range`<br>- `net.ipv4.ip_unprivileged_port_start`<br>- `net.ipv4.tcp_syncookies`<br>- `net.ipv4.ping_group_range`<br>- `net.ipv4.ip_local_reserved_ports` (since Kubernetes 1.27)<br>- `net.ipv4.tcp_keepalive_time` (since Kubernetes 1.29)<br>- `net.ipv4.tcp_fin_timeout` (since Kubernetes 1.29)<br>- `net.ipv4.tcp_keepalive_intvl` (since Kubernetes 1.29)<br>- `net.ipv4.tcp_keepalive_probes` (since Kubernetes 1.29)|

### Restricted[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted)

**The _Restricted_ policy is aimed at enforcing current Pod hardening best practices, at the expense of some compatibility.** It is targeted at operators and developers of security-critical applications, as well as lower-trust users. The following listed controls should be enforced/disallowed:

#### Note:

In this table, wildcards (`*`) indicate all elements in a list. For example, `spec.containers[*].securityContext` refers to the Security Context object for _all defined containers_. If any of the listed containers fails to meet the requirements, the entire pod will fail validation.

|   |   |
|---|---|
|**Control**|**Policy**|
|_Everything from the Baseline policy_|   |
|Volume Types|The Restricted policy only permits the following volume types.<br><br>**Restricted Fields**<br><br>- `spec.volumes[*]`<br><br>**Allowed Values**<br><br>Every item in the `spec.volumes[*]` list must set one of the following fields to a non-null value:<br><br>- `spec.volumes[*].configMap`<br>- `spec.volumes[*].csi`<br>- `spec.volumes[*].downwardAPI`<br>- `spec.volumes[*].emptyDir`<br>- `spec.volumes[*].ephemeral`<br>- `spec.volumes[*].persistentVolumeClaim`<br>- `spec.volumes[*].projected`<br>- `spec.volumes[*].secret`|
|Privilege Escalation (v1.8+)|Privilege escalation (such as via set-user-ID or set-group-ID file mode) should not be allowed. _[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#os-specific-policy-controls) in v1.25+ `(spec.os.name != windows)`_<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.allowPrivilegeEscalation`<br>- `spec.initContainers[*].securityContext.allowPrivilegeEscalation`<br>- `spec.ephemeralContainers[*].securityContext.allowPrivilegeEscalation`<br><br>**Allowed Values**<br><br>- `false`|
|Running as Non-root|Containers must be required to run as non-root users.<br><br>**Restricted Fields**<br><br>- `spec.securityContext.runAsNonRoot`<br>- `spec.containers[*].securityContext.runAsNonRoot`<br>- `spec.initContainers[*].securityContext.runAsNonRoot`<br>- `spec.ephemeralContainers[*].securityContext.runAsNonRoot`<br><br>**Allowed Values**<br><br>- `true`<br><br>The container fields may be undefined/`nil` if the pod-level `spec.securityContext.runAsNonRoot` is set to `true`.|
|Running as Non-root user (v1.23+)|Containers must not set runAsUser to 0<br><br>**Restricted Fields**<br><br>- `spec.securityContext.runAsUser`<br>- `spec.containers[*].securityContext.runAsUser`<br>- `spec.initContainers[*].securityContext.runAsUser`<br>- `spec.ephemeralContainers[*].securityContext.runAsUser`<br><br>**Allowed Values**<br><br>- any non-zero value<br>- `undefined/null`|
|Seccomp (v1.19+)|Seccomp profile must be explicitly set to one of the allowed values. Both the `Unconfined` profile and the _absence_ of a profile are prohibited. _[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#os-specific-policy-controls) in v1.25+ `(spec.os.name != windows)`_<br><br>**Restricted Fields**<br><br>- `spec.securityContext.seccompProfile.type`<br>- `spec.containers[*].securityContext.seccompProfile.type`<br>- `spec.initContainers[*].securityContext.seccompProfile.type`<br>- `spec.ephemeralContainers[*].securityContext.seccompProfile.type`<br><br>**Allowed Values**<br><br>- `RuntimeDefault`<br>- `Localhost`<br><br>The container fields may be undefined/`nil` if the pod-level `spec.securityContext.seccompProfile.type` field is set appropriately. Conversely, the pod-level field may be undefined/`nil` if _all_ container- level fields are set.|
|Capabilities (v1.22+)|Containers must drop `ALL` capabilities, and are only permitted to add back the `NET_BIND_SERVICE` capability. _[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#os-specific-policy-controls) in v1.25+ `(.spec.os.name != "windows")`_<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.capabilities.drop`<br>- `spec.initContainers[*].securityContext.capabilities.drop`<br>- `spec.ephemeralContainers[*].securityContext.capabilities.drop`<br><br>**Allowed Values**<br><br>- Any list of capabilities that includes `ALL`<br><br>---<br><br>**Restricted Fields**<br><br>- `spec.containers[*].securityContext.capabilities.add`<br>- `spec.initContainers[*].securityContext.capabilities.add`<br>- `spec.ephemeralContainers[*].securityContext.capabilities.add`<br><br>**Allowed Values**<br><br>- Undefined/nil<br>- `NET_BIND_SERVICE`|

## Policy Instantiation[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#policy-instantiation)

Decoupling policy definition from policy instantiation allows for a common understanding and consistent language of policies across clusters, independent of the underlying enforcement mechanism.

As mechanisms mature, they will be defined below on a per-policy basis. The methods of enforcement of individual policies are not defined here.

**[Pod Security Admission](Pod%20Security%20Admission.md)Controller**

- [Privileged namespace](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/security/podsecurity-privileged.yaml)
- [Baseline namespace](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/security/podsecurity-baseline.yaml)
- [Restricted namespace](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/security/podsecurity-restricted.yaml)

### Alternatives[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#alternatives)

**Note:** This section links to third party projects that provide functionality required by Kubernetes. The Kubernetes project authors aren't responsible for these projects, which are listed alphabetically. To add a project to this list, read the [](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content) before submitting a change. [](https://kubernetes.io/docs/concepts/security/pod-security-standards/#third-party-content-disclaimer)

Other alternatives for enforcing policies are being developed in the Kubernetes ecosystem, such as:

- [Kubewarden](https://github.com/kubewarden)
- [Kyverno](https://kyverno.io/policies/pod-security/)
- [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)

## Pod OS field[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#pod-os-field)

Kubernetes lets you use nodes that run either Linux or Windows. You can mix both kinds of node in one cluster. Windows in Kubernetes has some limitations and differentiators from Linux-based workloads. Specifically, many of the Pod `securityContext` fields [](https://kubernetes.io/docs/concepts/windows/intro/#compatibility-v1-pod-spec-containers-securitycontext).

#### Note:

Kubelets prior to v1.24 don't enforce the pod OS field, and if a cluster has nodes on versions earlier than v1.24 the Restricted policies should be pinned to a version prior to v1.25.

### Restricted Pod Security Standard changes[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted-pod-security-standard-changes)

Another important change, made in Kubernetes v1.25 is that the _Restricted_ policy has been updated to use the `pod.spec.os.name` field. Based on the OS name, certain policies that are specific to a particular OS can be relaxed for the other OS.

#### OS-specific policy controls[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#os-specific-policy-controls)

Restrictions on the following controls are only required if `.spec.os.name` is not `windows`:

- Privilege Escalation
- Seccomp
- Linux Capabilities

## User namespaces[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#user-namespaces)

User Namespaces are a Linux-only feature to run workloads with increased isolation. How they work together with Pod Security Standards is described in the [](https://kubernetes.io/docs/concepts/workloads/pods/user-namespaces/#integration-with-pod-security-admission-checks) for Pods that use user namespaces.

## FAQ[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#faq)

### Why isn't there a profile between Privileged and Baseline?[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#why-isn-t-there-a-profile-between-privileged-and-baseline)

The three profiles defined here have a clear linear progression from most secure (Restricted) to least secure (Privileged), and cover a broad set of workloads. Privileges required above the Baseline policy are typically very application specific, so we do not offer a standard profile in this niche. This is not to say that the privileged profile should always be used in this case, but that policies in this space need to be defined on a case-by-case basis.

SIG Auth may reconsider this position in the future, should a clear need for other profiles arise.

### What's the difference between a security profile and a security context?[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#what-s-the-difference-between-a-security-profile-and-a-security-context)

[Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) configure Pods and Containers at runtime. Security contexts are defined as part of the Pod and container specifications in the Pod manifest, and represent parameters to the container runtime.

Security profiles are control plane mechanisms to enforce specific settings in the Security Context, as well as other related parameters outside the Security Context. As of July 2021, [Pod Security Policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/) are deprecated in favor of the built-in [Pod Security Admission Controller](https://kubernetes.io/docs/concepts/security/pod-security-admission/).

### What about sandboxed Pods?[](https://kubernetes.io/docs/concepts/security/pod-security-standards/#what-about-sandboxed-pods)

There is not currently an API standard that controls whether a Pod is considered sandboxed or not. Sandbox Pods may be identified by the use of a sandboxed runtime (such as gVisor or Kata Containers), but there is no standard definition of what a sandboxed runtime is.

The protections necessary for sandboxed workloads can differ from others. For example, the need to restrict privileged permissions is lessened when the workload is isolated from the underlying kernel. This allows for workloads requiring heightened permissions to still be isolated.

Additionally, the protection of sandboxed workloads is highly dependent on the method of sandboxing. As such, no single recommended profile is recommended for all sandboxed workloads.