# node-label-inheritor
This reconciler will update pod labels to include node labels of the node the pod is running on.

Check out the deployment example at `examples/deployment.yaml`.

## Install
```
helm upgrade --install node-label-inheritor oci://ghcr.io/protosam/charts/node-label-inheritor
```
