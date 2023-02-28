---
template: main.html
---

# Experiment Resource

<!-- An appspec resource is a [Kubernetes configmap](https://kubernetes.io/docs/concepts/configuration/configmap/) that is used to describe an app to the Iter8 service. An example of an appspec resource is as follows.

## Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: recommender
  namespace: default
  labels:
    iter8.tools/role: appspec
data:
  appspec.yaml: |
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
An appspec resource has three distinguishing characteristics.

1. It has a label with key `iter8.tools/role` and value `appspec`.
2. Its `data` section has a single key, [`appspec.yaml`](#appspecyaml-reference). The value for this key provides the parameters related to the app.

## `appspec.yaml` reference

Parameters related to an app.

| Field | Type | Description |
| --------- | ------------------------------ | ------------------ |
| versions  | [][Version](#version) | List of app versions |
| finalizer | bool | Iter8 adds a finalizer to the app's resources to provide readiness [guarantees](abn/sdk.md#guarantees) for the Iter8 SDK `GetRoute` API. If you intend to use this API or other traffic engineering features of Iter8, set this to true. Default is true |
| seed | int | Iter8 service uses consistent hashing to map users to versions and [guarantee weighted routing and user stickiness](abn/sdk.md#guarantees) for the Iter8 SDK `GetRoute` API. This field is the seed value for the hash. Changing the seed will change the hash function used to map users to versions, while preserving the guarantees. The seed value should not be changed during the course of an A/B/n testing experiment, in order to preserve user stickiness[^1]. Default is 0 |

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
The `<namespace>/<name>` combination of an appspec acts as the app identifier. For instance, in the [above example](#example), `default/recommender` is the app ID. The app ID is used as part of [Iter8 SDK](abn/sdk.md) API calls.


[^1]: The `seed` is a mechanism to provide a fairness guarantee in Iter8. Consider an A/B testing experiment with two versions of an app with equal weights. Weighted routing guarantees that in the experiment, half the users are mapped to the stable version (Version 1) in expectation, and the remaining users are mapped to the candidate version (Version 2). Suppose you perform several such experiments for the same app with new candidate versions. You may wish to rotate the pool of users that are mapped to the candidate (Version 2) in each experiment. You can accomplish this by changing the seed after an experiment ends, and before a new one begins.
 -->
