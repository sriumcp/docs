---
template: main.html
title: Install Iter8
hide:
- toc
---

# Install or upgrade Iter8 CLI

--8<-- "docs/getting-started/installbrewbins.md"

--8<-- "docs/getting-started/installghaction.md"

# Install or upgrade Iter8 service

Iter8 service can be installed or upgraded using [Helm](https://helm.sh) as follows.

=== "All namespaces"
    Enable the Iter8 service to work with apps in any namespace.

    ```shell
    helm upgrade iter8-service iter8 --install --repo https://iter8-tools.github.io/hub --set role=svc \
    --set "appNamespaces=*" --version 0.14.x -n iter8-system --create-namespace
    ```

=== "Single namespace"
    Restrict the Iter8 service to work with apps in a single namespace (`pluto` in the example below).

    ```shell
    helm upgrade iter8-service iter8 --install --repo https://iter8-tools.github.io/hub --set role=svc \
     --version 0.14.x -n pluto --create-namespace
    ```

=== "Selected namespaces"
    Restrict the Iter8 service to work with apps inside a pre-existing set of namespace (`mercury`, `earth`, and `pluto` in the example below).

    ```shell
    helm upgrade iter8-service iter8 --install --repo https://iter8-tools.github.io/hub --set role=svc \
    --set "appNamespaces={mercury,earth,pluto}"  --version 0.14.x -n iter8-system --create-namespace
    ```

The use of `iter8-system` namespace with the `-n` flag in the above commands is recommended but not required. For a complete list of Iter8 service configuration options, and for production usage considerations, please see [here](../user-guide/topics/abn/service.md).


