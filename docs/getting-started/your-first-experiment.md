---
template: main.html
---

# Your First Experiment

> Deploy an app. Iter8 will detect the app and run a performance testing experiment for it. Whenever the app gets updated, Iter8 will re-run the experiment.

![HTTP load testing](images/http.png)

???+ warning "Before you begin"
    1. Get access to a Kubernetes cluster. For learning and local development, you can use [Kind](https://kind.sigs.k8s.io/) or [Minikube](https://minikube.sigs.k8s.io/docs/).
    2. [Install Iter8 CLI](install.md#install-iter8-cli).
    ```shell
    go install github.com/iter8-tools/iter8@v0.14
    ```
    3. [Install Iter8 service](install.md#install-iter8-service) such that it can manage apps in the `default` namespace. For example:
    ```shell
    helm upgrade --install iter8-service iter8-service \
    --repo https://iter8-tools.github.io/iter8 --version 0.14.x \
    --set "appNamespaces=*" -n iter8-system
    ```
    The above command installs Iter8 service in the `iter8-system` namespace. The service can manage apps in all namespaces.

***

## Create application

Create a sample application with a deployment resource and a service resource. A `subject` in Iter8 is a special type of [configmap](https://kubernetes.io/docs/concepts/configuration/configmap/) that lists the app's resources. It is an integral part of specifying an app. Create the subject along with the deployment and service.


=== "deployment and service"

    Create the deployment and service.

    ```shell
    kubectl create deploy httpbin --image=kennethreitz/httpbin --port=80
    kubectl expose deploy httpbin --port=80
    ```

    Label the deployment and service so that Iter8 can detect them.
    ```shell
    kubectl label deploy httpbin iter8.tools/detect=true
    kubectl label svc httpbin iter8.tools/detect=true
    ```

=== "subject"
    
    Create the subject as follows.

    ```shell
    cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: ConfigMap
    metadata:
      # httpbin is the name of the subject
      name: httpbin
      labels:
        # standard Iter8 labels applied on subjects
        app.kubernetes.io/managed-by: iter8
        iter8.tools/kind: subject
        iter8.tools/version: v0.14
    data:
      spec: |
        # httpbin has a single variant
        variants:
        # the variant has a service named httpbin and a deployment named httpbin
        - resources:
          - gvr: svc
            name: httpbin
          - gvr: deploy
            name: httpbin
    EOF
    ```

## Create experiment

An `experiment` in Iter8 is a special type of [secret](https://kubernetes.io/docs/concepts/configuration/secret/) that specifies an Iter8 experiment for a subject. Iter8 can launch new runs of an experiment whenever it detects changes to the app defined by the subject. Create the `experiment` as follows.

```shell
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  # name of the experiment
  name: performance-test
  labels:
    # standard Iter8 labels applied on experiment specs
    app.kubernetes.io/managed-by: iter8   
    iter8.tools/kind: iter8/experiment
    iter8.tools/version: v0.14
stringData:
  spec: |
    # trigger a new run of the experiment whenever there is a change to
    # the signature of variant 1 of the subject httpbin
    # older runs will be garbage collected automatically
    owner: httpbin/1/sig
    tasks:
    # check if Kubernetes resources involved in the experiment are ready
    - name: ready
      svc: httpbin
      deploy: httpbin
    # generate requests for HTTP service and collect metrics
    - name: http
      url: http://httpbin.default/get
    # assess httpbin variant using metrics and service-level objectives (SLOs)
    - name: assess
      SLOs:
        upper: 
          http/latency-mean: 50
          http/error-count: 0
EOF
```

***

## Watch experiment run events
Iter8 will run the `performance-test` experiment. Watch the events related to this experiment run as follows.
```shell
kubectl get events -w --field-selector='source=httpbin/performance-test'
```

In ~1 min, you should observe an event that indicates that the experiment run has completed.

***

## View experiment run report
Create the experiment run report as follows.

```shell
iter8 report -s httpbin -e performance-test -o html > report.html # view in a browser
```

??? note "Sample report"    
    ![HTML report](images/report.html.png)
        
***

Congratulations :tada: You ran your first Iter8 experiment.

***

## Trigger experiment re-run

Update the sample app.

```shell
kubectl set env deploy/httpbin IMPROVE=everything
```

The above changes variant 1 of `httpbin`. As a result, Iter8 will re-run the `performance-test` experiment. Watch events and view report for the second experiment run in the same manner as the first.

***

Congratulations :tada: You ran your second Iter8 experiment.

***


## Cleanup
Remove the app and experiment specs.

```shell
kubectl delete svc/httpbin deploy/httpbin cm/httpbin secret/httpbin-performance-test
```

