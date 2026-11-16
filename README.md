<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="HashiCorp Vault logo" width="128">
  </picture>
</p>

<h1 align="center">aac-hashivault</h1>

<p align="center">Self-hosted deployment definition for HashiCorp Vault, a single-node secrets manager with integrated Raft storage.</p>

<p align="center">
  <a href="https://github.com/hashicorp/vault"><img alt="Upstream" src="https://img.shields.io/badge/upstream-hashicorp%2Fvault-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-hashicorp%2Fvault%3A2.2.1-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/hashicorp/vault/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-BUSL--1.1-green"></a>
</p>

## About

HashiCorp Vault is a secrets management tool. It stores secrets, issues dynamic credentials,
encrypts data as a service and keeps an audit trail of every request. This deployment runs a
single Vault node with its web UI enabled and integrated (Raft) storage, so no external
storage backend is required. TLS is terminated by Traefik, while Vault itself listens on
plain HTTP inside the Docker networks only.

Key features (as provided by upstream):

- Secret storage with versioning and access policies
- Dynamic secrets and leases with configurable TTLs
- Encryption as a service (transit engine)
- Integrated Raft storage, with support for multi-node clusters
- File audit devices for request logging
- Built-in web UI

## Upstream

- Project: <https://github.com/hashicorp/vault>
- Documentation: <https://developer.hashicorp.com/vault/docs>
- Container image: `hashicorp/vault:2.2.1`
- License: BUSL-1.1 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external routing network must exist first:

```bash
docker network create services-secured

docker compose up -d
```

Vault starts sealed and uninitialised. Initialisation and unsealing are manual steps (see
below) and are deliberately not automated.

### First start (runbook)

1. Open the public URL of the service in a browser, or run
   `docker exec <container> vault operator init` on the host.
2. Store the unseal keys and the initial root token **outside** this repository, for example
   in a password manager or an offline vault. They are never committed.
3. Unseal the instance with the required number of unseal keys (`vault operator unseal`).
4. Enable the file audit device once, through the API (audit devices are not part of the
   configuration file), writing to `/vault/logs/audit.log`.
5. Revoke the root token after creating administrative identities and policies.

## Configuration

This deployment has no static bootstrap credentials. Nothing needs to be provided in `.env`
for the service to start. Environment variables (defaults from `service.yml`):

| Variable | Default | Description |
|---|---|---|
| `TZ` | `Europe/Berlin` | Time zone |
| `VAULT_UI` | `true` | Enable the web UI |
| `VAULT_API_ADDR` | `https://{{ service.hostname }}.{{ config.domain_name }}` | Public URL that clients are redirected to (the Traefik router FQDN) |
| `VAULT_RAFT_NODE_ID` | `aac-hashivault-1` | Stable Raft node identity, kept across container recreation |
| `VAULT_XFF_TRUSTED` | `172.16.0.0/12,192.168.0.0/16` | Networks trusted for `X-Forwarded-For`, so audit logs show the real client IP |
| `VAULT_DEFAULT_LEASE_TTL` | `768h` | Default lease and token TTL |
| `VAULT_MAX_LEASE_TTL` | `768h` | Maximum lease and token TTL |

Fixed container settings (set by the deployment, not configurable here):

| Variable | Value | Description |
|---|---|---|
| `VAULT_ADDR` | `http://127.0.0.1:8200` | Address for the local CLI inside the container |
| `SKIP_SETCAP` | `true` | Skips `setcap` in the entrypoint, which fails in unprivileged containers |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `8200` | TCP | Vault API and web UI (`api`), routed through Traefik over HTTPS |

Volumes:

| Mount | Description |
|---|---|
| `vault_file` -> `/vault/file` | Raft integrated storage. This is the data; back it up with `vault operator raft snapshot save` |
| `vault_logs` -> `/vault/logs` | File audit device target (`audit.log`) |
| `vault_config` -> `/vault/config` (read-only) | Rendered Vault configuration (`config.hcl`) |

Volume ownership: uid `100` and gid `1000`, the `vault` user of the official image.

Networks: the service joins `secured` and `stack_internal`.

Healthcheck: the container reports healthy when Vault answers on its health endpoint. A
sealed or uninitialised instance counts as healthy, so the router and the UI needed for
unsealing stay reachable.

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-hashivault>.

## Credits and license

HashiCorp Vault is developed by HashiCorp and licensed under BUSL-1.1. This repository only
contains the deployment definition for it and is not a fork of the application. The logo is
provided by the upstream project; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for
icon sources and licenses.
