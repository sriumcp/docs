---
template: main.html
---

# A/B Experiments

This tutorial describes [A/B testing](../../user-guide/topics/ab_testing.md) of a backend component in a distributed Kubernetes app.

***
 
## Launch Iter8 A/B/n service

```shell
cat << EOF > values.yaml
pvc: iter8pvc
abn:
- name: recommender
  namespace: test
  tracks:
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
EOF
```

```shell
helm install --repo https://iter8-tools.github.io/hub ic iter8ctrl \
-f values.yaml -n iter8-system
```

???+ note "Documentation for the values.yaml file"
    ```yaml
    # name of the Iter8 PVC configured as part of Iter8 (cluster) install
    pvc: iter8pvc
    # abn section describes the Iter8 controller configuration for abn experiments
    abn:
      # name of the component that is being AB tested
      # component names must be unique
    - name: recommender
      # namespace of the component
      namespace: test
      # a component can have multiple tracks
      # a component has a stable version and (possibly) multiple candidate versions at any time
      # it is best practice to associate the stable version with track 1
      # and associate candidate versions with the other tracks
      tracks:
        # users are split across tracks in proportion to their weights
        # weights must be positive
      - weight: 3
        # each track can have multiple k8s resources associated with it
        # the first track has Kubernetes service and deployment associated with it
        resources:
          # name of the resource
        - name: recommender-stable-svc
          # type of the resource
          # svc is shorthand for Kubernetes service
          type: svc
        - name: recommender-stable
          # deploy is shorthand for Kubernetes deployment
          type: deploy
        # second track with the default weight of 1
      - resources:
        - name: recommender-candidate-svc
          type: svc
        - name: recommender-candidate
          type: deploy
    ```

## Create the sample application

=== "frontend"
    === "node"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-node:0.13
        kubectl expose deployment frontend --name=frontend --port=8090
        ```

    === "Go"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-go:0.13
        kubectl expose deployment frontend --name=frontend --port=8090
        ```
    
=== "backend"
    Create track `1` of the backend component.

    ```shell
    kubectl create deployment recommender-stable --image=iter8/abn-sample-backend:0.13-v1
    kubectl expose deployment recommender-stable --name=recommender-stable-svc --port=8091
    ```

## Generate load

Generate load. In separate shells, port-forward requests to the frontend component and generate load for multiple users.  A [script](https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh) is provided to do this. To use it:
    ```shell
    kubectl port-forward service/frontend 8090:8090
    ```
    ```shell
    curl -s https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh | sh -s --
    ```

## Create track 2 of the backend component

```shell
kubectl create deployment backend-candidate --image=iter8/abn-sample-backend:0.13-v2
kubectl expose deployment backend-candidate --name=backend-candidate-svc --port=8091
```

Note: Previously we had a version associated with each track. Now, what constitutes a new version of a track?

## Launch experiment

```shell
iter8 k launch \
--set abnmetrics.component=recommender \
--set "tasks={abnmetrics}" \
--set runner=cronjob \
--set cronjobSchedule="*/1 * * * *"
```


## Inspect experiment report

Inspect the metrics:

```shell
iter8 k report
```

??? note "Sample output from report"
    ```
    Experiment summary:
    *******************

    Experiment completed: true
    No task failures: true
    Total number of tasks: 1
    Number of completed tasks: 1
    Number of completed loops: 3

    Latest observed values for metrics:
    ***********************************

    Metric                   | Track 1 | Track 2
    -------                  | -----        | -----
    abn/sample_metric/usercount  | 282        | 629
    abn/sample_metric/count  | 35.00        | 28.00
    abn/sample_metric/max    | 99.00        | 100.00
    abn/sample_metric/mean-per-count   | 56.31        | 52.79
    abn/sample_metric/mean-per-user   | 561.31        | 522.79
    abn/sample_metric/min    | 0.00         | 1.00
    abn/sample_metric/stddev-per-count | 28.52        | 31.91
    abn/sample_metric/stddev-per-user | 283.52        | 314.91
    ```
The output allows you to compare the versions against each other and select a winner. Since the experiment runs periodically, the values in the report will change over time.

Once a winner is identified, the experiment can be terminated, the winner can be promoted, and the candidate version(s) can be deleted.

To delete the experiment:

```shell
iter8 k delete
```

## Promote candidate version

Delete the candidate version:

```shell
kubectl delete deployment backend-candidate-1 
kubectl delete service backend-candidate-1
```

Update the version associated with the baseline track identifier *backend*:

```shell
kubectl set image deployment/backend abn-sample-backend=iter8/abn-sample-backend:0.13-v2
```

## Cleanup

### Delete sample application

```shell
kubectl delete \
deploy/frontend deploy/backend deploy/backend-candidate-1 \
service/frontend service/backend service/backend-candidate-1
```

### Uninstall the A/B/n service

```shell
helm delete iter8ctrl -n iter8-system
```
