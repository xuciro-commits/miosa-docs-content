---
slug: "opencomputers/devices"
title: "Devices"
description: "The device model - one id-addressed surface for running commands, moving files, exposing ports, and driving lifecycle across runtime kinds."
section: "Computers"
order: 3021
---

A device is any runtime MIOSA can run a command on.
`<device>` is an id, not a name, and the device surface does not care which kind of runtime answers.

Use it when a script has to work across runtime kinds, or when you have an id and do not know what it is.

> **Note: Devices and hosts are different surfaces**
>
> The `/api/v1/devices` endpoints resolve sandbox workers and managed Computers - the runtimes MIOSA provisions for you.
> A machine you connect with OpenComputers is a **host** and is addressed by its host ID through the [OpenComputers endpoints](/docs/api-reference/open-computers) instead.
> The [`miosa device`](/docs/cli/devices) command is the CLI's unified view; for a host, use the typed [`miosa host`](/docs/cli/platform#your-own-hosts) family.

## List and inspect

**miosa CLI**

```bash
miosa device list
miosa device list --status running --name web --limit 20
miosa device get <device>
miosa device capabilities <device>
```

Aliases: `miosa devices`, `show` for `get`, `caps` for `capabilities`.

**REST API**

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices?status=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/capabilities" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

`GET /api/v1/devices` accepts `status`, `name`, and `limit` (default 50).

A device record carries `id`, `name`, `slug`, `template`, `kind`, `state`, `ip_address`, `node_id`, `running`, `terminal`, and its `capabilities`.

## Check capabilities before you act

Capabilities are the difference between a script that works everywhere and one that fails on half your devices.

| Capability | Sandbox worker | Computer |
|---|---|---|
| `exec` | yes | yes |
| `files` | yes | yes |
| `browser` | no | yes |
| `artifacts` | yes | no |
| `live_server` | yes | yes |
| `connectors` | yes | yes |
| `pause`, `resume` | yes | yes |
| `snapshots` | yes | no |

Read the capability rather than assuming it from the device's name.

## Run a command

**miosa CLI**

```bash
miosa device exec <device> -- uname -a
miosa device exec <device> -c "npm test" --timeout-ms 120000
```

`miosa device exec` exits with the remote command's status, like `miosa exec`.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/exec" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command":"uname -a","timeout_ms":30000}'
```

The body takes `command` (required) and `timeout_ms` or `timeout` in seconds (default 30).

Exec on a device resolves a **running** target.
A device that is not running returns `DEVICE_NOT_RUNNING` with the current `state` rather than starting it for you.

## Files

**miosa CLI**

```bash
miosa device files <device> --path /workspace
miosa device read <device> --path /workspace/out.json
miosa device write <device> --path /workspace/app.py --content "print(1)"
```

Aliases: `ls` for `files`.

**REST API**

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files/read?path=/workspace/out.json" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files/write" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path":"/workspace/app.py","content":"print(1)"}'
```

`files` lists a directory and defaults to `/workspace`. `files/read` takes a required `path` and returns the content base64-encoded with `encoding: "base64"`. `files/write` takes `path` plus either `content_base64` or `content`.

`miosa device read` returns the file contents base64-encoded, and `miosa device write` takes the contents from `--content` or a JSON body with `--file`.

## Expose a port

```bash
miosa device expose <device> 3000

curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/expose" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port":3000}'
```

On a host, the typed [tunnel commands](/docs/opencomputers/tunnels) give you the fuller surface: access mode, slug control, and list and delete.

## Browser URL

```bash
miosa device browser <device>
```

`GET /api/v1/devices/{id}/browser` returns the streamed desktop for a computer, with `desktop_url`, `ws_url`, `token`, `expires_at`, `computer_id`, and `slug`, and a one-hour token TTL.
A sandbox returns `BROWSER_UNSUPPORTED_FOR_SANDBOX`; use a [Computer](/docs/computer-use) when you need a desktop.

## Lifecycle

```bash
miosa device extend <device> --timeout 2h
miosa device pause <device>
miosa device resume <device>
miosa device stop <device>
miosa device rm <device> --yes
```

| Action | REST | Notes |
|---|---|---|
| Extend | `POST /devices/{id}/extend` with `timeout_sec` | Replaces the time to live |
| Pause | `POST /devices/{id}/pause` | Keeps the disk |
| Resume | `POST /devices/{id}/resume` | |
| Stop | `POST /devices/{id}/stop` | Keeps the disk |
| Destroy | `DELETE /devices/{id}` | Permanent; returns state `destroyed` |

Each lifecycle call returns the device's new `state`.

## Error codes

| Code | Status | Meaning |
|---|---|---|
| `MISSING_PARAM` | 400 | A required field is absent; the response names it |
| `INVALID_PARAM` | 400 | A field is present but not usable |
| `INVALID_BASE64` | 400 | File content was not valid base64 |
| `NOT_FOUND` | 404 | The device does not exist, or is not yours |
| `DEVICE_NOT_RUNNING` | 409 | The device is not running; the response carries its `state` |
| `UNSUPPORTED_FOR_COMPUTER` | 422 | The operation does not apply to a computer |
| `BROWSER_UNSUPPORTED_FOR_SANDBOX` | 422 | Browsing requires a computer |
| `DEVICE_OPERATION_FAILED` | 422 | The runtime rejected the operation |

- [CLI device reference](/docs/cli/devices): Every miosa device command and flag.

- [Computers](/docs/computers/overview): The managed desktop runtime behind computer devices.
