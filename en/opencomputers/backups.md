---
slug: "opencomputers/backups"
title: "Backups and snapshots"
description: "Schedule directory backups on your own hosts, keep a rolling snapshot history, and restore a snapshot onto the same host or a different one."
section: "Computers"
order: 3019
---

A backup configuration names one directory on one host, a schedule, and a retention policy.
Each run produces a snapshot, and a snapshot can be restored onto any host in your organization.

This is the backup path for machines you own. Managed sandboxes and computers use their own [snapshots](/docs/api-reference/snapshots).

## Create a backup configuration

**miosa CLI**

```bash
miosa host backup list <host>
miosa host backup new <host> --name nightly --path /data --schedule "0 3 * * *" --keep 7
miosa host backup new <host> --name logs --path /var/log --schedule "0 * * * *" --compression 6 --exclude "*.tmp"
```

Aliases: `miosa host backups`, and `create` for `new`.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/backup-configs" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"nightly","path":"/data","schedule_cron":"0 3 * * *","timezone":"UTC","retention_count":7}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | 1 to 100 characters; unique per host |
| `path` | string | Yes | Directory on the host to back up, up to 1024 characters |
| `schedule_cron` | string | No | Cron expression, parsed and validated when the configuration is saved |
| `timezone` | string | No | Timezone the schedule is read in; default `UTC` |
| `retention_count` | integer | No | How many snapshots to keep, 1 to 365 |
| `retention_days` | integer | No | How many days to keep snapshots, 1 to 3650 |
| `exclude_patterns` | string[] | No | Globs to exclude |
| `compression_level` | integer | No | 1 to 22; default 19 |

A configuration is `active` or `paused`.
`last_snapshot_at` and `last_snapshot_size_bytes` record the most recent run.

## Run and manage schedules

```bash
miosa host backup trigger <host> nightly
miosa host backup pause <host> nightly
miosa host backup resume <host> nightly
miosa host backup update <host> nightly --schedule "0 4 * * *"
miosa host backup rm <host> nightly --yes
```

| Action | REST |
|---|---|
| Trigger now | `POST .../backup-configs/{config_id}/trigger` |
| Pause | `POST .../backup-configs/{config_id}/pause` |
| Resume | `POST .../backup-configs/{config_id}/resume` |
| Update | `PATCH .../backup-configs/{config_id}` |
| Delete | `DELETE .../backup-configs/{config_id}` |

A trigger returns `202` with the new snapshot while the upload is still running.
Triggering a backup on a host that is not connected returns `503`.

## Snapshots

**miosa CLI**

```bash
miosa host backup snapshot list <host> nightly --limit 10
miosa host backup snapshot list <host> nightly --before 2026-09-01T00:00:00Z
miosa host backup snapshot get <snapshot>
miosa host backup snapshot rm <snapshot> --yes
```

Aliases: `miosa host backup snapshots`.

**REST API**

```bash
curl --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/backup-configs/$CONFIG_ID/snapshots?limit=10" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

| Snapshot field | Meaning |
|---|---|
| `id` | Snapshot ID, used by every snapshot route |
| `config_id` | The configuration that produced it |
| `host_id` | The host it was taken from |
| `state` | `uploading`, `complete`, `failed`, or `expired` |
| `total_bytes` | Uncompressed size |
| `compressed_bytes` | Stored size |
| `chunk_count` | Number of stored chunks |
| `started_at`, `completed_at` | Timing |
| `error` | Failure reason, when `state` is `failed` |

Only a snapshot in state `complete` can be restored.
Snapshot routes are tenant-scoped, so `GET /api/v1/opencomputers/snapshots/{id}` and `DELETE /api/v1/opencomputers/snapshots/{id}` take the snapshot ID alone, without a host.

## Restore a snapshot

**miosa CLI**

```bash
miosa host backup snapshot fork <snapshot> --to-host <host> --to-path /restore
miosa host backup snapshot fork <snapshot> --overwrite
```

`--to-host` defaults to the host the snapshot came from, and `--overwrite` replaces files that already exist.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/snapshots/$SNAPSHOT_ID/restore" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"target_host_id":"<host-id>","target_path":"/restore","overwrite":false}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `target_host_id` | string | Yes | Host to write onto |
| `target_path` | string | No | Directory to write into |
| `overwrite` | boolean | No | Replace existing files; default `false` |

A restore returns `202` with a `restore_id`.
It is asynchronous: watch the target host's filesystem or the [audit log](/docs/opencomputers/security#audit-log) to confirm the files landed.

> **Caution: Restoring is a write to a live machine**
>
> A restore writes real files onto a real host. Without `overwrite`, existing files are left alone, and the restore can still leave a partially restored directory if it fails midway.
> Restore into a new path first, compare, and only then replace the live directory.

## What a backup does and does not cover

- It covers the files under one directory tree on one host. It does not image the operating system, and it does not capture running process state.
- It does not back up a database consistently on its own. Quiesce or dump the database into the backed-up directory before the schedule runs, or use the database's own backup tooling.
- It is not a substitute for a plan that covers losing the machine itself. Keep the credentials and configuration needed to rebuild the host somewhere other than the host.
