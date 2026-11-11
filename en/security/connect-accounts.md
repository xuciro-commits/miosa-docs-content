---
slug: "security/connect-accounts"
title: "Connect Accounts (OAuth)"
description: "Authorize accounts (GitHub, Google, Slack, …) once. MIOSA manages token refresh; your sandbox sees an env var."
section: "Sandboxes"
order: 2019
---

Instead of pasting a personal-access token or API key, **authorize an account** directly with the upstream provider. MIOSA stores the access + refresh tokens, refreshes them in the background, and exposes the current valid access token to your sandbox as a plain environment variable.

## When to use this over Secrets

| You have… | Use |
|---|---|
| An API key string (e.g., OpenAI `sk-…`, Stripe `sk_live_…`) | [Secrets](/docs/security/secrets) |
| An account where the provider supports OAuth (GitHub, Google, Slack, …) | **Connect Accounts** |
| A machine-to-machine OAuth2 client (client_id + client_secret) | **Connect Accounts** (Machine OAuth) |

## Connect a GitHub account (the typical flow)

### From the dashboard

```text
1. Sandbox detail → Secrets tab → "+ Connect Account"
2. Pick "GitHub" from the provider grid
3. Browser opens to github.com/login/oauth/authorize
4. Approve the scopes you want to grant
5. You bounce back to MIOSA. Done.
```

A new entry appears in your Secrets list:

```yaml
Name:     github_token
Type:     OAuth Connect
Provider: GitHub
Scopes:   repo, read:user
Expires:  in 8 hours (auto-refreshed)
```

Inside your sandbox:

```bash
$ echo $GITHUB_TOKEN
miosa-tok-9a3c5e7b1f8d2a4c...
```

Use it exactly like a real token  -  the swap happens on egress:

```python
requests.get(
    "https://api.github.com/user/repos",
    headers={"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
)
```

When the access token expires, MIOSA uses the refresh token in the background to mint a new one. Your code never notices.

### From the SDK

**Python**

```python
flow = sandbox.secrets.connect(
    provider="github",
    expose_as_env="GITHUB_TOKEN",
)

# `flow.authorize_url` is a URL the end user opens in their browser.
print("Authorize at:", flow.authorize_url)

# Block until the user completes the OAuth dance:
flow.wait_for_completion(timeout=300)
```

**TypeScript**

```ts
const flow = await sandbox.secrets.connect({
  provider: 'github',
  exposeAsEnv: 'GITHUB_TOKEN',
});

console.log('Authorize at:', flow.authorizeUrl);
await flow.waitForCompletion({ timeoutSec: 300 });
```

**Go**

```go
flow, err := sandbox.Secrets.Connect(ctx, miosa.OAuthConnect{
    Provider:    "github",
    ExposeAsEnv: "GITHUB_TOKEN",
    Scopes:      []string{"repo", "read:user"},
})
if err != nil { panic(err) }

fmt.Println("Authorize at:", flow.AuthorizeURL)
if err := flow.WaitForCompletion(ctx, 300); err != nil { panic(err) }
```

**Java**

```java
OAuthFlow flow = sandbox.secrets().connect(OAuthConnect.builder()
    .provider("github")
    .exposeAsEnv("GITHUB_TOKEN")
    .scopes(List.of("repo", "read:user"))
    .build());

System.out.println("Authorize at: " + flow.getAuthorizeUrl());
flow.waitForCompletion(Duration.ofSeconds(300));
```

**Elixir**

```elixir
{:ok, flow} = Miosa.Secrets.connect(sandbox.id, %{
  provider: "github",
  expose_as_env: "GITHUB_TOKEN",
  scopes: ["repo", "read:user"]
})

IO.puts("Authorize at: #{flow.authorize_url}")
{:ok, _} = Miosa.Secrets.wait_for_oauth(flow.id, timeout: 300)
```

## Supported providers

| Provider | Status |
|---|---|
| GitHub | ✅ available |
| Google (Gmail, Drive, Calendar) | rolling out |
| Slack | rolling out |
| Linear | rolling out |
| Notion | rolling out |
| Stripe | planned |
| HubSpot | planned |
| Custom OAuth2 provider | planned  -  file a request |

Adding a new well-known provider is a config-only change on our side  -  no SDK release needed.

## Machine OAuth (client credentials)

For provider APIs that use the OAuth2 client-credentials flow (no end-user, machine-to-machine):

```python
sandbox.secrets.add_machine_oauth(
    name="my_internal_api",
    client_id="abc...",
    client_secret="def...",
    token_url="https://my-api.example.com/oauth/token",
    scopes=["read", "write"],
    expose_as_env="INTERNAL_API_TOKEN",
)
```

MIOSA mints + caches access tokens against `token_url`, refreshes ahead of expiry, and injects them as `INTERNAL_API_TOKEN`.

## What the proxy injects

For an OAuth Connect secret, the proxy injects:

- **Access token** by default (the most common case  -  `Authorization: Bearer …`)
- **Refresh-on-401**: if the upstream returns a 401, the proxy attempts a refresh before bubbling the error to your code

The refresh token itself is **never** exposed to the sandbox. It stays encrypted in MIOSA's vault.

## Rotation, revocation, scope changes

- **Rotation**: happens automatically. You don't do anything.
- **Revocation**: open the Secrets tab → click "Disconnect" → MIOSA destroys the refresh token immediately. The next outbound request fails with 401.
- **Scope changes**: requires re-authorizing. Click "Re-authorize" on the secret; you'll go through the OAuth dance again with the new scopes.

