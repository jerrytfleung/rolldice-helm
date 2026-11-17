# rolldice-helm

[![Lint and Test Charts](https://github.com/jerrytfleung/rolldice-helm/actions/workflows/lint-test.yml/badge.svg)](https://github.com/jerrytfleung/rolldice-helm/actions/workflows/lint-test.yml)
[![Release Charts](https://github.com/jerrytfleung/rolldice-helm/actions/workflows/release.yml/badge.svg)](https://github.com/jerrytfleung/rolldice-helm/actions/workflows/release.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

A Helm chart for deploying the **rolldice** application (`ghcr.io/jerrytfleung/rolldice`) to Kubernetes.

| Chart version | App version |
|---------------|-------------|
| `0.1.1`       | `v1.0.0`    |

## Prerequisites

- Kubernetes cluster
- [Helm](https://helm.sh/docs/intro/install/) 3.x
- (Optional) An Ingress controller, if `ingress.enabled=true`
- (Optional) [Gateway API](https://gateway-api.sigs.k8s.io/guides/) CRDs and a controller, if `httpRoute.enabled=true`

## Installation

### From the Helm repository

Charts are published to GitHub Pages via [chart-releaser](https://github.com/helm/chart-releaser-action).

```bash
helm repo add rolldice https://jerrytfleung.github.io/rolldice-helm
helm repo update
helm install my-rolldice rolldice/rolldice-helm
```

### From source

```bash
git clone https://github.com/jerrytfleung/rolldice-helm.git
cd rolldice-helm
helm install my-rolldice ./charts/rolldice-helm
```

### Accessing the application

With the default `ClusterIP` service, port-forward to the pod:

```bash
kubectl port-forward svc/my-rolldice-rolldice-helm 8080:80
curl http://127.0.0.1:8080/
```

The post-install notes (`helm status my-rolldice`) print the exact commands for your service/ingress configuration.

### Uninstall

```bash
helm uninstall my-rolldice
```

## Configuration

Override values with `--set key=value` or `-f my-values.yaml`. Commonly used parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas (ignored when autoscaling is enabled) | `1` |
| `image.repository` | Container image repository | `ghcr.io/jerrytfleung/rolldice` |
| `image.tag` | Image tag (falls back to chart `appVersion`) | `v1.0.0` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `imagePullSecrets` | Secrets for pulling from a private registry | `[]` |
| `nameOverride` / `fullnameOverride` | Override generated resource names | `""` |
| `serviceAccount.create` | Create a ServiceAccount | `true` |
| `serviceAccount.automount` | Automount ServiceAccount API credentials | `true` |
| `serviceAccount.name` | ServiceAccount name (generated if empty) | `""` |
| `podAnnotations` / `podLabels` | Extra pod annotations / labels | `{}` |
| `podSecurityContext` / `securityContext` | Pod / container security context | `{}` |
| `service.type` | Kubernetes Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port the app listens on | `8080` |
| `ingress.enabled` | Create an Ingress | `false` |
| `ingress.className` | IngressClass name | `""` |
| `ingress.hosts` | Ingress host and path rules | `chart-example.local` → `/` |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| `httpRoute.enabled` | Create a Gateway API HTTPRoute | `false` |
| `httpRoute.parentRefs` | Gateways the route attaches to | `[{name: gateway, sectionName: http}]` |
| `httpRoute.hostnames` | Hostnames to match | `[chart-example.local]` |
| `httpRoute.rules` | Route rules and filters | `PathPrefix /headers` |
| `resources` | CPU/memory requests and limits | `{}` |
| `livenessProbe` / `readinessProbe` | Health probes | `httpGet /` on port `http` |
| `autoscaling.enabled` | Enable HorizontalPodAutoscaler | `false` |
| `autoscaling.minReplicas` / `maxReplicas` | HPA replica bounds | `1` / `100` |
| `autoscaling.targetCPUUtilizationPercentage` | HPA CPU target | `80` |
| `volumes` / `volumeMounts` | Additional volumes and mounts | `[]` |
| `nodeSelector` / `tolerations` / `affinity` | Pod scheduling constraints | `{}` / `[]` / `{}` |

See [`charts/rolldice-helm/values.yaml`](charts/rolldice-helm/values.yaml) for the full list with comments.

### Example: expose via Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: rolldice.example.com
      paths:
        - path: /
          pathType: Prefix
```

```bash
helm install my-rolldice rolldice/rolldice-helm -f my-values.yaml
```

## Development

Render and lint the chart locally:

```bash
helm lint charts/rolldice-helm
helm template my-rolldice charts/rolldice-helm
```

Run the chart test after installing:

```bash
helm test my-rolldice
```

### CI/CD

- **Lint and Test Charts** (`.github/workflows/lint-test.yml`) — on pull requests to `main`, runs [chart-testing](https://github.com/helm/chart-testing) `lint` and `install` against a [kind](https://kind.sigs.k8s.io/) cluster for changed charts.
- **Release Charts** (`.github/workflows/release.yml`) — on push to `main`, packages charts and publishes them to the `gh-pages` branch with chart-releaser.

> Bump `version` in `charts/rolldice-helm/Chart.yaml` with every chart change; chart-testing and chart-releaser rely on it.

## License

Licensed under the [Apache License 2.0](LICENSE).
