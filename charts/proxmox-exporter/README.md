# proxmox-exporter

![Version: 1.1.3](https://img.shields.io/badge/Version-1.1.3-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 3.5.3](https://img.shields.io/badge/AppVersion-3.5.3-informational?style=flat-square)

A Helm chart for Prometheus Proxmox VE Exporter

## Installation

Create a secret with your Proxmox VE credentials:

```bash
# Create the pve.yml file with your credentials
cat > pve.yml << EOF
default:
    user: prometheus@pve
    password: sEcr3T!
    verify_ssl: true

# For multiple hosts with different credentials
pve1:
    user: prometheus@pve
    token_name: "your-token-id-1"
    token_value: "your-token-value-1"
    verify_ssl: true

pve2:
    user: prometheus@pve
    token_name: "your-token-id-2"
    token_value: "your-token-value-2"
    verify_ssl: true
EOF

# Create the Kubernetes secret
kubectl create secret generic proxmox-credentials \
  --from-file=pve.yml=./pve.yml -n your-namespace
```

## Configuration

### Monitoring Multiple Proxmox Hosts

To monitor multiple Proxmox hosts, configure the ServiceMonitor with multiple targets:

```yaml
serviceMonitor:
  enabled: true
  extraLabels:
    release: prometheus
  targets:
    - name: pve1
      url: 192.168.1.2
      additionalMetricsRelabels:
        datacenter: dc1
    - name: pve2
      url: 192.168.1.3
      additionalMetricsRelabels:
        datacenter: dc2
```

### External Secret Configuration

The chart expects an external secret containing the Proxmox credentials. Configure the secret reference:

```yaml
secretRef:
  name: "proxmox-credentials"
  key: "pve.yml"
```

The content of the secret should be in the format described in the Prometheus PVE Exporter documentation. For multiple hosts, configure your pve.yml with different modules:

```yaml
default:
  user: prometheus@pve
  password: sEcr3T!
  verify_ssl: true

pve1:
  user: prometheus@pve
  token_name: "your-token-id-1"
  token_value: "your-token-value-1"
  verify_ssl: true

pve2:
  user: prometheus@pve
  token_name: "your-token-id-2"
  token_value: "your-token-value-2"
  verify_ssl: true
```

Then, in your ServiceMonitor configuration, specify which module to use for each target:

```yaml
serviceMonitor:
  enabled: true
  module: default # Default module
  targets:
    - name: pve1
      url: 192.168.1.2
    - name: pve2
      url: 192.168.1.3
      # Override default module
      additionalMetricsRelabels:
        module: pve2
```

## Examples

### Basic installation with two Proxmox hosts

```yaml
# values.yaml
secretRef:
  name: "proxmox-credentials"
  key: "pve.yml"

serviceMonitor:
  enabled: true
  extraLabels:
    release: prometheus
  targets:
    - name: pve1
      url: 192.168.1.2
    - name: pve2
      url: 192.168.1.3
```

### Disabling specific collectors for large deployments

```yaml
# values.yaml
extraArgs:
  - "--no-collector.config"
```

## Schema

For now it only has a simple schema generated with:

```bash
helm schema . --no-dependencies -k additionalProperties -k required
```

## Maintainers

| Name    | Email                 | Url                          |
| ------- | --------------------- | ---------------------------- |
| ojsef39 | <me+github@jhofer.de> | <https://github.com/ojsef39> |

## Requirements

| Repository                | Name | Version |
| ------------------------- | ---- | ------- |
| <https://charts.jhofer.org> | base | 1.x.x   |

## Values

| Key                                | Type   | Default                             | Description |
| ---------------------------------- | ------ | ----------------------------------- | ----------- |
| annotations                        | object | `{}`                                |             |
| containerPorts.http                | int    | `9221`                              |             |
| env                                | list   | `[]`                                |             |
| extraArgs                          | list   | `[]`                                |             |
| image.repository                   | string | `"prompve/prometheus-pve-exporter"` |             |
| image.tag                          | string | `""`                                |             |
| ingress.backend.servicePort        | string | `"http"`                            |             |
| ingress.enabled                    | bool   | `false`                             |             |
| ingress.hostname                   | string | `"proxmox-exporter.local"`          |             |
| ingress.path                       | string | `"/"`                               |             |
| ingress.pathType                   | string | `"Prefix"`                          |             |
| livenessProbe.enabled              | bool   | `true`                              |             |
| livenessProbe.failureThreshold     | int    | `6`                                 |             |
| livenessProbe.initialDelaySeconds  | int    | `180`                               |             |
| livenessProbe.path                 | string | `"/"`                               |             |
| livenessProbe.periodSeconds        | int    | `20`                                |             |
| livenessProbe.port                 | int    | `9221`                              |             |
| livenessProbe.successThreshold     | int    | `1`                                 |             |
| livenessProbe.timeoutSeconds       | int    | `5`                                 |             |
| readinessProbe.enabled             | bool   | `true`                              |             |
| readinessProbe.failureThreshold    | int    | `6`                                 |             |
| readinessProbe.initialDelaySeconds | int    | `180`                               |             |
| readinessProbe.path                | string | `"/"`                               |             |
| readinessProbe.periodSeconds       | int    | `20`                                |             |
| readinessProbe.port                | int    | `9221`                              |             |
| readinessProbe.successThreshold    | int    | `1`                                 |             |
| readinessProbe.timeoutSeconds      | int    | `5`                                 |             |
| replicaCount                       | int    | `1`                                 |             |
| secretRef.key                      | string | `"pve.yml"`                         |             |
| secretRef.name                     | string | `"proxmox-credentials"`             |             |
| service.nodePorts                  | object | `{}`                                |             |
| service.ports.http                 | int    | `9221`                              |             |
| service.type                       | string | `"ClusterIP"`                       |             |
| serviceMonitor.enabled             | bool   | `false`                             |             |
| serviceMonitor.extraLabels         | object | `{}`                                |             |
| serviceMonitor.interval            | string | `"30s"`                             |             |
| serviceMonitor.module              | string | `"default"`                         |             |
| serviceMonitor.scrapeTimeout       | string | `"30s"`                             |             |
| serviceMonitor.targets             | list   | `[]`                                |             |

---

Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
