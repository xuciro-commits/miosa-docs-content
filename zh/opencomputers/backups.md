---
slug: "opencomputers/backups"
title: "备份和快照"
description: "在您自己的主机上安排目录备份，保留滚动快照历史记录，并将快照还原到同一主机或不同主机上。"
section: "Computers"
order: 3019
---

备份配置命名一台主机上的一个目录、计划和保留策略。
每次运行都会生成一个快照，并且可以将快照恢复到组织中的任何主机上。

这是您拥有的计算机的备份路径。托管沙箱和计算机使用自己的[快照](/docs/api-reference/snapshots)。

## 创建备份配置

**MIOSA CLI**

```bash
miosa host backup list <host>
miosa host backup new <host> --name nightly --path /data --schedule "0 3 * * *" --keep 7
miosa host backup new <host> --name logs --path /var/log --schedule "0 * * * *" --compression 6 --exclude "*.tmp"
```

别名：`miosa host backups` 和 `create` 代表 `new`。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/backup-configs" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"nightly","path":"/data","schedule_cron":"0 3 * * *","timezone":"UTC","retention_count":7}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `name` |字符串|是的 | 1 到 100 个字符；每个主机都是唯一的|
| `path` |字符串|是的 |主机上要备份的目录，最多 1024 个字符 |
| `schedule_cron` |字符串|没有 | Cron 表达式，在保存配置时解析和验证 |
| `timezone` |字符串|没有 |读入时间表的时区；默认 `UTC` |
| `retention_count` |整数 |没有 |要保留多少个快照，1 到 365 |
| `retention_days` |整数 |没有 |快照保留多少天，1 到 3650 |
| `exclude_patterns` |字符串[] |没有 |要排除的 Glob |
| `compression_level` |整数 |没有 | 1至22；默认 19 |

配置为 `active` 或 `paused`。
`last_snapshot_at` 和 `last_snapshot_size_bytes` 记录最近的运行。

## 运行和管理计划

```bash
miosa host backup trigger <host> nightly
miosa host backup pause <host> nightly
miosa host backup resume <host> nightly
miosa host backup update <host> nightly --schedule "0 4 * * *"
miosa host backup rm <host> nightly --yes
```

|行动|休息 |
|---|---|
|立即触发 | `POST .../backup-configs/{config_id}/trigger` |
|暂停 | `POST .../backup-configs/{config_id}/pause` |
|简历 | `POST .../backup-configs/{config_id}/resume` |
|更新 | `PATCH .../backup-configs/{config_id}` |
|删除| `DELETE .../backup-configs/{config_id}` |

当上传仍在运行时，触发器会返回 `202` 和新快照。
在未连接的主机上触发备份会返回 `503`。

## 快照

**MIOSA CLI**

```bash
miosa host backup snapshot list <host> nightly --limit 10
miosa host backup snapshot list <host> nightly --before 2026-09-01T00:00:00Z
miosa host backup snapshot get <snapshot>
miosa host backup snapshot rm <snapshot> --yes
```

别名：`miosa host backup snapshots`。

**休息API**

```bash
curl --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/backup-configs/$CONFIG_ID/snapshots?limit=10" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

|快照字段 |意义|
|---|---|
| `id` |快照 ID，由每个快照路由使用 |
| `config_id` |产生它的配置 |
| `host_id` |其来源主机 |
| `state` | `uploading`、`complete`、`failed` 或 `expired` |
| `total_bytes` |未压缩大小|
| `compressed_bytes` |储存尺寸 |
| `chunk_count` |存储块的数量 |
| `started_at`、`completed_at` |时间 |
| `error` |失败原因，当`state`为`failed` |

只能恢复状态为`complete`的快照。
快照路由是租户范围的，因此 `GET /api/v1/opencomputers/snapshots/{id}` 和 `DELETE /api/v1/opencomputers/snapshots/{id}` 单独获取快照 ID，而不使用主机。

## 恢复快照

**MIOSA CLI**

```bash
miosa host backup snapshot fork <snapshot> --to-host <host> --to-path /restore
miosa host backup snapshot fork <snapshot> --overwrite
```

`--to-host` 默认为快照来自的主机，`--overwrite` 替换已存在的文件。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/snapshots/$SNAPSHOT_ID/restore" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"target_host_id":"<host-id>","target_path":"/restore","overwrite":false}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `target_host_id` |字符串|是的 |主机写入 |
| `target_path` |字符串|没有 |要写入的目录 |
| `overwrite` |布尔 |没有 |替换现有文件；默认 `false` |

恢复返回 `202` 和 `restore_id`。
它是异步的：观察目标主机的文件系统或[审核日志](/docs/opencomputers/security#audit-log)以确认文件已登陆。

> **警告：恢复是对实时机器的写入**
>
> 恢复将真实文件写入真实主机。如果没有 `overwrite`，现有文件将被单独保留，并且如果中途失败，恢复仍会留下部分恢复的目录。
> 先恢复到新路径，比较，然后才替换live目录。

## 备份涵盖和不涵盖哪些内容

- 它涵盖一台主机上一个目录树下的文件。它不镜像操作系统，也不捕获正在运行的进程状态。
- 它本身不会一致地备份数据库。在计划运行之前将数据库停顿或转储到备份目录中，或者使用数据库自​​己的备份工具。
- 它不能替代机器本身丢失的保险。将重建主机所需的凭据和配置保留在主机以外的其他位置。
