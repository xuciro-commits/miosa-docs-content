---
slug: "faq/sandboxes-and-computers"
title: "Sandboxes and computers"
description: "Sandbox vs Computer, the default sandbox, run length, states, pausing, and snapshots."
section: "Help"
order: 16002
---

## Sandbox or Computer?

Use a [Sandbox](/docs/develop/sandboxes) for code, files, and processes.
Use a [Computer](/docs/computers/overview) when you need a visible desktop, a browser login, or human takeover.
Use **Docker Host** when you need a durable, always-on URL.

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa list                     # see all your sandboxes
```

## What do I get in a default sandbox?

A headless microVM that is persistent by default, with `/workspace` as the editable root, booted from a template (usually `miosa-sandbox`).
The default size `small` is 2 vCPU, 4096 MiB RAM, and 10240 MiB disk, and responses carry a versioned `resource_contract`.
See [Sandboxes](/docs/develop/sandboxes) and [Sizing and limits](/docs/develop/sandboxes/sizing).

## What is different about a Computer?

A Computer is a durable cloud Linux desktop (Xfce and Firefox) with screenshot, click, type, and streaming, plus human takeover.
You embed or share it through `computer.urls()["desktop_url"]`.
See [Computers](/docs/computers/overview) and [Embed a live desktop](/docs/computers/embedding).

## How long can a sandbox run?

`timeout_sec` is 1 to 86400 seconds, default 3600.
`always_on` disables timeout enforcement subject to tenant policy.
The timeout counts from the last activity, and expiry stops a persistent sandbox and destroys a non-persistent one.
A sandbox also stops after 30 minutes idle by default on accounts created on or after October 11, 2026 (set `idle_timeout_sec` to `0` to disable), and a stopped sandbox wakes on the next request.
See [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts).

## What are the sandbox states?

`creating`, `running`, `stopped`, `paused`, `error`, and `destroyed`.
`paused` exists only on a runtime that advertises pause, which is Firecracker today; SOMA has no pause.
See [Sandbox lifecycle](/docs/develop/sandboxes/lifecycle).

## Does stopping a sandbox lose my files?

No. Stop keeps the sandbox id and the disk, including files, installed packages, and enabled services; resume brings it back.
Stop never keeps running processes.
Pause (Firecracker only) keeps processes for up to 30 minutes, then behaves like a stop.
Persistent means the sandbox survives a stop, not the loss of its host; the `durability` field says how many hosts hold the saved state.
SOMA does not support persistence, pause, or snapshots yet.
Destroy is what removes saved state.
See [Snapshots and persistence](/docs/develop/snapshots).

## What is the difference between a snapshot and a fork?

Restore builds a new sandbox from a checkpoint; fork branches a running sandbox into an independent one.
Both leave the source unchanged.
See [Snapshots and persistence](/docs/develop/snapshots).

```bash
miosa snapshot save my-box before-upgrade   # pin a named restore point
miosa snapshot fork before-upgrade          # branch it into an independent sandbox
```

## Does a snapshot save my database?

No. A snapshot captures the sandbox disk and, where the runtime advertises it, memory only.
External databases, object storage, connectors, and third-party services are not rolled back with it.
Keep durable state in a data service.
See [Data overview](/docs/data/overview).
