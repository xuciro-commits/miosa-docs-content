---
slug: "adk"
title: "智能体开发套件 (ADK)"
description: "构建使用 MIOSA 沙箱、桌面的 LLM 智能体，并使用已连接的工具调用循环进行部署。首先是 TypeScript ADK；接下来是Python。"
section: "Agents"
order: 6012
---

MIOSA ADK 可让您构建本机使用的生产就绪型 LLM 智能体
MIOSA 原语 - 沙箱、完整桌面环境和一键式
部署 - 工具调用循环已经连接。

如果 `@miosa/sdk` 是“从代码调用 MIOSA API”，`@miosa/adk` 是 **“构建一个
使用 MIOSA 的智能体“**。

```bash
npm install @miosa/adk @anthropic-ai/sdk
```

---

## 30 秒示例

```typescript
import { Agent, anthropicProvider } from "@miosa/adk";

const agent = new Agent({
  provider: anthropicProvider({
    apiKey: process.env.ANTHROPIC_API_KEY!,
    model: "claude-opus-4-7",
  }),
  miosaApiKey: process.env.MIOSA_API_KEY!,
});

const result = await agent.run({
  prompt:
    "Create a MIOSA sandbox, write a Python hello-world, run it, then destroy the sandbox.",
  maxIterations: 10,
});

console.log(result.finalText);
```

就是这样。智能体启动沙箱，写入文件，运行它，打印
输出，破坏沙箱，返回最终响应。无工具循环
样板文件。

---

## ADK 与 SDK

- **@miosa/sdk**：围绕 MIOSA 的 REST API 的原始 TypeScript SDK。您调用方法：

  ```typescript
  const computer = await client.computers.create({ name: "x" });
  await computer.exec.bash("echo hello");
  ```

  当您**编写与 MIOSA 通信的应用程序代码**时，请使用此选项。
- **@miosa/adk (this)**：在顶部添加智能体循环。你给它一个提供者+提示：

  ```typescript
  const agent = new Agent({ provider, miosaApiKey });
  await agent.run({ prompt: "build a flask app and run it" });
  ```

  当您**构建应使用 MIOSA 的智能体**时，请使用此选项。

---

## 内置 MIOSA 工具

当您通过 `miosaApiKey` 时，智能体会开箱即用这些工具：

|工具|是否 |
|------|------|
| `create_sandbox` |启动 Debian 12 microVM (Python + Node)。 |
| `create_desktop` |启动完整的桌面环境。 |
| `list_sandboxes` |列出您的活动计算机。 |
| `get_sandbox` |通过 id（状态/IP）获取一个沙箱。 |
| `destroy_sandbox` |删除一台计算机。 |
| `exec` |运行 bash 命令。 |
| `exec_python` |内联运行 Python 代码片段。 |
| `read_file` |从虚拟机读取文件。 |
| `write_file` |在虚拟机内写入一个文件。 |
| `list_files` |列出虚拟机内的目录。 |

在 <https://miosa.ai/dashboard/api-keys> 获取 API 密钥，或运行
`npx -y @miosa/cli mcp install` 通过基于浏览器的身份验证连接。

---

## 自定义工具

与 MIOSA 工具一起（或代替）编写您自己的工具目录：

```typescript
import {
  Agent,
  anthropicProvider,
  miosaTools,
  type Tool,
} from "@miosa/adk";
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sendSlack: Tool = {
  name: "send_slack",
  description: "Post a message to the #builds Slack channel.",
  inputSchema: {
    type: "object",
    properties: { text: { type: "string" } },
    required: ["text"],
  },
  async execute(args) {
    const text = String(args["text"] ?? "");
    // your Slack call here
    return `Sent: ${text}`;
  },
};

const agent = new Agent({
  provider: anthropicProvider({ apiKey: process.env.ANTHROPIC_API_KEY! }),
  tools: [...miosaTools({ client: miosa }), sendSlack],
});
```

---

## 流式传输步骤事件

当智能体运行时驱动实时 UI：

```typescript
const result = await agent.run({
  prompt: "...",
  onStep: (step) => {
    console.log(`[turn ${step.index}] ${step.text}`);
    for (const call of step.toolCalls) {
      console.log(
        `  → ${call.name}(${JSON.stringify(call.input)}) → ${call.output.slice(0, 80)}`,
      );
    }
  },
});
```

`AgentStep` 形状：

```typescript
interface AgentStep {
  index: number;
  text: string;
  toolCalls: Array<{
    id: string;
    name: string;
    input: Record<string, unknown>;
    output: string;
    isError: boolean;
  }>;
  usage?: { inputTokens: number; outputTokens: number };
}
```

