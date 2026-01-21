# HTTP Mirror

A Kubernetes-native solution for mirroring HTTP directory listings and serving them via a web server.

## Overview

HTTP Mirror consists of two components:
- **Web Server** (Deployment): Always-running web server that serves mirrored files
- **Updater** (CronJob): Scheduled job that downloads/updates files from configured HTTP directory listings

## Features

- ✅ Smart mirroring with change detection (HTTP HEAD requests)
- ✅ Built-in web server with custom directory listings
- ✅ Kubernetes-native scheduling with timezone support
- ✅ Configurable rate limiting and retry logic
- ✅ Persistent storage for mirrored files
- ✅ Health checks and monitoring support
- ✅ Multi-target support with individual configurations

## Installation

### Add the Helm repository

```bash
helm repo add jhofer-cloud https://charts.jhofer.org
helm repo update
```

### Install the chart

```bash
helm install my-mirror jhofer-cloud/http-mirror \
  --set targets[0].name=example \
  --set targets[0].url=http://example.com/files/ \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=mirror.example.com
```

### Install with values file

```yaml
# values.yaml
targets:
  - name: "documentation"
    url: "http://docs.example.com/files/"
  - name: "downloads"
    url: "https://releases.site.com/downloads/"
    rateLimit: "200k"  # Slower rate for this target

updater:
  schedule:
    cron: "0 8 * * *"  # Daily at 8AM
    timezone: "Europe/Berlin"

server:
  replicaCount: 2

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: "mirror.example.com"
      paths:
        - path: "/"
          pathType: Prefix
  tls:
    - secretName: mirror-tls
      hosts:
        - mirror.example.com

storage:
  size: "50Gi"
```

```bash
helm install my-mirror jhofer-cloud/http-mirror -f values.yaml
```

## Configuration

### Targets

Each target represents an HTTP directory listing to mirror:

```yaml
targets:
  - name: "my-files"           # Unique name for the target
    url: "http://example.com/files/"  # URL to mirror
    # All other settings inherit from defaults
    
  - name: "slow-server"
    url: "https://slow.example.com/downloads/"
    rateLimit: "100k"         # Override rate limit
    waitBetweenRequests: 5    # Wait 5 seconds between requests
    maxDepth: 10              # Deeper recursion
```

### Scheduling

The updater runs on a configurable schedule:

```yaml
updater:
  schedule:
    cron: "0 2 * * 0"          # Weekly on Sunday at 2AM
    timezone: "America/New_York"  # Any IANA timezone
```

### Storage

Configure persistent storage for mirrored files:

```yaml
storage:
  enabled: true
  size: "100Gi"
  storageClassName: "fast-ssd"
  accessModes:
    - ReadWriteOnce
```

### Ingress

Multiple ingress patterns are supported:

```yaml
ingress:
  enabled: true
  hosts:
    # Serve all targets with directory listing
    - host: "mirror.example.com"
      paths:
        - path: "/"
          pathType: Prefix
    
    # Serve specific target only
    - host: "docs.mirror.example.com" 
      paths:
        - path: "/"
          pathType: Prefix
          target: "documentation"  # Specific target name
```

## Values Reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `defaults.userAgent` | string | `"Mozilla/5.0 (compatible; HttpMirror/1.0; +https://github.com/jhofer-cloud/http-mirror) Friendly Educational Mirror"` | Default User-Agent for requests |
| `defaults.rateLimit` | string | `"500k"` | Default rate limit (e.g., "500k", "1m", "100") |
| `defaults.retries` | int | `3` | Default number of retries for failed requests |
| `defaults.maxDepth` | int | `5` | Default maximum recursion depth |
| `defaults.timeout` | int | `30` | Default request timeout in seconds |
| `defaults.waitBetweenRequests` | int | `1` | Default wait time between requests in seconds |
| `targets` | list | `[]` | List of mirror targets (see above) |
| `updater.enabled` | bool | `true` | Enable the updater CronJob |
| `updater.schedule.cron` | string | `"0 8 * * *"` | Cron schedule for updates |
| `updater.schedule.timezone` | string | `"Europe/Berlin"` | Timezone for the schedule |
| `server.enabled` | bool | `true` | Enable the web server |
| `server.replicaCount` | int | `1` | Number of server replicas |
| `storage.enabled` | bool | `true` | Enable persistent storage |
| `storage.size` | string | `"10Gi"` | Storage size |
| `ingress.enabled` | bool | `false` | Enable ingress |

## Monitoring

The chart supports Prometheus monitoring via ServiceMonitor:

```yaml
serviceMonitor:
  enabled: true
  labels:
    monitoring: "true"
```

## Architecture

```
┌─────────────────┐    ┌─────────────────┐
│   CronJob       │    │   Deployment    │
│   (Updater)     │    │   (Web Server)  │
│                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   Mirror    │ │    │ │ File Server │ │
│ │   Logic     │ │    │ │ + Directory │ │
│ │             │ │    │ │ Listings    │ │
│ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘
         │                       │
         └───────────────────────┘
                    │
            ┌─────────────────┐
            │ Persistent      │
            │ Volume          │
            │ (Shared Data)   │
            └─────────────────┘
```

## License

[Apache License 2.0](../../LICENSE)