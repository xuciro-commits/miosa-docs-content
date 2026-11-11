---
slug: "optimal-engine/operations"
title: "Operate a self-hosted Optimal Engine"
description: "Backups and restore, upgrades and migrations, size profiles and rate limits, replication between Engines, moving a workspace, erasure and retention, and metrics for a self-hosted Optimal Engine."
section: "Optimal Engine"
order: 8004
---

This page is for a self-hosted Engine.
On MIOSA, backups, upgrades, and resizing are done for you; see [Optimal Engine on MIOSA](/docs/optimal-engine/miosa#backups-upgrades-and-size).

## Backups

A backup is an online copy of the SQLite database, taken with `VACUUM INTO` while the Engine keeps serving.

```bash
mix optimal.backup /var/backups/engine-$(date +%F).db --verify
```

`--verify` runs an integrity check on the copy.
Over HTTP, an `admin` key can call `POST /api/backup` with an optional `{"name"}`; the file is written in the backup folder and the response has its `path`, `size_bytes`, `checksum_sha256`, and `integrity_status`.

### Offsite backups

With `OPTIMAL_BACKUP_KEY` and the `OPTIMAL_S3_*` settings, backups can be encrypted and sent to S3-compatible storage:

```bash
mix optimal.backup /tmp/snapshot.db --offsite
mix optimal.backup --list-offsite
```

Set `OPTIMAL_OFFSITE_BACKUP_INTERVAL_MS` (60000 or more) to take them on a schedule.
The Engine keeps the newest 14 offsite and 3 local copies by default (`OPTIMAL_OFFSITE_BACKUP_KEEP`, `OPTIMAL_OFFSITE_BACKUP_KEEP_LOCAL`).

### Restore

Stop the Engine first.

```bash
mix optimal.restore /var/backups/engine-2026-10-23.db
mix optimal.restore --from-offsite <backup id>
```

The restore checks the copy's checksum and integrity before it changes anything, and keeps the previous database beside it as `<db>.pre-restore-<time>`.

> **Caution: What a backup does not include**
>
> A backup is the database only.
> Workspace folders under `OPTIMAL_ENGINE_ROOT`, local asset files, and the RocksDB graph folder are not in it; back them up with the volume or folder.
> Local backups are not encrypted, and they contain the encrypted connector keys, so store them as secrets.
> With Docker, the simplest full backup is the whole `/data` volume.

## Upgrades

1. Take a backup.
2. Install the new version: pull the new tag and rebuild, or start the new image on the same volume.
3. Start it. Migrations run at boot, each in its own transaction, before the API answers.

If a migration fails, it is rolled back, the Engine refuses to open the store, and it exits with `migration <number> failed and was rolled back: <cause>`.
Migrations that ran before it in the same boot stay applied, so going back means restoring the backup from step 1.

Check what is running and what it expects:

```bash
curl -s http://127.0.0.1:4200/api/version -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
mix optimal.migrate --status
```

`mix optimal.conformance --url <url> --key <key>` runs the conformance suite against a live Engine, prints a JSON report, and exits non-zero on any failure, so it can gate an upgrade.

A release build runs the same maintenance with the API off: `bin/optimal maintenance migrate`, `health`, `embed_backfill`, `graph_backfill`, `facts`, or `claims`.
It exits `0` when done, `1` on failure, and `78` when the Engine is in the wrong state for the task.

## Sizing

`OPTIMAL_SIZE_PROFILE` sets the rate limit per key and the advised limits the Engine measures itself against.

| Profile | Machine | Disk | Vectors | Memories | API requests per minute per key |
|---|---|---|---|---|---|
| `agent` | 1 vCPU, 1 GiB | 2 GiB | 50,000 | 100,000 | 300 |
| `workspace` | 1 vCPU, 2 GiB | 10 GiB | 250,000 | 500,000 | 600 |
| `business` (default) | 2 vCPU, 4 GiB | 25 GiB | 1M | 2M | 1,200 |
| `platform` | 4 vCPU, 8 GiB | 100 GiB | 5M | 10M | 3,000 |
| `scale` | 8 vCPU, 16 GiB or more | 250 GiB or more | more | more | 6,000 |

The vector and memory numbers are advice, not hard caps.
At 70% of any of them, `GET /api/storage` and `/api/metrics` recommend the next profile.
A key can get its own rate limit with `rate_limit_per_minute` in its metadata.

The Engine is a single writer: run one Engine per data folder.

## Replication

Two SQLite Engines can keep a workspace in sync.

| Route | What it does |
|---|---|
| `POST /api/sync/enable` | Start recording changes in a workspace |
| `POST /api/sync/peers` | Add a peer: `{"workspace", "url", "api_key", "policy", "max_class"}` (`admin`) |
| `POST /api/sync/run` | Sync now |
| `GET /api/sync/status`, `/changes`, `/conflicts` | State, the change log, and conflicts |
| `POST /api/sync/conflicts/:id/resolve` | Resolve one conflict |

`policy` is `push`, `pull`, `both`, or `none`.
`max_class` is the most sensitive data class that may leave: `public`, `internal`, `confidential` (default), or `restricted`.
The peer's key is stored encrypted with `CONNECTOR_KEY`.
Peers sync every 20 seconds by default.
Assets, connectors, API keys, jobs, and tools are not replicated.

## Move a workspace

A workspace bundle is one file with a whole workspace in it.

| Route | What it does |
|---|---|
| `POST /api/workspaces/:id/bundle` | Write a bundle |
| `POST /api/bundles/import` | Import one |
| `POST /api/bundles/jobs/export`, `POST /api/bundles/jobs/import` | The same as background jobs, for large workspaces |
| `GET /api/bundles/jobs/:id` | Follow a job |
| `POST /api/workspaces/:id/finalize-move` | Remove the workspace from the old Engine after the move |

Bundles are limited to 5 GiB (`OPTIMAL_BUNDLE_MAX_BYTES`).

## Erasure and retention

```bash
mix optimal.compliance dsar <principal id>
mix optimal.compliance erase <principal id> --preview
mix optimal.compliance erase <principal id>
mix optimal.compliance hold place <signal id> --by <actor> --reason "litigation"
mix optimal.compliance hold list
mix optimal.compliance retention sweep --dry-run
```

| Action | Effect |
|---|---|
| `dsar` | Exports what the Engine holds about one person |
| `erase` | Deletes it, and what was derived from it. `--preview` shows what would go; `--force` overrides legal holds. |
| `hold` | Places, lists, or releases a legal hold that stops deletion |
| `retention sweep` | Applies each source's retention rule |
| `POST /api/workspaces/:id/archive` | Archives a workspace; its rows are kept |

An erasure does not reach backups, offsite copies, exported bundles, or replicas that already synced the data.
SQLite does not overwrite freed pages, so deleted text can remain in the database file until it is vacuumed.

## Observability

| Route | What it returns |
|---|---|
| `GET /api/health` | `up`, `degraded`, or `down`, with the check that failed |
| `GET /api/metrics` | JSON counters, latencies, connector state, and size gauges |
| `GET /api/metrics/prometheus` | Prometheus text, such as `optimal_engine_search_latency_ms`, `optimal_engine_memories_total`, `optimal_engine_uptime_seconds` |
| `GET /api/viability` | A report on the workspace as an operating system, with alerts |

The Prometheus route needs a key when keys are required; it is only exempt from rate limits.
`deploy/grafana-dashboard.json` in the repository is a ready dashboard for it.
Logs go to standard output as `[level] message`.
