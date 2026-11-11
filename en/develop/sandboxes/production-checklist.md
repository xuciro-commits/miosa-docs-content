---
slug: "develop/sandboxes/production-checklist"
title: "Production checklist"
description: "The defaults to set on sandboxes before relying on them in production: timeouts, persistence, idempotency, templates, secrets, and event subscription."
section: "Sandboxes"
order: 2012
---

Before relying on sandboxes in production:

- Set an explicit `timeout_sec`. Interactive app-building workflows should usually use `1h`; short one-shot commands can use less.
- Keep the default `persistent=True` for agent workspaces so timeout preserves `/workspace` and dependency installs.
- Set `idle_timeout_sec` for user-facing dev environments so abandoned sessions stop compute and preserve state.
- Pass `idempotency_key` on `sandboxes.create` calls in agent retry loops. The platform deduplicates creates with the same key within a 24-hour window, and the key also makes an uncertain placement outcome retryable: the API answers a retryable `503 SANDBOX_PLACEMENT_RECONCILING` instead of a terminal `409`.
- Use a published template instead of reinstalling the same dependencies for every sandbox.
- Fork before destructive operations when you need an independent branch of the current running state.
- Subscribe to `sbx.events.stream()` to detect unexpected exits and trigger retries.
- Never store long-lived secrets in `env`; pass them per-exec or use the Secrets API.
- Filter by `external_workspace_id` in your list calls to avoid scanning your entire tenant's sandbox set.

> **Note: Related**
>
> [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts) explains `timeout_sec` and usage measurement, [Sizing and limits](/docs/develop/sandboxes/sizing) explains `persistent` and `always_on`, and [Sandbox events](/docs/develop/sandboxes/events) covers the event stream.
