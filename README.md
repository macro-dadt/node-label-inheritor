# node-label-inheritor
A reconciler to copy node labels onto pods.

This is useful for building service selectors that only connect to pods scheduled on specific nodes. It may provide some extra tooling to work around some of the short comings of [Topology Aware Hints](https://kubernetes.io/docs/concepts/services-networking/topology-aware-hints/).

## Quick Start
If you need a cluster to test with, you can setup a [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster with with `examples/kind-config.yaml`.
```sh
curl https://raw.githubusercontent.com/protosam/node-label-inheritor/master/examples/kind-config.yaml | kind create cluster --config -
```

Helm is used to install `node-label-inheritor`.
```sh
helm install --namespace kube-system node-label-inheritor oci://ghcr.io/protosam/charts/node-label-inheritor
```

Applying `examples/deployment.yaml` to the kind cluster create previously will be seen by `node-label-inheritor`.
```sh
curl https://raw.githubusercontent.com/protosam/node-label-inheritor/master/examples/deployment.yaml | kubectl apply -f -
```

These are the resulting pods.
```sh
$ kubectl get pods -owide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx-deployment-5fc748d44b-9fwcm   1/1     Running   0          14s   10.244.2.3   kind-worker    <none>           <none>
nginx-deployment-5fc748d44b-dxtsm   1/1     Running   0          14s   10.244.1.4   kind-worker2   <none>           <none>
```

If you inspect the pods, the `node-label-inheritor` controller will eventually update the labels as follows.
```sh
$ kubectl get pod nginx-deployment-5fc748d44b-9fwcm -oyaml | yq .metadata.labels
app: nginx
pod-template-hash: 5fc748d44b
topology.kubernetes.io/region: us-nowhere-1
topology.kubernetes.io/zone: us-nowhere-1a
```

Both `topology.kubernetes.io/region` and `topology.kubernetes.io/zone` were added by `node-label-inheritor` to match the same labels of the nodes running the pods.

The configuration for these keys is just done with comma-separated values via annotation.
```yaml
metadata:
  annotations:
    nodeLabelsInherited: topology.kubernetes.io/region, topology.kubernetes.io/zone
```

For the value, `node-label-inheritor` is pretty forgiving  with formatting. It trims any whitespace encountered after it splits on commas.

These are valid as well.
```yaml
metadata:
  annotations:
    nodeLabelsInherited: |
      topology.kubernetes.io/region, 
      topology.kubernetes.io/zone
---
metadata:
  annotations:
    nodeLabelsInherited: topology.kubernetes.io/region,topology.kubernetes.io/zone
```

Lastly, this shows where `topology.kubernetes.io/region` and `topology.kubernetes.io/zone` were sourced from.
```sh
$ kubectl get nodes       
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   16m   v1.25.3
kind-worker          Ready    <none>          16m   v1.25.3
kind-worker2         Ready    <none>          16m   v1.25.3
kind-worker3         Ready    <none>          16m   v1.25.3

$ kubectl get node kind-worker2 -oyaml | yq .metadata.labels
beta.kubernetes.io/arch: arm64
beta.kubernetes.io/os: linux
kubernetes.io/arch: arm64
kubernetes.io/hostname: kind-worker2
kubernetes.io/os: linux
topology.kubernetes.io/region: us-nowhere-1
topology.kubernetes.io/zone: us-nowhere-1b
```
