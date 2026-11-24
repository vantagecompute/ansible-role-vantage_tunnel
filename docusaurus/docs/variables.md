---
title: Variables
description: Every input of the vantage_tunnel role, with its default and meaning
---

All inputs start with `vantage_tunnel_`.

## Required

The role stops before changing anything when one of these is empty.

| Variable | Meaning |
|---|---|
| `vantage_tunnel_client_id` | The cluster's Keycloak client ID. The tunnel client signs in with it, and the proxy accepts only tokens with this audience |
| `vantage_tunnel_client_secret` | The client secret. Keep it in `ansible-vault` |
| `vantage_tunnel_cluster_name` | The cluster's name in Vantage |
| `vantage_tunnel_oidc_base_url` | The Keycloak base URL without `/realms/vantage`, for example `https://auth.vantagecompute.ai` |
| `vantage_tunnel_vantage_url` | The URL of the Vantage platform the tunnel client connects to, for example `https://app.vantagecompute.ai` |

## Optional

| Variable | Default | Meaning |
|---|---|---|
| `vantage_tunnel_artifacts_url` | `https://vantage-artifacts.vantagecompute.ai` | Where the binaries are downloaded from |
| `vantage_tunnel_auth_proxy_version` | `latest` | The `vantage-auth-proxy` version. Pin it in production |
| `vantage_tunnel_client_version` | `latest` | The `vantage-tunnel-client` version. Pin it in production |

## Fixed values

The auth proxy has no other settings. Its configuration is fixed on purpose and deployed as is.
Any other configuration is unsupported. These are constants in the role's `vars/main.yml`, listed
so you know where to look on a node.

| What | Value |
|---|---|
| Proxy listen address | `127.0.0.1:8888` |
| Where the proxy forwards | `127.0.0.1:6820` on the same node, the proxy's built-in default |
| Where the tunnel client delivers traffic | `localhost:8888`, the proxy |
| Keycloak realm | `/realms/vantage`, added to `vantage_tunnel_oidc_base_url` |
| Binary architecture | `linux-amd64` |
| Download cache | `/var/cache/vantage-tunnel` |
| Install record | `/etc/vantage/artifacts.lock` |
| Proxy settings | `/etc/vantage/vantage-auth-proxy/env` |
| Tunnel client settings | `/etc/vantage/vantage-tunnel-client/env` |