---

## 提供商

`@miosa/adk` 附带用于托管、市场和
本地 OpenAI 兼容模型，以及 Anthropic 和 Gemini 特定适配器。的
相同的提供商接口也适用于自定义内部路由器。

**人择**

```typescript
import { anthropicProvider } from "@miosa/adk/providers/anthropic";

const provider = anthropicProvider({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: "claude-opus-4-7", // default
  maxTokens: 4096, // default
});
```

将 `@anthropic-ai/sdk` 消息 API 与本机工具结合使用。支持任何
支持工具调用的Claude模型。

**盈余**

```typescript
import { surplusProvider } from "@miosa/adk/providers/surplus";

const provider = surplusProvider({
  apiKey: process.env.SURPLUS_API_KEY!,
  model: "claude-opus-4.8",
});
```

Surplus 使用与 OpenAI 兼容的聊天完成合约。平台内
运行时配置文件，更喜欢通过 MIOSA 的智能 API/模型路由进行路由，因此
密钥可以被计量、审计、撤销，并用作克劳德代码的后备燃料，
Codex、Hermes、OSA 或 MIOSA 沙箱内的自定义运行框架。

**MIOSA路线**

```typescript
import { miosaRouteProvider } from "@miosa/adk/providers/miosa";

const provider = miosaRouteProvider({
  miosaApiKey: process.env.MIOSA_API_KEY!,
  model: "optimal",
});
```

当智能体应该调用 MIOSA 的 Intelligence API 而不是
直接第三方提供商。随着模型路线的发布，模型可以成为
路线ID，例如`route:cheap-claude-code`； MIOSA 然后拥有提供商选择权，
后备、计量和秘密处理。

**兼容 OpenAI**

```typescript
import { openAIProvider, groqProvider, openAICompatibleProvider } from "@miosa/adk";

const openai = openAIProvider({
  apiKey: process.env.OPENAI_API_KEY!,
  model: "gpt-5",
});

const groq = groqProvider({
  apiKey: process.env.GROQ_API_KEY!,
  model: "moonshotai/kimi-k2-instruct-0905",
});

const internal = openAICompatibleProvider({
  name: "internal-router",
  baseUrl: "https://llm.example.com/v1/chat/completions",
  apiKey: process.env.INTERNAL_LLM_KEY,
  model: "best-agent-model",
});
```

捆绑的 OpenAI 兼容提供商涵盖 OpenAI、Groq、DeepSeek、
OpenRouter、Together、Fireworks、Mistral、Cerebras、Perplexity、xAI、Surplus、
MIOSA Intelligence、Ollama、LM Studio 和自定义路由器。

**定制提供商**

当端点需要自定义工具时，直接实现`Provider`接口
或流协议。

---

## 终止

`agent.run()` 在以下情况下返回：

- 模型发出 `end_turn` ，没有进一步的工具调用（正常完成）
- `stop_sequence` 被击中
- 达到 `maxIterations`（默认 10） - 防止失控循环的安全防护
- 提供者抛出（返回为 `stopReason: "error"`）

```typescript
const result = await agent.run({ prompt: "...", maxIterations: 20 });
if (result.stopReason === "max_iterations") {
  console.warn("Agent didn't finish in 20 turns");
}
```

---

## 快速比较

| | `@miosa/adk` |人类克劳德智能体SDK | OpenAI 智能体 SDK |谷歌 ADK |
|--|--|--|--|--|
|工具调用循环| ✓ | ✓ | ✓ | ✓ |
|内置 MIOSA 工具 | **✓** |  - |  - |  - |
|人类提供者| ✓ | ✓ |  - |  - |
| OpenAI 兼容提供商 | ✓ |  - | ✓ |  - |
|分智能体| （路线图）| ✓ | ✓ | ✓ |
|内存| （路线图）| ✓ | （基本）| ✓ |
|成本|免费|免费|免费|免费|

ADK 故意做得很小。如果你想要 MIOSA 作为执行底层
有了干净的智能体循环，您就来对地方了。

---

## 接下来会发生什么- **Python ADK** (`pip install miosa-adk`) - 相同的表面，Python 人体工程学。
- **托管模型路由** - 通过 MIOSA Intelligence API 路由 ADK 和沙箱内运行框架调用。
- **子智能体** - 使用限定范围的工具子集生成子智能体。
- **内存** - 内置对话内存，带 TTL + 摘要。
- **流式输出** - 通过provider.streamStep逐个令牌。

如果存在问题，请在 <https://github.com/Miosa-osa/miosa/issues> 上提出问题
您需要的东西更快。
