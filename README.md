# kubeflow-gitops-demo

Bootstrap a self-service AI platform with GitOps

## Reference environment

Single-node K3s cluster on `pi2.2xlarge.4` ECS instance running on [Huawei Cloud](https://www.huaweicloud.com/intl/en-us/). The ECS instance has 8 vCPU, 32G memory, 128G system disk and 1 NVIDIA Tesla T4 datacenter GPU.

The K3s `config.yaml` given below. Traefik ingress is disabled and the per-node pod limit is raised from 110 to 250.

```yaml
disable:
  - traefik
kubelet-arg:
  - "max-pods=250"
```

## Prerequisites

Deploy `v0.18.0` of [argoproj-labs/argocd-operator](https://github.com/argoproj-labs/argocd-operator) with `make deploy` and set the following environment variable in the operator workload.

```bash
ARGOCD_CLUSTER_CONFIG_NAMESPACES=argocd
```

Now deploy the cluster ArgoCD instance in namespace `argocd` ensuring the namespace already exists.

```yaml
---
apiVersion: argoproj.io/v1beta1
kind: ArgoCD
metadata:
  name: argocd
spec: {}
```

## Bootstrapping the self-service AI platform

Apply the Kustomize build under directory `bootstrap/overlays/v1/` to namespace `argocd`. This repository uses "app-of-apps" pattern to bootstrap the Kubeflow AI reference platform with GPU support enabled.

```bash
kubectl kustomize bootstrap/overlays/v1/ | \
    kubectl -n argocd apply -f - --server-side
```

## License

[Apache 2.0](./LICENSE)
