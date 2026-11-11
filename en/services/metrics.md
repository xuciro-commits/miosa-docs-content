---
slug: "services/metrics"
title: "Service metrics"
description: "Requests, errors and latency per service, and what the series does not measure."
section: "Services & projects"
order: 1017
---

Every service reports its own request series: requests, errors, error rate, and
response latency at p50 and p95.

```bash
miosa service metrics lumen API
miosa service metrics lumen API --range 24h --json
```

The same series is at `GET /api/v1/projects/:id/services/:sid/metrics/series?range=`
with `15m`, `1h`, `6h`, `24h` or `7d`, scope `projects:read`.

## Reading it honestly

The server buckets the series and reports `step_seconds`: 60 for `15m` and `1h`,
300 for `6h`, 900 for `24h`, and 3600 for `7d`. The full timeline comes back even
in a quiet window, with null fields where nothing happened; a point is empty when
`requests` is null, and the CLI prints a dash rather than a zero.

`available` names what is actually measured. For a service that is `requests` and
`latency`; **cpu, memory and network are not collected for services**, and the CLI
says so under the table instead of printing empty columns.

> **Note: Web vitals**
>
> Real-user web vitals (LCP, INP, FCP, TTFB, CLS) are collected for static and web
> services that include the beacon, and come back under `web_vitals` with
> `available.web_vitals`. They are reported separately because they measure the
> browser, not the server.
