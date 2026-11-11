---
slug: "optimal/isolation"
title: "Isolation and permissions"
description: "Optimal acts only inside your organization and workspace, with your permissions, and never on MIOSA's own keys."
section: "Optimal"
order: 7009
---

Optimal acts only inside your organization and workspace, with your permissions.
It never reaches anything you could not reach yourself.

## Scoped per chat

Every key Optimal holds is scoped to one chat:

- a machine key bound to one sandbox or computer; or
- a workspace key bound to your organization, workspace, and user.

Both are minted per chat from the chat's saved scope, never from a message or a tool argument.
They are short-lived, and are never logged or saved in a tool result.

## Every call re-checks the scope

Each tool call re-derives your organization, workspace, and user from the chat.
An id in a tool argument is ignored.
A foreign organization, a sibling workspace, and an unknown id all get the same answer: not found, or no access.

## Nothing runs on MIOSA's host

For a hosted chat, no shell or file tool runs on the Optimal host.
Those tools run only on the machine the chat is bound to.
Tool output is cleaned before the model or the stream sees it.

## Dispatched agents get none of Optimal's authority

When Optimal dispatches a harness run, the run request carries no key, token, or scope.
The harness runs on your own account, and Optimal's own reasoning bills wallet credits.

## Related

- [Approvals](/docs/optimal/approvals): what Optimal asks before it acts.
- [Optimal API](/docs/optimal/api#authentication): the scoped Optimal token.
- [Agent confinement](/docs/agents/confinement): what an agent can reach from inside its machine.
- [Optimal](/docs/optimal): the product overview.
