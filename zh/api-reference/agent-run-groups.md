---
slug: "api-reference/agent-run-groups"
title: "智能体运行组"
description: "创建持久的多智能体编排组、调度运行清单、检查计数并取消分组工作。"
section: "API reference"
order: 12035
---

智能体运行组是多智能体扇出的持久控制平面对象。
当一个用户请求创建多个相关智能体运行时使用组
沙箱、计算机或自定义运行时。

基本路径：`/api/v1/agent-run-groups`

> **注意：可以使用排队调度**
>
> 对您希望在其中创建子运行的小扇出使用同步调度
> 回应。使用 `async: true` 进行排队扇出：MIOSA 存储每个清单
> 条目，在工作人员声明条目时强制执行组的并发限制，以及
> 立即返回排队条目记录。

## 端点

|方法|路径|描述 |
|---|---|---|
| `GET` | `/api/v1/agent-run-groups` |列出租户组 |
| `POST` | `/api/v1/agent-run-groups` |创建群组 |
| `GET` | `/api/v1/agent-run-groups/{id}` |获取一组 |
| `POST` | `/api/v1/agent-run-groups/{id}/dispatch` |调度子智能体运行的有界清单 |
| `GET` | `/api/v1/agent-run-groups/{id}/events` |重播或流式传输组中每个孩子的持久事件 |
| `POST` | `/api/v1/agent-run-groups/{id}/cancel` |取消分组并运行子进程 |

## 创建

```http
POST /api/v1/agent-run-groups
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "name": "lumen-browser-checks",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "concurrency_limit": 25,
  "expected_runs": 100,
  "metadata": {
    "user_task_id": "task_123"
  }
}
```

## 调度

Dispatch 接受 `runs` 数组。每个条目使用相同的字段
[智能体运行](/docs/api-reference/agent-runs)，加上可选
`orchestration_role` 和 `parent_agent_run_id`。

```http
POST /api/v1/agent-run-groups/{id}/dispatch
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "runs": [
    {
      "sandbox_id": "sbx_code_1",
      "provider": "claude-code",
      "cwd": "/workspace",
      "prompt": "Implement the pricing page and export /workspace/report.html",
      "orchestration_role": "coder",
      "metadata": {
        "artifact_paths": ["/workspace/report.html"]
      }
    },
    {
      "computer_id": "cmp_browser_1",
      "provider": "osa",
      "prompt": "Open the preview, test signup, and capture screenshots.",
      "orchestration_role": "browser-qa"
    }
  ],
  "async": true
}
```

当组应将条目排队并在
创建子智能体运行。 MIOSA 为每个清单项目创建一个持久条目，
然后调度员工作人员在组的 `concurrency_limit` 下领取条目。

当前调度批次的上限是每个请求的 `100` 条目。对于较大的
舰队，将逻辑工作分成多个组或从您的系统中批量分派
后端。

## 回应

```json
{
  "data": {
    "group": {
      "id": "grp_123",
      "name": "lumen-browser-checks",
      "status": "running",
      "concurrency_limit": 25,
      "expected_runs": 100,
      "counts": {
        "total": 2,
        "running": 0,
        "succeeded": 2,
        "failed": 0,
        "canceled": 0
      },
      "entry_counts": {
        "total": 2,
        "queued": 0,
        "running": 0,
        "succeeded": 2,
        "failed": 0,
        "canceled": 0
      }
    },
    "results": [
      {
        "index": 0,
        "ok": true,
        "run": {
          "id": "run_123",
          "agent_run_group_id": "grp_123",
          "orchestration_role": "coder",
          "status": "succeeded"
        }
      }
    ],
    "entries": []
  }
}
```

异步调度立即返回 `entries` 并且通常返回空
`results` 数组：

```json
{
  "data": {
    "group": {
      "id": "grp_123",
      "status": "running",
      "concurrency_limit": 25,
      "entry_counts": {
        "total": 2,
        "queued": 2,
        "running": 0,
        "succeeded": 0,
        "failed": 0,
        "canceled": 0
      }
    },
    "results": [],
    "entries": [
      {
        "id": "agre_123",
        "agent_run_group_id": "grp_123",
        "index": 0,
        "status": "queued",
        "attempts": 0
      }
    ]
  }
}
```

