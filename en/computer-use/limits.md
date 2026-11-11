---
slug: "computer-use/limits"
title: "Computer use limits"
description: "Latency, coordinate space, sizing, authentication, and rate limits for MIOSA Computer desktop control."
section: "Computers"
order: 3011
---

## Latency

Desktop methods run synchronously over authenticated HTTP RPC. Typical round-trip from the MIOSA edge:

| Method type | Typical latency |
|---|---|
| Click, key, scroll, type | 30-80 ms |
| Screenshot | 100-300 ms (compression-bound) |
| Window list | 50-100 ms |
| `bash()` / `python()` | 50 ms plus command execution time |

For high-frequency loops, prefer `bash()` for multi-step operations over many individual GUI calls.
`GET /desktop/window-state` returns the screenshot and the accessibility tree together, so perception costs one round trip instead of two.

## Coordinate space

Coordinates are screen pixels with the origin at the top-left, matching what `GET /desktop/screen-size` and `GET /desktop/cursor` report.
A vision model may be shown a scaled-down screenshot; map its output back to screen pixels before sending an action.

## Screenshots

Screenshots are PNG.
A full desktop image is typically a few hundred kilobytes; regions are smaller.
Use `/desktop/screenshot/region` when only part of the screen matters, and `window-state` when you need pixels and structure at once.

## Sizing

| Size | vCPU | RAM | Disk | Typical use |
|---|---|---|---|---|
| `xs` | 1 | 2 GB | 10 GB | Lightweight headless tasks |
| `small` | 2 | 4 GB | 20 GB | **Default** for a single agent and basic browsing |
| `medium` | 4 | 8 GB | 40 GB | Heavier desktop apps, faster page loads |
| `large` | 8 | 16 GB | 80 GB | Multi-app, large-context workloads |
| `xl` | 16 | 32 GB | 160 GB | Intensive parallelism |

GPU support for accelerated desktop and GPU-aware workflows is plan-dependent.

## Lifecycle

create creates provisioning; provisioning goes to running on ready; running goes to stopped on stop, and back to running on start.

running goes to recovering on host maintenance; recovering goes to running on restored.

recovering goes to error on recovery fails.

running or stopped goes to deleted on delete.

Stopping a Computer pauses compute and preserves the filesystem and browser profile.
Restarting resumes from that state.
Desktop endpoints only accept requests while the Computer is `running`; otherwise they return `409 COMPUTER_NOT_RUNNING`.

## Authentication

Desktop methods require the same workspace API key as every other MIOSA resource.
Call them server-side.
For browser-side control, mint a scoped token instead of shipping the API key - see [Embedding & streaming](/docs/computers/embedding).

> **Caution: Never expose the API key**
>
> Mint stream URLs and scoped tokens on your backend.
> Send only the short-lived URL or token to the browser.

## Rate limits

Rate limits vary by key, tenant policy, and endpoint family.
Honor `X-RateLimit-*` and `Retry-After` response headers.
Individual desktop actions are fast, but a tight perceive-act loop still issues many requests; batch where the work does not need pixels.

## Errors

| Status | Code | Cause |
|---|---|---|
| `409` | `COMPUTER_NOT_RUNNING` | The Computer is stopped, provisioning, or recovering |
| `502` | `AGENT_UNAVAILABLE` | The agent inside the Computer did not answer |
| `429` | - | Rate limited |
