---
slug: "optimal-engine/cli-sdks-mcp"
title: "Optimal Engine CLI、SDK 和 MCP"
description: "最佳客户端 CLI、用于 Claude Code 和 Codex 的引擎 MCP 服务器、操作员的混合任务以及 TypeScript 和 Python 客户端 - 如何从存储库安装每个客户端并将其指向引擎。"
section: "Optimal Engine"
order: 8009
---

下面的每个接口都与相同的 HTTP API 通信，因此每个接口的工作区看起来都是相同的。
所有这些都通过 [Miosa-osa/OptimalEngine](https://github.com/Miosa-osa/OptimalEngine) 的签出安装。

|接口 |对于 |哪里 |
|---|---|---|
| `optimal` CLI（也`oe`）|终端中的人员和智能体 | `apps/cli` |
| MCP服务器|智能体Claude Code、Codex等客户端| `apps/mcp` |
|混合任务 |引擎自己的机器上的操作员| `lib/mix/tasks` |
| TypeScript 客户端 |应用程序 | `sdks/typescript` |
| Python 客户端 |应用程序和脚本 | `sdks/python` |

> **注意：不适用于 npm 或 PyPI**
>
> 客户端和 MCP 服务器未发布到 npm 或 PyPI。
> 从存储库构建它们，如下所示。

## 最佳 CLI

需要 `sh`、`curl` 7.55 或更高版本以及 `awk` 的单个 shell 脚本。
它将密钥发送到标准输入上的curl，因此该密钥永远不会显示在进程列表中，也永远不会打印该密钥。

```bash
apps/cli/install.sh            # installs optimal and oe into /usr/local/bin or ~/.local/bin
optimal --version
```

### 将其指向引擎

设置按此顺序读取，第一个匹配获胜：

1. 标志：`--api <url>`、`--key-file <path>`、`-w <workspace>`、`--profile <name>`。
2. 命名配置文件，来自 `--profile` 或 `OPTIMAL_PROFILE`。
3. 环境：`OPTIMAL_ENGINE_URL`、`OPTIMAL_ENGINE_API_KEY` 或 `OPTIMAL_ENGINE_API_KEY_FILE`、`OPTIMAL_WORKSPACE`。
4. 默认配置文件 `~/.config/optimal/profile`。
5. `http://127.0.0.1:4200`。

配置文件包含相同的设置：

```text
url=http://127.0.0.1:4200
key_file=/home/me/.config/optimal/key
workspace=default:notes
```

`optimal doctor` 报告每个设置的来源以及引擎是否响应。
在 MIOSA 上，`miosa engine connect` 为您编写此配置文件；请参阅[MIOSA 上的Optimal Engine](/docs/optimal-engine/miosa#from-a-terminal)。

### 命令

|集团|命令 |
|---|---|
|会议| `boot`、`today`、`fires`、`brief`、`protocol`、`commitments` |
|循环 | `loops list`、`loops add`、`loops close`、`loops drop`、`loops reopen`、`loops extract` |
|检索 | `search` (`find`)、`grep`、`assemble`、`read`、`ask` (`rag`)、`reconstruct`、`memory`、`claims`、 `review` |
|记录| `capture`、`aware`、`note`、`lesson`、`decision`、`task`、`close`、`remember` |
|聊天和政策 | `session commit`、`session list`、`session show`、`policy`、`org-section` |
|规则| `routing`、`authority effective`、`authority decide`、`authority request`、`authority export` |
|设置和健康状况| `init`、`size`、`viability`、`chain`、`check`、`analyses`、`jobs` |
|发动机| `status`、`health`、`version`、`doctor`、`url`、`workspaces`、`organizations`、`whoami` |
|任何路线| `api <METHOD> /api/<path> [json]` |

```bash
optimal capture "Decided: annual plans get two months free." -w default:notes --title "Pricing"
optimal search "annual plans" -w default:notes -n 5
optimal ask "What did we decide about pricing?" -w default:notes
optimal api GET /api/version
```

常用选项：`-n` 用于限制，`--as-of` 用于时间点，`--mode lexical|hybrid|auto`、`--field a.b` 用于打印一个字段，`--raw` 用于无格式 JSON。

|退出代码 |意义|
|---|---|
| 0 |完成 |
| 1 |失败 |
| 64 | 错误用法|
| 65 | 无法读取引擎的答案|
| 69 | 引擎没有回答|
| 77 | 密钥被拒绝（401 或 403）|
| 78 | 设置错误、命令被拒绝或客户端太旧 (426)|

## MCP 服务器

通过相同 API 的 stdio MCP 服务器，用于智能体客户端。
它需要 Node 20 或更高版本。

```bash
cd apps/mcp
npm ci && npm run build
```

它在启动时从环境或配置文件中读取与 CLI 相同的设置。
密钥更改后重新启动它。

使用克劳德代码注册它：

```bash
claude mcp add --scope user \
  --env OPTIMAL_ENGINE_URL=http://127.0.0.1:4200 \
  --env OPTIMAL_ENGINE_API_KEY_FILE=/home/me/.config/optimal/key \
  --env OPTIMAL_WORKSPACE=default:notes \
  optimal-engine -- node /path/to/OptimalEngine/apps/mcp/dist/index.js
```

仅为其智能体提供一把密钥，其中包含其智能体所需的内容，例如一个工作区上的 `read` 和 `write`。

|集团|工具|
|---|---|
|操作循环| `boot`、`brief`、`protocol`、`loops_list`、`loops_add`、`loops_close` |
|检索 | `search`、`grep`、`ask`、`assemble`、`read`、`recall`、`profile`、`wiki_get`、`workspaces` |
|记录| `capture`、`record`、`close`、`add_memory`、`forget_memory` |
|聊天和政策 | `session_commit`、`sessions_list`、`session_get`、`org_policy` |
|规则和检查| `authority_decide`、`authority_effective`、`check_draft`、`viability`、`action_step`、`action_chain` |
|渲染| `render_context`, `render_specs` |

## 混合任务在引擎自己的机器上，结帐有直接在商店中工作的`mix optimal.*`任务。
有用的：

|任务|它有什么作用 |
|---|---|
| `mix optimal.auth` |创建、列出、撤销和删除 API 密钥 |
| `mix optimal.bootstrap` |编译、迁移和加载示例工作区 |
| `mix optimal.status`, `mix optimal.health` |活力和十个商店诊断 |
| `mix optimal.ingest`, `mix optimal.ingest_workspace` |摄取文本或文件夹 |
| `mix optimal.import_notion` |导入 Notion 导出 |
| `mix optimal.connector`, `mix optimal.pull` |设置并运行应用程序连接器 |
| `mix optimal.search`、`mix optimal.rag`、`mix optimal.assemble` |从命令行读取 |
| `mix optimal.claims`, `mix optimal.facts` |审查索赔并列出事实|
| `mix optimal.backup`、`mix optimal.restore`、`mix optimal.migrate` |运营;请参阅[操作](/docs/optimal-engine/operations) |
| `mix optimal.embed_backfill`, `mix optimal.index` |嵌入丢失的块，或重新索引 |
| `mix optimal.compliance` |数据导出、删除、合法保留和保留 |
| `mix optimal.conformance` |针对实时引擎运行一致性套件 |

存储库中的 `docs/guides/mix-tasks.md` 列出了所有这些。

## TypeScript 客户端

```bash
cd sdks/typescript
npm install && npm run build
```

```ts
import { OptimalEngine } from "@optimal-engine/client";

const engine = new OptimalEngine({
  baseUrl: "http://127.0.0.1:4200",
  apiKey: process.env.OPTIMAL_ENGINE_API_KEY,
  workspace: "default:notes",
});

await engine.ingest("Decided: annual plans get two months free.", { title: "Pricing" });
const hits = await engine.search("annual plans");
const context = await engine.ask("What did we decide about pricing?");
console.log(context.envelope.body, context.envelope.sources);
```

它还具有 `grep`、`assemble`、`profile`、`createNode` 以及 `memory`、`workspaces`、`wiki`、`recall`、`subscriptions` 的子客户端， `surface` 和 `render`。
Vercel AI SDK 和 OpenAI Agents SDK 的适配器位于 `@optimal-engine/client/adapters/ai-sdk` 和 `/adapters/openai-agents`。

## Python 客户端

```bash
pip install ./sdks/python
```

```python
from optimal_engine import OptimalEngine

engine = OptimalEngine(base_url="http://127.0.0.1:4200", workspace="default:notes", api_key="oe_...")
results = engine.search("annual plans")
memory = engine.memory.create(content="Customer Example Co renews in March.")
```

它需要 Python 3.10 或更高版本，并且具有同步 `OptimalEngine` 和异步 `AsyncOptimalEngine`。
它涵盖搜索、grep、召回、内存和工作区。

> **注意：Python 询问**
>
> 在当前的Python客户端中，`ask()`从错误的字段读取答案并返回空的`body`。
> 直接调用`POST /api/rag`，或者使用TypeScript客户端，直到修复。
