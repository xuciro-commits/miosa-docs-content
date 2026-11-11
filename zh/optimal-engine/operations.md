---
slug: "optimal-engine/operations"
title: "运行自托管优化引擎"
description: "备份和恢复、升级和迁移、大小配置文件和速率限制、引擎之间的复制、移动工作区、擦除和保留以及自托管Optimal Engine的指标。"
section: "Optimal Engine"
order: 8004
---

此页面适用于自托管引擎。
在 MIOSA 上，备份、升级和调整大小都会为您完成；请参阅[MIOSA 上的Optimal Engine](/docs/optimal-engine/miosa#backups-upgrades-and-size)。

## 备份

备份是 SQLite 数据库的在线副本，在引擎保持服务时使用 `VACUUM INTO` 获取。

```bash
mix optimal.backup /var/backups/engine-$(date +%F).db --verify
```

`--verify` 对副本运行完整性检查。
通过 HTTP，`admin` 键可以使用可选的 `{"name"}` 调用 `POST /api/backup`；文件写入备份文件夹中，响应具有其 `path`、`size_bytes`、`checksum_sha256` 和 `integrity_status`。

### 异地备份

使用 `OPTIMAL_BACKUP_KEY` 和 `OPTIMAL_S3_*` 设置，可以加密备份并将其发送到 S3 兼容存储：

```bash
mix optimal.backup /tmp/snapshot.db --offsite
mix optimal.backup --list-offsite
```

设置 `OPTIMAL_OFFSITE_BACKUP_INTERVAL_MS`（60000 或更多）以按计划进行。
默认情况下，引擎保留最新的 14 个异地副本和 3 个本地副本（`OPTIMAL_OFFSITE_BACKUP_KEEP`、`OPTIMAL_OFFSITE_BACKUP_KEEP_LOCAL`）。

### 恢复

首先停止发动机。

```bash
mix optimal.restore /var/backups/engine-2026-10-23.db
mix optimal.restore --from-offsite <backup id>
```

恢复会在更改任何内容之前检查副本的校验和和完整性，并将先前的数据库保留在其旁边作为 `<db>.pre-restore-<time>`。

> **注意：备份不包括什么**
>
> 备份仅是数据库。
> `OPTIMAL_ENGINE_ROOT`下的工作区文件夹、本地资源文件、RocksDB图文件夹不在其中；使用卷或文件夹备份它们。
> 本地备份未加密，并且包含加密的连接器密钥，因此将它们存储为机密。
> 对于 Docker，最简单的完整备份是整个 `/data` 卷。

## 升级

1. 进行备份。
2. 安装新版本：拉取新标签并重建，或在同一卷上启动新映像。
3. 启动它。迁移在启动时运行，每个迁移都在其自己的事务中，然后 API 响应。

如果迁移失败，则会回滚，引擎拒绝打开存储，并以 `migration <number> failed and was rolled back: <cause>` 退出。
在同一引导中之前运行的迁移仍然适用，因此返回意味着从步骤 1 恢复备份。

检查正在运行的内容以及它的期望：

```bash
curl -s http://127.0.0.1:4200/api/version -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
mix optimal.migrate --status
```

`mix optimal.conformance --url <url> --key <key>` 针对实时引擎运行一致性套件，打印 JSON 报告，并在任何失败时以非零值退出，因此它可以控制升级。

发布版本在 API 关闭的情况下运行相同的维护：`bin/optimal maintenance migrate`、`health`、`embed_backfill`、`graph_backfill`、`facts` 或 `claims`。
完成时退出 `0`，失败时退出 `1`，当引擎处于错误的任务状态时退出 `78`。

## 尺码

`OPTIMAL_SIZE_PROFILE` 设置每个键的速率限制以及引擎自行测量的建议限制。

|规格 |机器 |磁盘 |向量 |记忆 |每个密钥每分钟的 API 请求数 |
|---|---|---|---|---|---|
| `agent` | 1 个 vCPU，1 GiB | 2 GiB | 50,000 | 100,000 | 300 |
| `workspace` | 1 个 vCPU，2 GiB | 10 GiB | 250,000 | 500,000 | 600 |
| `business`（默认）| 2 个 vCPU，4 GiB | 25 GiB | 1M | 2M | 1,200 |
| `platform` | 4 个 vCPU，8 GiB | 100 GiB | 5M | 10M | 3,000 |
| `scale` | 8 个 vCPU、16 GiB 或更多 | 250 GiB 或更多 |更多 |更多 | 6,000 |

向量和内存数字是建议，而不是硬性上限。
其中 70% 的人 `GET /api/storage` 和 `/api/metrics` 推荐下一个配置文件。
密钥可以通过其元数据中的 `rate_limit_per_minute` 获得自己的速率限制。

该引擎是单个编写器：每个数据文件夹运行一个引擎。

## 复制

两个 SQLite 引擎可以保持工作空间同步。

|路由 |它有什么作用 |
|---|---|
| `POST /api/sync/enable` |开始记录工作区中的更改 |
| `POST /api/sync/peers` |添加对等点： `{"workspace", "url", "api_key", "policy", "max_class"}` (`admin`) |
| `POST /api/sync/run` |立即同步 |
| `GET /api/sync/status`、`/changes`、`/conflicts` |状态、变更日志和冲突 |
| `POST /api/sync/conflicts/:id/resolve` |解决一个冲突 |

`policy` 是 `push`、`pull`、`both` 或 `none`。
`max_class` 是可能留下的最敏感数据类：`public`、`internal`、`confidential`（默认）或 `restricted`。
对等方的密钥使用 `CONNECTOR_KEY` 加密存储。
默认情况下，对等点每 20 秒同步一次。
资产、连接器、API 密钥、作业和工具不会被复制。

## 移动工作空间

工作区包是一个包含整个工作区的文件。

|路由 |它有什么作用 |
|---|---|
| `POST /api/workspaces/:id/bundle` |写一个包|
| `POST /api/bundles/import` |导入一个|
| `POST /api/bundles/jobs/export`, `POST /api/bundles/jobs/import` |与后台作业相同，适用于大型工作空间 |
| `GET /api/bundles/jobs/:id` |关注工作 |
| `POST /api/workspaces/:id/finalize-move` |移动后从旧引擎中删除工作区 |

捆绑包大小限制为 5 GiB (`OPTIMAL_BUNDLE_MAX_BYTES`)。

## 删除和保留

```bash
mix optimal.compliance dsar <principal id>
mix optimal.compliance erase <principal id> --preview
mix optimal.compliance erase <principal id>
mix optimal.compliance hold place <signal id> --by <actor> --reason "litigation"
mix optimal.compliance hold list
mix optimal.compliance retention sweep --dry-run
```

|操作 |效果 |
|---|---|
| `dsar` |导出引擎保存的关于一个人的信息 |
| `erase` |删除它以及从中衍生出的内容。 `--preview` 显示将会发生的事情； `--force` 优先于合法保留。 |
| `hold` | 设置、列出或解除阻止删除的合法保留 |
| `retention sweep` |应用每个来源的保留规则 |
| `POST /api/workspaces/:id/archive` |归档工作空间；它的行被保留|

擦除不会到达备份、异地副本、导出的捆绑包或已同步数据的副本。
SQLite 不会覆盖释放的页面，因此删除的文本可以保留在数据库文件中，直到被清理为止。

## 可观察性

|路由 |它返回什么 |
|---|---|
| `GET /api/health` | `up`、`degraded` 或 `down`，以及失败的检查项 |
| `GET /api/metrics` | JSON 计数器、延迟、连接器状态和大小计量表 |
| `GET /api/metrics/prometheus` | Prometheus 文本，如 `optimal_engine_search_latency_ms`、`optimal_engine_memories_total`、`optimal_engine_uptime_seconds` |
| `GET /api/viability` |关于工作区作为操作系统的报告，带有警报 |

在需要密钥时，Prometheus 路由同样需要密钥；它只是不受速率限制。
存储库中的 `deploy/grafana-dashboard.json` 是一个现成的仪表板。
日志以 `[level] message` 的形式发送到标准输出。
