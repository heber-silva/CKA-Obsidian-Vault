A [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) is an API object used to store non-confidential data in key-value pairs. [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a [volume](https://kubernetes.io/docs/concepts/storage/volumes/).

A ConfigMap allows you to decouple environment-specific configuration from your [](https://kubernetes.io/docs/reference/glossary/?all=true#term-image), so that your applications are easily portable.

#### Caution:

ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential, use a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) rather than a ConfigMap, or use additional (third party) tools to keep your data private.

## Motivation[](https://kubernetes.io/docs/concepts/configuration/configmap/#motivation)

Use a ConfigMap for setting configuration data separately from application code.

For example, imagine that you are developing an application that you can run on your own computer (for development) and in the cloud (to handle real traffic). You write the code to look in an environment variable named `DATABASE_HOST`. Locally, you set that variable to `localhost`. In the cloud, you set it to refer to a Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) that exposes the database component to your cluster. This lets you fetch a container image running in the cloud and debug the exact same code locally if needed.

#### Note:

A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed 1 MiB. If you need to store settings that are larger than this limit, you may want to consider mounting a volume or use a separate database or file service.

## ConfigMap object[](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-object)

A ConfigMap is an [](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects) that lets you store configuration for other objects to use. Unlike most Kubernetes objects that have a `spec`, a ConfigMap has `data` and `binaryData` fields. These fields accept key-value pairs as their values. Both the `data` field and the `binaryData` are optional. The `data` field is designed to contain UTF-8 strings while the `binaryData` field is designed to contain binary data as base64-encoded strings.

The name of a ConfigMap must be a valid [](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names).

Each key under the `data` or the `binaryData` field must consist of alphanumeric characters, `-`, `_` or `.`. The keys stored in `data` must not overlap with the keys in the `binaryData` field.

Starting from v1.19, you can add an `immutable` field to a ConfigMap definition to create an [](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-immutable).

## ConfigMaps and Pods[](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods)

You can write a Pod `spec` that refers to a ConfigMap and configures the container(s) in that Pod based on the data in the ConfigMap. The Pod and the ConfigMap must be in the same [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces).

#### Note:

The `spec` of a [static Pod](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) cannot refer to a ConfigMap or any other API objects.