---
title: Contact and support
description: Where to ask for help with the vantage_tunnel role, report a bug, or report a vulnerability
---

## Report a bug or ask for a feature

Open an issue at
[github.com/vantagecompute/ansible-role-vantage_tunnel/issues](https://github.com/vantagecompute/ansible-role-vantage_tunnel/issues).

Include:

1. What you expected and what happened.
2. The role version (`ansible-galaxy role list | grep vantage_tunnel`) and your Ansible version
   (`ansible --version`).
3. The name of the task that failed and its message.
4. The contents of `/etc/vantage/artifacts.lock` on the node.
5. The relevant log, for example `sudo journalctl -u vantage-tunnel-client -n 50 --no-pager`.

Remove secrets first. `/etc/vantage/vantage-tunnel-client/env` holds the client secret, so do not
paste it.

## Report a vulnerability

Email security@vantagecompute.ai. Do not open a public issue.

## Commercial support

For supported deployments of Vantage Compute, write to support@vantagecompute.ai or visit
[vantagecompute.ai](https://vantagecompute.ai).

## Contribute

See [Contributing](./contributing.md).
