# Prometheus PVE Exporter Helm Chart

This Helm chart deploys the Prometheus Proxmox VE Exporter, which exposes information from Proxmox VE nodes for monitoring with Prometheus.

## Prerequisites

- A secret with Proxmox credentials

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

## Values

| Key                        | Type   | Default                                                     | Description                                          |
| -------------------------- | ------ | ----------------------------------------------------------- | ---------------------------------------------------- |
| image                      | string | `"prompve/prometheus-pve-exporter:{{ .Chart.AppVersion }}"` | Docker image for the Prometheus PVE Exporter         |
| replicaCount               | int    | `1`                                                         | Number of replicas                                   |
| secretRef.name             | string | `"proxmox-credentials"`                                     | Name of the external secret with Proxmox credentials |
| secretRef.key              | string | `"pve.yml"`                                                 | Key in the secret containing the pve.yml content     |
| serviceMonitor.enabled     | bool   | `false`                                                     | Enable ServiceMonitor for Prometheus Operator        |
| serviceMonitor.extraLabels | object | `{}`                                                        | Extra labels for the ServiceMonitor                  |
| serviceMonitor.targets     | list   | `[]`                                                        | List of Proxmox targets to monitor                   |
| extraArgs                  | list   | `[]`                                                        | Additional command-line arguments for the exporter   |
| env                        | list   | `[]`                                                        | Environment variables for the exporter container     |

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
