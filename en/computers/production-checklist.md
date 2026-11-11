---
slug: "computers/production-checklist"
title: "Production checklist"
description: "The defaults to verify before relying on a Computer in production: workspace attribution, access, secrets, state, observability, and recovery."
section: "Computers"
order: 3006
---

Before you rely on a Computer in production, verify these points.

| Area | Check |
|---|---|
| Workspace attribution | Set `external_workspace_id`, `external_user_id`, or workspace/project fields when building a white-label app |
| Access | Use platform auth or stream tickets for normal users |
| Secrets | Put API keys in backend code or encrypted env, not inside frontend bundles |
| State | Use stop/start for durable sessions and delete only when state should be removed |
| Observability | Log the Computer ID, external IDs, and task/run ID together |
| Recovery | Design the app so a user can reopen the same running Computer after refreshing the browser |

> **Note: Related**
>
> [Lifecycle and sizing](/docs/computers/lifecycle) covers the states and sizes these checks assume, and [Embedding & streaming](/docs/computers/embedding) covers the access model and stream tickets.