## White-label OAuth (your branding, not MIOSA's)

By default, end-users see "Authorize MIOSA to access your GitHub" in the OAuth consent screen. For white-label deployments, register your own OAuth app and route the dance through your branding  -  see [White-label Credentials](/docs/security/white-label).

## Real-world example  -  agent reads GitHub issues and posts Slack updates

Connect both GitHub and Slack via OAuth. The agent reads open issues and posts a daily digest to a Slack channel. No credentials ever enter the sandbox.

**Python**

```python
import os, threading
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")

# Connect GitHub
github_flow = sandbox.secrets.connect(
    provider="github",
    expose_as_env="GITHUB_TOKEN",
    scopes=["repo", "read:user"],
)
print("1. Authorize GitHub at:", github_flow.authorize_url)
github_flow.wait_for_completion(timeout=300)
print("   GitHub connected.")

# Connect Slack
slack_flow = sandbox.secrets.connect(
    provider="slack",
    expose_as_env="SLACK_BOT_TOKEN",
    scopes=["chat:write", "channels:read"],
)
print("2. Authorize Slack at:", slack_flow.authorize_url)
slack_flow.wait_for_completion(timeout=300)
print("   Slack connected.")

# Write and run the agent
agent_code = '''
import os, requests, json

GITHUB_TOKEN = os.environ["GITHUB_TOKEN"]
SLACK_TOKEN  = os.environ["SLACK_BOT_TOKEN"]
REPO         = "your-org/your-repo"
CHANNEL      = "#engineering-digest"

# Fetch open issues
issues = requests.get(
    f"https://api.github.com/repos/{REPO}/issues?state=open&per_page=10",
    headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
).json()

# Build digest message
lines = [f"*Open issues digest for {REPO}*"]
for i in issues:
    lines.append(f"• <{i['html_url']}|#{i['number']} {i['title']}>")
message = "\\n".join(lines)

# Post to Slack
resp = requests.post(
    "https://slack.com/api/chat.postMessage",
    headers={"Authorization": f"Bearer {SLACK_TOKEN}"},
    json={"channel": CHANNEL, "text": message},
)
print("Posted:", resp.json().get("ok"))
'''

sandbox.files.write("/workspace/digest.py", agent_code)
result = sandbox.exec("python /workspace/digest.py")
print(result.stdout)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.create({ template: 'node-web' });

// Connect GitHub
const githubFlow = await sandbox.secrets.connect({
  provider: 'github',
  exposeAsEnv: 'GITHUB_TOKEN',
  scopes: ['repo', 'read:user'],
});
console.log('1. Authorize GitHub at:', githubFlow.authorizeUrl);
await githubFlow.waitForCompletion({ timeoutSec: 300 });
console.log('   GitHub connected.');

// Connect Slack
const slackFlow = await sandbox.secrets.connect({
  provider: 'slack',
  exposeAsEnv: 'SLACK_BOT_TOKEN',
  scopes: ['chat:write', 'channels:read'],
});
console.log('2. Authorize Slack at:', slackFlow.authorizeUrl);
await slackFlow.waitForCompletion({ timeoutSec: 300 });
console.log('   Slack connected.');

// Run the digest agent
const result = await sandbox.exec('node /workspace/digest.js');
console.log(result.stdout);
```

## Troubleshooting

**`flow.wait_for_completion` times out  -  user authorized but the flow never resolves.**
Usually means the OAuth callback URL is wrong. Verify the callback URL in your OAuth app settings on the provider side matches what MIOSA expects. For white-label setups, make sure your proxy endpoint is forwarding `code` and `state` parameters to `client.oauth.complete()`.

**GitHub OAuth returns "bad_verification_code".**
The authorization code is single-use with a short TTL (10 minutes for GitHub). If `wait_for_completion` took too long, or the callback was called twice, the code is stale. Re-initiate `sandbox.secrets.connect()` to get a fresh flow.

**Token expired  -  sandbox gets 401 from GitHub.**
MIOSA auto-refreshes tokens. If refresh fails (token revoked by user), the dashboard shows "Reconnect needed." Call `sandbox.secrets.reauthorize(provider="github")` and have the user re-authorize.

**"invalid_grant" during token refresh.**
GitHub (and most providers) invalidate the refresh token when the user revokes the OAuth app permission. Re-connect via `sandbox.secrets.reauthorize()`.

**Slack posts succeed but the token appears in the message body.**
This happens if the Slack SDK puts the token in the request body instead of the `Authorization` header. The MIOSA proxy scans bodies when buffered, but Slack's standard `Bearer` header is the reliable path. Ensure you're using `Authorization: Bearer $SLACK_BOT_TOKEN`.

For more issues, see the [Security troubleshooting hub](/docs/security/troubleshooting).

## Frequently asked

**My OAuth provider isn't on the list. Can I add it manually?**
Yes  -  file a request, or use the Custom OAuth2 path when it ships (Phase 3). For now, any provider that issues static access tokens works as a regular API key via [Secrets](/docs/security/secrets).

**Does the sandbox see the refresh token?**
No. Only the current access token is injected. Refresh tokens stay encrypted in MIOSA.

**Can I bind one OAuth account to many sandboxes?**
Yes. Save the secret at the `tenant`, `workspace`, or `user` scope; every sandbox in that scope inherits the binding.

**What happens if the provider revokes my token from their side?**
The proxy's next refresh attempt will fail. We surface this in the dashboard with a "Reconnect needed" badge on the secret, and your next outbound request returns 401.
