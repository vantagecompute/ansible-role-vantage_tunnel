---
title: Usage
description: Apply the vantage_tunnel role, pin versions, upgrade, and check the result
---

## A complete play

The role must run as root, so set `become: true`.

```yaml
- hosts: cluster_head
  become: true
  vars:
    vantage_tunnel_client_id: my-cluster
    vantage_tunnel_client_secret: "{{ vault_vantage_client_secret }}"
    vantage_tunnel_cluster_name: my-cluster
    vantage_tunnel_oidc_base_url: https://auth.vantagecompute.ai
    vantage_tunnel_vantage_url: https://app.vantagecompute.ai
    vantage_tunnel_auth_proxy_version: "X.Y.Z"
    vantage_tunnel_client_version: "X.Y.Z"
  roles:
    - vantagecompute.vantage_tunnel
```

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass
```

Every variable is described in [Variables](./variables.md).

## Keep the secret in ansible-vault

One input is a secret: `vantage_tunnel_client_secret`. Put it in a vault file and refer to it from
the play, as above:

```bash
ansible-vault create group_vars/cluster_head/vault.yml
```

```yaml
vault_vantage_client_secret: "..."
```

## Pin the versions

Each binary's version defaults to `latest`, which can change between two runs. In production, set
both version variables to exact releases.

Whatever the version, the role resolves it to a checksum on every run and reinstalls a binary only
when its checksum differs from the one in `/usr/bin`.

## Upgrade

Change a version variable and run the play again. The role downloads the new build, runs its
`systemd-setup`, and restarts the service.

## Run it again

A second run with the same inputs changes nothing. It is safe to apply the role on a schedule.

## With the vantage_agents role

The two roles are independent. To apply both, list the tunnel first, which is the order `v8x`
uses:

```yaml
  roles:
    - vantagecompute.vantage_tunnel
    - vantagecompute.vantage_agents
```

## Check the result

On the node:

```bash
systemctl status vantage-auth-proxy vantage-tunnel-client
ss -ltn 'sport = :8888'
cat /etc/vantage/artifacts.lock
```

The proxy listens on `127.0.0.1:8888` only. `/etc/vantage/artifacts.lock` has one line per
installed binary: its name, the version you asked for, and its sha256.
