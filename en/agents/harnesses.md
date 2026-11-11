---
slug: "agents/harnesses"
title: "Harnesses and auth"
description: "The agents MIOSA can run on your machine, how each one authenticates, and how MIOSA gets each one ready."
section: "Agents"
order: 6008
---

A harness is a coding agent MIOSA runs on a machine, on your own account.
MIOSA never falls back to its own keys.

## The harnesses

| Harness | `runner` |
|---|---|
| Claude Code | `claude-code` |
| Codex | `codex` |
| OSA | `osa` |
| pi | `pi` |
| OpenCode | `opencode` |
| Prime Agent | `prime` |
| Kimi Code | `kimi` |
| Mistral Vibe | `mistral` |
| Hermes Agent | `hermes` |

There is also `custom` for a gateway model, which runs on MIOSA's AI Gateway.

## How MIOSA gets them ready

Runs are fast because a machine rarely installs anything at run time.

| Ready | Harnesses |
|---|---|
| Prebaked in the machine image | Claude Code, Codex, OSA |
| Installed on first use | pi, OpenCode, Prime Agent, Kimi Code, Mistral Vibe |
| Add-on, installed at create | Hermes Agent |

Hermes Agent is a large install, about 2.5 GB, so it is added at create and cached per host.

## Which account powers which harness

A Connection you make once is reused by every harness that accepts it.
Connect a ChatGPT plan once, and Codex, pi, OpenCode, Prime Agent, and Hermes Agent all use it.

| Harness | Plans | API keys |
|---|---|---|
| Claude Code | Claude plan | Anthropic, OpenRouter, DeepSeek; Amazon Bedrock |
| Codex | ChatGPT plan | OpenAI, DeepSeek; Amazon Bedrock |
| OSA | | Ollama Cloud; Anthropic, OpenAI, OpenRouter |
| pi | Claude plan, ChatGPT plan, Mistral Vibe key | Anthropic, OpenAI, OpenRouter, Mistral, LLM Gateway |
| OpenCode | ChatGPT plan, Mistral Vibe key | Anthropic, OpenAI, OpenRouter, Mistral, LLM Gateway |
| Prime Agent | Claude plan, ChatGPT plan, Mistral Vibe key | Anthropic, OpenAI, OpenRouter, Mistral, LLM Gateway |
| Kimi Code | Kimi Code plan | Moonshot |
| Mistral Vibe | Mistral Vibe key | Mistral |
| Hermes Agent | ChatGPT plan, Claude plan (Max with extra usage) | Anthropic, OpenAI, OpenRouter |

Two caveats to know:

- **OpenCode cannot use a Claude plan.** Anthropic does not allow it, and OpenCode removed the plugins that did it.
- **Hermes Agent on a Claude plan needs a Claude Max plan with extra usage credits.** A Claude Pro plan is refused.

## Hermes Agent

Hermes Agent is a first-class runner in the run API: send `runner: "hermes"` to `POST /api/v1/runs`.
It is not in the CLI `--harness` list yet, so use the API or the console until it is.

MIOSA runs the platform's pinned Hermes version, and never installs a newer one.
A Hermes run without an account returns `OWN_CREDENTIALS_REQUIRED`, the same as any other harness.

## The harness list as an API

`GET /api/v1/agents/harnesses` returns each harness with:

- `supported_auth`: the accounts that can power it, in the order the run path tries them.
  Each has `id`, `label`, `kind`, `vendor`, `injection`, `model`, `source`, and `note`.
- `also_supports`: what the agent's own CLI takes that MIOSA does not connect yet.
- `unsupported_auth`: what a person may expect to work but the vendor refuses, with `reason` and `source`.

Each model carries `served_by`, the account ids that can run it.

Refresh tokens never reach a machine.
The control plane is the single refresh owner, and hands the machine a short-lived access token that outlives the run.

## Related

- [Quickstart](/docs/quickstart): run an agent in a sandbox.
- [Agents, chats and runs](/docs/cli/agents): the CLI surface.
- [Chat with an agent on a machine](/docs/agents/chat-on-a-machine): pick a harness from the chat.
