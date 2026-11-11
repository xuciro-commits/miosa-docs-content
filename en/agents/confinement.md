---
slug: "agents/confinement"
title: "Agent confinement"
description: "An agent runs inside its machine and cannot use MIOSA's own platform key unless an Environment grants it a scoped one."
section: "Agents"
order: 6009
---

An agent runs inside its machine and stays there.
It cannot use MIOSA's own platform key to reach the rest of the platform.

## The platform key is blanked

Every sandbox boots with a platform key and its ids in the environment, so a plain command can call MIOSA.

Harness runs do not get it.
When MIOSA starts a harness, it blanks the platform key and the ids at launch, and unsets them in the run script.
So Claude Code, Codex, OSA, pi, OpenCode, Prime Agent, Kimi Code, Mistral Vibe, and Hermes Agent cannot borrow MIOSA's key.

## Grant a scoped key with an Environment

When you want an agent to reach MIOSA on purpose, an [Environment](/docs/cli/environments) can grant it a scoped machine key.
That key is bound to the machine, not to your account.

## What is kept

- A value the run itself supplies is kept.
- Environment grants are not touched.

## Related

- [Isolation and permissions](/docs/optimal/isolation): what a hosted chat can touch.
- [Runtime environment](/docs/guides/runtime-env): variables by scope and target.
