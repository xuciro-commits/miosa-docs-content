---
slug: "integrations/webhooks"
title: "Webhooks"
description: "Subscribe to sandbox and deployment lifecycle events, verify HMAC-SHA256 signatures, handle retries, and test locally."
section: "Platform builders"
order: 9012
---

MIOSA sends HTTP `POST` requests to your endpoint whenever a sandbox or deployment changes state. Use webhooks to update your database, notify users, trigger downstream actions, or bill for compute.

---

## Event taxonomy

### Sandbox events

| Event | Fires when |
|---|---|
| `sandbox.created` | The sandbox row is inserted. The VM may still be booting. |
| `sandbox.running` | The sandbox is fully booted and ready. `external_user_id` is available. |
| `sandbox.paused` | The sandbox was paused due to an idle timeout or an explicit API call. |
| `sandbox.resumed` | A paused sandbox was resumed. |
| `sandbox.destroyed` | The sandbox was permanently deleted. |
| `sandbox.error` | The sandbox failed to boot or encountered a runtime error. |

### Deployment events

| Event | Fires when |
|---|---|
| `deployment.created` | A new deployment was created. |
| `deployment.build_started` | A build started for a new version. |
| `deployment.build_succeeded` | A build finished successfully. |
| `deployment.build_failed` | A build failed. The `error` field contains the failure reason. |
| `deployment.published` | A version was promoted to the active release. |
| `deployment.domain_attached` | A custom domain was verified and attached. |

### Forge events

Every Forge event names its repository in the payload (`repository.id`, `repository.full_name`, `repository.owner`, `repository.slug`), so one organization subscription can route events from every repository. Subscriptions cannot be limited to a single repository.

| Event | Fires when |
|---|---|
| `forge.push` | A push to a repository was accepted. |
| `forge.pull_request` | A pull request was opened, closed, reopened, merged or reviewed. |
| `forge.check` | A check reported on a pull request or a commit. The payload includes `commit_sha`. |

---

## Payload structure

Every webhook delivery has the same envelope:

```json
{
  "id": "evt_01hzqmrkntq6g9gxnqhvpa8c7t",
  "type": "sandbox.running",
  "created_at": "2026-05-25T14:32:10Z",
  "data": {
    "id": "sbx_01hzq5pnmpgt6vdwp0r8d23c5n",
    "state": "running",
    "external_user_id": "user_42",
    "external_project_id": "proj_lumen_1",
    "metadata": {
      "customer_id": "user_42",
      "project_name": "brand-refresh",
      "source": "lumen-whitelabel"
    },
    "preview_url": "https://3000-sbx01hzq.sandbox.example.com"
  }
}
```

The `id` field is a unique event ID. Use it to deduplicate deliveries.

---

## HMAC-SHA256 signature verification

Every delivery includes a `Miosa-Signature` header:

```yaml
Miosa-Signature: t=1716645130,v1=a3f2c8...
```

- `t`  -  Unix timestamp (seconds) when MIOSA signed the request
- `v1`  -  HMAC-SHA256 hex digest of `"<t>.<raw_body>"`

The signed payload is `timestamp + "." + raw request body bytes`. Verify before parsing the JSON.

> **Caution: Replay protection**
>
> Reject events where the timestamp `t` is more than 300 seconds (5 minutes) in the past or future. This prevents replay attacks using captured valid signatures.

**Python**

```python
import hashlib
import hmac
import time

def verify_miosa_signature(body: bytes, header: str, secret: str) -> bool:
    """Return True if the signature is valid."""
    try:
        parts = dict(item.split("=", 1) for item in header.split(","))
        timestamp = parts["t"]
        received = parts["v1"]
    except (KeyError, ValueError):
        return False

    # Reject stale events.
    if abs(time.time() - int(timestamp)) > 300:
        return False

    signed = f"{timestamp}.{body.decode('utf-8')}"
    expected = hmac.new(
        secret.encode("utf-8"),
        signed.encode("utf-8"),
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, received)

# Flask example
from flask import request, jsonify

@app.post("/webhooks/miosa")
def miosa_webhook():
    raw = request.get_data()
    sig = request.headers.get("Miosa-Signature", "")

    if not verify_miosa_signature(raw, sig, WEBHOOK_SECRET):
        return jsonify({"error": "invalid signature"}), 401

    event = request.get_json()
    handle_event(event)
    return jsonify({"ok": True}), 200
```

**TypeScript**

