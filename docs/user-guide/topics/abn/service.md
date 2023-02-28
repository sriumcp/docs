---
template: main.html
---

# Iter8 Service

<!-- Iter8 service facilitates A/B/n testing of distributed Kubernetes applications. It keeps track of the [versions](../../../glossary.md#version) of [apps](../../../glossary.md#app) that are described using [`appspec` resources](../appspec.md), and provides the server-side implementation of the [gRPC](https://grpc.io/)-based [Iter8 SDK](sdk.md) methods.

Iter8 service can be installed or upgraded as described [here](../../../getting-started/install.md#install-or-upgrade-iter8-service). The install or upgrade can be parameterized through a Helm `values.yaml` file. These parameters are described below.

## `values.yaml` reference
Helm parameters that can be supplied during Iter8 service install or upgrade. All these parameters are optional.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| storage  | [Storage](#storage) | Iter8 service uses [persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) in Kubernetes to manage its state. This field contains parameters related to the persistent storage |
| requests  | [Requests](#Requests) | Resource requests |
| limits  | [Limits](#Limits) | Resource limits |
| hpa | [HPA](#HPA) | Parameters related to [Kubernetes horizontal pod autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) |

### Storage
Configuration parameters related to the persistent storage used by Iter8 service.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| className  | string | Name of the [Kubernetes storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/). A [Kubernetes persistent volume claim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) will be created using this storage class during Iter8 service installation or upgrade. By default, this field is left unspecified, which results in the [cluster's default storage class](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1) to be used by the PVC |
| accessMode  | string enum | [Access mode for persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes). Default is `ReadWriteOnce`. |

### Requests
Configuration parameters related to resource requests.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| memory  | [Quantity](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | Memory request for Iter8 service pod. Default is 2Gi  |
| cpu  | [Quantity](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | CPU request for Iter8 service pod. Default is 500m |
| storage | [Quantity](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | Storage request for the PVC created during Iter8 service install. Default is 5Gi |

### Limits
Configuration parameters related to resource limits.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| memory  | [Quantity](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | Memory limit for Iter8 service pod. Default is 4Gi  |
| cpu  | [Quantity](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | CPU limit for Iter8 service pod. Default is 1 |

### HPA
Configuration parameters related to horizontal pod autoscaler.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| minReplicas  | Integer | Minimum number of replicas of Iter8 service pods. Default is 1  |
| maxReplicas  | Integer | Maximum number of replicas of Iter8 service pods. Default is 10 |
| targetCPUUtilizationPercentage | Integer | Average CPU utilization of Iter8 service pods. Default is 60 |

## Production considerations
### Monitoring
Monitoring is essential for understanding how the Iter8 service behaves in production and ensuring its reliability. You can use [standard tools for resource usage monitoring](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/) and your choice of [Kubernetes monitoring systems such as Prometheus](https://landscape.cncf.io/card-mode?category=monitoring&project=graduated,incubating,member,no&grouping=category&sort=stars) for this purpose. The following are metrics that especially useful to monitor.

1. CPU/memory requests and limits vs. actual usage to ensure that CPU and memory resources do not have contention and are not wasted.
2. Actual replica count vs. minimum and maximum replica counts to ensure that HPA is configured properly.
3. Persistent volume utilization to ensure that Iter8 SDK calls do not result in errors due to lack of free space.

### Storage access mode
The default value of the `storage.accessMode` parameter is [`ReadWriteOnce`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes). This mode corresponds to the access mode supported by the default storage classes in many managed Kubernetes services. However, `ReadWriteOnce` access mode implies that all pods backing the Iter8 service will be scheduled on the same node. This may lead to resource contention when the rate of Iter8 SDK API is high. You can solve this issue by setting the `storage.accessMode` parameter to `ReadWriteMany`, and using an appropriate `storage.className` that supports this access mode.

### Storage volume expansion
The persistent volume used by the Iter8 service is dynamically provisioned during its installation. If the utilization of this volume reaches its capacity, Iter8 SDK calls will result in errors due to lack of free space. You can solve this issue by using a storage class that supports [volume expansion](https://kubernetes.io/docs/concepts/storage/storage-classes/#allow-volume-expansion), monitoring the persistent volume utilization, and resizing it by [upgrading the Iter8 service](../../../getting-started/install.md#install-or-upgrade-iter8-service) with a higher [storage request](#requests).

 -->
