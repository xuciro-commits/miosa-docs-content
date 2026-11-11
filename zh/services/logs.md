---
slug: "services/logs"
title: "服务日志"
description: "按副本读取服务的运行时日志、某次发布的构建日志，以及导致其状态的事件。"
section: "Services & projects"
order: 1016
---

服务面板有一个 **Logs** 标签页，包含三个视图。

| 视图 | 显示内容 |
|---|---|
| 运行时 | 每个副本打印的最新日志行，以及该副本的状态。 |
| 构建 | 生成某次发布的构建日志，以及构建是否完成。 |
| 事件 | 某次发布的状态转换，以及失败时的原因。 |

运行容器镜像的服务没有构建过程，因此其构建视图为空。

## 运行时日志

运行时日志按副本提供。
每个副本会报告其状态和日志行；如果无法读取其日志，则会报告错误。
每个副本单独列出，使用 `replica` 参数可以读取其中一个。

```text
GET /api/v1/projects/:id/services/:sid/logs?tail=200&replica=<id>
```

## 构建日志

```text
GET /api/v1/projects/:id/services/:sid/builds/latest/logs?tail=200
GET /api/v1/projects/:id/services/:sid/builds/:build_id/logs?after=<offset>
```

响应包含文本、继续读取所需的偏移量；构建完成后还会包含 `complete`。
要跟踪正在运行的构建，请将 `after` 设为上一个偏移量并再次请求。

## 事件与失败原因

```bash
miosa service logs shop API
```

CLI 会打印最新发布的状态转换及其失败原因。
它不会流式输出运行时日志。

## 下一步

- [发布与回滚](/docs/services/releases)：事件所描述的发布状态。
- [副本与扩缩容](/docs/services/replicas)：什么是副本。
