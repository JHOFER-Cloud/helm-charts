<h1 align="center">
  <br><img src="project-logo.svg" height="192px">
  <br>
  Helm Charts
  <br>
</h1>

<h4 align="center">My own public Helm Charts repository.</h4>

<p align="center">
  <a href="https://github.com/jhofer-cloud/helm-charts/actions/workflows/lint-test.yaml">
      <img src="https://github.com/jhofer-cloud/helm-charts/actions/workflows/lint-test.yaml/badge.svg">
  </a>
  <a href="https://github.com/jhofer-cloud/jhofer-cloud/actions/workflows/release.yaml">
      <img src="https://github.com/jhofer-cloud/helm-charts/actions/workflows/release.yaml/badge.svg">
  </a>
  <a href="https://artifacthub.io/packages/search?repo=jhofer-cloud-charts">
      <img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/jhofer-cloud-charts">
  </a>
</p>

<p align="center">
  <a href="#getting-started">Getting Started</a> •
  <a href="#chart-sources">Chart Sources</a>
</p>

# Getting Started

Add my repository to helm and use one of my chart:

```
helm repo add jhofer-cloud https://charts.jhofer.org
helm install proxmox-pve-exporter jhofer-cloud/proxmox-pve-exporter
```

# Chart Sources

- `jhofer-cloud/proxmox-exporter`: Chart for Prometheus [Proxmox exporter](https://github.com/prometheus-pve/prometheus-pve-exporter).
