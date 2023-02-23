---
template: main.html
---

# Iter8

Iter8 is the Kubernetes release optimizer built for DevOps, MLOps, SRE and data science teams. Iter8 makes it easy to ensure that Kubernetes apps and ML models perform well and maximize business value.

## Iter8 experiment
Iter8 introduces the notion of an *experiment* to facilitate various release optimization use-cases for apps and ML models, as illustrated in the picture below.

![Iter8 experiment](../images/iter8-intro-dark.png)
<!-- {: style="width:80%"} -->

## Use-cases

1.  A/B/n testing of apps and ML models, with traffic splitting, user stickiness, business metrics collection, and assessment.
2.  Performance testing and service-level objective (SLO) validation of HTTP and gRPC services.
3.  Canary testing with traffic splitting and SLO validation with metrics from any metrics store (e.g., Prometheus).
4.  Traffic mirroring and SLO validation with metrics from any metrics store (e.g., Prometheus).

## How it works
A brief overview of how Iter8 works for the various use-cases is presented below.

=== "A/B/n testing"
    Following is an example of Iter8 A/B testing in a distributed application. The **frontend** uses **Iter8 SDK** to route users to **backend versions** and push business metrics. **Iter8 experiment** fetches metrics and assesses versions. **Iter8 service** backs the Iter8 SDK and supports the Iter8 experiment.

    ![A/B testing](../tutorials/abn/images/abn.backend.png){: style="width:80%"}


=== "Performance testing"
    **Iter8 experiment** checks if the app is ready, generates load, collects metrics, and validates service-level objectives (SLOs).
    === "HTTP"

        ![HTTP performance testing](images/http.png)

    === "gRPC"

        ![gRPC performance testing](../tutorials/images/grpc.png)

=== "Canary testing"
    **Iter8 experiment** checks if the app is ready, fetches metrics from a metrics store (e.g., Prometheus), and validates service-level objectives (SLOs). **Service mesh or reverse proxy** handles traffic splitting.

    ![Canary testing](../tutorials/custom-metrics/images/two-or-more-versions.png)

=== "Traffic mirroring"
    **Iter8 experiment** checks if the app is ready, fetches metrics from a metrics store (e.g., Prometheus), and validates service-level objectives (SLOs). **Service mesh or reverse proxy** handles traffic mirroring.
    
    ![Canary testing](../tutorials/custom-metrics/images/two-or-more-versions.png)

## Features

* Designed to work with any Kubernetes resource (including custom resources), any ML serving/app technology in Kubernetes, any service mesh, any CI/CD platform, and any metrics store
* Simple business metrics collection using Iter8 SDK
* Automated experiments with Iter8 AutoX controller. Manual experiments with Iter8 CLI
* HTML and text reports of experiments with metrics-based version assessment
* Simple declarative experiment specification
* Single and multi-loop experiments
* Iter8 GitHub action
* GitHub and Slack integrations


## Development Status
Iter8 is actively developed by the community. Iter8 builds on a few awesome open source projects including:

- [Helm](https://helm.sh)
- [Fortio](https://github.com/fortio/fortio)
- [ghz](https://ghz.sh)
- [plotly.js](https://github.com/plotly/plotly.js)

Iter8 releases can be found [here](https://github.com/iter8-tools/iter8).