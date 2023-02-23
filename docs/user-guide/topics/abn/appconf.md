---
template: main.html
---

# Appconf Resource

An appconf resource is a [Kubernetes configmap](https://kubernetes.io/docs/concepts/configuration/configmap/) that is used to describe an app to the Iter8 service. An example of an appconf resource is as follows.

## Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: recommender
  namespace: default
  labels:
    iter8.tools/role: appconf
data:
  appconf.yaml: |
    versions:
    - weight: 3
      resources:
      - name: recommender-stable
        type: svc
      - name: recommender-stable
        type: deploy
    - resources:
      - name: recommender-candidate
        type: svc
      - name: recommender-candidate
        type: deploy
```

## Distinguishing characteristics
An appconf resource has two characteristics that distinguishes it from a regular configmap.

1. It has the `iter8.tools/role` set to the value `appconf`.
2. Its `data` section has an [`appconf.yaml`](#appconfyaml-reference) field, whose value describes the app.

## `appconf.yaml` reference

Parameters related to an app.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| versions  | [][Version](#version) | List of app versions |

### Version
Parameters related to an app version.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| weight  | int | Proportion of users routed to this version. Must be positive. Default is 1  |
| resources  | [][Kubernetes resource](#kubernetes-resource) | Parameters related to a Kubernetes resource |

### Kubernetes Resource
Parameters related to a Kubernetes resource.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| name  | string | Name of the resource  |
| type  | string enum | Identifies the group version kind (GVK) of the resource. Valid values are `svc`, `deploy`, `isvc` and `ksvc` which correspond to [Kubernetes service](https://kubernetes.io/docs/concepts/services-networking/service/), [Kubernetes deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [KServe inference service](https://github.com/kserve/kserve), and [Knative service](https://knative.dev/docs/) respectively |



## App ID
The `<namespace>/<name>` combination of an appconf acts as the app identifier. For instance, in the [above example](#example), `default/recommender` is the app ID. The app ID is used as part of [Iter8 SDK](sdk.md) API calls.
