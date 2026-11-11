---
slug: "optimal/pricing"
title: "Pricing and usage"
description: "What an Optimal chat costs, how the cost strip and the Usage page report it, what happens when credits run out, and how harness brains are billed."
section: "Optimal"
order: 7008
---

An Optimal chat has two costs: the model and the machines it uses.
Both are paid from wallet credits, where 1 credit is 1 cent.

## The Optimal model

| | Price |
|---|---|
| Input | $1.90 per 1M tokens |
| Output | $8.00 per 1M tokens |
| A typical message | About 3 credits |

Billing is in whole credits.
The part of a credit left over from one call carries to the next, so small calls are not each rounded up.

Every chat has the full 1M-token context, with automatic compaction when it fills.
See [Chats and modes](/docs/optimal/chats#context-and-compaction).

The live price is also returned by the API in `GET /models`, under `pricing`.
See the [Optimal API](/docs/optimal/api#models).

## Machines

A sandbox or computer that Optimal creates or uses bills at its normal size price, while it runs.
A new sandbox that Optimal creates has a 30-minute idle timeout.
See [Pricing](/docs/pricing) for machine rates.

## Harnesses and saved agents

A harness or saved agent bills its model to your own account, never to MIOSA credits.
The cost strip says so, for example "Billed to your Claude plan".
The machine it runs on still bills in credits.

## The cost strip

The cost strip sits under the composer and updates while Optimal works.
For an Optimal chat it reads, for example:

```text
12 credits model · 7 credits machines · 19 credits so far · 4,180 left · Add credits
```

| Part | Meaning |
|---|---|
| model | Model calls settled for this chat |
| machines | Machines this chat ran, settled for this chat |
| so far | The two together |
| left | Your wallet balance, in credits |

A running machine settles every few minutes, so the machine figure catches up shortly after the work.
The (i) tooltip explains each figure.

## The Usage page

**Usage** in the Optimal sidebar shows Optimal's spend in the workspace:

- **Total cost**, **Tokens**, **Input**, and **Output**;
- your credit balance;
- **Model activity**: requests and cost for Optimal.

## When credits run out

Optimal needs a positive credit balance.
With no credits left, it does not start a chat or answer a message.
On the free plan, the chat shows "Optimal runs on credits" with **Upgrade** and **Add credits**.
Over the API, the call answers `402` with the code `OPTIMAL_CREDITS_REQUIRED`.

Add credits from **Billing**, or upgrade your plan, then send again.

> **Tip: One chat's cost from the API**
>
> `GET https://api.miosa.ai/api/v1/credits/session/{chat_id}` returns `model_credits`, `machine_credits`, and `total_credits` for one chat, read from the ledger.
> `attributed` is `false` when no machine has been tagged with the chat yet, so the total is the model only.

## Related

- [Pricing](/docs/pricing): every MIOSA price.
- [Brains and agents](/docs/optimal/brains): which brain bills where.