## 活动

当您的 UI 需要一个流来完成整个多智能体任务时，请使用组事件。
MIOSA 记录子智能体运行事件并通过组重播它们
序列顺序。这可以让您的产品从编码器、测试人员那里呈现进度，
浏览器 QA 智能体和部署验证程序，无需为每个子项打开一个流
跑。

将现有事件重播为 JSON：

```http
GET /api/v1/agent-run-groups/{id}/events
Authorization: Bearer msk_u_...
```

通过 SSE 流式传输现有活动和现场活动：

```http
GET /api/v1/agent-run-groups/{id}/events?stream=true
Accept: text/event-stream
Authorization: Bearer msk_u_...
```

事件有效负载包括组 ID、子运行 ID、事件序列、事件类型、
消息、负载和时间戳：

```json
{
  "data": [
    {
      "id": "ev_123",
      "agent_run_group_id": "grp_123",
      "agent_run_id": "run_123",
      "sequence": 4,
      "type": "command_finished",
      "message": "Command completed",
      "payload": {
        "exit_code": 0
      },
      "created_at": "2026-06-15T21:20:00Z"
    }
  ]
}
```

终端组状态在重放后关闭 SSE 响应。跑步团体保持
在等待新的子事件时，流打开并发出心跳注释。

## SDK

**打字稿**

```typescript
const group = await miosa.agentRunGroups.create({
  name: "lumen-browser-checks",
  workspaceId: "ws_123",
  concurrencyLimit: 25,
  expectedRuns: 100,
});

const result = await miosa.agentRunGroups.dispatch(
  group.id,
  [
    {
      sandboxId: "sbx_code_1",
      provider: "claude-code",
      cwd: "/workspace",
      prompt: "Implement the pricing page and export /workspace/report.html",
      orchestrationRole: "coder",
      metadata: { artifact_paths: ["/workspace/report.html"] },
    },
  ],
  { async: true },
);

const withRuns = await miosa.agentRunGroups.get(group.id, { includeRuns: true });

const events = await miosa.agentRunGroups.events(group.id);

for await (const event of miosa.agentRunGroups.streamEvents(group.id)) {
  console.log(event.agent_run_id, event.type, event.message);
}
```

**Python**

```python
group = client.agent_run_groups.create(
    name="lumen-browser-checks",
    workspace_id="ws_123",
    concurrency_limit=25,
    expected_runs=100,
)

result = client.agent_run_groups.dispatch(
    group["id"],
    [
        {
            "sandbox_id": "sbx_code_1",
            "provider": "claude-code",
            "cwd": "/workspace",
            "prompt": "Implement the pricing page and export /workspace/report.html",
            "orchestration_role": "coder",
            "metadata": {"artifact_paths": ["/workspace/report.html"]},
        }
    ],
    async_=True,
)

with_runs = client.agent_run_groups.get(group["id"], include_runs=True)

events = client.agent_run_groups.events(group["id"])

for event in client.agent_run_groups.stream_events(group["id"]):
    print(event["agent_run_id"], event["type"], event.get("message"))
```

## 命令行界面

```bash
miosa api POST /run-groups -d '{
  "name": "lumen-browser-checks",
  "workspace_id": "ws_123",
  "concurrency_limit": 25,
  "expected_runs": 100
}'

miosa api POST /run-groups/grp_123/dispatch -d '{
  "runs": [
    {
      "sandbox_id": "sbx_code_1",
      "provider": "claude-code",
      "prompt": "Build and export report",
      "metadata": {"artifact_paths": ["/workspace/report.html"]}
    }
  ],
  "async": true
}'

miosa api GET /run-groups/grp_123?include=runs
miosa api GET /run-groups/grp_123/activity
miosa api GET "/run-groups/grp_123/activity?stream=true&limit=100"
miosa api POST /run-groups/grp_123/cancel
```

## 下一步

- [智能体运行](/docs/api-reference/agent-runs)：将一个提示分派到沙箱或计算机中并收集事件和工件。
- [智能体编排](/docs/guides/agent-orchestration)：设计分配设备、流式传输进度和存储输出的产品后端。
- [运行时环境](/docs/api-reference/runtime-env)：为智能体和部署配置继承的环境变量。
