---
slug: "opencomputers/tunnels"
title: "Tunnels and exposing services"
description: "Expose a port on a host at a MIOSA-managed address, choose who can reach it, and understand what a tunnel does and does not do."
section: "Computers"
order: 3020
---

A tunnel publishes a port on one of your hosts at a MIOSA-managed address.
The host keeps dialing out to MIOSA; MIOSA routes inbound requests back down that session to your service.

An outbound management session and a publicly reachable application tunnel are different things.
Connecting a host does not expose anything. A tunnel is an explicit, separate exposure with its own access policy.

## Create a tunnel

```bash
miosa host tunnel new <host> --port 3000 --slug my-app-dev --access organization
miosa host tunnel list <host>
miosa host tunnel get <host> <tunnel>
miosa host tunnel rm <host> <tunnel> --yes
```

| Flag | Meaning |
|---|---|
| `--port int` | Port on the host to expose |
| `--slug string` | Address name (default: generated) |
| `--access string` | Who may open it: `organization` (default) or `public` |

The same call over REST:

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/tunnels" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"target_port":3000,"auth_mode":"tenant_only","slug":"my-app-dev"}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `target_port` | integer | Yes | Local port to expose, `1` to `65535` |
| `auth_mode` | string | No | `public` \| `tenant_only` \| `password` (default: `tenant_only`) |
| `slug` | string | No | Address name, 4 to 63 lowercase alphanumeric or hyphen characters |
| `target_host` | string | No | Host to dial from the agent (default: `127.0.0.1`) |
| `password` | string | No | Required in practice when `auth_mode` is `password`; hashed, never returned |

```json
{
  "id": "tun_abc",
  "host_id": "host_abc123",
  "slug": "my-app-dev",
  "target_port": 3000,
  "auth_mode": "tenant_only",
  "public_url": "https://<public-host>/t/my-app-dev",
  "enabled": true,
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

Use the returned `public_url`. Do not construct a hostname yourself: the routing domain is set per environment, so read it from the response instead of hardcoding one.

## Change or disable a tunnel

```bash
miosa host tunnel new <host> --port 8080 --slug my-app-dev   # re-create with a new port or slug
```

Over REST, `PATCH /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}` changes a tunnel in place:

| Field | Type | Description |
|---|---|---|
| `state` | string | `active`, `paused`, or `revoked` |
| `auth_mode` | string | Change the access mode |
| `target_host` | string | Change the host the agent dials |
| `slug` | string | Change the address name |
| `rotate_slug` | boolean | Generate a new slug |
| `password` | string | Set a new password (re-hashed) |

`DELETE /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}` returns `204 No Content` and revokes the tunnel.

The response adds live counters you can use to check that traffic is flowing: `state`, `custom_domain`, `request_count`, `lifetime_bytes_ingress`, and `lifetime_bytes_egress`.

## Access modes

| Mode | Who can open it |
|---|---|
| `organization` | Members of your MIOSA organization only (CLI default) |
| `tenant_only` | The tenant that owns the host |
| `public` | Anyone with the URL (API default) |

> **Caution: Do not open a private service to troubleshoot it**
>
> A public URL cannot make an unavailable local service healthy, and it does not fix an authentication problem.
> Confirm the service answers on the host's own port before you change a tunnel's access mode.

## Expose a device port

`miosa device expose` publishes a port for any device kind, including a host:

```bash
miosa device expose <device> 3000
```

Use the typed `miosa host tunnel` command when you want the fuller surface - list, get, access mode, and slug control.

## What a tunnel does not do

- It does not change who owns the application. Your service still runs on your machine, under your process supervision.
- It does not remove the need for the service to be listening. The tunnel reaches a port; it does not start anything.
- It does not make the traffic private in transit beyond the transport MIOSA terminates. Set the access mode and the application's own authentication to match the data you serve.

- [Tunnels in the API reference](/docs/api-reference/open-computers#tunnels): Request fields, response shape, and access configuration.

- [Security model](/docs/opencomputers/security): What the outbound session grants and what it does not.

- [OpenComputers troubleshooting](/docs/opencomputers/troubleshooting#tunnel-does-not-reach-the-application): What to check when a tunnel does not reach the application.
