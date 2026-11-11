---
slug: "data/email"
title: "Transactional Email"
description: "Configure MIOSA-managed transactional email for project auth, password reset, email confirmation, and platform notifications."
section: "Databases & storage"
order: 4009
---

MIOSA uses transactional email for platform notifications and for per-project
end-user auth flows such as password reset and email confirmation.

This page separates three different email patterns:

| Pattern | Who owns it | Examples |
|---|---|---|
| **MIOSA-managed auth email** | MIOSA/operator | Password reset, email confirmation, security notices |
| **App-owned transactional email** | The generated app/customer | Receipts, booking confirmations, workflow notices |
| **Enterprise/private sender setup** | MIOSA + customer/operator | White-label sender domains, private SMTP, compliance-bound delivery |

> **Note: Current production model**
>
> The live MIOSA-managed auth path uses Resend today. That does not mean Resend is
> the only email provider a customer app can use. It means the built-in Project
> Auth sender currently reads the platform Resend configuration.

## What email covers

| Flow | Uses this config? | Notes |
|---|---:|---|
| Project Auth password reset | Yes | Sends a reset link for an end user of a generated app. |
| Project Auth email confirmation | Yes | Sends a confirmation link after signup when email verification is enabled. |
| Platform admin/user notifications | Yes | Uses the same Resend API key, with platform mailer settings. |
| Customer application transactional email | No | Wire the app to its own provider integration or runtime secret. |
| Customer marketing/campaign email | No | Usually belongs in the customer app or marketing stack, not MIOSA auth. |

## MIOSA-managed auth email

MIOSA operators configure the shared delivery provider and sender identity for the built-in Auth-as-a-Service product.
Customer applications do not receive or manage the platform delivery credential.

If platform delivery is unavailable, auth can still generate reset or confirmation tokens, but the corresponding email is not delivered.
Do not treat a successful password-reset response as proof that a message reached the provider or inbox.

## Other provider options

For application-level email, the generated app can use whichever provider the
customer chooses. Common patterns:

| Provider model | Good for | How it should be wired |
|---|---|---|
| **Resend** | Developer-friendly transactional email | Store the app's `RESEND_API_KEY` as a project/runtime secret and call Resend from app code. |
| **Postmark** | High-deliverability transactional email | Store a server token as a secret; app owns templates and sender verification. |
| **SendGrid** | Larger enterprise email programs | Store API key as a secret or enterprise integration; app owns templates/categories. |
| **Mailgun** | Domain-heavy transactional email | Store API key/domain as secrets; app owns routing and logs. |
| **SMTP relay** | Enterprise/private mail infrastructure | Store SMTP host/user/password as secrets or configure a private deployment path. |
| **Customer-owned service** | Regulated or on-prem environments | Scope network path, credentials, audit, and data handling in the enterprise deployment. |

The important rule: app-owned providers are not the same as MIOSA's built-in
auth mailer. App email belongs to the application runtime; MIOSA auth email
belongs to the platform control plane.

## Platform mailer and notifications

The platform mailer and Auth-as-a-Service use operator-managed delivery configuration.
Platform notification and app-auth sender policies can differ even when they use the same provider.

Do not treat a successful password-reset API response as proof an email was
sent. Check provider configuration, logs, and the provider event dashboard.

## Verification checklist

After configuring the env vars:

1. Enable Project Auth for a fresh project or sandbox.
2. Create a test end user through `{AUTH_URL}/signup`.
3. Call `{AUTH_URL}/password-reset` for that email.
4. Confirm the backend does not log `email_not_configured`.
5. Confirm the configured provider accepts the message.
6. Confirm the reset link reaches the inbox and `{AUTH_URL}/password-reset/confirm` accepts the token.

## White-label and enterprise senders

Today the built-in auth mailer is platform-operated. For a white-label or
enterprise customer that needs mail from its own domain, scope these items
explicitly:

- sender domain and DNS records;
- SPF, DKIM, DMARC, bounce, and complaint handling;
- whether MIOSA sends through a managed provider or the customer's private SMTP;
- per-tenant branding and template ownership;
- audit logs and delivery visibility;
- whether the setup is cloud, private-network, or on-prem.

Do not promise a dashboard toggle that lets each project configure its own auth sender.

## App-owned email

Generated apps can still send their own email by storing provider credentials
as project/runtime secrets and calling the provider directly from app code.
That is the right path for product emails such as receipts, invites, booking
confirmations, reports, or customer-specific notifications.

## Next

- [Project Auth](/docs/data/auth): Enable per-project signup, login, JWTs, password reset, and email confirmation.
- [Project integrations](/docs/integrations): Connect app-level services such as Stripe, Resend, Twilio, and webhooks.
- [Data overview](/docs/data/overview): Managed databases, storage, Redis, Qdrant, Auth, Volumes, and Fractal.
