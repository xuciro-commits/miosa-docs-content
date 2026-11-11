---
slug: "data/auth"
title: "Auth-as-a-Service"
description: "Opt-in end-user signup, login, and JWT issuance for a sandbox or deployment."
section: "Databases & storage"
order: 4008
---

> **Tip: Available**
>
> Managed end-user auth is live.
> Auth is **opt-in** and disabled by default.
> Enable it for a sandbox or deployment and MIOSA provisions the auth schema in the linked managed database.
> The runtime receives `AUTH_URL`, `AUTH_JWT_SECRET`, and `AUTH_DATABASE_URL`.

## Opt-in model

Auth is disabled by default.
A sandbox or deployment has no managed signup surface until you enable it.
This prevents accidental exposure of authentication endpoints.

**Enable via API:**
```bash
curl -X POST https://api.miosa.ai/api/v1/project-auth/enable \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource_type": "sandbox",
    "resource_id": "'"$SANDBOX_ID"'",
    "config": {
      "signup_enabled": true,
      "email_confirm_required": false,
      "token_expiry_sec": 3600
    }
  }'
```

If the linked database is still provisioning, the endpoint returns `202` with `state: "pending"`.
Poll the status endpoint until auth is active:

```bash
curl \
  "https://api.miosa.ai/api/v1/project-auth/status?resource_type=sandbox&resource_id=$SANDBOX_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

For running sandboxes, MIOSA attempts to synchronize the auth variables into the current runtime.
Deployment changes follow the deployment environment and release lifecycle.

---

Auth is the opt-in managed-auth experience.
When enabled, MIOSA provisions auth tables in the linked database.
The app receives:
```bash

AUTH_URL=https://api.miosa.ai/api/v1/app-auth/sandbox/<sandbox-id>
AUTH_JWT_SECRET          # generated per project, 64 hex chars
AUTH_DATABASE_URL=postgresql://...
```

The generated app uses them like this:
```typescript
// Sign up a new end user
const res = await fetch(`${process.env.AUTH_URL}/signup`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ email, password }),
})
const { access_token, user } = await res.json()

// Verify a JWT in the app's own API
import jwt from "jsonwebtoken"
const claims = jwt.verify(token, process.env.AUTH_JWT_SECRET!)
const userId = claims.sub
```

That's the agent-on-day-zero experience: agent writes a signup flow on day zero, never has to ask the user to configure auth.

## What's offered

| Feature | Status |
|---|---|
| Email + password signup / login | Available |
| JWT access tokens | Available |
| Token verify | Available |
| Email verification | Available (requires email delivery configured) |
| Password reset | Available (requires email delivery configured) |
| Organization SSO (SAML) | Available (configure per organization; enforce SSO for members) |

Not offered today: refresh tokens, magic links, OAuth providers such as Google and GitHub, MFA and TOTP, and passkeys and WebAuthn. Plan for the flows above, and hold your own session lifetime outside the access token. See [Known limitations](/docs/known-limitations) for the full list.

## Endpoints
`AUTH_URL` already points at the resource-specific base (`.../app-auth/<type>/<id>`).
Append the action path:
```http
POST   {AUTH_URL}/signup                   create an end-user account → JWT
POST   {AUTH_URL}/login                    authenticate → JWT
POST   {AUTH_URL}/verify                   verify a JWT, returns the claims
POST   {AUTH_URL}/password-reset           request a reset link (sends email*)
POST   {AUTH_URL}/password-reset/confirm   complete the reset with the token
POST   {AUTH_URL}/email/confirm            confirm an email with the token
```
\* `/password-reset` and `/email/confirm` only send mail once email delivery is
configured; see below. The endpoints succeed regardless; the email is the part
that needs setup.

## Configuring email delivery

Password reset and email verification generate the token and call the send, but
the email itself only goes out once an email provider is wired up. **Until then,
those endpoints return success but no email is sent** (the platform logs
`email_not_configured` and skips it; it never crashes the auth flow).

MIOSA currently sends built-in auth email through an operator-managed delivery provider.
See [Transactional Email](/docs/data/email) for the verification checklist and white-label sender boundaries.
There is no self-service per-resource sender credential in the current public contract.

> Today email sends from a single platform-wide address. Per-project and
> custom-domain senders, so each white-label brand's emails come from its own
> domain, are not offered today.

## End users are not MIOSA users

Important distinction:

- **MIOSA platform user** = the developer / platform builder. Authenticates with `msk_*` API keys or JWT sessions issued by MIOSA. Has access to the MIOSA dashboard.
- **End user of a generated app** = the visitor to an app YOUR PLATFORM built (e.g. a customer logging into an app your platform built). Authenticates against the per-project `AUTH_URL`. Has no MIOSA account.

These are completely separate auth systems. The same email can be a MIOSA platform user AND an end user of someone's app; they're different records, with different password hashes.

## Per-project isolation

Each enabled resource receives an isolated auth scope and JWT secret.
A token issued for one resource cannot authenticate against a different resource.

This makes the "end user signs up to a generated app" pattern safe even when the same MIOSA tenant hosts thousands of independent end-user apps.

## Row-level security (Postgres pattern)

When combined with managed [Postgres](/docs/data/postgres), the auth JWT carries the user ID (`sub` claim). Your runtime code uses it for row-level filtering:
```typescript
const claims = jwt.verify(token, process.env.AUTH_JWT_SECRET!)

const orders = await pool.query(
  "SELECT * FROM orders WHERE customer_id = $1",
  [claims.sub]
)
```

Or use Postgres RLS policies if your Postgres has them enabled:
```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY order_owner ON orders
  USING (customer_id = current_setting('jwt.claims.sub')::uuid);
```

## Bring your own auth?

Yes. Skip the managed auth, set your own `AUTH_URL` and `AUTH_JWT_SECRET` to point at Clerk / Auth0 / your own service. MIOSA's runtime injects whatever you set.

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Get current managed-auth status and config
status = miosa.project_auth.status(
    resource_type="sandbox",
    resource_id=sandbox_id,
)
print(status)  # {"enabled": false, ...}

# Enable project auth (with optional config kwargs)
miosa.project_auth.enable(
    resource_type="sandbox",
    resource_id=sandbox_id,
    signup_enabled=True,
)

# Update config after enabling
miosa.project_auth.update(
    resource_type="sandbox",
    resource_id=sandbox_id,
    signup_enabled=False,
)

# Disable when no longer needed
miosa.project_auth.disable(
    resource_type="sandbox",
    resource_id=sandbox_id,
)
```

## See also

- [Overview](/docs/data/overview)  -  the broader data plane
- [Postgres](/docs/data/postgres)  -  pairs naturally with auth for RLS
- [Platform / Browser Tokens](/docs/platform/browser-tokens)  -  for MIOSA-level browser access; distinct from your app's end-user auth
