---
slug: "develop/sandboxes/connect"
title: "Connecting to a running sandbox"
description: "Four ways into a live sandbox: the SDK, the CLI, SSH over an authenticated tunnel, and preview URLs. What each is good for, and what does not work."
section: "Sandboxes"
order: 2009
---

A sandbox has no public SSH endpoint and no open inbound port other than the ones you expose. Every way in goes through the MIOSA API and an authenticated tunnel. There are four, and they are good at different things.

> **Note: Advertised, not assumed**
>
> SSH and TCP tunnels are advertised capabilities: confirm them on the runtime you target with `GET /api/v1/runtimes` before you depend on one. A guest cannot reach the platform's internal network; see [Internet access and egress](/docs/develop/sandboxes/internet-access).

| Way in | Best for |
|---|---|
| SDK | Application code: run commands, stream output, move files |
| CLI | Interactive and scripted use from your terminal |
| SSH | A real `ssh` session for tools that expect one |
| Preview URL | Letting a browser reach a service running in the sandbox |

## 1. The SDK

The SDK is the path for code that talks to a sandbox. Run a command and read its output, or stream it as it runs.

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.get("sbx_123")

result = sandbox.exec("uname -a")
print(result.stdout, result.exit_code)

for event in sandbox.exec_stream("pnpm install"):
    print(event.line, end="")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.get('sbx_123');

const result = await sandbox.exec.run('uname -a');
console.log(result.stdout, result.exitCode);
```

Execution and file operations are covered in full in [Files and exec](/docs/develop/files-and-exec). `exec` on a paused persistent sandbox resumes it before dispatch.

## 2. The CLI

The CLI is the fastest way to poke at a running sandbox from a terminal.

```bash
miosa exec my-box -- ls -la /workspace        # run one command, get its exit code
miosa console my-box                          # interactive PTY shell
miosa files cp ./app.py my-box:/workspace/    # move files in and out
```

- **`miosa exec`** runs a command and exits with the remote command's exit code, so it drops into scripts and CI. A sandbox that is still starting refuses commands; `exec` retries with backoff until it is ready.
- **`miosa console`** opens a full-duplex terminal over the `miosa-terminal-v1` WebSocket: a real PTY, working window resizing, and Ctrl-C reaching the remote process. Use a command after `--` to run a program instead of a login shell.
- **`miosa files`** copies, lists, and inspects files with a `<sandbox>:<path>` syntax.
- **`miosa proxy my-box 5432:5432`** forwards a local port to a sandbox port over a WebSocket tunnel for any TCP service.

See [Run commands and move files](/docs/cli/work-inside) for the full command set.

## 3. SSH

`miosa ssh` gives you a real SSH session without any setup in `~/.ssh`. It makes a one-time key, asks the API for a short-lived certificate (15 minutes, valid for that one sandbox), and runs the system `ssh` through MIOSA's authenticated tunnel. Where the server has no certificate authority, it authorizes the one-time key in the sandbox instead. The key, certificate, and host file live in a private temporary directory that is removed when ssh ends.

```bash
miosa ssh my-box                                  # interactive login shell
miosa ssh my-box -- bash -lc "cd app && npm test"  # run a command
miosa ssh my-box -- bash -s < ./setup.sh           # pipe a script in
```

Standard input is piped through. With `ssh` or `ssh-keygen` unavailable, or with `--api`, the command runs over the API instead.

> **Caution: There is no public SSH endpoint**
>
> `ssh user@host` and `scp` pointed straight at a sandbox do not work. The tunnel is the only SSH path. It is also available to code: the Go SDK exposes `sandboxssh.Dial`, and the Python SDK ships `miosa.ssh_tunnel` as an OpenSSH `ProxyCommand` and a `paramiko` / `asyncssh` socket.
>
> ```text
> # ~/.ssh/config
> Host miosa
>     HostName <sandbox id>
>     User root
>     ProxyCommand python3 -m miosa.ssh_tunnel %h
> ```

## 4. Preview URLs

To let a browser reach a service running inside the sandbox, expose its port as an HTTPS preview URL. Previews are the only inbound path; nothing else in the sandbox is reachable.

```bash
miosa preview create my-box 3000 --visibility public
miosa url my-box
```

Previews carry their own token model for embedding and sharing. See [Previews](/docs/develop/previews).

## Which to use

- **Automation and applications:** the SDK.
- **You, at a terminal, right now:** `miosa console` for interactive work, `miosa exec` for scripts.
- **A tool that needs SSH:** `miosa ssh`, or the SDK's SSH tunnel from your own code.
- **A person or a browser that needs to open a page:** a preview URL.

## See also

- [Files and exec](/docs/develop/files-and-exec): Run commands, stream output, and move files with the SDK.
- [Run commands and move files](/docs/cli/work-inside): exec, console, ssh, files, processes, services, and ports in the CLI.
- [Previews](/docs/develop/previews): Expose a sandbox port as a live HTTPS URL.
