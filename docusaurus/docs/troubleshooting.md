---
title: Troubleshooting
description: Failures of the vantage_tunnel role and of the services it installs, with the cause and the fix for each
---

## The run stops in the first checks

| Message | Cause and fix |
|---|---|
| `This role supports Ubuntu 26.04 on x86_64 only.` | The node runs something else. Other platforms are not supported yet. |
| `The variable vantage_tunnel_... is required and is empty.` | Set that variable. See [Variables](./variables.md#required). |

## A task fails with its output hidden

Tasks that handle the secret hide their output, so a failure reads
`the output has been hidden due to the fact that 'no_log: true' was specified`. Find the cause from
the task's name:

| Task | Likely cause |
|---|---|
| `Get a Keycloak token for the artifact CDN` | A wrong client ID or secret, a wrong `vantage_tunnel_oidc_base_url` (it must not include `/realms/vantage`), or no route to Keycloak |
| `Fetch the checksum of ...` | A version that does not exist, a wrong `vantage_tunnel_artifacts_url`, or no route to the CDN |
| `Download ...` | A dropped connection, or a download that does not match its checksum. Run again |

To see the detail, repeat the request by hand from the node. Do not turn `no_log` off on a shared
machine: the secret would be written to the terminal and to any log of the run.

## A service is not running

Read its log as root. A regular user sees no entries.

```bash
systemctl status vantage-auth-proxy vantage-tunnel-client
sudo journalctl -u vantage-auth-proxy -n 50 --no-pager
sudo journalctl -u vantage-tunnel-client -n 50 --no-pager
```

| Service | Check |
|---|---|
| `vantage-auth-proxy` | That nothing else holds port 8888: `ss -ltnp 'sport = :8888'`. That the node can reach the Keycloak realm, which the proxy needs to check tokens |
| `vantage-tunnel-client` | `vantage_tunnel_vantage_url`, the client ID and secret, and outbound HTTPS from the node to Vantage |

## Vantage cannot reach the service behind the proxy

Work from the node outward:

1. Is something listening where the proxy forwards?

   ```bash
   ss -ltn 'sport = :6820'
   ```

   On a Slurm node this is slurmrestd: `systemctl status slurmrestd`. On a node without Slurm,
   nothing listens there unless you run a service on that address, and every request fails at this
   step.

2. Is the proxy listening?

   ```bash
   ss -ltn 'sport = :8888'
   ```

3. Does the proxy reject requests? A token issued for another cluster is refused, so check that
   `vantage_tunnel_client_id` is this cluster's client ID.

4. Is the tunnel client connected? Read its log, as above.

## Which build is installed

```bash
cat /etc/vantage/artifacts.lock
sha256sum /usr/bin/vantage-auth-proxy
```

The checksum in the lock file matches the binary in `/usr/bin` after a successful run.
