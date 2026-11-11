---
slug: "opencomputers/troubleshooting"
title: "OpenComputers troubleshooting"
description: "Diagnose offline hosts, rejected credentials, failed commands, container, mesh, runner, backup, and tunnel problems."
section: "Computers"
order: 3024
---

Start with the exact host ID, organization, and operation that failed.
Check both the control-plane record and the machine itself.
A working login, a healthy API endpoint, or a saved host key is not proof that the host can execute your request.

The fastest first check is always the same:

```bash
miosa host get <host>
miosa host jobs <host>
```

`state` and `connected` describe now; the job list describes what actually happened.

## Host stays offline

1. Confirm the machine is awake and its network connection works.
2. Check the agent process on the host; for the systemd setup in [Connect your first host](/docs/opencomputers/connect), use the commands below.
3. Check outbound HTTPS and WebSocket access to `api.miosa.ai` on port 443; an HTTP health check alone does not test WebSocket proxying.
4. Confirm the service runs as the user whose `~/.osa/open_computers.toml` contains the intended host configuration.
5. Check that the host has not been revoked, then confirm a fresh heartbeat in MIOSA.

```bash
systemctl status osa-opencomputers.service --no-pager
journalctl -u osa-opencomputers.service -n 50 --no-pager
```

Use `sudo` for logs if your account cannot read the system journal.
`osa opencomputers status` can help inspect configuration, but its session output describes the runtime queried by that command.
Confirm the actual service and control-plane heartbeat before concluding that a separate background process is offline.

## Key saved but no connection

