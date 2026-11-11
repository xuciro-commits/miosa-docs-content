---
slug: "troubleshooting"
title: "Troubleshooting"
description: "The front door for things that are not working: sandbox, exec, preview, secret, network, and deploy failures, and where to look for each."
section: "Help"
order: 16011
---

Start with the exact resource id and the operation that failed. Most requests also return an `x-request-id` header  -  include it in any support request.

```bash
miosa doctor   # checks CLI version, config, endpoint, key, and clock skew
```

A skewed clock breaks signed webhooks and expiring tokens, so `miosa doctor` is a fast first check.

## Sandbox will not start

- **Still `provisioning`.** The runtime is not ready yet. Poll `GET /api/v1/sandboxes/{id}` and wait for `state` to become `running` and `ready` to be `true`.
- **`error`.** A boot or runtime failure. Check `sbx.data["metadata"]` for `last_error`, then destroy and create a new sandbox.
- **`exec` is refused.** A sandbox that is still starting rejects commands. The CLI's `exec` retries with backoff for up to 90 seconds; SDK callers should do the same or wait for `ready`.

## Commands fail or hang

- **A command returns non-zero.** The sandbox exits with the remote command's status. Read `stderr`; nothing is silently swallowed.
- **`exec` times out.** The sandbox `exec` endpoint accepts a `timeout` of 1 to 86,400 seconds (default 3,600) regardless of the sandbox's own timeout. For long work, start a [background process](/docs/develop/sandboxes/long-running-tasks) and poll its logs instead.
- **A file upload fails.** The maximum file upload is 500 MB. Split larger artifacts or write them from inside the sandbox.
- **HTTP `429`.** You hit a rate limit. Wait for the seconds in the `Retry-After` header. See [Rate limits](/docs/rate-limits).

## Previews do not load

- **`404` or connection refused.** Nothing is listening on the port you exposed. Confirm the server is bound to `0.0.0.0`, not just `127.0.0.1`, and that the port number matches.
- **A private preview needs a token.** Private previews require a valid token. See [Previews](/docs/develop/previews) for visibility and sharing.

## A secret is not working

Secrets are covered in depth in [Security troubleshooting](/docs/security/troubleshooting). That page catalogs the most common issues  -  a placeholder that is not swapped, an unbound or mismatched scope, and rotation that seems not to take effect  -  with root causes and fixes, alongside Connect Accounts, the Network Allowlist, the Audit Log, and white-label credentials.

## The network is blocked

A sandbox starts in audit-only mode, so nothing is blocked until you enforce an allowlist. If a call is blocked, check the pending queue (`sandbox.network.pending()`) or filter the audit log by `action=reject`; each row names the rule in `rejected_by`. Common gaps (`pip`, `npm`, CDNs) are listed in [Internet access and egress](/docs/develop/sandboxes/internet-access).

## A deployment failed

- **Build fails.** Read the build logs for the deployment version; see the [Docker Host deploy guide](/docs/docker-host/docker-deploy) for container build paths.
- **A custom domain does not resolve.** Apex records need ALIAS / ANAME support in your DNS provider.
- **You need the previous version back.** Roll back to a prior release; see [Releases](/docs/docker-host/releases) and [Rollback](/docs/docker-host/rollback).

For host, tunnel, and bring-your-own-cloud problems, see [OpenComputers troubleshooting](/docs/opencomputers/troubleshooting).

## SSH and connections

There is no public SSH endpoint. `miosa ssh`, the SDK SSH tunnel, and preview URLs are the only ways in. See [Connecting to a running sandbox](/docs/develop/sandboxes/connect).

## Still stuck

Open a support ticket from the MIOSA dashboard and include the resource id, the operation, a timestamp with timezone, and the `x-request-id`. Remove API keys, tokens, and private URLs from anything you paste. See [Support](/docs/support) for other channels.

- [Security troubleshooting](/docs/security/troubleshooting): Secrets, OAuth, network allowlist, audit log, and white-label issues.
- [OpenComputers troubleshooting](/docs/opencomputers/troubleshooting): OpenComputer connectivity, failed commands, desktops, and tunnels.
- [Known limitations](/docs/known-limitations): What does not work yet, and what is intentionally out of scope.
