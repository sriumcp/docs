---
template: main.html
title: Install Iter8
hide:
- toc
---

# Install Iter8

## Install Iter8 CLI

Use [Go](https://go.dev/ref/mod#go-install) to install Iter8 CLI on your local machine.

```shell
go install github.com/iter8-tools/iter8@v0.14
```

## Install or upgrade Iter8 service

Use [Helm](https://helm.sh) to install or upgrade Iter8 service in the Kubernetes cluster.

=== "All namespaces"
    ```shell
    helm upgrade --install iter8-service iter8-service \
    --repo https://iter8-tools.github.io/iter8 --version 0.14.x \
    --set "appNamespaces=*" -n iter8-system
    ```
    The above command installs Iter8 service in the `iter8-system` namespace. The service can manage apps in all namespaces.

=== "Single namespace"
    ```shell
    helm upgrade --install iter8-service iter8-service \
    --repo https://iter8-tools.github.io/iter8 --version 0.14.x \
    -n pluto
    ```
    The above command installs Iter8 service in the `pluto` namespace. The service can manage apps in the `pluto` namespace only.

=== "Selected namespaces"
    ```shell
    helm upgrade --install iter8-service iter8-service \
    --repo https://iter8-tools.github.io/iter8 --version 0.14.x \
    --set "appNamespaces={mercury,earth,pluto}" -n iter8-system
    ```
    The above command installs Iter8 service in the `iter8-system` namespace. The service can manage apps in `mercury`, `earth`, and `pluto` namespaces only.
