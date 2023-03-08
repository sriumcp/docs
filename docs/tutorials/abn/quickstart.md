---
template: main.html
---

# Quick Start

> Deploy a client server application. A/B test two variants of the server (backend). The client will use the Iter8 SDK to map users to backend variants and report business metrics. Iter8 SDK will ensure user stickiness and weighted routing during the A/B testing process.
<!-- A/B testing an app involves the following steps.

1. Deploying a candidate Variant of the app.
2. Splitting user traffic between the stable and candidate Variants, while ensuring [user stickiness](sticky).
3. Collecting business metrics and assessing Variants based on them.
4. Promoting the winning Variant as the latest stable Variant.

Iter8 detects new candidate Variants after step 1, automates steps 2 and 3, and assists in step 4 through notifications.

In this tutorial, we focus on A/B testing [an upstream service (backend app/ML model)](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/intro/terminology) in a distributed application. The following picture illustrates the approach of this tutorial. -->

![A/B testing a backend](images/abn.backend.png)
 
***

???+ warning "Before you begin"
    [Install or upgrade the Iter8 service](../../getting-started/install.md#install-or-upgrade-iter8-service) so that it manages apps in the `default` namespace.

***

## Deploy application

Deploy the client server application along with a subject. For the client, choose a specific language and deploy the client in that language. For the backend, deploy both variants.

=== "Client"

    Deploy the client in the language of your choice. 

    === "Node"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-node:0.14
        kubectl expose deployment frontend --name=frontend --port=8090
        ```

    === "Python"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-python:0.14
        kubectl expose deployment frontend --name=frontend --port=8090
        ```

    === "Go"
        ```shell
        kubectl create deployment frontend --image=iter8/abn-sample-frontend-go:0.14
        kubectl expose deployment frontend --name=frontend --port=8090
        ```
    
=== "Backend variant 1"

     Deploy variant 1 (stable version) of the backend.

    ```shell
    kubectl create deployment recommender-stable --image=iter8/abn-sample-backend:0.14-v1
    kubectl expose deployment recommender-stable --port=8091
    ```

    Label backend resources so that Iter8 can detect them. These resources will be referenced in the [Iter8 `subject` spec](#create-iter8-specs).

    ```shell
    kubectl label deploy recommender-stable iter8.tools/detect=true
    kubectl label svc recommender-stable iter8.tools/detect=true
    ```

=== "Backend variant 2"

     Deploy variant 2 (candidate version) of the backend.

    ```shell
    kubectl create deployment recommender-candidate --image=iter8/abn-sample-backend:0.14-v2
    kubectl expose deployment recommender-candidate --port=8091
    ```

    Label backend resources so that Iter8 can detect them. These resources will be referenced in the [Iter8 `subject` spec](#create-iter8-specs).

    ```shell
    kubectl label deploy recommender-candidate iter8.tools/detect=true
    kubectl label svc recommender-candidate iter8.tools/detect=true
    ```

=== "subject"

    Create the `subject` for the `backend` server. The subject also includes variant weights: the number of users mapped to each variant is proportional to its weight.

    ```shell
    cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: recommender
      labels:
        app.kubernetes.io/managed-by: iter8    
        iter8.tools/kind: subject
        iter8.tools/version: v0.14
    data:
      spec: |
        variants: 
          # Variant 1 gets 3/4th of the users. Variant 2 gets the rest.
        - weight: 3
          resources:
          - gvr: svc
            name: recommender-stable
          - gvr: deploy
            name: recommender-stable
        - resources:
          - gvr: svc
            name: recommender-candidate
          - gvr: deploy
            name: recommender-candidate
        routing:
          # turn on Iter8 SDK. Default is false
          sdk: true
    EOF
    ```

## Create experiment

Create the `experiment`.

```shell
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: abn-test
  labels:
    app.kubernetes.io/managed-by: iter8   
    iter8.tools/kind: iter8/experiment
    iter8.tools/version: v0.14
stringData:
  spec: |
    subject: recommender
    # this experiment has no triggers
    # so, a single run will be launched and it will never be updated
    # the experiment run will be garbage collected automatically when the subject is deleted
    tasks:
    # Fetch Iter8 SDK metrics from the Iter8 service
    - name: sdkmetrics
      subject: recommender
      # address of the Iter8 service
      endpoint: iter8-service.iter8-system.svc.cluster.local:50051
    # this is a multi-loop experiment that uses a cronjob under the covers; 
    # loops are scheduled once per minute
    cronjobSchedule: "*/1 * * * *"
EOF
```

## Simulate users
Your application will receive requests from real users in production. For the purposes of this tutorial, use [this script](https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh) to simulate user traffic.

First, port-forward the frontend.
```shell
kubectl port-forward service/frontend 8090:8090
```

In a separate terminal, run the script.
```shell
curl -s https://raw.githubusercontent.com/iter8-tools/docs/main/samples/abn-sample/generate_load.sh | sh -s --
```

<!-- ??? note "Behind the scenes"
    Suppose the app under consideration is an online store. Users can use the `List()` API of the frontend to view a ranked list of products. They can use the `Buy()` API of the frontend to buy items in their shopping cart. When the frontend receives a `List()` request, it invokes the `Rank()` API of the bankend to receive a list of product recommendations, and surfaces this to the user.

    The following sequence diagram illustrates how users, frontend, backend, and the Iter8 service interact. The frontend invokes the `Lookup(name, user)` API of the Iter8 service to determine the backend Variant; here, `name` is the name of the app being A/B tested (`recommender`) in this tutorial, and `user` is the unique ID of the user. It uses this Variant to receive recommendations from the backend and answer the user query. When the user buys items, it uses the `WriteMetric(name, user, metric, value)` API of the Iter8 service to report business metrics (such as total price, and the number of items in this example).

    ```mermaid
    sequenceDiagram
        actor U1 as User1
        actor U2 as User2
        participant F as Frontend
        participant B1 as Backend <br> Variant 1
        participant B2 as Backend <br> Variant 2
        participant I as Iter8 service
        U1->>F: List()
        activate F
        F->>I: Lookup(name, user)
        activate I
        I-->>F: Variant 1
        deactivate I
        F->>B1: Rank()
        B1-->>F: Recommendations
        F-->>U1: Products
        deactivate F
        U2->>F: List()
        activate F
        F->>I: Lookup(name, user)
        activate I
        I-->>F: Variant 2
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
        I-->>F: Variant 2
        deactivate I
        F->>B2: Rank()
        B2-->>F: Recommendations
        F-->>U2: Products
        deactivate F
    ```

    The Iter8 service provides the following guarantees:
    
    1. Weight: the number of users assigned to each variant is (approximately) proportional to its weight.
    
    2. User stickiness: when two or more variants of a backend are deployed, for a given user, every `Lookup(name, user)` call returns the same variant.
    
    3. Readiness: when a variant is returned by a `Lookup(name, user)` call, the Kubernetes resources corresponding to this variant are guaranteed to exist, and be ready to serve frontend requests. In particular, this guarantee implies that to you can update any variant, or delete candidate variants at any point in time with zero downtime (i.e., without loss of requests from the frontend). -->

## View experiment report

Create the experiment report as follows.

```shell
iter8 report -s recommender -e abn-test -o html > report.html # view in a browser
```

??? note "Sample report"
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

    Metric                                  | Variant 1   | Variant 2
    -------                                 | -----     | -----
    abn/recommender/duration                | 368 (hr)  | 120 (hr)
    abn/recommender/numusers                | 628       | 149
    abn/recommender/lookups                 | 2725      | 822
    abn/recommender/sales/mean              | 51.31     | 52.79
    abn/sample_metric/stddev                | 82.52     | 69.89
    abn/recommender/sales/conversion        | 82        | 43
    abn/recommender/sales/conversion-rate   | 13.05 (%) | 28.85 (%)
    ```

The above experiment report allows you to compare variants and select a winner. Since this is [multi-loop experiment](../../getting-started/concepts.md#iter8-experiment), the values in the report are updated with each loop. If the candidate variant is deemed the winner, it can be promoted as the latest stable version, and resources corresponding to variant 2 (candidate) can be deleted.

## Promote variant 2

Promote variant 2 (candidate) as the latest stable version.

```shell
kubectl set image deployment/recommender-stable \
abn-sample-backend=iter8/abn-sample-backend:0.14-v2
```

## Cleanup

Remove the sample application and Iter8 specs.

```shell
kubectl delete \
deploy/frontend deploy/recommender-stable deploy/recommender-candidate \
service/frontend service/recommender-stable service/recommender-candidate
kubectl delete cm/recommender secret/abn-test
```