```typescript
import crypto from "node:crypto";
import type { Request, Response } from "express";

function verifyMiosaSignature(
  rawBody: Buffer,
  header: string,
  secret: string,
): boolean {
  const parts = Object.fromEntries(
    header.split(",").map((p) => p.split("=") as [string, string]),
  );
  const { t: timestamp, v1: received } = parts;
  if (!timestamp || !received) return false;

  // Reject stale events.
  if (Math.abs(Date.now() / 1000 - Number(timestamp)) > 300) return false;

  const signed = `${timestamp}.${rawBody.toString("utf8")}`;
  const expected = crypto
    .createHmac("sha256", secret)
    .update(signed)
    .digest("hex");

  return crypto.timingSafeEqual(
    Buffer.from(expected, "hex"),
    Buffer.from(received, "hex"),
  );
}

// Express example
app.post("/webhooks/miosa", express.raw({ type: "*/*" }), (req: Request, res: Response) => {
  const sig = req.headers["miosa-signature"] as string ?? "";
  if (!verifyMiosaSignature(req.body, sig, process.env.MIOSA_WEBHOOK_SECRET!)) {
    return res.status(401).json({ error: "invalid signature" });
  }
  const event = JSON.parse(req.body.toString());
  handleEvent(event);
  res.json({ ok: true });
});
```

**Go**

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "io"
    "math"
    "net/http"
    "strconv"
    "strings"
    "time"
)

func verifyMiosaSignature(body []byte, header, secret string) bool {
    parts := map[string]string{}
    for _, part := range strings.Split(header, ",") {
        kv := strings.SplitN(part, "=", 2)
        if len(kv) == 2 {
            parts[kv[0]] = kv[1]
        }
    }
    ts, ok1 := parts["t"]
    received, ok2 := parts["v1"]
    if !ok1 || !ok2 {
        return false
    }

    tsInt, err := strconv.ParseInt(ts, 10, 64)
    if err != nil || math.Abs(float64(time.Now().Unix()-tsInt)) > 300 {
        return false
    }

    signed := fmt.Sprintf("%s.%s", ts, string(body))
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write([]byte(signed))
    expected := hex.EncodeToString(mac.Sum(nil))

    return hmac.Equal([]byte(expected), []byte(received))
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    body, _ := io.ReadAll(r.Body)
    sig := r.Header.Get("Miosa-Signature")

    if !verifyMiosaSignature(body, sig, webhookSecret) {
        http.Error(w, "invalid signature", http.StatusUnauthorized)
        return
    }

    // parse and handle body ...
    w.WriteHeader(http.StatusOK)
}
```

---

## Idempotency

MIOSA may deliver the same event more than once (network retries, at-least-once semantics). Make your handler idempotent by storing the event `id` and discarding duplicates:

```python
processed_events = set()  # use a database or cache in production

def handle_event(event: dict) -> None:
    event_id = event["id"]
    if event_id in processed_events:
        return  # duplicate  -  skip
    processed_events.add(event_id)

    match event["type"]:
        case "sandbox.running":
            notify_user(event["data"]["external_user_id"])
        case "sandbox.destroyed":
            release_resources(event["data"]["id"])
        case "sandbox.error":
            alert_oncall(event["data"])
```

---

## Retry policy

| Attempt | Delay |
|---|---|
| 1 (initial) | Immediate |
| 2 | 1 minute |
| 3 | 5 minutes |
| 4 | 30 minutes |
| 5 | 1 hour |
| 6 | 2 hours |
| 7 | 4 hours |
| 8 | 8 hours |

MIOSA makes up to eight delivery attempts over roughly 15.5 hours (within a 24-hour window), then drops the event. If your endpoint is unreachable, check the webhook delivery log in the MIOSA dashboard.

Return a `2xx` status code within **10 seconds** to acknowledge receipt. If your processing takes longer, acknowledge immediately and handle the event asynchronously.

---

## Register a webhook

**Python**

```python
hook = client.webhooks.create(
    url="https://example.com/webhooks/miosa",
    events=[
        "sandbox.created",
        "sandbox.running",
        "sandbox.paused",
        "sandbox.destroyed",
        "sandbox.error",
    ],
    description="Lumen lifecycle events",
)

# The secret is returned only on creation. Store it immediately.
print(hook["secret"])  # MIOSA_WEBHOOK_SECRET=...
```

**TypeScript**

```typescript
const hook = await client.webhooks.create({
  url: "https://example.com/webhooks/miosa",
  events: [
    "sandbox.created",
    "sandbox.running",
    "sandbox.paused",
    "sandbox.destroyed",
    "sandbox.error",
  ],
  description: "Lumen lifecycle events",
});

// The secret is returned only on creation. Store it immediately.
console.log(hook.secret); // MIOSA_WEBHOOK_SECRET=...
```

> **Caution: Copy the secret now**
>
> The webhook secret is returned once at creation. If you lose it, delete the webhook and create a new one.

---

## Local testing

Use a tunnel to expose your local server during development:

```bash
# Option 1  -  ngrok
ngrok http 8000
# Copy the https URL, set MIOSA_WEBHOOK_URL in .env

# Option 2  -  cloudflared
cloudflared tunnel --url http://localhost:8000
```

Then run the one-time setup script to register the tunnel URL as your webhook endpoint. When you restart the tunnel and get a new URL, update the webhook:

```python
# Update an existing webhook's URL
client.webhooks.update("whk_abc123", url="https://new-tunnel.ngrok.io/webhooks/miosa")
```

---

## See also

- [End-to-end white-label walkthrough](/docs/platform/white-label)
- [Preview Tokens](/docs/develop/previews)
- [Sandbox API reference](/docs/api-reference/sandboxes)
