---
title: Installation
description: What a node needs before the vantage_tunnel role runs, and how to install the role from Ansible Galaxy
---

## What you need

On the machine that runs Ansible (the control machine):

- Ansible 2.16 or newer. The role uses only `ansible.builtin` modules, so `ansible-core` is enough.

On the node the role is applied to:

- Ubuntu 26.04 on x86_64. Other platforms are not supported yet, and the role stops on them.
- Outbound HTTPS to the artifact CDN, the Keycloak realm and Vantage.
- A service listening on `127.0.0.1:6820` for the proxy to forward to. On a Slurm node that is
  slurmrestd. The role does not check for it, so the node does not have to run Slurm for the role
  to apply.

No inbound firewall rule is needed. The tunnel is an outbound connection, and the proxy listens on
the loopback address only.

From Vantage:

- A cluster, its name, and its Keycloak client credentials.

## Install the role

From Ansible Galaxy:

```bash
ansible-galaxy role install vantagecompute.vantage_tunnel
```

To install an exact version, which is what a production deployment should do:

```bash
ansible-galaxy role install vantagecompute.vantage_tunnel,0.1.0
```

Or name it in a `requirements.yml` beside your playbook:

```yaml
roles:
  - name: vantagecompute.vantage_tunnel
    version: "0.1.0"
```

```bash
ansible-galaxy role install -r requirements.yml
```

A role version is a git tag of the
[role's repository](https://github.com/vantagecompute/ansible-role-vantage_tunnel/tags).

## Check the installation

```bash
ansible-galaxy role list | grep vantage_tunnel
```

Then write a play as shown in [Usage](./usage.md).
