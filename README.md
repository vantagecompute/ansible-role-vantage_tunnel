# vantage_tunnel

An Ansible role that installs the Vantage tunnel client and the Vantage auth proxy on a Slurm node, so Vantage can reach the node's slurmrestd through a proxy that checks the Vantage token on every request.

- **vantage-auth-proxy** validates the Keycloak bearer token (signature, issuer, expiry and this cluster's audience), strips inbound `x-slurm-*` headers, and forwards to slurmrestd on the same node.
- **vantage-tunnel-client** connects the node to Vantage and delivers platform traffic to the proxy.

The Vantage Compute `v8x` tool applies this role on every Slurm single-node deployment, with no opt-out. You can apply the same role to your own nodes. It is independent of the `vantage_agents` role and can be applied on its own.

> Working name: the repository name for this role is provisional.

## Requirements

- Ubuntu 26.04 on x86_64. Other platforms are not supported yet.
- Ansible 2.16 or newer on the control machine, and `become: true` on the play.
- A Vantage cluster and its Keycloak client credentials.
- slurmrestd on the node at `127.0.0.1:6820`, which is where the proxy forwards.
- Outbound HTTPS from the node to the artifact CDN, the Keycloak realm and Vantage.

## Variables

Required (the role fails when one is empty):

| Variable | Meaning |
| --- | --- |
| `vantage_tunnel_client_id` | The cluster's Keycloak client ID. The proxy uses it as the token audience. |
| `vantage_tunnel_client_secret` | The client secret. Keep it in `ansible-vault`. |
| `vantage_tunnel_cluster_name` | The cluster's name in Vantage. |
| `vantage_tunnel_oidc_base_url` | Keycloak base URL without `/realms/vantage`, for example `https://auth.vantagecompute.ai`. |
| `vantage_tunnel_vantage_url` | The Vantage platform URL the tunnel client connects to. |

Optional:

| Variable | Default | Meaning |
| --- | --- | --- |
| `vantage_tunnel_artifacts_url` | `https://vantage-artifacts.vantagecompute.ai` | Where the binaries are downloaded from. |
| `vantage_tunnel_auth_proxy_version` | `latest` | Auth proxy version. Pin it in production. |
| `vantage_tunnel_client_version` | `latest` | Tunnel client version. Pin it in production. |

The auth proxy has no other settings. Its configuration is fixed and deployed as is: it listens on `127.0.0.1:8888` only, forwards to slurmrestd on the same node, and checks the Vantage token first. Any other configuration is unsupported.

## Example

```yaml
- hosts: slurm_head
  become: true
  vars:
    vantage_tunnel_client_id: my-cluster
    vantage_tunnel_client_secret: "{{ vault_vantage_client_secret }}"
    vantage_tunnel_cluster_name: my-cluster
    vantage_tunnel_oidc_base_url: https://auth.vantagecompute.ai
    vantage_tunnel_vantage_url: https://vantage.example.com
    vantage_tunnel_auth_proxy_version: "X.Y.Z"
    vantage_tunnel_client_version: "X.Y.Z"
  roles:
    - vantage_tunnel
```

## How it works

For each binary the role gets a Keycloak token with the cluster's credentials, downloads the binary and its `.sha256` file from the artifact CDN, and checks the checksum. When `/usr/bin` holds a different build it runs the binary's own `systemd-setup` command, which creates the service user, directories and unit. The role writes the settings files (root-only, `0600`), then starts the proxy first and the tunnel client second. A second run changes nothing. Upgrading means changing the version variable and running again.

The checksum comes from the same CDN as the binary, so it catches a damaged download, not a compromised CDN. Tasks that handle secrets use `no_log`.

## Testing

`just lint` runs ansible-lint. `just test <inventory>` runs the role twice against a throwaway Ubuntu 26.04 host that you point it at, with a stub token endpoint and artifact server, and checks that the second run changes nothing. It proves the Ansible logic only. It does not prove the real binaries or the real `systemd-setup`; for that, apply the role to a host with a real cluster.

## License

GPL-3.0-or-later. See `LICENSE`.
