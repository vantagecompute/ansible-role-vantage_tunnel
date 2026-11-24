---
title: Security
description: What the Vantage auth proxy enforces, which secret the vantage_tunnel role handles, and what its download check does and does not prove
---

## What the proxy enforces

Every request that arrives through the tunnel passes the proxy before it reaches the service on
the node. The proxy:

| Check | What it stops |
|---|---|
| Validates the token's signature against the Keycloak realm | A forged token |
| Checks the issuer | A token from another sign-in service |
| Checks the expiry | An old token |
| Checks that the audience is this cluster's client ID | A valid token issued for another cluster |
| Removes inbound `x-slurm-*` headers | A caller naming another Slurm user for itself |

This configuration is fixed. The role has no variable that loosens it, and any other configuration
is unsupported.

## Network exposure

- The proxy listens on `127.0.0.1:8888` only. Nothing outside the node can connect to it directly.
- The tunnel is an outbound connection from the node to Vantage. The role opens no inbound port
  and needs no inbound firewall rule.
- The proxy forwards to `127.0.0.1:6820` on the same node, and nowhere else.

## The secret the role handles

| Secret | Input | Where it ends up on the node |
|---|---|---|
| The cluster's client secret | `vantage_tunnel_client_secret` | `/etc/vantage/vantage-tunnel-client/env`, `root:root`, `0600` |

The proxy's settings file holds no secret: it checks tokens against the realm's public keys.

systemd reads each settings file as root before it starts the service as its own user, so the
service user cannot read the file.

## Keeping the secret out of Ansible's output

Every task that handles the secret or the token sets `no_log: true`: the token request, the
downloads (which send the token) and the tunnel client's settings template. The cost is that a
failure in one of those tasks shows no detail.
[Troubleshooting](./troubleshooting.md#a-task-fails-with-its-output-hidden) explains how to find
the cause.

Keep `vantage_tunnel_client_secret` in `ansible-vault`, not in a plain variables file.

## What the download check proves

The role checks each binary against a `.sha256` file from the same CDN, fetched with the same
token. That catches a damaged or incomplete download. It does not protect against a compromised
CDN, which could serve a matching binary and checksum.

## Reporting a vulnerability

Email security@vantagecompute.ai. Do not open a public issue.
