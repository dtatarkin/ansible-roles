# AGENTS.md

## Overview

Ansible Galaxy collection `dtatarkin.infra` — infrastructure roles for provisioning VPS hosts with Nebula overlay networking and K3s Kubernetes services. Targets Ubuntu 24 (Noble).

## Collection Structure

Defined in `galaxy.yml` (namespace: `dtatarkin`, name: `infra`). All roles live under `roles/`.

**Two categories of roles:**

1. **Host-level roles** (run directly on the target host):
   - `ubuntu24` — base OS packages
   - `nebula` — Nebula mesh VPN (install binary, configure, systemd service)
   - `hcloud_volumes` — format and mount Hetzner Cloud volumes

2. **K3s manifest-based roles** (deploy Kubernetes resources via K3s auto-deploy):
   - `cert_manager`, `clickhouse`, `cnpg`, `docker_registry`, `hcloud_csi`, `local_path_slow`, `monitoring`, `traefik`, `valkey`, `woodpecker`
   - These roles template manifests (HelmChart CRDs, Secrets, etc.) into `/var/lib/rancher/k3s/server/manifests/`

## Key Patterns

**State-driven roles:** Every role uses a `<role_name>_state` variable (`present`/`absent`). The `tasks/main.yml` conditionally includes `deploy.yml` or `remove.yml` based on this.

**Typical K3s role task flow:** `main.yml` → `secret.yml` (read from SOPS) → `deploy.yml` (template manifests) → wait for Helm job → `verify.yml` (check deployment health).

**Secrets management:** Secrets are read from SOPS-encrypted inventory files using `sops -d --extract` from `{{ playbook_dir }}/../inventory/group_vars/all.sops.yml`, delegated to localhost.

**Deploy mechanism:** K3s HelmChart CRD manifests are templated as Jinja2 (`.j2`) and placed in `/var/lib/rancher/k3s/server/manifests/`. K3s auto-applies them.

**Role defaults:** Each role's `defaults/main.yml` contains chart versions, namespaces, resource sizing, and storage config. Resources are typically tuned for Hetzner cx23 instances.

## Conventions

- All tasks use FQCNs (`ansible.builtin.*`)
- Minimum Ansible version: 2.15
- Templates use `.j2` extension in `templates/`
- Helm job completion is awaited with `kubectl wait --for=condition=complete` with retries
- Sensitive tasks use `no_log: true`
