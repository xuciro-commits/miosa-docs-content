---
slug: "optimal/brains"
title: "Brains and agents"
description: "Choose who answers an Optimal chat - the built-in Optimal model, a coding harness on your own account, or a saved agent you built in Agent Studio."
section: "Optimal"
order: 7003
---

The brain is who answers a chat.
Pick it in the composer's brain picker (tooltip "Choose who answers") before the first message.

## The three kinds of brain

| Brain | Billed to | Where it runs | Best for |
|---|---|---|---|
| **Optimal** | Wallet credits | MIOSA, working on your machines through scoped tools | Questions about your workspace, building, investigating |
| **A harness** | Your own model account | On the sandbox or computer you pick | Coding with Claude Code, Codex, OSA, and others |
| **A saved agent** | Your own model account | On the sandbox or computer you pick | An agent with your own instructions, tools, and permissions |

## Optimal

Optimal is the model built into MIOSA.
The picker shows it as "Built into MIOSA" and "Uses your wallet credits", with the live price under it.

- It is one model, with a 1M-token context.
- You cannot pick a different model for it, and it does not use your own model keys.
- It is the only brain that offers Investigate mode.

On the free plan with no credits left, Optimal is disabled in the picker with "Add credits to use Optimal".

## Harnesses

A harness is a coding agent that runs on a machine on your own account.
MIOSA credits never pay for its model.
The picker lists the harnesses under **Your agents**:

- Claude Code, Codex, and OSA;
- OpenCode, pi, Prime Agent, Kimi Code, Mistral Vibe, and Hermes, when your organization can connect them.

Each row shows its connection state: "Connected" with the account, "Not connected", or "Needs signing in again".
**Connect** or **Reconnect** opens the sign-in for that harness, then returns you to Optimal.
See [Harnesses and auth](/docs/agents/harnesses) for what each harness accepts.

A harness always asks where to work, in any mode, because each message is a run on a machine.
The cost strip says the work is billed to your own plan.

## Saved agents

A saved agent is an agent you built in Agent Studio, with its own instructions, harness, model, tools, and permissions.
Saved agents appear under **Your agents** next to the harness they use.
An archived agent is not listed.

To build one, choose **Build an agent** at the bottom of the brain picker, or open [Agent Studio](/docs/agents/agent-studio).
From the CLI:

```bash
miosa agent new reviewer --harness claude-code --instructions-file review.md
miosa agent list
```

## Switching brains

The brain is fixed after the first message.
The picker's tooltip then reads "This chat keeps its brain. Start a new chat to change it."
Picking another brain in a started chat starts a new chat.

> **Note: Your keys stay yours**
>
> Harnesses and saved agents run on your own model account.
> Optimal's model is MIOSA's, and is paid only in wallet credits.

## Related

- [Chat with an agent on a machine](/docs/agents/chat-on-a-machine): the same brains from a computer or sandbox page.
- [Agent confinement](/docs/agents/confinement): what an agent can reach from inside its machine.
- [Pricing and usage](/docs/optimal/pricing): how each brain is billed.
