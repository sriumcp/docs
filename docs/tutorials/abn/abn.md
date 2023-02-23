---
template: main.html
---

# A/B testing

The process of A/B testing an app/ML model involves the following steps.

1. Deploying a candidate version of the app/ML model.
2. Splitting user traffic between the stable and candidate version, while ensuring [user stickiness](sticky).
3. Collecting business metrics and assessing versions based on them.
4. Promoting the winning version as the latest stable version.

Iter8 automatically tracks deployed versions during step 1, automates steps 2 and 3, and assists step 4 through notifications. 

The following picture illustrates a typical design for A/B testing [an upstream service (backend app/ML model)](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/intro/terminology) in a distributed application using Iter8. We will use this design in this tutorial.

![A/B testing a backend](images/abn.backend.png){ width="65%" }
 
***

???+ warning "Before you begin"
    1. [Install or upgrade the Iter8 service](../../getting-started/install.md#install-or-upgrade-iter8-service) so that it works with apps in the `default` namespace.
    2. Try [your first experiment](../../getting-started/your-first-experiment.md). Understand the main [concepts](../../getting-started/concepts.md) behind Iter8 experiments.

***

## Create appconf

Provide a description of the app being A/B tested by the Iter8 service using an [appconf](../../user-guide/topics/abn/appconf.md)

```shell
cat << EOF > appconf.yaml
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
EOF
```

```shell
kubectl create configmap recommender --from-file=appconf.yaml
kubectl label configmap recommender iter8.tools/role=appconf
```

??? note "Documentation for `recommender.yaml`"
    ```yaml
    # an app must always have a stable version; it may have multiple candidate versions
    # by convention, Version 1 is treated as the stable version; 
    # Versions 2 or more are treated as the candidate version
    versions:
      # users are split across versions in proportion to version weights
      # weights must be positive (default 1)
    - weight: 3
      # each version can have multiple resources associated with it
      resources:
        # name of the resource
      - name: recommender-stable
        # type of the resource
        # svc is shorthand for k8s service
        type: svc
      - name: recommender-stable
        # deploy is shorthand for k8s deployment
        type: deploy
      # second version with the default weight of 1
    - resources:
      - name: recommender-candidate
        type: svc
      - name: recommender-candidate
        type: deploy
    ```

## Deploy sample application
Deploy the sample application.
=== "Frontend"
    Deploy the frontend in the language of your choice.
    === "node"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-node:0.14
        kubectl expose deployment frontend --name=frontend --port=8090
        ```

    === "Go"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-go:0.14
        kubectl expose deployment frontend --name=frontend --port=8090
        ```
    
=== "Backend: stable version"
     Deploy the stable version of the backend as follows.

    ```shell
    kubectl create deployment recommender-stable --image=iter8/abn-sample-backend:0.14-v1
    kubectl expose deployment recommender-stable --port=8091
    ```

    > The resources created above are the same as the resources listed under Version 1 in the Iter8 service configuration.


## Generate load

Use [this script](https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh) to generate load. The script simulates multiple users that make requests to the frontend. First, port-forward the frontend as follows.
```shell
kubectl port-forward service/frontend 8090:8090
```

In a separate terminal, run the load generation script.
```shell
curl -s https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh | sh -s --
```

## Deploy candidate version
Deploy the candidate version of the backend. The candidate version is associated with Version 2.

```shell
kubectl create deployment recommender-candidate --image=iter8/abn-sample-backend:0.14-v2
kubectl expose deployment recommender-candidate --port=8091
```

> The resources created above are the same as the resources listed under Version 2 in the Iter8 service configuration.

??? note "Behind the scenes"
    Suppose the app under consideration is an online store. Users can use the `List()` API of the frontend to view a ranked list of products. They can use the `Buy()` API of the frontend to buy items in their shopping cart. When the frontend receives a `List()` request, it invokes the `Rank()` API of the bankend to receive a list of product recommendations, and surfaces this to the user.

    The following sequence diagram illustrates how users, frontend, backend, and the Iter8 service interact. The frontend invokes the `Lookup(name, user)` API of the Iter8 service to determine the backend version; here, `name` is the name of the app being A/B tested (`recommender`) in this tutorial, and `user` is the unique ID of the user. It uses this version to receive recommendations from the backend and answer the user query. When the user buys items, it uses the `WriteMetric(name, user, metric, value)` API of the Iter8 service to report business metrics (such as total price, and the number of items in this example).

    ```mermaid
    sequenceDiagram
        actor U1 as User1
        actor U2 as User2
        participant F as Frontend
        participant B1 as Backend <br> Version 1
        participant B2 as Backend <br> Version 2
        participant I as Iter8 service
        U1->>F: List()
        activate F
        F->>I: Lookup(name, user)
        activate I
        I-->>F: Version 1
        deactivate I
        F->>B1: Rank()
        B1-->>F: Recommendations
        F-->>U1: Products
        deactivate F
        U2->>F: List()
        activate F
        F->>I: Lookup(name, user)
        activate I
        I-->>F: Version 2
        deactivate I
        F->>B2: Rank()
        B2-->>F: Recommendations
        F-->>U2: Products
        deactivate F
        U2->>F: Buy()
        activate F
        F->>I: WriteMetric(name, user, metric, value)
        deactivate F
        U2->>F: List()
        activate F
        F->>I: Lookup(name, user)
        activate I
        I-->>F: Version 2
        deactivate I
        F->>B2: Rank()
        B2-->>F: Recommendations
        F-->>U2: Products
        deactivate F
    ```

    The Iter8 service provides the following guarantees:
    
    1. Weight: the number of users assigned to each version is (approximately) proportional to its weight.
    
    2. User stickiness: when two or more versions of a backend are deployed, for a given user, every `Lookup(name, user)` call returns the same version.
    
    3. Readiness: when a version is returned by a `Lookup(name, user)` call, the Kubernetes resources corresponding to this version are guaranteed to exist, and be ready to serve frontend requests. In particular, this guarantee implies that to you can update any version, or delete candidate versions at any point in time with zero downtime (i.e., without loss of requests from the frontend).

## Launch experiment

Use the Iter8 CLI to launch the Iter8 experiment in the cluster.

```shell
iter8 k launch \
--set abnmetrics.name=recommender \
--set "tasks={abnmetrics}" \
--set runner=cronjob \
--set cronjobSchedule="*/1 * * * *"
```

??? note "About this experiment"
    This experiment consists of a single [task](../../getting-started/concepts.md#iter8-experiment), namely, [abnmetrics](../../user-guide/tasks/abnmetrics.md).

    The abnmetrics task fetches metrics from the [Iter8 service](iter8service), and is parameterized by the name of the app being A/B tested (`recommender` in this tutorial).

    This is a [multi-loop experiment](../../getting-started/concepts.md#iter8-experiment). Hence, its [runner](../../getting-started/concepts.md#how-it-works) value is set to `cronjob`. The `cronjobSchedule` expression specifies that each experiment loop (i.e., the experiment task) is scheduled for execution periodically once every minute. This enables Iter8 to refresh the metric values during each loop.

View the experiment report as follows.

```shell
iter8 k report
```

??? note "Sample experiment report"
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

    Metric                                  | Version 1   | Version 2
    -------                                 | -----     | -----
    abn/recommender/duration                | 368 (hr)  | 120 (hr)
    abn/recommender/numusers                | 628       | 149
    abn/recommender/lookups                 | 2725      | 822
    abn/recommender/sales/mean              | 51.31     | 52.79
    abn/sample_metric/stddev                | 82.52     | 69.89
    abn/recommender/sales/conversion        | 82        | 43
    abn/recommender/sales/conversion-rate   | 13.05 (%) | 28.85 (%)
    ```

The above experiment report allows you to compare versions and select a winner. Since this is [multi-loop experiment](../../getting-started/concepts.md#iter8-experiment), the values in the report are updated with each loop. 

If the candidate version is deemed the winner, it can be promoted as the latest stable version, and resources corresponding to Version 2 can be deleted.

## Promote candidate version

Promote the candidate version as the latest stable version.

```shell
kubectl set image deployment/recommender-stable \
abn-sample-backend=iter8/abn-sample-backend:0.14-v2
```

At this point, Versions 1 and 2 both have the same image. You can delete all the resources associated with Version 2, leaving Version 1 as the one and only deployed version of the app.

## Cleanup

Remove the Iter8 experiment, the sample application, and the Iter8 service from the Kubernetes cluster.

```shell
iter8 k delete
kubectl delete \
deploy/frontend deploy/recommender-stable deploy/recommender-candidate \
service/frontend service/recommender-stable service/recommender-candidate
helm delete iter8-service -n iter8-system
```
