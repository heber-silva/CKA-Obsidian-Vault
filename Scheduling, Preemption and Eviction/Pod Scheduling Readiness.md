[Doc](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
FEATURE STATE: `Kubernetes v1.30 [stable]`

Pods were considered ready for scheduling once created. Kubernetes scheduler does its due diligence to find nodes to place all pending Pods. However, in a real-world case, some Pods may stay in a "miss-essential-resources" state for a long period. These Pods actually churn the scheduler (and downstream integrators like Cluster AutoScaler) in an unnecessary manner.

By specifying/removing a Pod's `.spec.schedulingGates`, you can control when a Pod is ready to be considered for scheduling.

## Configuring Pod schedulingGates[](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/#configuring-pod-schedulinggates)

The `schedulingGates` field contains a list of strings, and each string literal is perceived as a criteria that Pod should be satisfied before considered schedulable. This field can be initialized only when a Pod is created (either by the client, or mutated during admission). After creation, each schedulingGate can be removed in arbitrary order, but addition of a new scheduling gate is disallowed.

[Open: Pasted image 20241117160231.png](Images/1c746930af2f4c3d26c40ca895790bc3_MD5.jpeg)
![1c746930af2f4c3d26c40ca895790bc3_MD5](Images/1c746930af2f4c3d26c40ca895790bc3_MD5.jpeg)

Figure. Pod SchedulingGates

## Usage example[](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/#usage-example)

To mark a Pod not-ready for scheduling, you can create it with one or more scheduling gates like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  schedulingGates:
  - name: example.com/foo
  - name: example.com/bar
  containers:
  - name: pause
    image: registry.k8s.io/pause:3.6
```

After the Pod's creation, you can check its state using:

```bash
kubectl get pod test-pod
```

The output reveals it's in `SchedulingGated` state:

```none
NAME       READY   STATUS            RESTARTS   AGE
test-pod   0/1     SchedulingGated   0          7s
```

You can also check its `schedulingGates` field by running:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.schedulingGates}'
```

The output is:

```none
[{"name":"example.com/foo"},{"name":"example.com/bar"}]
```

To inform scheduler this Pod is ready for scheduling, you can remove its `schedulingGates` entirely by reapplying a modified manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: pause
    image: registry.k8s.io/pause:3.6
```

You can check if the `schedulingGates` is cleared by running:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.schedulingGates}'
```

The output is expected to be empty. And you can check its latest status by running:

```bash
kubectl get pod test-pod -o wide
```

Given the test-pod doesn't request any CPU/memory resources, it's expected that this Pod's state get transited from previous `SchedulingGated` to `Running`:

```none
NAME       READY   STATUS    RESTARTS   AGE   IP         NODE
test-pod   1/1     Running   0          15s   10.0.0.4   node-2
```

## Observability[](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/#observability)

The metric `scheduler_pending_pods` comes with a new label `"gated"` to distinguish whether a Pod has been tried scheduling but claimed as unschedulable, or explicitly marked as not ready for scheduling. You can use `scheduler_pending_pods{queue="gated"}` to check the metric result.

## Mutable Pod scheduling directives[](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/#mutable-pod-scheduling-directives)

You can mutate scheduling directives of Pods while they have scheduling gates, with certain constraints. At a high level, you can only tighten the scheduling directives of a Pod. In other words, the updated directives would cause the Pods to only be able to be scheduled on a subset of the nodes that it would previously match. More concretely, the rules for updating a Pod's scheduling directives are as follows:

1. For `.spec.nodeSelector`, only additions are allowed. If absent, it will be allowed to be set.
    
2. For `spec.affinity.nodeAffinity`, if nil, then setting anything is allowed.
    
3. If `NodeSelectorTerms` was empty, it will be allowed to be set. If not empty, then only additions of `NodeSelectorRequirements` to `matchExpressions` or `fieldExpressions` are allowed, and no changes to existing `matchExpressions` and `fieldExpressions` will be allowed. This is because the terms in `.requiredDuringSchedulingIgnoredDuringExecution.NodeSelectorTerms`, are ORed while the expressions in `nodeSelectorTerms[].matchExpressions` and `nodeSelectorTerms[].fieldExpressions` are ANDed.
    
4. For `.preferredDuringSchedulingIgnoredDuringExecution`, all updates are allowed. This is because preferred terms are not authoritative, and so policy controllers don't validate those terms.
    

## What's next[](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/#what-s-next)

- Read the [PodSchedulingReadiness KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/3521-pod-scheduling-readiness) for more details