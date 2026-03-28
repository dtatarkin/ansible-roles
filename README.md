# dtatarkin.infra

Ansible Galaxy collection for provisioning a full-stack infrastructure on a single VPS &mdash; from base OS setup and mesh networking to a production-ready Kubernetes platform with databases, monitoring, CI/CD, and GitOps.

Built around **K3s** on **Ubuntu 24 (Noble)** with **Hetzner Cloud** support and **Nebula** overlay networking.

## Key Features

- **State-driven roles** &mdash; every role supports `present` / `absent` via a `<role>_state` variable, making teardown as easy as setup
- **Zero-touch Helm** &mdash; K3s HelmChart CRDs auto-apply manifests; no external Helm binary required
- **SOPS secrets** &mdash; encrypted inventory files, decrypted at runtime, never stored on disk
- **Small-VPS-friendly** &mdash; resource defaults tuned for Hetzner cx23 (2 vCPU / 4 GB RAM)
- **GitOps-ready** &mdash; optional Flux role for continuous deployment from a Git repository

## Roles

### Host-Level

| Role | Description |
|------|-------------|
| **ubuntu24** | Base OS packages (jq, net-tools, mc, etc.) |
| **nebula** | Nebula mesh VPN overlay (v1.10.3) |
| **hcloud_volumes** | Format & mount Hetzner Cloud volumes |

### Kubernetes (K3s)

| Role | Description |
|------|-------------|
| **cert_manager** | Automated TLS via Let's Encrypt (cert-manager v1.17.1) |
| **traefik** | Ingress controller configuration (K3s built-in) |
| **hcloud_csi** | Hetzner Cloud CSI driver (v2.12.0) |
| **local_path_slow** | Local storage provisioner for HDD / slow volumes |
| **cnpg** | PostgreSQL operator & managed clusters (CloudNativePG v0.27.1) |
| **valkey** | In-memory cache &mdash; Redis fork (v0.9.3) |
| **clickhouse** | Analytics database (v0.14.0) |
| **monitoring** | Victoria Metrics + Grafana + Alertmanager (v0.72.2) |
| **temporal** | Workflow engine (v1.29) |
| **woodpecker** | CI/CD pipelines (v3.5.1 server + v3.13.0 agent) |
| **docker_registry** | Private Docker registry with UI |
| **oauth2_proxy** | OAuth2 authentication proxy (Google, GitHub, etc.) |
| **metabase** | Business intelligence & analytics dashboards |
| **homepage** | Service discovery dashboard |
| **flux** | GitOps continuous deployment (Flux v2.14.0) |

## Requirements

- **Ansible** >= 2.15
- **Target OS** &mdash; Ubuntu 24.04 (Noble)
- **K3s** installed on the target host (for Kubernetes roles)
- `community.docker` collection (required by the `woodpecker` role)

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install dtatarkin.infra
```

### From Git

```bash
ansible-galaxy collection install git+https://github.com/dtatarkin/ansible-roles.git
```

## Quick Start

Use roles in a playbook by their fully qualified collection name:

```yaml
- hosts: vps
  roles:
    - role: dtatarkin.infra.ubuntu24

    - role: dtatarkin.infra.nebula
      nebula_state: present

    - role: dtatarkin.infra.cert_manager
      cert_manager_state: present

    - role: dtatarkin.infra.cnpg
      cnpg_state: present
      cnpg_databases:
        - name: myapp
          owner: myapp

    - role: dtatarkin.infra.monitoring
      monitoring_state: present
```

Every role exposes sensible defaults in `defaults/main.yml`. Override what you need:

```yaml
- role: dtatarkin.infra.valkey
  valkey_state: present
  valkey_storage_size: 2Gi
  valkey_maxmemory: 512mb
```

## Recommended Deployment Order

```
ubuntu24 / hcloud_volumes     # OS & storage
        |
      nebula                  # Mesh networking
        |
  local_path_slow / hcloud_csi  # Storage classes
        |
    cert_manager              # TLS certificates
        |
      traefik                 # Ingress
        |
       cnpg                   # PostgreSQL
        |
  valkey / clickhouse         # Cache & analytics DB
        |
  monitoring / temporal /     # Platform services
  woodpecker / docker_registry /
  oauth2_proxy / metabase /
  homepage
        |
       flux                   # GitOps (last)
```

## Configuration

All roles follow a consistent variable naming convention:

| Pattern | Example | Purpose |
|---------|---------|---------|
| `<role>_state` | `cnpg_state: present` | Enable or disable a role |
| `<role>_namespace` | `cnpg_namespace: cnpg-system` | Kubernetes namespace |
| `<role>_storage_class` | `cnpg_storage_class: local-path` | PVC storage class |
| `<role>_storage_size` | `cnpg_storage_size: 5Gi` | PVC size |
| `<role>_chart_version` | `cnpg_chart_version: 0.27.1` | Helm chart version |
| `<role>_resources` | `cnpg_resources: {requests: ...}` | CPU / memory requests |
| `<role>_domain` | `docker_registry_domain: registry.example.com` | Service hostname |

See each role's `defaults/main.yml` for the full list of options.

## License

[MIT](https://opensource.org/licenses/MIT)
