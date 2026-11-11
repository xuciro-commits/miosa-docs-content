---
slug: "security/white-label"
title: "End-user credentials and OAuth"
description: "How to scope secrets and OAuth connections to your own end-users on top of MIOSA - with your branding, not ours."
section: "Platform builders"
order: 9010
---

If you're building a product **on top of MIOSA** (an AI coding agent, a vertical SaaS, an agency tool) and your end-users create sandboxes through *your* UI, MIOSA Security supports them as first-class identities  -  even though they don't have MIOSA accounts.

This page covers:

1. Attributing sandboxes to your end-users with `external_user_id`
2. Storing credentials scoped to those end-users
3. Branding the OAuth consent flow as your own product
4. Auditing per end-user, per workspace

## Background  -  external attribution

Every Computer and Sandbox already supports three caller-supplied attribution fields:

| Field | Purpose |
|---|---|
| `external_user_id` | Your end-user's identity in your system |
| `external_workspace_id` | Your workspace/team grouping |
| `external_project_id` | Your project identity |

These are labels, not authentication boundaries. But MIOSA Security uses them as **credential-scoping keys**, which means you can store secrets that resolve only for *that* end-user's resources.

See [External Attribution](/docs/platform/attribution) for the broader picture.

## Stamping attribution at create time

When your backend creates a sandbox for one of your end-users, pass the attribution:

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="python-ds",
    external_user_id="dana@example.net",
    external_workspace_id="acme",
    external_project_id="intake-flow",
)
```

**TypeScript**

```typescript
const sandbox = await client.sandboxes.create({
  template: 'node-web',
  externalUserId: 'dana@example.net',
  externalWorkspaceId: 'acme',
  externalProjectId: 'intake-flow',
});
```

**Go**

```go
sandbox, err := client.Sandboxes.Create(ctx, miosa.SandboxCreate{
    Template:            "python-ds",
    ExternalUserID:      "dana@example.net",
    ExternalWorkspaceID: "acme",
    ExternalProjectID:   "intake-flow",
})
```

**Java**

```java
Sandbox sandbox = client.sandboxes().create(SandboxCreate.builder()
    .template("python-ds")
    .externalUserId("dana@example.net")
    .externalWorkspaceId("acme")
    .externalProjectId("intake-flow")
    .build());
```

**Elixir**

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(%{
  template_id: "python-ds",
  external_user_id: "dana@example.net",
  external_workspace_id: "acme",
  external_project_id: "intake-flow"
})
```

The values you choose are opaque to MIOSA  -  they're whatever IDs make sense in your system.

## Storing a credential scoped to an end-user

Suppose Dana pastes her own OpenAI key into your UI. Save it scoped to her:

```python
client.secrets.set(
    name="openai_key",
    value="sk-proj-her-personal-key...",
    scope="external_user",
    external_user_id="dana@example.net",
    expose_as_env="OPENAI_API_KEY",
)
```

Now any sandbox you create with `external_user_id="dana@example.net"` resolves `OPENAI_API_KEY` to her key. Other members in the same account do **not** see it.

For an account-wide credential (for example a shared company key):

```python
client.secrets.set(
    name="ehr_api_key",
    value="...",
    scope="external_workspace",
    external_workspace_id="acme",
    expose_as_env="EHR_API_KEY",
)
```

## How the proxy decides which credential to use

When a sandbox makes an outbound call and the proxy needs to swap a placeholder, it walks (in priority order):

```text
1. Direct binding on this specific sandbox
2. Secret scoped to user (owner_user_id  -  for MIOSA-native users)
3. Secret scoped to external_user (external_user_id match)
4. Secret scoped to external_workspace (external_workspace_id match)
5. Secret scoped to MIOSA workspace
6. Tenant-wide secret
```

First match wins. If nothing matches, the placeholder stays in the request and the upstream call fails (the right behavior  -  better than silently leaking).

## White-label OAuth (your branding, not MIOSA's)

By default, when one of your end-users clicks "Connect GitHub" in your UI, they're sent to GitHub with the consent screen showing **"Authorize MIOSA"**. For a true white-label experience, override that:

### 1. Register your own OAuth app on the provider

For GitHub:

```text
https://github.com/settings/applications/new

Application name:  Lumen Cloud
Homepage URL:      https://lumen.example
Authorization callback URL:  https://app.lumen.example/oauth/callback
```

### 2. Drop the credentials into MIOSA

