# Istio-enabled [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

An example repository showing how the kube-prometheus-stack Helm chart could be
patched to support Istio without modifying Kubernetes resources after installing
the chart.

Based on <https://github.com/prometheus-community/helm-charts/issues/145>.

I used the `post-renderer` technique with kustomize (see [1], [2]) to patch the
existing and inject extra resources into the helm-generated k8s manifests.

## Prerequisites

- [Kustomize](https://github.com/kubernetes-sigs/kustomize/releases) 3.5+
  installed in `$PATH`.
- [Helm](https://github.com/helm/helm/releases) 3.1+ installed.
- A k8s cluster (or use `helm template` instead of `helm upgrade -i` in the
  command below)

## Usage

1. Create the `metrics` namespace and enable Istio injection for the namespace.
1. Make the [`kustomize-pipe`](kustomize-pipe)
   executable: `chmod +x kustomize-pipe`.
1. Run:
   ```shell
   helm upgrade -i monitoring prometheus-community/kube-prometheus-stack \
     --namespace metrics \
     --values values.yaml \
     --post-renderer ./kustomize-pipe \
     --debug --dry-run
   ```

The command will install the kube-prometheus-stack chart
using [`values.yaml`](values.yaml) with Istio-specific patches applied by
Kustomize (see [`kustomization.yaml`](kustomization.yaml)):

- Enforces Istio mTLS for the namespace while allowing Prometheus k8s service
  discovery ([`networking.yaml`](networking.yaml)).
- Patches specific `ServiceMonitors` to use the injected Istio mTLS
  certificates.
- Sets `appProtocol: http` to known `Service` ports to fix the
  Istio [protocol selection][3].

[1]: https://helm.sh/docs/topics/advanced/#post-rendering

[2]: https://github.com/thomastaylor312/advanced-helm-demos/tree/master/post-render

[3]: https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection
