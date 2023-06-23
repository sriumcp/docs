Install Iter8 inside a Kubernetes cluster using Helm or Kustomize.

=== "Helm"
    Install Iter8 using `helm` as follows.

    ```shell
    helm install \
    --repo https://iter8-tools.github.io/iter8 iter8-traffic traffic \
    -n iter8-system
    ```
    
=== "Kustomize"
    Install Iter8 using `kustomize` as follows.

    ```shell
    kubectl apply \
    -k 'https://github.com/iter8-tools/iter8.git/kustomize/traffic/clusterScoped?ref=v0.14.8' \
    -n iter8-system
    ```

