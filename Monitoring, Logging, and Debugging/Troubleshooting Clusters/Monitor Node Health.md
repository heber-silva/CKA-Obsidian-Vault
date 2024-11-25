[Docs](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/)

_Node Problem Detector_ is a daemon for monitoring and reporting about a node's health. You can run Node Problem Detector as a `DaemonSet` or as a standalone daemon. Node Problem Detector collects information about node problems from various daemons and reports these conditions to the API server as Node [](https://kubernetes.io/docs/concepts/architecture/nodes/#condition)s or as [Event](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/event-v1/)s.

To learn how to install and use Node Problem Detector, see [Node Problem Detector project documentation](https://github.com/kubernetes/node-problem-detector).

## Before you begin[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#before-you-begin)

You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) or you can use one of these Kubernetes playgrounds:

- [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

## Limitations[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#limitations)

- Node Problem Detector uses the kernel log format for reporting kernel issues. To learn how to extend the kernel log format, see [](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#support-other-log-format).

## Enabling Node Problem Detector[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#enabling-node-problem-detector)

Some cloud providers enable Node Problem Detector as an [Addon](https://kubernetes.io/docs/concepts/cluster-administration/addons/). You can also enable Node Problem Detector with `kubectl` or by creating an Addon DaemonSet.

### Using kubectl to enable Node Problem Detector[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#using-kubectl)

`kubectl` provides the most flexible management of Node Problem Detector. You can overwrite the default configuration to fit it into your environment or to detect customized node problems. For example:

1. Create a Node Problem Detector configuration similar to `node-problem-detector.yaml`:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-problem-detector-v0.1
  namespace: kube-system
  labels:
    k8s-app: node-problem-detector
    version: v0.1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: node-problem-detector  
      version: v0.1
      kubernetes.io/cluster-service: "true"
  template:
    metadata:
      labels:
        k8s-app: node-problem-detector
        version: v0.1
        kubernetes.io/cluster-service: "true"
    spec:
      hostNetwork: true
      containers:
      - name: node-problem-detector
        image: registry.k8s.io/node-problem-detector:v0.1
        securityContext:
          privileged: true
        resources:
          limits:
            cpu: "200m"
            memory: "100Mi"
          requests:
            cpu: "20m"
            memory: "20Mi"
        volumeMounts:
        - name: log
          mountPath: /log
          readOnly: true
      volumes:
      - name: log
        hostPath:
          path: /var/log/
```
#### Note: 
You should verify that the system log directory is right for your operating system distribution.
2. Start node problem detector with `kubectl`:

```shell
kubectl apply -f https://k8s.io/examples/debug/node-problem-detector.yaml
```

### Using an Addon pod to enable Node Problem Detector[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#using-addon-pod)

If you are using a custom cluster bootstrap solution and don't need to overwrite the default configuration, you can leverage the Addon pod to further automate the deployment.

Create `node-problem-detector.yaml`, and save the configuration in the Addon pod's directory `/etc/kubernetes/addons/node-problem-detector` on a control plane node.

## Overwrite the configuration[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#overwrite-the-configuration)

The [default configuration](https://github.com/kubernetes/node-problem-detector/tree/v0.8.12/config) is embedded when building the Docker image of Node Problem Detector.

However, you can use a [`ConfigMap`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) to overwrite the configuration:

1. Change the configuration files in `config/`
    
2. Create the `ConfigMap` `node-problem-detector-config`:
    
    ```shell
    kubectl create configmap node-problem-detector-config --from-file=config/
    ```
    
3. Change the `node-problem-detector.yaml` to use the `ConfigMap`:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-problem-detector-v0.1
  namespace: kube-system
  labels:
	k8s-app: node-problem-detector
	version: v0.1
	kubernetes.io/cluster-service: "true"
spec:
  selector:
	matchLabels:
	  k8s-app: node-problem-detector  
	  version: v0.1
	  kubernetes.io/cluster-service: "true"
  template:
	metadata:
	  labels:
		k8s-app: node-problem-detector
		version: v0.1
		kubernetes.io/cluster-service: "true"
	spec:
	  hostNetwork: true
	  containers:
	  - name: node-problem-detector
		image: registry.k8s.io/node-problem-detector:v0.1
		securityContext:
		  privileged: true
		resources:
		  limits:
			cpu: "200m"
			memory: "100Mi"
		  requests:
			cpu: "20m"
			memory: "20Mi"
		volumeMounts:
		- name: log
		  mountPath: /log
		  readOnly: true
		- name: config # Overwrite the config/ directory with ConfigMap volume
		  mountPath: /config
		  readOnly: true
	  volumes:
	  - name: log
		hostPath:
		  path: /var/log/
	  - name: config # Define ConfigMap volume
		configMap:
		  name: node-problem-detector-config
```

4. Recreate the Node Problem Detector with the new configuration file:
```shell
    # If you have a node-problem-detector running, delete before recreating
    kubectl delete -f https://k8s.io/examples/debug/node-problem-detector.yaml
    kubectl apply -f https://k8s.io/examples/debug/node-problem-detector-configmap.yaml
```

#### Note:

This approach only applies to a Node Problem Detector started with `kubectl`.

Overwriting a configuration is not supported if a Node Problem Detector runs as a cluster Addon. The Addon manager does not support `ConfigMap`.

## Problem Daemons[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#problem-daemons)

A problem daemon is a sub-daemon of the Node Problem Detector. It monitors specific kinds of node problems and reports them to the Node Problem Detector. There are several types of supported problem daemons.

- A `SystemLogMonitor` type of daemon monitors the system logs and reports problems and metrics according to predefined rules. You can customize the configurations for different log sources such as [filelog](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor-filelog.json), [kmsg](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor.json), [kernel](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor-counter.json), [abrt](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/abrt-adaptor.json), and [systemd](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/systemd-monitor-counter.json).
    
- A `SystemStatsMonitor` type of daemon collects various health-related system stats as metrics. You can customize its behavior by updating its [configuration file](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/system-stats-monitor.json).
    
- A `CustomPluginMonitor` type of daemon invokes and checks various node problems by running user-defined scripts. You can use different custom plugin monitors to monitor different problems and customize the daemon behavior by updating the [configuration file](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/custom-plugin-monitor.json).
    
- A `HealthChecker` type of daemon checks the health of the kubelet and container runtime on a node.
    

### Adding support for other log format[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#support-other-log-format)

The system log monitor currently supports file-based logs, journald, and kmsg. Additional sources can be added by implementing a new [log watcher](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/pkg/systemlogmonitor/logwatchers/types/log_watcher.go).

### Adding custom plugin monitors[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#adding-custom-plugin-monitors)

You can extend the Node Problem Detector to execute any monitor scripts written in any language by developing a custom plugin. The monitor scripts must conform to the plugin protocol in exit code and standard output. For more information, please refer to the [plugin interface proposal](https://docs.google.com/document/d/1jK_5YloSYtboj-DtfjmYKxfNnUxCAvohLnsH5aGCAYQ/edit).

## Exporter[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#exporter)

An exporter reports the node problems and/or metrics to certain backends. The following exporters are supported:

- **Kubernetes exporter**: this exporter reports node problems to the Kubernetes API server. Temporary problems are reported as Events and permanent problems are reported as Node Conditions.
    
- **Prometheus exporter**: this exporter reports node problems and metrics locally as Prometheus (or OpenMetrics) metrics. You can specify the IP address and port for the exporter using command line arguments.
    
- **Stackdriver exporter**: this exporter reports node problems and metrics to the Stackdriver Monitoring API. The exporting behavior can be customized using a [configuration file](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/exporter/stackdriver-exporter.json).
    

## Recommendations and restrictions[](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/#recommendations-and-restrictions)

It is recommended to run the Node Problem Detector in your cluster to monitor node health. When running the Node Problem Detector, you can expect extra resource overhead on each node. Usually this is fine, because:

- The kernel log grows relatively slowly.
- A resource limit is set for the Node Problem Detector.
- Even under high load, the resource usage is acceptable. For more information, see the Node Problem Detector [](https://github.com/kubernetes/node-problem-detector/issues/2#issuecomment-220255629).