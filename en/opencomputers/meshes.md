---
slug: "opencomputers/meshes"
title: "Meshes"
description: "Link several of your hosts on a private encrypted WireGuard network so they reach each other without going over the public internet."
section: "Computers"
order: 3017
---

A mesh is a named private network across several of your hosts.
MIOSA assigns each member an address in the mesh's `/24` and distributes WireGuard keys, so members reach each other directly without exposing anything on the public internet.

Use a mesh when services on one host need to talk to services on another: a database on one box, an application on a second, a GPU worker on a third.

## Create a mesh

**miosa CLI**

```bash
miosa host mesh list
miosa host mesh new lab --host <host> --host <host>
miosa host mesh get lab
```

Aliases: `miosa host meshes`.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/meshes" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"lab","host_ids":["<host-1>","<host-2>"]}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | 1 to 80 lowercase alphanumeric or hyphen characters |
| `host_ids` | string[] | No | Hosts to add at creation; default `[]` |

A mesh carries a `cidr` in the form `10.x.x.0/24`, a `state`, and a `listen_port_base` that defaults to `51820`.
Mesh states are `provisioning`, `active`, `failed`, and `torn_down`.

## Manage membership

```bash
miosa host mesh add-host lab <host>
miosa host mesh remove-host lab <host> --yes
miosa host mesh rm lab --yes
```

| Action | REST |
|---|---|
| Add a host | `POST /opencomputers/meshes/{id}/hosts/{host_id}` |
| Remove a host | `DELETE /opencomputers/meshes/{id}/hosts/{host_id}` |
| Delete the mesh | `DELETE /opencomputers/meshes/{id}` |

Adding a host returns the new peer.
Removing a host leaves the host itself in place; deleting the mesh tears down the WireGuard configuration before the record is removed.

> **Note: A mesh caps at 253 hosts**
>
> The mesh's `/24` address space allows 253 members. Adding one past that returns `422` with `mesh /24 is full (max 253 hosts)`.
> If you outgrow a single mesh, contact support rather than nesting meshes yourself.

## Inspect peers

`miosa host mesh get <mesh>` and `GET /opencomputers/meshes/{id}` return the mesh with its current peer list.

| Peer field | Meaning |
|---|---|
| `id` | Peer record ID |
| `host_id` | The host this peer represents |
| `assigned_ip` | Address in the mesh's `10.x.x.x` range |
| `pubkey` | The peer's WireGuard public key |
| `endpoint` | Last seen endpoint |
| `latest_handshake` | When the tunnel last completed a handshake |
| `rx_bytes`, `tx_bytes` | Traffic counters |
| `state` | `pending_key`, `configured`, `active`, or `failed` |

Peer states are the fastest way to answer "is this host actually reachable over the mesh": an `active` peer with a recent `latest_handshake` is, and a `pending_key` peer never joined.

## Events

**miosa CLI**

```bash
miosa host mesh events lab
```

Streams host joins, key rotation, and state changes until you press Ctrl-C.

**REST API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/meshes/$MESH_ID/events" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Named SSE events: `mesh_peer_active`, `mesh_peer_down`, and `mesh_status`.

## Security notes

- Mesh traffic is encrypted and stays inside the mesh. It does not become reachable from the internet because a host is a member.
- MIOSA distributes the WireGuard keys. An operator with control-plane access can add a host to a mesh, so treat mesh membership as a privileged change and review it in the [audit log](/docs/opencomputers/security#audit-log).
- A mesh does not grant a host access to anything else on another host beyond the ports that host's firewall and services allow.
