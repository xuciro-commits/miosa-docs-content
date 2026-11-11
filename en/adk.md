---
slug: "adk"
title: "Agent Development Kit (ADK)"
description: "Build LLM agents that use MIOSA sandboxes, desktops, and deploys with the tool-call loop already wired. TypeScript ADK first; Python next."
section: "Agents"
order: 6012
---

The MIOSA ADK lets you build production-ready LLM agents that natively use
MIOSA primitives  -  sandboxes, full desktop environments, and one-click
deploys  -  with the tool-call loop already wired.

If `@miosa/sdk` is "call MIOSA APIs from code", `@miosa/adk` is **"build an
AGENT that uses MIOSA"**.

```bash
npm install @miosa/adk @anthropic-ai/sdk
```

---

## 30-second example

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

That's it. The agent boots a sandbox, writes the file, runs it, prints
output, destroys the sandbox, returns the final response. No tool-loop
boilerplate.

---

## ADK vs SDK

- **@miosa/sdk**: The raw TypeScript SDK around MIOSA's REST API. You call methods:

  ```typescript
  const computer = await client.computers.create({ name: "x" });
  await computer.exec.bash("echo hello");
  ```

  Use this when you're **writing application code** that talks to MIOSA.
- **@miosa/adk (this)**: Adds the agent loop on top. You give it a provider + prompt:

  ```typescript
  const agent = new Agent({ provider, miosaApiKey });
  await agent.run({ prompt: "build a flask app and run it" });
  ```

  Use this when you're **building an agent** that should use MIOSA.

---

## Built-in MIOSA tools

When you pass `miosaApiKey`, the agent gets these tools out of the box:

| Tool | Does |
|------|------|
| `create_sandbox` | Boot a Debian 12 microVM (Python + Node). |
| `create_desktop` | Start a full desktop environment. |
| `list_sandboxes` | List your active computers. |
| `get_sandbox` | Fetch one sandbox by id (status / IP). |
| `destroy_sandbox` | Delete a computer. |
| `exec` | Run a bash command. |
| `exec_python` | Run a Python snippet inline. |
| `read_file` | Read a file from the VM. |
| `write_file` | Write a file inside the VM. |
| `list_files` | List a directory inside the VM. |

Get an API key at <https://miosa.ai/dashboard/api-keys>, or run
`npx -y @miosa/cli mcp install` to wire one via browser-based auth.

---

## Custom tools

Compose your own tool catalogue alongside (or instead of) the MIOSA tools:

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

## Streaming step events

Drive a live UI as the agent runs:

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

The `AgentStep` shape:

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

## Providers

`@miosa/adk` ships with native tool-call providers for hosted, marketplace, and
local OpenAI-compatible models, plus Anthropic and Gemini-specific adapters. The
same provider interface also works for custom internal routers.

**Anthropic**

```typescript
import { anthropicProvider } from "@miosa/adk/providers/anthropic";

const provider = anthropicProvider({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: "claude-opus-4-7", // default
  maxTokens: 4096, // default
});
```

Uses `@anthropic-ai/sdk` Messages API with native tool-use. Supports any
Claude model that supports tool calling.

**Surplus**

```typescript
import { surplusProvider } from "@miosa/adk/providers/surplus";

const provider = surplusProvider({
  apiKey: process.env.SURPLUS_API_KEY!,
  model: "claude-opus-4.8",
});
```

Surplus uses the OpenAI-compatible chat completions contract. In platform
runtime profiles, prefer routing through MIOSA's Intelligence API/model route so
keys can be metered, audited, revoked, and used as fallback fuel for Claude Code,
Codex, Hermes, OSA, or custom harnesses inside MIOSA sandboxes.

**MIOSA route**

```typescript
import { miosaRouteProvider } from "@miosa/adk/providers/miosa";

const provider = miosaRouteProvider({
  miosaApiKey: process.env.MIOSA_API_KEY!,
  model: "optimal",
});
```

Use this when the agent should call MIOSA's Intelligence API rather than a
third-party provider directly. As model routes ship, the model can become a
route id such as `route:cheap-claude-code`; MIOSA then owns provider selection,
fallbacks, metering, and secret handling.

**OpenAI-compatible**

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

The bundled OpenAI-compatible provider covers OpenAI, Groq, DeepSeek,
OpenRouter, Together, Fireworks, Mistral, Cerebras, Perplexity, xAI, Surplus,
MIOSA Intelligence, Ollama, LM Studio, and custom routers.

**Custom provider**

Implement the `Provider` interface directly when an endpoint needs a custom tool
or streaming protocol.

---

## Termination

`agent.run()` returns when:

- The model emits an `end_turn` with no further tool calls (normal completion)
- A `stop_sequence` is hit
- `maxIterations` is reached (default 10)  -  safety guard against runaway loops
- The provider throws (returned as `stopReason: "error"`)

```typescript
const result = await agent.run({ prompt: "...", maxIterations: 20 });
if (result.stopReason === "max_iterations") {
  console.warn("Agent didn't finish in 20 turns");
}
```

---

## Quick comparison

| | `@miosa/adk` | Anthropic Claude Agent SDK | OpenAI Agents SDK | Google ADK |
|--|--|--|--|--|
| Tool-call loop | ✓ | ✓ | ✓ | ✓ |
| MIOSA tools built-in | **✓** |  -  |  -  |  -  |
| Anthropic provider | ✓ | ✓ |  -  |  -  |
| OpenAI-compatible providers | ✓ |  -  | ✓ |  -  |
| Sub-agents | (roadmap) | ✓ | ✓ | ✓ |
| Memory | (roadmap) | ✓ | (basic) | ✓ |
| Cost | free | free | free | free |

The ADK is intentionally small. If you want MIOSA-as-execution-substrate
with a clean agent loop, you're in the right place.

---

## What's coming next

- **Python ADK** (`pip install miosa-adk`)  -  same surface, Python ergonomics.
- **Managed model routes**  -  route ADK and in-sandbox harness calls through MIOSA Intelligence API.
- **Sub-agents**  -  spawn child agents with scoped tool subsets.
- **Memory**  -  built-in conversation memory with TTL + summarization.
- **Streaming output**  -  token-by-token via provider.streamStep.

Open an issue at <https://github.com/Miosa-osa/miosa/issues> if there's
something you need sooner.
