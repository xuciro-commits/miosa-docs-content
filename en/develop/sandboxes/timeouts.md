---
slug: "develop/sandboxes/timeouts"
title: "Timeouts and auto-stop"
description: "The sandbox active-session timeout, how to extend it, and how runtime, CPU, and network usage are measured and reported."
section: "Sandboxes"
order: 2007
---

`timeout_sec` is the idle-anchored timeout: it counts from the last activity.
It defaults to 3600 seconds and accepts values from 1 through 86400.
`POST /api/v1/sandboxes/{id}/extend` replaces that timeout; it does not add seconds to the previous value.
An omitted extend body preserves the current timeout.
Set `always_on: true` only when policy permits a sandbox to run until an explicit lifecycle action.

## Auto-stop semantics

`timeout_sec` counts from the last activity, not from when the session started.
Exec calls, file operations, preview HTTP traffic, and terminal stdin all reset it.
When it expires:

- A **persistent** sandbox (`persistent: true`, the default) is stopped and moves to `stopped`. The disk is preserved, so resume brings it back with `/workspace` and installed dependencies intact. Running processes do not survive a stop. Billing drops from the compute rate to the cold storage rate.
- A **non-persistent** sandbox (`persistent: false`) is destroyed, and the disk is discarded.

SOMA sandboxes do not support persistence yet, so a SOMA sandbox is disposable and is deleted when its timeout expires.

## Idle auto-stop and wake on request

`idle_timeout_sec` stops a persistent sandbox after it has been idle.

New accounts (created on or after October 11, 2026): sandboxes without `idle_timeout_sec` stop after 30 minutes idle and wake on the next request. Set `idle_timeout_sec` to `0` to disable. Existing accounts keep their current behavior (idle timeout off unless set).

A stopped persistent sandbox **wakes on request**: a request to its preview URL resumes it and is held for up to 10 seconds. If the sandbox is not running by then, the request returns `503` with `Retry-After` while the resume finishes in the background, so a retry finds the sandbox running. Exec, file operations, and deploys on a stopped or paused persistent sandbox resume it first and then run. A non-persistent sandbox is never auto-resumed: the call returns `409`. Authorization is checked first, so a private preview never wakes for anonymous traffic, and a request for a port that is not exposed does not wake the sandbox.

> **Note: Extend, do not accumulate**
>
> `POST /api/v1/sandboxes/{id}/extend` replaces the timeout. It does not add seconds to the previous value, so compute the new absolute TTL when you extend. Omitting the body preserves the current timeout. See [Sandbox lifecycle](/docs/develop/sandboxes/lifecycle) for the stop transition this triggers.

## Sandbox usage

`GET /api/v1/sandboxes/{id}/usage` returns `runtime_sec`, `provisioned_vcpu_ms`, `active_cpu_ms`, network ingress and egress bytes, estimated cost, and timeout visibility.
`runtime_sec` and `provisioned_vcpu_ms` are allocation measurements.
Use `measurement_status` before interpreting active CPU or network values:

| Status | Meaning | Client behavior |
|---|---|---|
| `measured` | The value was measured for this sandbox. | Display and aggregate it normally. |
| `unavailable` | No trustworthy measurement is available. The related value is `null`. | Show "Unavailable"; never coerce it to zero. |
| `stale` | The value is the latest retained sample but is not current. | Label it stale and avoid presenting it as real-time usage. |

Provisioned resources are always reported with `measurement_status.provisioned_resources: "measured"`.
`timeout_remaining_ms` can be `null` before start or when timeout enforcement is disabled.

> **Note: Related**
>
> [Sandbox lifecycle](/docs/develop/sandboxes/lifecycle) covers what a persistent sandbox becomes when the timeout elapses, and [Sizing and limits](/docs/develop/sandboxes/sizing) covers the `always_on`, `persistent`, and `idle_timeout_sec` fields.