In your tenant admin dashboard → Settings → OAuth Providers → "Add provider override":

```text
Provider:       github
Display name:   GitHub
Client ID:      <from GitHub>
Client Secret:  <from GitHub>
Callback URL:   https://app.lumen.example/oauth/callback
Branding:       { "button_label": "Connect GitHub to Lumen" }
```

From now on, when one of your end-users connects GitHub, they see "Authorize Lumen Cloud"  -  your name, your branding, your callback domain.

### 3. Make sure your callback proxies to MIOSA

Your `https://app.lumen.example/oauth/callback` route receives the OAuth `code` parameter and forwards it to MIOSA's callback endpoint:

```python
@app.get("/oauth/callback")
def oauth_callback(request):
    code = request.query_params["code"]
    state = request.query_params["state"]
    # Forward to MIOSA
    return miosa.oauth.complete(provider="github", code=code, state=state)
```

MIOSA exchanges the code for tokens on your behalf and stores them in the vault.

## Per-user auditing

Audit log carries the full identity chain on every row, so you can query by your own user IDs:

```python
# All activity by one end-user
events = client.audit.list(
    since="7d",
    external_user_id="dana@example.net",
)

# All activity in one workspace
events = client.audit.list(
    since="30d",
    external_workspace_id="acme",
)
```

Same filters available in the UI.

## Privacy boundary

What a tenant admin (you, as the white-label operator) can do:

- ✅ See that a particular `external_user_id` has secrets bound to them
- ✅ Read audit rows showing which credential resolved for each call
- ✅ Disable a secret to immediately revoke access
- ❌ Re-read the encrypted value of any stored secret

The "you can't read stored values" rule applies to every actor in MIOSA  -  including the platform engineers. If one of your end-users ever asks "can Lumen see my OpenAI key after I save it"  -  the answer is "no, the value is encrypted; only the request body that leaves your sandbox ever contains it, briefly, in transit."

## Disabling end-user-scoped credentials entirely

Some white-label customers want all credentials managed centrally  -  end-users can't bring their own keys. Toggle this in your tenant settings:

```text
Settings → Security → "Allow end-user-scoped credentials" → Off
```

When off, only `tenant` and `external_workspace` scoped secrets resolve. End-user-scoped secret creation returns a 403.

## Real-world example  -  Lumen: one sandbox per end user

Lumen is a SaaS built on MIOSA. Each of Lumen's end users gets their own sandbox. They supply their own OpenAI key through the Lumen settings page. The audit log is surfaced in Lumen's own dashboard. GitHub OAuth shows "Authorize Lumen", not "Authorize MIOSA".

**Python**

```python
import os
from miosa import Miosa
from flask import Flask, request, jsonify, redirect

app_server = Flask(__name__)
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_or_create_sandbox(member_id: str, account_id: str):
    """Return existing sandbox or create a new one for this member."""
    sandboxes = client.sandboxes.list(external_user_id=member_id)
    if sandboxes:
        return sandboxes[0]
    return client.sandboxes.create(
        template_id="python-ds",
        external_user_id=member_id,
        external_workspace_id=account_id,
        external_project_id="agent-workspace",
    )

@app_server.post("/api/members/<member_id>/openai-key")
def save_openai_key(member_id: str):
    """Member saves their own OpenAI key through the Lumen settings page."""
    data = request.json
    client.secrets.set(
        name="openai_key",
        value=data["api_key"],
        scope="external_user",
        external_user_id=member_id,
        expose_as_env="OPENAI_API_KEY",
    )
    return jsonify({"status": "saved"})

@app_server.post("/api/members/<member_id>/connect-github")
def connect_github(member_id: str):
    """Start GitHub OAuth for a member."""
    account_id = request.json["account_id"]
    sbx = get_or_create_sandbox(member_id, account_id)
    flow = sbx.secrets.connect(
        provider="github",
        expose_as_env="GITHUB_TOKEN",
        scopes=["repo", "read:user"],
    )
    return jsonify({"authorize_url": flow.authorize_url})

@app_server.get("/oauth/callback")
def oauth_callback():
    """OAuth callback forwarded from Lumen's branded domain."""
    client.oauth.complete(
        provider="github",
        code=request.args["code"],
        state=request.args["state"],
    )
    return redirect("/dashboard?connected=github")

@app_server.get("/api/members/<member_id>/activity")
def member_activity(member_id: str):
    """Per-member audit feed for the Lumen compliance dashboard."""
    events = client.audit.list(
        since="7d",
        external_user_id=member_id,
        limit=200,
    )
    return jsonify([{
        "time": e.timestamp,
        "host": e.host,
        "action": e.action,
        "status": e.status_code,
    } for e in events])
```

