---
template: main.html
---

# Iter8

Iter8 is the Kubernetes release optimizer built for DevOps, MLOps, SRE and data science teams. Iter8 detects new versions of apps and ML models and automates the following:

1. A/B/n testing with user stickiness and business metrics
2. Performance testing for HTTP and gRPC services
3. Traffic engineering using any service mesh or proxy
4. Version assessment using metrics from any provider

## Iter8 experiment
Iter8 introduces the notion of an *experiment* to facilitate various release optimization use-cases for apps and ML models as illustrated in the picture below.

![Iter8 experiment](../images/iter8-intro-dark.png)
<!-- {: style="width:80%"} -->

## How it works

Iter8 consists of three components, namely, Iter8 service, Iter8 SDK (optional), and the Iter8 CLI.

1. **Iter8 service:** runs inside the cluster; detects changes to apps, automates Iter8 experiments, manages service mesh resources, and implements the server-side of Iter8 SDK. Users control Iter8 and app behavior through four types of declarative specifications, namely, `subject`, `experimentspec`, `weights`, and `routingpolicy`.

2. **Iter8 SDK:** gRPC-based APIs embedded within end-user application logic; simplifies A/B/n testing; enables applications to receive or report routing information, and report business metrics.

3. **Iter8 CLI:** runs on the local machine; creates Iter8 experiment reports, and validates specs.

## Features
* Simple, powerful, automated, and declarative app/ML testing and experimentation
* Flexible integration with Kubernetes eco-system. Use Iter8 with:
    *   Any Kubernetes resource including custom resources
    *   Any ML serving/app technology
    *   Any service mesh, proxy or load balancer
    *   Any CI/CD/GitOps platform
    *   Any metrics provider
* Easy ways to collect and analyze business and performance metrics
* Intuitive experiment reports
* Single and multi-loop experiments
* GitHub and Slack notifications

## Development Status
Iter8 is actively developed by the community. Iter8 builds on a few awesome open source projects including:

- [Helm](https://helm.sh)
- [Fortio](https://github.com/fortio/fortio)
- [ghz](https://ghz.sh)
- [plotly.js](https://github.com/plotly/plotly.js)

Iter8 releases can be found [here](https://github.com/iter8-tools/iter8).