`osa opencomputers login` saves a host key, and `connect` is an alias for that command.
The agent still has to start in host mode; saving the credential alone does not start the session.
Follow the [startup steps](/docs/opencomputers/connect#4-start-the-host-session).
Do not substitute a platform API key for the host key.

## Identity rejected after reinstall

The machine's identity file must match the identity pinned when the host first connected.
Preserve `~/.osa/open_computers.ed25519`, along with the host configuration, during upgrades.
If the identity is lost, revoke the host and register a new one rather than copying another machine's identity or disabling verification.

## Registration fails

| Error | Cause |
|---|---|
| `402 plan_limit_reached` | Your plan's host limit is reached; the response carries `limit` and `current` |
| `422 name is required` | `name` is missing, empty, or over 64 characters |
| `501 open_computers_disabled` | OpenComputers is not enabled for this environment |

Check the plan's host allowance in [Billing and limits](/docs/opencomputers/billing) before assuming a bug.

## Command fails or appears stuck

| Symptom | What to check |
|---|---|
| `cmd is required` | Send the `cmd` field to the exec endpoint; `command` is the field name on the newer `POST .../jobs` route, not on `/exec` |
| `409 host_not_connected` | The host is offline. Bring the agent up before dispatching |
| `422 invalid_command` | The command is empty or over 8192 characters |
| `422 invalid_args` | More than 256 arguments, or a non-string argument |
| `422 invalid_timeout` | The timeout is outside 1 to 3600 seconds |
| `502 dispatch_failed` | The session dropped while dispatching; check the agent, then retry once |

- **Permission denied or command unavailable:** run the same command locally as the agent user and check its permissions, working directory, PATH, and supported operations.
- **HTTP request remains open:** exec streams by default; inspect SSE events until `exec_result` or `exec_error`, or use `stream: false` for a short command.
- **Timeout or connection loss:** save the returned job ID, inspect the job and the local process, and establish the outcome before retrying.

```bash
curl --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/exec/$JOB_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

A timeout or a disconnected client is not proof that the process stopped.
Avoid retrying a deployment, payment, file mutation, or other operation with side effects merely because the client disconnected.

## Containers

| Symptom | What to check |
|---|---|
| `503 host_not_connected` | The container runtime is managed by the agent on the host; an offline host cannot create or start containers |
| `402 plan_limit_reached` | The container and compose slot budget is full; see [Billing and limits](/docs/opencomputers/billing) |
| Stuck in `provisioning` | The host is pulling the image, or the image reference is not pullable by that host's runtime |
| `409` on start | A container only starts from `stopped` or `failed`; check `state` first |
| `last_error` set | The runtime rejected the container; read `last_error` before retrying |

Pulling a large image over a slow link can take minutes. Watch the host's disk and network with `miosa host metrics <host>` rather than recreating the container.

## Meshes

| Symptom | What to check |
|---|---|
| Peer stuck in `pending_key` | The host has not completed its key exchange; confirm the host is online and the agent is current |
| Peer `active` but traffic fails | The services on the other host may not be listening on the address you dialed, or a host firewall blocks the mesh interface |
| `422 mesh /24 is full` | The mesh has reached 253 members |
| `422 one or more hosts not found` | A host ID belongs to another organization or does not exist |

Use `miosa host mesh get <mesh>` and read `latest_handshake`, `rx_bytes`, and `tx_bytes` to tell "never connected" from "connected but no traffic".

## GitHub Actions runners

| Symptom | What to check |
|---|---|
| `422 GitHub API rejected the PAT` | The token lacks repository or organization runner permissions, or has expired |
| `422 GitHub repo/org not found` | The `repo_url` is wrong, or the token cannot see it. GitHub Enterprise Server URLs are not supported |
| `429 GitHub API rate limit exceeded` | GitHub throttled the registration call; wait and retry |
| Runner `offline` | The host is offline, or the runner process died. Use `miosa host gha-runner refresh <host> <runner>` |
| Runner `failed` | Read the runner events; a failed install is usually a missing dependency on the host |

A workflow that never schedules is usually a label mismatch: check that the workflow's `runs-on` labels match the runner's `labels`.

## Backups

| Symptom | What to check |
|---|---|
| `503` on trigger | The source host must be connected to take a snapshot |
| Snapshot `failed` | Read `error` on the snapshot; a path that disappeared or a permission change are the common causes |
| `422 snapshot has no chunks` | The snapshot did not complete; only `complete` snapshots restore |
| `503 target host not connected` | Restores are pushed by the target host's agent, so the target must be online |
| `403 cross-organization restore not allowed` | The target host is not in the same organization as the snapshot |

## Tunnel does not reach the application

1. On the host, confirm the application is running and responds on the intended local port.
2. Check the tunnel's configured `target_port` and the host's connection.
3. Open the returned `public_url`; do not construct a hostname yourself.
4. Confirm the viewer meets the tunnel's access policy.

| Proxy error | Meaning |
|---|---|
| `connection_refused` | Nothing is listening on `target_port` on the host |
| `target_port_not_allowed` | The port is outside the range the host agent permits |
| `tunnels_disabled_on_host` | Tunnels are disabled on that host |
| `too_many_concurrent_tunnels` | The host is at its concurrent tunnel limit |
| `502 host_offline` | The host dropped while the request was in flight |
| `503 tunnel paused` | The tunnel is in state `paused` |
| `401` with `WWW-Authenticate: Basic` | The tunnel is in `password` mode and the request did not carry credentials |

A public URL cannot make an unavailable local service healthy.
Do not change a private tunnel to public solely to troubleshoot authentication.

## Commands work but desktop does not

A headless host can execute commands without providing a desktop.
Verify the selected host advertises desktop support and has the required session and streaming backend.
Check the agent's current platform support; registering macOS or Windows does not guarantee a working desktop stream.
For a MIOSA-managed Computer, follow [desktop readiness](/docs/computers/fast-start) instead of installing a desktop on an arbitrary host.

## Customer-cloud resources exist but placement is unavailable

Cloud infrastructure existing is only one onboarding step.
Verify provider acceptance, a successful live canary, and host readiness before expecting placement.
See the [BYOC overview](/docs/infrastructure/byoc/overview) for the applicable gates.

## Information to include when asking for help

Include the host ID, the operation or job ID, a timestamp with timezone, the agent version, the OS and architecture, and the relevant error text.
State whether the command works locally as the agent user and whether the heartbeat is fresh.
Remove API keys, host keys, identity files, private URLs containing tokens, and sensitive command output from shared logs.

- [Connect your first host](/docs/opencomputers/connect): Repeat the connection and completed-command checks.

- [Billing and limits](/docs/opencomputers/billing): Check plan limits before treating a 402 as a fault.
