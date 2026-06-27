# AIDoc — Fleet Deployment

GitOps deployment manifests for the [AIDoc](https://github.com/hoffeloffe/AIDoc)
app, delivered to a Kubernetes cluster with [Rancher Fleet](https://fleet.rancher.io/).

Fleet watches this repo and reconciles the cluster to match the manifests under
[`fleet/`](./fleet) — there is no manual `kubectl apply`.

## What gets deployed

Namespace: `aidoc-system`

| Manifest | Resource | Notes |
| --- | --- | --- |
| [`fleet/deployment.yaml`](./fleet/deployment.yaml) | `Deployment` aidoc | 2 replicas, liveness/readiness probes, CPU/memory requests + limits |
| [`fleet/service.yaml`](./fleet/service.yaml) | `Service` aidoc-service | ClusterIP, `80 -> 3000` |
| [`fleet/ingress.yaml`](./fleet/ingress.yaml) | `Ingress` aidoc-ingress | nginx, host `aidoc.local` |
| [`fleet/fleet.yaml`](./fleet/fleet.yaml) | Fleet bundle config | targets clusters labelled `env: prod` |

The container image is **pinned by digest** (not `:latest`) so deploys are
reproducible and rollback-able. Bump the digest when a new image is published.

> The Deployment runs on ARM64 nodes (`nodeSelector: kubernetes.io/arch: arm64`)
> for a Raspberry Pi cluster.

## How it works

1. The app repo builds and pushes a container image.
2. Bump the image digest in `fleet/deployment.yaml` and commit.
3. Fleet detects the change and rolls it out to matching clusters.

## Local validation

Validate the manifests before committing:

```bash
kubeconform -strict -ignore-missing-schemas \
  fleet/deployment.yaml fleet/service.yaml fleet/ingress.yaml
```

> `fleet/values.yaml` is currently unused (these are raw manifests, not a Helm
> chart). It's kept as a reference for a possible future Helm-based bundle.
