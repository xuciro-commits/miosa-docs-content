---
slug: "services/middleware"
title: "Edge middleware"
description: "Ordered path rules and an optional function that run at the edge, in front of a server service."
section: "Networking & domains"
order: 5004
---

Edge middleware runs in front of a server service, before the request reaches
your process: redirects, rewrites, header changes, and optionally a published
function that can answer, continue or rewrite the request.

It is a structured rule set, so it is written as a file rather than a pile of
flags:

```bash
miosa middleware shop API --from middleware.json
miosa middleware shop API           # what is configured now
miosa middleware shop API --off     # remove the rules and the function
```

```json
{
  "rules": [
    { "path": "/old/*", "redirect": { "to": "/new", "status": 308 } },
    { "path": "/api/*", "request_headers": { "set": { "x-plan": "pro" } } }
  ],
  "function": { "slug": "gate", "paths": ["/app/*"], "timeout_ms": 250, "on_error": "closed" }
}
```

Rules run in order and there can be up to fifty of them. A `path` is either an
exact path or a prefix ending in `/*`, where `$1` is the rest of the path. Each
rule carries exactly one of a `redirect` (with an optional `301`, `302`, `303`,
`307` or `308`, default `307`), a `rewrite`, and/or header changes.

The function answers `x-miosa-middleware: next` to let the request continue, and
may add `x-miosa-middleware-request-<name>` or `x-miosa-middleware-rewrite`. Any
other answer is returned to the client. With `on_error: "closed"`, a failing
function answers `503 edge_middleware_unavailable`; with `open`, the request
continues.

Protected header names are refused, including `host`, `content-length`,
`transfer-encoding`, `connection`, `upgrade`, `x-internal-*`, `x-miosa-*` and
`x-forwarded-*`.

> **Note: It applies in seconds**
>
> A change applies within about five seconds at every edge node. Validation errors
> name the field (`settings.edge_middleware.<path>`), so a typo in a rule is easy to
> find.
