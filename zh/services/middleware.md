---
slug: "services/middleware"
title: "边缘中间件"
description: "在服务器服务前方、于边缘运行的有序路径规则和可选函数。"
section: "Networking & domains"
order: 5004
---

边缘中间件在服务器服务前方运行，在请求到达你的进程之前处理：重定向、重写、请求头变更，以及一个可选发布的函数，该函数可以响应、继续或重写请求。

它是一组结构化规则，因此以文件形式编写，而不是一堆命令行标志：

```bash
miosa middleware shop API --from middleware.json
miosa middleware shop API           # what is configured now
miosa middleware shop API --off     # remove the rules and the function
```

```json
{
  "rules": [
    { "path": "/old/*", "redirect": { "to": "/new", "status": 308 } },
    { "path": "/api/*", "request_headers": { "set": { "x-plan": "pro" } } }
  ],
  "function": { "slug": "gate", "paths": ["/app/*"], "timeout_ms": 250, "on_error": "closed" }
}
```

规则按顺序执行，最多可以有五十条。`path` 可以是精确路径，也可以是以 `/*` 结尾的前缀，其中 `$1` 表示路径的剩余部分。每条规则必须且只能包含一个 `redirect`（可选状态码 `301`、`302`、`303`、`307` 或 `308`，默认 `307`）或一个 `rewrite`，并且可以附带请求头变更。

函数通过返回 `x-miosa-middleware: next` 让请求继续，并且可以添加 `x-miosa-middleware-request-<name>` 或 `x-miosa-middleware-rewrite`。任何其他响应都会返回给客户端。当 `on_error: "closed"` 时，函数失败会返回 `503 edge_middleware_unavailable`；当为 `open` 时，请求继续。

受保护的请求头名称会被拒绝，包括 `host`、`content-length`、`transfer-encoding`、`connection`、`upgrade`、`x-internal-*`、`x-miosa-*` 和 `x-forwarded-*`。

> **注意：几秒内生效**
>
> 更改会在约五秒内应用到每个边缘节点。验证错误会指出字段名（`settings.edge_middleware.<path>`），因此规则中的拼写错误很容易找到。
