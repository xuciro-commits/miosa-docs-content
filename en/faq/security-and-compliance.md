---
slug: "faq/security-and-compliance"
title: "Security and compliance"
description: "SOC 2, HIPAA, where data lives, how secrets are stored, retention, and publishable keys."
section: "Help"
order: 16008
---

Short answers on certifications, data location, secrets, and retention.

## Are you SOC 2 compliant?

Not yet. Our SOC 2 audit is in progress and not complete, so there is no report to share today.
Until it is issued, we can share our security questionnaire and controls documentation under NDA.
See [Trust](https://miosa.ai/trust) for the current status.

## Do you support HIPAA?

MIOSA runs HIPAA-compliant infrastructure for healthcare workloads, and business associate agreements (BAAs) are available for Enterprise customers.
See [Trust](https://miosa.ai/trust).

## Where does my data live?

In United States data centers we provision, network, and manage directly; MIOSA is not a reseller of a hyperscaler.
The three live regions are all in the US, and a resource cannot move between them after creation.
See [Regions](/docs/regions) and [Trust](https://miosa.ai/trust).

## How are secrets stored?

Values are encrypted with ChaCha20-Poly1305.
Sandboxes see an opaque `miosa-tok-<hex>` placeholder that is swapped at egress, so no one - including MIOSA engineers - can re-read a stored value.
See [Secrets](/docs/security/secrets).

## How long is data retained?

Egress audit events are retained 90 days.
Usage and credit records are kept separately.
There is no published zero-retention mode.
See [Data retention](/docs/data-retention).

## Are there publishable keys for the browser?

No. There are no `pk_*` publishable keys: API keys are server-only.
Browser and embed access uses short-lived scoped tokens instead (`mp_` preview, `ms_` share, and browser tokens).
See [API keys](/docs/platform/api-keys) and [Browser tokens](/docs/platform/browser-tokens).
