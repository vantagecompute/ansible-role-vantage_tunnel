# vantage_tunnel

An Ansible role that installs the Vantage tunnel client and the Vantage auth proxy on a node, so Vantage can reach a service on that node through a proxy that checks the Vantage token on every request. The usual case is a Slurm node, where that service is slurmrestd, but the node does not have to run Slurm.

- **vantage-auth-proxy** validates the Keycloak bearer token (signature, issuer, expiry and this cluster's audience), strips inbound `x-slurm-*` headers, and forwards to `127.0.0.1:6820` on the same node, which is where slurmrestd listens.
- **vantage-tunnel-client** connects the node to Vantage and delivers platform traffic to the proxy.

The Vantage Compute `v8x` tool applies this role, together with `vantage_agents`, when it deploys the `slurm-single-qemu` application. You can apply the same role to your own nodes. It is independent of the `vantage_agents` role and can be applied on its own.

## Install

```bash
ansible-galaxy role install vantagecompute.vantage_tunnel
```

Pin an exact version in production: `ansible-galaxy role install vantagecompute.vantage_tunnel,X.Y.Z`. A version is a git tag of this repository.

Full documentation is at https://docs.vantagecompute.ai/developer/ansible-vantage-tunnel/ and its source is in `docusaurus/`.

## Requirements

- Ubuntu 26.04 on x86_64. Other platforms are not supported yet.
- Ansible 2.16 or newer on the control machine, and `become: true` on the play.
- A Vantage cluster and its Keycloak client credentials.
- A service on the node at `127.0.0.1:6820`, which is where the proxy forwards. On a Slurm node that is slurmrestd. The role does not check for it, so it applies to a node without Slurm too; requests through the tunnel then succeed only when something listens there.
- Outbound HTTPS from the node to the artifact CDN, the Keycloak realm and Vantage.

## Terms

- **slurmrestd** is Slurm's REST API service. The proxy forwards requests to it.
- **Keycloak and OIDC**: Keycloak is the sign-in service Vantage uses, and OIDC is the protocol it speaks. The cluster signs in with a client ID and secret and receives a short-lived token.
- **Audience**: the cluster name written inside a token. The proxy rejects a token whose audience is another cluster.
- **Bearer token**: the login token a client sends in an `Authorization: Bearer ...` header. The proxy checks it before anything reaches Slurm.
- **x-slurm-* headers**: request headers that tell Slurm which user is calling. The proxy removes any it receives, so a caller cannot pretend to be someone else.
- **Loopback**: the address `127.0.0.1`, reachable only from the same machine. The proxy listens only there.
- **systemd-setup**: a command each Vantage binary has that installs itself as a systemd service (its own user, directories and unit file).
- **Tunnel**: an outbound connection from the node to Vantage, so Vantage can reach the node without an inbound firewall rule.

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

The auth proxy has no other settings. Its configuration is fixed and deployed as is: it listens on `127.0.0.1:8888` only, forwards to `127.0.0.1:6820` on the same node, and checks the Vantage token first. Any other configuration is unsupported.

## Example

```yaml
- hosts: slurm_head
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

## How it works

For each binary the role gets a Keycloak token with the cluster's credentials, downloads the binary and its `.sha256` file from the artifact CDN, and checks the checksum. When `/usr/bin` holds a different build it runs the binary's own `systemd-setup` command, which creates the service user, directories and unit. The role writes the settings files (root-only, `0600`), then starts the proxy first and the tunnel client second. A second run changes nothing. Upgrading means changing the version variable and running again.

The role records what it installed, one line per binary (name, requested version, sha256), in `/etc/vantage/artifacts.lock`, the same file and format `v8x` writes.

The checksum comes from the same CDN as the binary, so it catches a damaged download, not a compromised CDN. Tasks that handle secrets use `no_log`.

## Testing and releasing

`just release X.Y.Z` tags a release from an up-to-date `main` and pushes the tag; CI then publishes that version to Ansible Galaxy and deploys the documentation.

`just lint` runs ansible-lint. `just test <inventory>` runs the role twice against a throwaway Ubuntu 26.04 host that you point it at, with a stub token endpoint and artifact server, and checks that the second run changes nothing. It proves the Ansible logic only. It does not prove the real binaries or the real `systemd-setup`; for that, apply the role to a host with a real cluster.

## License

GPL-3.0-or-later. See `LICENSE`.
