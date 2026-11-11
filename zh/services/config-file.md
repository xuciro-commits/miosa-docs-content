---
slug: "services/config-file"
title: "以代码方式管理服务设置"
description: "将服务的设置放在仓库根目录的文件中，让仓库说明服务如何运行，并在控制台中显示每个值的来源。"
section: "Services & projects"
order: 1008
---

服务可以将设置放在仓库根目录的一个文件中。同一个文件对 Docker Host 和 Services 都适用：

- `miosa.app.yml`、`miosa.app.yaml` 或 `miosa.app.json`，以及
- `miosa.json`。

`railway.json` 会作为兼容回退读取，因此从 Railway 导入的项目无需先修改。

## 规则

**只会更改文件中明确指定的字段。** 只包含一个键的文件会让其他所有设置保持项目当前的值。文件是追加，而不是替换。

**API 仍会验证。** 合并后的设置会经过与控制台保存相同的验证，因此文件无法写入 API 会拒绝的值。

**对于文件指定的字段，文件优先于控制台。** 控制台会显示哪些字段来自文件。

## 字段

| 文件键 | 设置 | 适用范围 |
|---|---|---|
| `deploy.start` / `startCommand` / `start_command` | 启动命令 | 全部 |
| `deploy.preDeploy` / `preDeployCommand` | 部署前命令 | 全部 |
| `deploy.healthcheckPath` / `healthcheck_path` | 健康检查路径 | Web 服务 |
| `deploy.healthcheckTimeout` | 健康检查超时，单位为秒 | Web 服务 |
| `deploy.restartPolicy` / `restartPolicyType` | `always`、`on_failure` 或 `never` | 全部 |
| `deploy.maxRetries` / `restartPolicyMaxRetries` | 放弃前的重试次数 | 全部 |
| `deploy.replicas` / `numReplicas` | 副本数 | 全部 |
| `deploy.cron` / `cronSchedule` | 五字段计划 | 定时任务服务 |
| `deploy.region` | `us-east`、`us-west` 或 `us-mia` | 全部 |
| `deploy.sleep` / `sleepApplication` | 空闲时休眠，默认五分钟 | 带公共地址的 Web 服务 |
| `build_command` | 构建命令，仅 Railpack 构建 | 静态服务 |
| `output_dir` | 要提供服务的目录，相对于应用 | 静态服务 |

`variables` 只按名称读取；值不会从文件中获取。

## 文件不能携带的内容

没有对应项的键会在计划和控制台中报告，而不是悄悄丢弃：`build.builder`、`build.dockerfilePath` 以及任何未知内容。

> **注意：值的来源**
>
> 文件设置的每个字段都会记录到设置它的文件和键上，因此几个月后也能回答“为什么启动命令是这个？”。
