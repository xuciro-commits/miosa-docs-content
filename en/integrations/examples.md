---
slug: "integrations/examples"
title: "Example Apps"
description: "Four full-stack white-label examples you can clone, run locally, and adapt to your own stack."
section: "Platform builders"
order: 9006
---

Four complete, runnable examples. Each demonstrates a different frontend stack and a different set of white-label capabilities. Clone the repo and follow the "Run locally" steps below each entry.

Every example should pass three proofs before you adapt it: the application starts locally, its MIOSA resource resolves to the intended Workspace and Project, and the browser never receives a server API key.

## Pick an example

| Goal | Start with | Completion proof |
|---|---|---|
| Server-rendered integration | `whitelabel-saas` | A customer Sandbox opens through a scoped Preview token |
| React product with embedded terminal | `whitelabel-react-saas` | Refresh resumes the same attributed Sandbox |
| Next.js marketplace or builder | `whitelabel-nextjs-marketplace` | Multiple Projects remain separated by canonical IDs |
| Vue learning environment | `whitelabel-vue-tutor` | Quotas and teardown apply to the correct end user |

> **Caution: Use examples as executable starting points**
>
> Run the example unchanged first.
> Record the successful local URL and MIOSA resource IDs, then adapt one concern at a time.

All examples live in `examples/` at the root of the [MIOSA GitHub repository](https://github.com/Miosa-osa/miosa).

---

## whitelabel-saas

**Stack:** Python (Flask) backend + plain HTML/JS frontend

**What it demonstrates:**

- One-time tenant setup: custom preview domain, branding, webhook registration
- Per-customer sandbox provisioning with `external_user_id`, `external_project_id`, and `metadata`
- Server-side preview token minting and iframe embed
- HMAC-SHA256 webhook signature verification
- Teardown by `external_user_id` (cancellation flow)

This is the canonical reference example. Every concept in the [white-label walkthrough](/docs/platform/white-label) maps directly to code in this example.

**Run locally:**

```bash
cd examples/whitelabel-saas
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

python setup.py      # one-time tenant setup
python app.py        # server on http://localhost:8000
# Open index.html in a browser for the embed demo
```

**GitHub:** [`examples/whitelabel-saas`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-saas)

---

## whitelabel-react-saas

**Stack:** React 18 (Vite) frontend + Express.js BFF

**What it demonstrates:**

- BFF pattern: Express mints browser tokens; React uses `@miosa/sdk` directly in the browser
- Live terminal inside an iframe powered by the TypeScript SDK
- Per-user sandbox state stored in the BFF session
- Sandbox resume on page refresh (poll for `state === "running"` before showing the embed)
- Token refresh on `miosa:preview:expired` message event

**Run locally:**

```bash
cd examples/whitelabel-react-saas
pnpm install

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

pnpm dev   # starts both Express (port 3001) and Vite (port 5173)
# Open http://localhost:5173
```

**GitHub:** [`examples/whitelabel-react-saas`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-react-saas)

---

## whitelabel-nextjs-marketplace

**Stack:** Next.js 15 (App Router) full-stack

**What it demonstrates:**

- Next.js Route Handlers as the BFF: token minting without a separate Express server
- Seller onboarding: sandbox provisioned on first login, re-used on return
- Multi-project listing: sandboxes filtered by `external_user_id` and rendered as cards
- Usage display: per-seller compute hours fetched with `client.usage.get(external_user_id)`
- Custom `next.config.js` to proxy sandbox previews through the Next.js server (avoid CORS for embedded iframes)

**Run locally:**

```bash
cd examples/whitelabel-nextjs-marketplace
pnpm install

cp .env.local.example .env.local
# Edit .env.local  -  fill in MIOSA_API_KEY

pnpm dev   # starts on http://localhost:3000
```

**GitHub:** [`examples/whitelabel-nextjs-marketplace`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-nextjs-marketplace)

---

## whitelabel-vue-tutor

**Stack:** Vue 3 (Vite) frontend + Express.js backend

**What it demonstrates:**

- Per-lesson sandbox lifecycle: fresh sandbox per lesson, destroyed on completion
- Quota enforcement: custom middleware rejects `sandbox.create` if the user has reached their limit
- Public share links with a 15-minute TTL for peer review
- File-tree component: reads the sandbox filesystem via `client.files.list(sandboxId, "/")`
- Run-button component: streams exec output via `client.exec.stream(sandboxId, "npm test")`

**Run locally:**

```bash
cd examples/whitelabel-vue-tutor
pnpm install

cp .env.example .env
# Edit .env  -  fill in MIOSA_API_KEY

pnpm dev   # starts both Express (port 3001) and Vite (port 5174)
# Open http://localhost:5174
```

**GitHub:** [`examples/whitelabel-vue-tutor`](https://github.com/Miosa-osa/miosa/tree/main/examples/whitelabel-vue-tutor)

---

## Feature matrix

| Capability | whitelabel-saas | whitelabel-react-saas | whitelabel-nextjs-marketplace | whitelabel-vue-tutor |
|---|:---:|:---:|:---:|:---:|
| Preview token (server-side) | Yes | Yes | Yes | Yes |
| Iframe embed | Yes | Yes | Yes | Yes |
| Webhook verification | Yes |  -  |  -  |  -  |
| Browser token (client SDK) |  -  | Yes |  -  |  -  |
| Live terminal |  -  | Yes |  -  | Yes |
| File tree |  -  |  -  |  -  | Yes |
| Exec streaming |  -  |  -  |  -  | Yes |
| Usage display |  -  |  -  | Yes |  -  |
| Quota enforcement |  -  |  -  |  -  | Yes |
| Public share links |  -  |  -  |  -  | Yes |
| Teardown by user | Yes |  -  |  -  |  -  |

---

## Contributing an example

If you build something on MIOSA and want to share it, open a pull request in the [MIOSA GitHub repository](https://github.com/Miosa-osa/miosa). Examples must:

- Include a working `README.md` with local setup instructions
- Not commit `MIOSA_API_KEY` or any secrets
- Use `external_user_id` for all resource attribution
- Verify webhook signatures if a webhook endpoint is included
- Include a smoke command that exits nonzero when the primary user journey fails
- Pin supported runtime and package-manager versions
- Explain expected output and cleanup steps

---

## See also

- [End-to-end walkthrough](/docs/platform/white-label)
- [Preview Tokens](/docs/develop/previews)
- [Webhooks reference](/docs/integrations/webhooks)