**TypeScript**

```typescript
import express from 'express';
import { Miosa } from '@miosa/sdk';

const app = express();
app.use(express.json());
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function getOrCreateSandbox(memberId: string, accountId: string) {
  const sandboxes = await client.sandboxes.list({ externalUserId: memberId });
  return sandboxes[0] ?? client.sandboxes.create({
    template: 'python-ds',
    externalUserId: memberId,
    externalWorkspaceId: accountId,
    externalProjectId: 'agent-workspace',
  });
}

app.post('/api/members/:memberId/openai-key', async (req, res) => {
  await client.secrets.set({
    name: 'openai_key',
    value: req.body.apiKey,
    scope: 'external_user',
    externalUserId: req.params.memberId,
    exposeAsEnv: 'OPENAI_API_KEY',
  });
  res.json({ status: 'saved' });
});

app.post('/api/members/:memberId/connect-github', async (req, res) => {
  const sbx = await getOrCreateSandbox(req.params.memberId, req.body.accountId);
  const flow = await sbx.secrets.connect({
    provider: 'github',
    exposeAsEnv: 'GITHUB_TOKEN',
    scopes: ['repo', 'read:user'],
  });
  res.json({ authorizeUrl: flow.authorizeUrl });
});

app.get('/oauth/callback', async (req, res) => {
  await client.oauth.complete({
    provider: 'github',
    code: req.query.code as string,
    state: req.query.state as string,
  });
  res.redirect('/dashboard?connected=github');
});

app.get('/api/members/:memberId/activity', async (req, res) => {
  const events = await client.audit.list({
    since: '7d',
    externalUserId: req.params.memberId,
    limit: 200,
  });
  res.json(events.map(e => ({ time: e.timestamp, host: e.host, action: e.action, status: e.statusCode })));
});
```

## Troubleshooting

**Sandbox can't find the secret I stored for an end-user  -  env var is empty.**
The sandbox must have been created with the same `external_user_id` you used for the secret. Attribution is stamped at creation time and cannot be changed. Verify with `client.sandboxes.get(sandbox_id).external_user_id`.

**`scope="external_user"` returns HTTP 403.**
Your tenant has "Allow end-user-scoped credentials" disabled. Enable it in Settings → Security, or use `scope="external_workspace"` for workspace-level credentials instead.

**The resolver skips my workspace-scoped secret.**
The sandbox must have been created with a matching `external_workspace_id`. If the sandbox was created without it, the workspace-level scope walk entry is blank and won't match.

**OAuth consent screen still says "Authorize MIOSA" after I added a provider override.**
Double-check that the `client_id` in your override matches the OAuth app you registered with the provider, not MIOSA's default client ID. Remove and re-add the override if needed.

**My end-user can see other users' sandboxes in the MIOSA dashboard.**
End-users should not have MIOSA accounts or API keys. If they do, create a browser token scoped to that user instead: `client.browser_tokens.create(external_user_id="...")`. Browser tokens are read-only and scoped to that user's resources.

For more issues, see the [Security troubleshooting hub](/docs/security/troubleshooting).

## Frequently asked

**My end-users don't have MIOSA accounts. Can they still use OAuth Connect?**
Yes. The OAuth flow attributes the resulting token to whatever `external_user_id` your backend passes when initiating the flow.

**Can the same `external_user_id` exist in multiple tenants?**
Yes  -  `external_user_id` is scoped within a tenant. The same string in two different tenants represents two independent identities.

**What if an end-user is in two of my workspaces?**
You can create credentials at the workspace scope for each  -  the proxy resolves the correct one based on which sandbox is making the request.

**Can my end-users see the audit log themselves?**
That's your call  -  MIOSA gives you the audit data; how you expose it in your white-labeled UI is up to you. Most white-label customers surface "your activity" to end-users in their own dashboards.

**How do I migrate existing in-app secret storage to MIOSA Secrets?**
Bulk import via SDK is the easiest path:

```python
for user_id, openai_key in your_existing_secret_store.items():
    client.secrets.set(
        name="openai_key",
        value=openai_key,
        scope="external_user",
        external_user_id=user_id,
        expose_as_env="OPENAI_API_KEY",
    )
```

Once verified, delete from your old store.
