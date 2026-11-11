---
slug: "glossary"
title: "MIOSA Glossary"
description: "Plain-language definitions for MIOSA resources, services, deployment stages, domains, on-premises infrastructure, OpenComputers, Optimal, agent workflows, data, and billing. Search by topic, study with flashcards, and link straight to any term."
section: "Optional"
order: 17002
---

Plain-language definitions for every MIOSA resource, API field, and billing term, with a shareable link for each. See how they fit together in the [Product model](/docs/concepts).

## Core primitives

The things you create, build, publish and run.

### App Engine Host

An App Engine Host is an always-on machine for one Workspace that runs many of its apps as separate containers.

Use it when a Workspace publishes lots of small apps, funnels, or client sites and a new production VM for each would be wasteful. MIOSA still routes, secures, and meters each Deployment individually.

In MIOSA, a Northwind Agency workspace with fifteen client sites runs them all on one App Engine Host. The host is capacity, not an access boundary.

Related: Deployment, Workspace, Runtime Instance, Host, App Engine

### Artifact

An Artifact is a file or result worth keeping from a build, command, evaluation, or Agent Run: a report, a screenshot, a build folder, a CSV.

Use Artifacts to hand outputs to people. The Sandbox stays the working surface; you export exactly the paths a user should receive instead of asking them to scrape terminal output.

In MIOSA, an agent writes `/workspace/artifacts/report.pdf`, your backend calls an export for that path, and your user clicks Download. A saved Sandbox state is a Snapshot, and a published app is a Release.

Related: Snapshot, Release, Agent Run

### Background process

A Background process is a command you start in a sandbox that keeps running on its own, such as a dev server or a long job. Its output is kept, so you can read it later.

Use `exec` for a command whose result you wait for, and a background process for servers and anything long.

In MIOSA, `miosa process start my-box --name api -- node server.js` starts one, and `miosa ps` lists what is running.

Related: Exec, Sandbox, Streaming exec, Preview, Logs

### Build

A Build installs dependencies and compiles or packages your source into something that can run. MIOSA does it in an isolated Builder from the frozen source snapshot.

You mostly let MIOSA pick: it detects a Dockerfile, a language manifest, or plain static files. Override with a build command when the defaults are not right.

In MIOSA, `miosa deploy builds team-portal` lists a deployment's builds, and a failed build leaves the old Version live.

Related: Builder, Publish, Release, Dockerfile, BuildSpec

### Builder

A Builder is the isolated worker that handles one publish. It takes the frozen source, runs the build, stores the result, and exits.

You rarely touch it directly. It is why a build cannot be influenced by whatever you are doing in the Sandbox at the same time, and why builds are repeatable.

In MIOSA, a publish for Lumen's team portal starts a Builder, which installs dependencies, runs the build command, and produces the Release. Builder time is billed in seconds during the publish.

Related: Publish, Release, Build, BuildSpec

### BuildSpec

A BuildSpec is a small JSON document that overrides MIOSA's default install, build, and start commands for a Sandbox or Deployment.

Use it when auto-detect guesses wrong, for example a monorepo with an unusual build step.

In MIOSA, a BuildSpec can set `install` to `pnpm install --frozen-lockfile`, `build` to `pnpm build`, and `start` to your server command, plus environment variables.

Related: Build, Builder, Dynamic deployment, Template

### Checkpoint

A Checkpoint is a fast, sandbox-local restore point. Take one before a risky step, such as upgrading dependencies, and restore from it if the step goes wrong.

Snapshots are the portable history of a machine and the basis for forking. Checkpoints are the quick, in-place kind. Use a Snapshot when you want to move or copy a machine, and a Checkpoint for a fast undo.

In MIOSA, `miosa checkpoint create my-box --comment "before deps"` takes one.

Related: Snapshot, Restore, Named snapshot, Sandbox

### Computer

A Computer is a durable cloud desktop: a real Linux machine with a screen, a browser, files, and a shell that you or an agent can operate, and that keeps its state when stopped.

Use one when the work depends on pixels or sessions: clicking through a website, a logged-in browser, desktop apps, or a human taking over where an agent stopped. For code and builds, a Sandbox is cheaper and faster.

In MIOSA, a Northwind Agency operator watches an agent fill in a client's web form on a Computer, then takes over the same desktop to finish. Sizes run from `xs` to `xl`, and `small` is the default.

Related: Desktop session, MIOSA-managed Computer, OpenComputer, Readiness, Capability, Computer use

### Container image

A Container image is an app that is already built and packaged, published to a registry like GitHub Container Registry or Docker Hub.

Use it when you build your app elsewhere and only want MIOSA to run it. MIOSA pulls the image and starts it without a build step. A private registry needs a registry secret.

In MIOSA, `miosa service deploy shop API --image ghcr.io/acme/api:1.4 --port 8080` deploys an image.

Related: Service, Container, Dockerfile, Build, Release

### Cron job

A Cron job runs one command on a schedule you write as a cron expression, for example every five minutes or every night at 3 am.

Use it for work nobody should have to remember: a health check, a nightly export, a cleanup. The command runs in the Sandbox you attach it to, so that Sandbox needs the script and the credentials it uses. You can also trigger a run right now to test it.

In MIOSA, cron jobs are created through the MCP `cron_create` tool, and agents can also be started on a schedule with `miosa agent triggers add`.

Related: Sandbox, Job, Webhook

### Deployment

A Deployment is the durable production identity of your app: its name, URL, history of Versions, domains, settings, and attached data. The Deployment stays put while the code behind it changes.

Use it as "the live app". When you publish and promote a new Version, the same Deployment and URL serve the new code. Databases and storage attach to the Deployment, not to each Release.

In MIOSA, Lumen's `team-portal` Deployment keeps `team-portal.example-org.miosa.app` and the custom domain `portal.example.net` stable across every Version.

Related: Version, Release, Promotion, Domain, Binding, Environment, Runtime Instance, Static deployment, Dynamic deployment, App Engine, Service

### Desired state

Desired state is what MIOSA has been told a resource should look like: running, stopped, on this size, with this configuration.

It matters because MIOSA keeps comparing the world with that record. If a host restarts and a Computer you wanted running is not, MIOSA brings it back instead of waiting for you.

In MIOSA, you stop a Computer on purpose and its desired state is stopped, so it stays stopped through host maintenance. A Computer that was running is restored and verified.

Related: Reconciliation, Control plane

### Desktop session

A desktop session is the live connection you use to see and control a Computer's screen. It is the window onto the machine, not the machine itself.

Use it when you or an agent need to watch, click, and type. Closing the session leaves the Computer running or stopped exactly as it was, and you can open a new one later.

In MIOSA, a support lead opens a live viewer onto the Computer an agent was using at Acme Corp, takes over to finish a task, and closes the viewer without ending anything.

Related: Computer, Readiness, Agent Run

### Dockerfile

A Dockerfile is a text recipe that says how to assemble a container image: the base system, the files to copy, and the command to run.

If one exists in your app's output folder, MIOSA builds your app from it. You can also use it to build a custom Sandbox template.

In MIOSA, sandboxes include Docker Engine, so you can test a Dockerfile inside a Sandbox first, then publish through App Engine.

Related: Container, Build, Template, App Engine

### Dynamic deployment

A Dynamic deployment runs code on a server: APIs, server-rendered pages, WebSockets, and background workers. MIOSA starts it, checks it, and keeps it running.

Choose it when the app must do work per request. You give a run command and a port so MIOSA knows how to start and reach it.

In MIOSA, `miosa deploy create --from-sandbox my-box --port 3000 --run-command "npm start" --wait` publishes a running server.

Related: Static deployment, Runtime Instance, Health check, Replica

### Environment

An environment is a named starting setup for new machines: which repositories to clone, which variables and secret files to inject, and which of your credentials the machine may use.

Use environments so every new Sandbox or Computer starts ready, and so staging and production do not share settings. Every Workspace has a default named `base`.

For Deployments, a Deployment Environment is the scope where variables, secrets, and data bindings apply. Every Project gets `production` and `staging`, and you can add others such as `preview/pr-42`.

Related: Template, Environment variable, Secret, Deployment, Workspace

### Exec

Exec runs one command inside a Sandbox and returns its output and exit code. Examples are `npm install`, `python app.py`, or `npm run build`.

It is how agents and scripts do the work: install, build, test, start a server. The CLI exits with the remote command's exit code, so it drops straight into scripts and CI.

In MIOSA, a Lumen agent runs the build through Exec and reads the output to decide what to fix next. A command has 30 seconds by default and 300 at most; run anything longer as a background process.

Related: Sandbox, Job, Event stream (SSE), Computer, Logs, Background process, Streaming exec

### Fork

A Fork creates a new Sandbox from a copy of an existing one's current state, leaving the original running. After the fork they are independent.

Use it to try something risky on a copy, or to give each customer a fresh copy of a prepared setup.

In MIOSA, `miosa fork my-box --ttl 2h` forks a running Sandbox. Forking from a named snapshot also works after the original machine is gone.

Related: Snapshot, Sandbox, Pause and resume

### Health check

A Health check is a quick request to a URL path in your running app. A success response (2xx) says it is healthy, so MIOSA sends it traffic.

Add one so a broken release never takes over. MIOSA only moves traffic to a new Version once its instances pass, and removes instances that stop passing.

In MIOSA, set `health_check_path` on a dynamic publish. Static sites skip it because there is no app process to check.

Related: Runtime Instance, Dynamic deployment, Promotion, Heartbeat

### Idle sleep

Idle sleep stops a Service that has had no requests for a while, so it does not run when nobody uses it. The next request wakes it and waits for it to start.

It applies only to services with a public address, because a request is what wakes them. Workers and private services never sleep. On the Free plan, services always sleep.

In MIOSA, set it with the `sleep` setting: `{enabled, idle_s}`. A sleeping release shows the state `sleeping`.

Related: Service, Public address, Release, Idle timeout

### Idle timeout

An Idle timeout pauses a persistent Sandbox after it has been inactive for a period, instead of waiting for the fixed Timeout.

Turn it on to save money on Sandboxes people forget about. Commands, file writes, preview traffic, and terminal input all count as activity and reset the clock.

In MIOSA, `miosa create my-box --idle-timeout 15m` pauses it after fifteen quiet minutes. It only applies when you set it, and a paused Sandbox wakes on request.

Related: Timeout, Pause and resume, Preview

### Job

A Job is one tracked command run with an ID, progress, and a final result such as an exit code.

It matters when work outlives one request: you can check later whether it finished, or cancel it. On OpenComputers, every command sent to a connected host is a Job.

In MIOSA, `miosa host job list <host>` shows recent commands on a machine you connected. Cancelling a Job is not proof the remote process stopped, so confirm the outcome before retrying work with side effects.

Related: Exec, Agent Run

### Logs

Logs are the lines of text a Sandbox, process, or deployment prints as it runs: boot messages, errors, and your app's own output.

Read them first when something fails. Follow them live while you fix it.

In MIOSA, `miosa logs my-box -n 100 -f` follows a Sandbox's system log, `miosa process logs <pid>` reads a background process, and `miosa deploy logs team-portal` reads a deployment.

Related: Exec, Build, Event stream (SSE), Log drain

### Named snapshot

A Named snapshot is a Snapshot you pin on purpose under a name. Unnamed snapshots are the machine's history and expire by retention; a named one stays until you remove the name, even after the machine is destroyed.

Use it for a setup you want to start from again and again, then fork a new machine from it whenever you need one.

In MIOSA, each organization keeps 10 named snapshots free. `miosa snapshot save my-box web-stack` pins one, and `miosa snapshot fork --name web-stack` starts a machine from it.

Related: Snapshot, Fork, Checkpoint, Template

### Pause and resume

Pause stops a running Sandbox's compute but keeps its disk and memory state. Resume brings it back with the same ID and files, and processes continue where they were.

Use it to stop paying for compute between work sessions. A paused Sandbox is billed only at the storage rate.

In MIOSA, `miosa pause my-box` and `miosa resume my-box` do it by hand. A paused persistent Sandbox can also wake on request: a hit on its preview URL resumes it.

Related: Persistence, Idle timeout, Timeout, Sandbox

### Persistence

Persistence means a Sandbox's files and installed tools survive when it stops or pauses. The Sandbox record and its `/workspace` outlive any single running session until you destroy it.

Use it for work you will come back to: a project an agent builds over several days, or a customer's workspace. Turn it off only for throwaway runs that should disappear.

In MIOSA, Sandboxes are persistent by default. When the timeout passes, a persistent Sandbox pauses and keeps its files; a non-persistent one is destroyed.

Related: Pause and resume, Timeout, Snapshot, Sandbox

### Preview

A Preview is a web address for something running inside a Sandbox. Start a dev server on a port, create a Preview for that port, and you get a URL you can open in a browser.

Use it to watch an app while an agent builds it, or to show work in progress. A Preview always shows the Sandbox's current state, so it is not a production URL and it stops working when the Sandbox is gone.

In MIOSA, a Preview looks like `https://3000-abc12345.sandbox.miosa.ai`. Previews are private by default; mint a short-lived share token to let a specific browser in.

Related: Sandbox, Preview domain, Publish, Platform URL, Share token, Idle timeout, Port forward, Sandbox share

### Preview environment

A Preview environment is a copy of your Project that runs the code from one pull request, with its own addresses.

Open a pull request and MIOSA forks the default environment as `pr-<number>`, deploys every service from the branch, and comments the addresses on the pull request. Close or merge it and the environment is deleted. A preview never changes the environment it came from.

In MIOSA, turn it on in the project's **Settings** under **Previews** by naming the repository.

Related: Environment, Project, Service, Preview

### Private service

A Private service answers requests from the other Services in the same Project and Environment, and is never reachable from the internet.

It suits an admin backend or a job that only calls out. Private networking between services is not available yet, so your other services cannot call it by name; give it a public address if they need to.

In MIOSA, `miosa service create shop billing --type private --repo https://github.com/acme/billing` creates one. Private networking between services is not available yet, so other services cannot reach it by name today.

Related: Service, Service type, Private address, Public address

### Project canvas

The Project canvas is the main view of a Project in the console. It draws every Service, database, and volume as a card and shows how they connect.

Open a card to change its settings, variables, or domains. The environment switcher in the canvas top bar moves you between production, staging, and preview environments.

In MIOSA, edits on the canvas become Staged changes, and **Deploy** applies them together.

Related: Project, Service, Staged changes, Environment

### Promotion

Promotion is the deliberate step that makes one verified Version the live one for a Deployment. Customers keep using the same URL; they simply start receiving the new Version.

Use it to separate "built" from "live". Publish produces a candidate, you check it, then promote. If something is wrong, promote an earlier Version instead.

In MIOSA, a Northwind Agency client site publishes Version 6, checks the staging URL, then promotes it. Nothing about the Domain or Deployment changes.

Related: Publish, Version, Deployment, Rollback

### Publish

Publish turns a Sandbox's current work into a frozen, buildable candidate. MIOSA snapshots the source, builds it in an isolated Builder, and records an immutable Release and a Version.

Use it when the app works in the Sandbox and you want a real, repeatable production candidate. The Sandbox can keep changing while the build runs. Publishing does not by itself move customer traffic; promotion does.

In MIOSA, `miosa deploy create --from-sandbox my-box --name team-portal --wait` publishes the Sandbox and waits for the result. Use the same name next time to add a new Version behind the same URL.

Related: Builder, Release, Version, Promotion, Source snapshot, Build, Static deployment, Dynamic deployment

### Readiness

Readiness is proof that a resource can do the specific thing you need right now. "Created" and "online" are not enough.

Check it before you send work. A headless server can be ready to run commands without being ready to stream a desktop, and a Sandbox is command-ready only once it is `running`.

In MIOSA, `miosa create my-box --wait` blocks until the Sandbox accepts commands, so `miosa create x --wait && miosa exec x -- ...` never races. For hosts, a fresh heartbeat plus a completed command is the evidence.

Related: Heartbeat, Evidence, Computer, Capability

### Reconciliation

Reconciliation is the automatic repair loop: MIOSA notices when a resource's real condition differs from what it should be, and fixes it.

You rely on it without doing anything. A crashed runtime gets replaced, a Computer on a host under maintenance is restored, and a missing replica is started.

In MIOSA, when a dynamic Lumen app instance dies, the reconciler schedules a replacement on another healthy host. A manual redeploy is a different thing you start yourself.

Related: Desired state, Control plane

### Release

A Release is the finished, immutable result of a successful publish: the built files for a static site, or a packaged server app. It never changes after it is created.

It answers "which exact bytes are we running?" That is why rollback is quick and why many copies can run identically. To change an app, publish again to get a new Release.

In MIOSA, a Lumen site's Release is content-addressed by sha256. Never repair a Release by editing it in place; publish a fix instead.

A Service has releases too: every `miosa service deploy` is a numbered, immutable release with a state, and rollback serves an earlier release's built image without rebuilding.

Related: Publish, Version, Deployment, Artifact, Service, Rollback

### Restore

Restore brings back saved state by creating a new sandbox from a checkpoint or snapshot. The original sandbox is not changed, so you can compare the two.

Use it to undo a bad change. For a sandbox that failed or errored, `miosa recover` does the same from its preserved disk.

In MIOSA, `miosa restore my-box cp_abc123` restores from a checkpoint and prints the new sandbox.

Related: Checkpoint, Snapshot, Fork, Sandbox

### Rollback

Rollback returns a Deployment to an earlier ready Version. It is quick because nothing is rebuilt: the old Release is still stored.

Use it when a new Version passes checks but misbehaves with real traffic, such as a bad migration or a regression you missed. It restores known application behavior, not data.

In MIOSA, Lumen rolls `team-portal` back to Version 16 with one command. Static sites recover in seconds; dynamic ones in seconds to about a minute. A database change is not undone.

Related: Promotion, Version, Release

### Runtime Instance

A Runtime Instance is one running copy of a dynamic Release, serving traffic for one Version. Static sites do not have them.

You do not pick where they run. MIOSA starts instances, waits for health checks, routes traffic to healthy ones, and replaces any that fail. A Deployment can have several instances for capacity.

In MIOSA, Lumen's API server runs as one instance by default. Publishing a new Version starts new instances first and only moves traffic once they are healthy.

Related: Deployment, Version, Release, App Engine Host, Health check, Replica

### Sandbox

A Sandbox is a private, disposable Linux machine in the cloud. You, or an AI agent, can install packages, run commands, edit files, and start servers in it without touching your laptop or production.

Use it as the workshop: build and test here, then publish what works. A Sandbox is mutable, so it is the wrong place to serve real customers. Publishing freezes a copy into an immutable Release.

In MIOSA, a Lumen builder agent creates a Sandbox, writes a web app into `/workspace`, starts the dev server, and opens a Preview. Sandboxes are persistent by default, so pausing one keeps its files.

Related: Preview, Snapshot, Template, Exec, Project, Firecracker, Sandbox size, Persistence, Pause and resume, Timeout, Fork

### Sandbox size

A Sandbox size is a named bundle of CPU, memory, and disk, such as `small` or `large`. You pick a name instead of numbers.

Choose the smallest size that fits. `small` (2 vCPU, 4 GB RAM, 10 GB disk) is the default and is right for most agent work, package installs, and ordinary builds. Move up to `medium` only when you measure that you need more.

In MIOSA, `miosa create my-box --size medium` gives a Sandbox 4 vCPU, 8 GB RAM, and 20 GB disk. Larger sizes cost more while running.

Related: Sandbox, Template, Timeout, Region

### Service

A Service is one app a Project runs, such as your API, a background worker, or a static site. It belongs to one Project and one Environment.

Every deploy of a Service creates an immutable Release, and you can roll back to an earlier one without rebuilding. A Service also has Variables, Replicas, Logs, and an address.

In MIOSA, `miosa service create shop API --type web --repo https://github.com/acme/api` creates one, and `miosa service deploy shop API --wait` ships it.

Related: Service type, Project, Release, Replica, Environment, Public address, Private address, Shared variable

### Service type

Every Service has a type that decides how it runs and whether it gets an address.

- **Web service**: serves HTTP and can have a public address
- **Worker**: runs with no port and no address
- **Private service**: serves other services in the project, never the internet
- **Static site**: pre-built files served from the edge
- **Cron job**: a command on a five-field schedule, with run history
- **Database service**: a managed Postgres or Redis bound to the project

In MIOSA, pick it with `--type` when you create the Service: `web`, `private`, `worker`, `static`, or `schedule`.

Related: Service, Worker, Private service, Cron job, Static deployment, Managed database

### Snapshot

A Snapshot is a saved copy of a Sandbox at one moment. You can restore it later or fork a new Sandbox from it, and the original is left untouched.

Take one before a risky change, or to keep a good starting point you reuse. A Snapshot is development state, not production: it never serves customers.

In MIOSA, a Northwind Agency developer sets up a project once, saves it as a named snapshot, then forks a fresh copy for each new client site.

Related: Sandbox, Artifact, Release, Template, Fork, Persistence, Named snapshot, Checkpoint, Restore

### Source snapshot

A Source snapshot is a frozen archive of your Sandbox's source, taken at the moment you publish. The build runs from it, not from the live Sandbox.

It is why a publish is repeatable: retries can rebuild from the exact same input, and you can keep editing while the build runs. Build output folders and caches are excluded.

In MIOSA, publishing Lumen's site freezes `/workspace` as a source snapshot, then builds the Release from that copy.

Related: Publish, Release, Build, Snapshot

### Staged changes

Staged changes are edits you make on the Project canvas, such as a new variable or a new replica count, that are queued instead of applied at once. A bar at the bottom lists them.

Choose **Deploy** to apply them together, or **Discard** to drop them. When the project keeps change sets on the server, Deploy applies the whole set at once: if one change fails, none are applied.

In MIOSA, a card on the canvas shows how many changes are staged for it, so you can review before anything ships.

Related: Project canvas, Release, Service, Environment variable

### Static deployment

A Static deployment is a site made of ready files: HTML, CSS, JavaScript, and images. MIOSA serves the files directly. There is no server process to keep alive.

Choose it for landing pages, single-page apps, docs, and generated HTML. It is the cheapest and fastest to roll back, and it is the default publish type.

In MIOSA, `miosa deploy create --dir ./dist --name my-site` uploads a built folder as a static deployment.

Related: Dynamic deployment, Release, Deployment, Publish

### Streaming exec

Streaming exec runs a command and sends its output to you as it happens, instead of waiting until the command finishes.

Use it for builds, installs, and test suites, where you want to watch progress or react to output early.

In MIOSA, `miosa exec my-box --stream -- npm test` streams the output, and so does any exec with a timeout above 300 seconds or `--timeout 0`.

Related: Exec, Background process, Event stream (SSE), Logs

### Template

A Template is the starting image a new Sandbox is built from: the operating system, languages, and tools that are already installed. MIOSA gives you two built-in choices, the default sandbox and the desktop, and your Organization can build its own.

Pick the closest match to your stack so the Sandbox is useful the moment it boots. `miosa-sandbox` is the default when you do not choose one.

In MIOSA, a Lumen team uses the default sandbox for its Next.js app and starts it on port 3000, or builds a custom template when they need a proprietary toolchain.

Related: Sandbox, Snapshot, Environment, Dockerfile, BuildSpec

### Timeout

A Timeout (also called a TTL) is the wall-clock time a Sandbox's active session may run. When it passes, a persistent Sandbox pauses and a non-persistent one is destroyed.

Set it so an abandoned Sandbox does not run, and bill, forever. Extending replaces the old value rather than adding to it, so work out the new total.

In MIOSA, `miosa create my-box --ttl 2h` sets two hours, and `miosa extend my-box --for 2h` replaces it with two hours from now.

Related: Idle timeout, Pause and resume, Persistence, Sandbox

### Version

A Version is one entry in a Deployment's history. It ties one Release to the configuration needed to run it, and its number only goes up.

Use Versions to compare, promote, and roll back. A Deployment has many Versions but one active at a time, and old ones stay bootable so going back is fast.

In MIOSA, Lumen's `team-portal` is on Version 17 while Version 18 is being checked. Promote 18 when it passes; roll back to 17 if it does not.

Related: Release, Deployment, Promotion, Rollback, Runtime Instance

### Worker

A Worker is a Service that does background work, such as processing a queue or sending emails, and never answers web requests.

Use one for anything that should keep running but that nobody visits. It has no public or private address, so other services reach it through a shared database or queue rather than by name.

In MIOSA, `miosa service create shop jobs --type worker --repo https://github.com/acme/api --start-command "node worker.js"` creates one.

Related: Service, Service type, Private service, Replica

## Organization

Who owns resources, and who may touch them.

### API key

An API key is a secret that lets software act on your behalf: servers, scripts, CI, SDKs, and the CLI. It is shown once, at creation.

Use one for anything that runs without a person typing a password. Keep keys on your backend and give browsers a short-lived browser token instead. Limit each key to the scopes it needs.

In MIOSA, `miosa api-key create ci --preset ci --expires-in 90` makes a 90-day key for sandboxes, deployments, and previews. If a key leaks, rotate it.

Related: Authentication, Browser token, Host key, Role, Scope, API key preset, Service account

### API key preset

An API key preset bundles the scopes a common kind of key needs, so you do not pick them one by one.

The named presets are `read-only`, `ci`, `agent`, `cli`, `automation`, and `full-access`. A preset is a starting point, not a ceiling: add `--scope` for anything it does not include, and the key lists exactly what it holds.

In MIOSA, `miosa api-key create ci --preset ci --expires-in 90` creates a key from a preset.

Related: API key, Scope, Service account

### Audit log

An Audit log is a record of actions: who did what, when, and from where. It exists so you can answer questions after something happens.

MIOSA keeps two kinds. The activity audit shows API actions in your Organization. A Sandbox's network audit log shows every outbound call that workload made.

In MIOSA, `miosa audit` shows recent activity, newest first. Revealing environment values is recorded there too.

Related: Egress, Logs, Secret, Member

### Authentication

Authentication is proving who is making a request: a person with a login, or a program with an API key or browser token.

It answers "who are you?" only. What you may do is a separate check, authorization.

In MIOSA, every API request carries `Authorization: Bearer <token>`. The token is either an API key (for servers) or a short-lived browser token (for a browser).

Related: Authorization, API key, User, Two-factor authentication

### Authorization

Authorization is deciding whether an identity that has already been proven may do a particular thing in a particular place.

A successful login does not make everything allowed. Roles limit people, and scopes limit API keys.

In MIOSA, a viewer can read a Workspace but not create a Sandbox, and a key without `deployments:write` cannot publish. The API tells you which scope is missing.

Related: Authentication, Role, Approval

### Feedback

Feedback is how you tell the MIOSA team about a bug, a missing feature, or anything that got in your way.

Send it from **Send feedback** in the console, where you can attach a screenshot, or from the terminal. Every message is read by a person, and replies come to your account email.

In MIOSA, `miosa feedback "miosa deploy hangs after the build finishes"` sends one, with your CLI version and operating system attached.

Related: CLI, Organization

### Host fingerprint

A Host fingerprint is the identifier for a machine's cryptographic identity. MIOSA uses it to recognize the same enrolled machine when it connects again.

It is why a machine can be told apart from a lookalike, and why its name, IP address, or host ID alone are not the identity.

In MIOSA, when your lab server reconnects after a reboot, its fingerprint is how MIOSA knows it is the same enrolled host.

Related: Host key, Host, OpenComputer

### Host key

A Host key is the secret an OpenComputer's agent uses to prove its connection is the machine you enrolled. It is printed once, when you register the host.

Save it right away, because it cannot be shown again. It is different from your API key: the Host key connects your server's agent, and your API key authorizes the request that asks it to run a command.

In MIOSA, `miosa host register studio --platform linux` prints the host ID and the Host key once.

Related: Host fingerprint, OpenComputer, API key

### Invitation

An Invitation lets you add someone who is not yet on MIOSA to an Organization or a Workspace. They get an email with a link and accept it.

Use a workspace invite to add them to one workspace, or an organization invite to add them to the whole organization first.

In MIOSA, `miosa org invite dana@example.com --role member` sends one. If the email already belongs to an org member, a workspace invite adds them directly.

Related: Member, Role, Organization, Workspace

### Member

A Member is a User who has been given a role inside one Organization. Membership is what actually grants access.

Use it to control who can see and change what. A User who signs in but has not been added to an Organization has access to nothing in it.

In MIOSA, you add Members with `miosa member add`, and their role (owner, admin, member, or viewer) decides what they can do. Only owners and admins can change membership.

Related: User, Role, Organization, Invitation

### Organization

An Organization is the top-level boundary for a customer or company on MIOSA. It owns members, plan, billing, domains, API keys, and policy.

Everything you create lives under one. It is the unit that gets billed and the wall that keeps one customer's resources away from another's, even if they use the same slug.

In MIOSA, the ownership chain runs Organization, Workspace, Project, then Sandbox or Deployment. Lumen is one Organization; its customers sit below it as Workspaces.

Related: Workspace, Member, Plan, Tenant, User

### Project

A Project groups the development and production resources for one product or app. It sits inside a Workspace.

Use it to keep Sandboxes, Deployments, data services, and environments for one product together. Every Project gets `production` and `staging` environments.

In MIOSA, Lumen's "Lead Magnet" is one Project holding the Sandboxes used to build it and the Deployment that serves it. If no project is given, a `default` project is used.

Related: Workspace, Sandbox, Deployment, external_project_id, Service, Project canvas, Preview environment

### Role

A Role is a named set of permissions given to a Member: owner, admin, member, or viewer.

Use the least powerful role that does the job. Viewers read, members create and manage resources, admins manage members and resources, and owners can also transfer and delete.

In MIOSA, a Northwind Agency designer is a viewer on a client's Workspace, while the account lead is an admin. A Role is permanent until changed; an approval is a one-time yes.

Related: Member, Authorization, Approval

### Scope

A Scope is one specific permission on an API key, such as `sandboxes:exec` or `deployments:write`. A key can only do what its scopes allow.

Give each key the fewest scopes it needs. Reads usually need the matching `:read` scope and writes need `:write`.

In MIOSA, presets bundle common sets: `read-only`, `ci`, `agent`, and `cli`. A call without the right scope gets a `403` that names the missing scope.

Related: API key, Role, Authorization, API key preset

### Service account

A Service account is an identity for a program, not a person. Its API keys keep working when the person who created them leaves the organization.

Use one for CI pipelines, backend automation, and anything in production. Prefer it over a personal key.

In MIOSA, `miosa service-account new deploy-pipeline` creates one, and `miosa service-account key new <account> --preset ci --expires-in 90` gives it a key.

Related: API key, API key preset, Member, Organization

### Two-factor authentication

Two-factor authentication asks for a second proof that it is you, a 6-digit code emailed to you, on top of your sign-in.

Turn it on for your own account, and an organization owner can require it for every member. A member without it is asked to turn it on before continuing.

In MIOSA, you turn it on from your account security settings.

Related: Authentication, Organization, Member, Role

### User

A User is one person or service identity that signs in to MIOSA. It is who you are, not what you own.

A User gets access by becoming a Member of an Organization, through an invitation or a role. One person can belong to several Organizations.

Be precise with "account". Use **User** for the identity that signs in and **Organization** for the customer boundary that owns resources and billing. In MIOSA, Alice at Acme Corp is a User; Acme Corp is the Organization.

Related: Member, Organization, Role, Authentication

### Workspace

A Workspace is an isolated area inside an Organization for one team, customer, or line of business. It holds Projects and everything created inside its boundary.

Use Workspaces to keep customers apart, split billing, and give a team its own space. In a platform you resell, each of your customers is typically one Workspace.

In MIOSA, a Northwind Agency Organization creates a Workspace per client. Resources, usage, and App Engine hosts are then separated per client.

Related: Organization, Project, external_workspace_id, App Engine Host, Slug

## Agents

AI agents, harnesses, runs, and the events they emit.

### Agent

An agent is a saved, versioned recipe for an AI worker: which harness runs it, which model, its instructions, which machine it uses, and what it is allowed to do.

Use a saved agent for work you repeat, like reviewing every pull request. You refer to it by name or id and run it on a Sandbox or Computer. Agents use your own model connections, never MIOSA platform keys.

In MIOSA, a Lumen team saves a `reviewer` agent once and runs `miosa agent run reviewer "review PR 42"` whenever a PR opens. App Engine is a hosting target, not an agent device.

Related: Harness, Agent Run, Runtime Profile, OSA, Approval, Computer use, Optimal, Agent Studio, Workflow

### Agent Run

An Agent Run is one tracked execution of an AI agent against a machine you chose. You give it a task in plain language, it works, and you get a stable record of what happened.

Use it when the task is "do this for me", not "run this exact command". An agent decides the steps; plain Exec runs a command you already know.

In MIOSA, a Lumen product sends "update the landing page and leave the preview running" to a Sandbox, and shows the run's output and status in its own UI.

Related: Agent, Agent Run Group, Job, Approval, Runtime Profile

### Agent Run Group

An Agent Run Group bundles many related Agent Runs so you can manage them as one operation. Use one when a single request fans out to many machines.

It tracks counts, streams events from every child, enforces a concurrency limit, and lets you cancel the whole group at once.

In MIOSA, a Northwind Agency request to "check 100 client pages" becomes one group with up to 25 runs at a time.

Related: Agent Run

### Agent Studio

Agent Studio is where you build and use your own agents. The chat is the main surface, with a workspace panel beside it.

Each chat gets one warm sandbox, and follow-ups reuse it. A paused sandbox is woken with its files; only a destroyed one is replaced, and the chat says so.

In MIOSA, an agent saved in Agent Studio can also be chosen as the brain of an Optimal chat.

Related: Agent, Brain, Sandbox, Harness

### AI Gateway

The AI Gateway sits between your applications and model providers. A routing policy says which model serves which requests and what to fall back to; a budget and limits cap spend and rate.

Use it to switch models without changing app code, and to see every request's trace.

In MIOSA, open **AI Gateway** in the console, or use `miosa gateway policy list` and `miosa gateway requests --limit 20`.

Related: Connection, Rate limit, Spend cap, Agent

### Approval

An Approval is a one-time yes that a person gives before a restricted step continues, such as a deploy or an external write.

Use it to keep a human in the loop for risky actions. It is not a permanent permission: it covers the step it was asked about.

In MIOSA, `miosa agent approvals list` shows waiting requests, and `miosa agent approvals approve` lets a paused run continue. Approval gating in the Computer-use protocol is a client-side convention, so gate sensitive steps in your own code too.

Related: Agent Run, Role, Authorization

### Brain

The Brain is the model or agent that does the work in an Optimal chat. You pick it before the first message.

- **Optimal**: MIOSA's own model, paid from wallet credits
- **A harness**: Claude Code, Codex, OSA, or another harness, running on a machine with your own account
- **A saved agent**: an agent you built in Agent Studio

In MIOSA, the brain is locked after the first message; start a new chat to change it.

Related: Optimal chat, Optimal, Harness, Agent Studio

### Computer use

Computer use is a loop where software looks at a screen, decides, takes one action, and looks again: screenshot, click, type, scroll. It lets an AI operate a real desktop.

Use it when the task depends on pixels: a logged-in browser, desktop apps, or a human taking over where automation stopped. For code and builds, use a Sandbox.

In MIOSA, the loop runs against a Computer through the SDKs, MCP tools, REST API, or live viewer. You bring your own agent; there is no managed loop.

Related: Computer, Desktop session, Agent, MCP

### Connection

A Connection is an account or key on another service that MIOSA keeps for you, so agents and workflows can use it without you pasting secrets into them.

There are three kinds: **Model provider** (Claude or ChatGPT sign-in, provider API keys), **App** (GitHub, Google, Slack and others, often by OAuth), and **Tool server**. For OAuth apps, MIOSA refreshes the tokens in the background.

In MIOSA, manage them under **Connections** in the console, or with `miosa connections`.

Related: Tool server, Secret, Agent, Harness

### Event stream (SSE)

An event stream keeps one connection open and pushes updates to you as they happen, using server-sent events (SSE). Command output and a final result arrive line by line instead of all at the end.

Use it to show progress live, like a build log or a terminal. It is one open request, unlike a Webhook, which MIOSA sends to your server later as a separate call.

In MIOSA, streaming exec ends with an exit event, and `miosa events my-box` streams a Sandbox's state changes.

Related: Webhook, Exec, Logs

### Evidence

Evidence is a recorded result that proves an operation actually reached the state you needed. It is a fact you can check, not a promise.

Use it instead of trusting "accepted" or "queued". A request being accepted proves only that MIOSA received it.

In MIOSA, publishing ends with proof that the route, runtime, and public URL work. For a connected host, a fresh heartbeat plus a completed command is the evidence.

Related: Readiness, Agent Run

### Harness

A harness is the agent program that actually does the work in a run: `claude-code`, `codex`, or `osa`. MIOSA provides the machine; the harness provides the thinking and tool use.

Pick one per agent or per run. A separate `--model` chooses the model it uses, and the harness defaults to `claude-code` if you do not say.

In MIOSA, `miosa prompt --new-machine --harness codex "write a README"` provisions a machine and has Codex write the README there.

Related: Agent, Agent Run, OSA, MCP

### MCP

MCP, the Model Context Protocol, is a standard way for AI apps to discover and use tools. MIOSA runs an MCP server, so a compatible client can create Sandboxes, run commands, and deploy by calling tools.

Use it when you want an assistant such as Claude Code, Cursor, or OSA to operate MIOSA directly, with the same API key as the CLI.

In MIOSA, `miosa mcp install --client claude` adds the server to Claude Code's config. The MIOSA REST API is separate; MCP is a tool layer over it.

Related: OSA, Harness, Agent, Tool server

### Memory

Memory is a workspace's store of knowledge. Optimal searches it to answer questions and cites what it used.

Ask Optimal in a chat to save a note, or import a Notion export from the terminal. Memory runs as an always-on machine in the workspace and is billed like one, so you start it on purpose.

In MIOSA, open it from **Optimal**, then **Memory**, to search what the workspace knows.

Related: Optimal, Optimal Engine, Optimal chat, Workspace

### Optimal

Optimal is the agent built into MIOSA. You describe what you want in a chat, and it answers from your live workspace, builds on a sandbox or computer you choose, or investigates a problem.

It works only inside your organization and workspace, with your permissions, and steps that change your workspace wait for your approval unless you allow them.

In MIOSA, open it from **Optimal** in the console sidebar, or use `miosa optimal chat "how many sandboxes are running?"` from a terminal.

Related: Optimal chat, Brain, Approval, Memory, Agent

### Optimal chat

An Optimal chat is one conversation with Optimal. Each chat is in one of three modes:

- **Ask** answers from live workspace context without touching a machine
- **Build** builds or changes something on a sandbox or computer
- **Investigate** traces evidence before reaching a conclusion

A chat belongs to the workspace it was started in and never moves. Chats are personal until you share them with the workspace.

In MIOSA, `miosa optimal chats` lists your chats, newest first.

Related: Optimal, Brain, Approval, Memory

### Optimal Engine

The Optimal Engine stores and searches knowledge for agents: notes, documents, and decisions, organized per workspace.

On MIOSA, a managed Engine runs like a managed database with engine `optimal`, with backups and upgrades. You can also run one yourself and keep a workspace in sync between a local and a cloud Engine.

In MIOSA, `miosa engine create` creates a managed Engine, and `miosa engine attach` connects a workspace to it.

Related: Memory, Optimal, Managed database, Backup

### OSA

OSA is MIOSA's open-source agent. It installs on your machine, works with your own Claude or ChatGPT subscription, and acts as an MCP client, so MIOSA's tools show up beside its built-ins.

Use it when you want an agent that already knows how to use MIOSA from your terminal or on a Computer. Computers ship with OSA installed but disabled until you activate it.

In MIOSA, `miosa prompt --harness osa --new-machine "summarize this repository"` runs it in a fresh machine.

Related: Harness, MCP, Agent, Computer, Optimal

### Runtime Profile

A Runtime Profile is a reusable set of defaults for how an agent runtime is prepared: which runner, tools, connectors, environment, and policy it gets.

Use one so you do not repeat the same settings on every run. A Profile shapes the runtime; each Agent Run still supplies the prompt, target, and timeout.

In MIOSA, Lumen makes a `lumen-claude` profile for Sandboxes and Computers, then references it from many runs.

Not to be confused with the `runtime_profile` field on sandbox create, which selects the VM engine. See [runtime_profile](/docs/glossary#runtime-profile-field).

Related: Agent Run, Agent, runtime_profile (sandbox field)

### Tool server

A Tool server gives an agent extra tools from another service through MCP. MIOSA keeps a catalog of official ones, including GitHub, Linear, Notion, Sentry, and Stripe.

Adding one asks for exactly the keys the vendor needs, tests the key before saving, and keeps it in the workspace vault.

In MIOSA, open **Connections**, then **Tool servers**, then **Browse catalog**.

Related: Connection, MCP, Agent

### Webhook

A Webhook is a message MIOSA sends to your own server when something happens, like a Sandbox becoming ready or a deployment succeeding. You do not have to keep asking.

Use it to update your database, notify a user, or start the next step. Each delivery is signed so you can check it really came from MIOSA.

In MIOSA, Lumen registers `https://example.com/hooks/miosa`, verifies the HMAC-SHA256 signature of the raw body, and uses the event `id` to ignore duplicates.

Related: Event stream (SSE), HMAC signature

### Workflow

A Workflow chains steps together, such as an agent run followed by a tool call, so a repeated process runs the same way each time.

Start a run by hand or from its triggers, pause the triggers when you need a break, and read each run's state.

In MIOSA, `miosa workflow run <workflow> --input '{"branch":"main"}'` starts a run, and `miosa workflow runs` lists recent ones.

Related: Agent, Agent Run, Cron job, Tool server

## Networking and domains

Hostnames, DNS, certificates, and routing.

### Apex domain

An Apex domain is the bare domain with no prefix: `example.net`, not `www.example.net` or `app.example.net`. It sits at the root of the domain.

It matters because the usual CNAME record is not allowed at the root, so you use A/AAAA, or a provider's ALIAS, ANAME, or flattened CNAME.

In MIOSA, if your DNS provider does not support flattening, such as GoDaddy, an apex domain needs a workaround. A subdomain like `app.example.net` is simpler.

Related: Custom domain, DNS record, DNS verification

### Custom domain

A Custom domain is one hostname you or your customer owns, such as `portal.example.net`, attached directly to a single Deployment.

Use it when a customer wants their own address on their app. You attach it, add the DNS record MIOSA shows you, and ask MIOSA to verify it. MIOSA then issues the HTTPS certificate.

In MIOSA, `miosa deploy domain-add team-portal portal.example.net` registers it and prints the record to add.

Related: Deployment domain, DNS verification, TLS certificate, Deployment, DNS record, Apex domain

### Deployment domain

A Deployment domain is a parent domain your Organization controls, used to give every production app a stable address like `lead-magnet.apps.example.com`.

Use it when you generate many apps and want them all under one branded domain. It is different from a Custom domain, which attaches one specific customer-owned hostname to one Deployment.

In MIOSA, if no deployment domain is set, apps use the managed fallback `{slug}.{organization}.miosa.app`. Follow [deployment domains](/docs/docker-host/domains) to choose the right setup.

Related: Custom domain, Preview domain, Deployment, Platform URL

### DNS record

A DNS record is an entry in your domain's settings that tells the internet where a name leads. The common ones are CNAME (point to another name) and A (point to an IP address).

When you attach a Custom domain, MIOSA shows the exact record to add at your DNS provider. Add it, then verify.

In MIOSA, `portal.example.net` uses a CNAME to the verification target. A bare domain like `example.net` needs an A/AAAA, ALIAS, ANAME, or flattened CNAME, depending on your provider.

Related: Custom domain, DNS verification, Apex domain, TLS certificate

### DNS verification

DNS verification is the check that proves the DNS record for your hostname points where MIOSA expects.

Do this after attaching a Custom domain. Until it passes, MIOSA will not route traffic or issue a certificate for that name.

In MIOSA, you add the CNAME or A record shown by `miosa deploy domains`, then run `miosa deploy domain-verify`. DNS changes can take time to spread, so retry if it fails right away.

Related: Custom domain, TLS certificate, Routing, DNS record

### Domain

A Domain is a hostname, like `portal.example.net`, that sends visitors to something on MIOSA: a Preview, a Deployment, a Computer, or another resource.

Use Domains so people reach your app at an address you can read and remember. A Domain routes to a Deployment, which serves its active Version, so the address stays the same when the code changes.

In MIOSA, Lumen's app answers at `lead-magnet.apps.example.com` and at Acme Corp's `portal.example.net`. Always show the `public_url` the API returns instead of building hostnames yourself.

Related: Deployment, Custom domain, Deployment domain, Routing

### Egress

Egress means outbound traffic: the calls a Sandbox or Computer makes to other services, such as an AI provider, GitHub, or a package registry.

Watch it to learn what your workload really contacts, then restrict it. This limits what code, especially AI-written code, can reach.

In MIOSA, new Sandboxes start in audit-only mode that records every outbound call and blocks none. When ready, lock down to an allowlist; `miosa egress allowlist` shows the allowed hosts.

Related: Network policy, Secret, Isolation, Sandbox

### Network policy

A network policy limits which outside addresses a machine may contact. It is a fence around what a workload can reach.

Use it to contain untrusted code or stop an AI agent from making surprise outbound calls. It limits where traffic can go; it does not log in to the destination.

In MIOSA, new Sandboxes start in audit-only mode that records every outbound call, and you can lock down to an allowlist later. Computers use `allowlist` policies with rules like `github.com` on port 443.

Related: Isolation, Sandbox, Egress

### Platform URL

A Platform URL is a MIOSA-provided address that works right away, with no DNS setup on your side.

Use it to test, to share a quick link, or as the always-available fallback while you set up a branded domain.

In MIOSA, a deployment gets a managed fallback such as `https://team-portal.example-org.miosa.app`. Previews also get platform hostnames ending in `.sandbox.miosa.ai`.

Related: Custom domain, Deployment domain, Preview, Public address

### Port forward

A Port forward makes a service inside a sandbox, such as a database or a debugger, appear on a port of your own computer.

Use it for anything that is not a web page, or when you do not want a public URL. For an HTTP port you want to share, create a Preview instead.

In MIOSA, `miosa proxy my-box 5432:5432` forwards local port 5432 to the sandbox's port 5432 until you press Ctrl-C.

Related: Preview, Sandbox, Tunnel, Scope

### Preview domain

A Preview domain is the branded parent domain used to build temporary Sandbox Preview addresses. It decides what the address of a Preview looks like.

Use it when you resell MIOSA under your own brand and want previews on your domain instead of MIOSA's. It is separate from the domain for production apps, so brand both if you want both branded.

For example, a made-up company called Lumen that owns `example.com` would see its previews at `https://5173-abc123.sandbox.example.com`. Lumen and `example.com` are examples, not real. DNS is one wildcard `CNAME`, `*`, pointing to `proxy.miosa.ai`.

Related: Preview, Deployment domain, Platform URL

### Private address

A Private address is the internal name one Service would use to reach another without going over the internet.

Private networking between services is not available on MIOSA yet: the API's `private_network` reads `{"enabled": false}`, and the older `private_domain` (`<slug>.miosa.internal`) and `MIOSA_PRIVATE_DOMAIN` do not resolve. Call another service at its public address.

Managed databases connect automatically: reference `${{Postgres.DATABASE_URL}}` or `${{Redis.REDIS_URL}}` in a Variable.

Related: Service, Private service, Public address, Network policy

### Public address

A Public address is the internet address of a web service or static site. It ends in `miosa.app`, and MIOSA issues and renews its TLS certificate.

Turn it on when the service should be reachable from the internet; turn it off and the service keeps running on its private address only. For your own domain, add a Custom domain.

In MIOSA, `miosa service domains enable shop API --label api` gives the service `https://api.<your-org>.miosa.app`.

Related: Service, Private address, Custom domain, Platform URL, TLS certificate

### Routing

Routing is how MIOSA decides where a request goes: from a hostname to the right Organization, Deployment, and active Version.

It is why changing the active Version, or rolling back, takes effect without touching DNS. The address stays; the route behind it moves.

In MIOSA, a request for `portal.example.net` is routed to Lumen's `team-portal` Deployment and served from its active Version. Edge routers refresh their route cache in under 60 seconds by default.

Related: Domain, Deployment, DNS verification

### Sandbox share

A Sandbox share is a link that lets someone watch a running sandbox without an account. They can look, not drive.

Use it to show a teammate or a customer what an agent is doing. Revoke the share and the link stops working at once.

In MIOSA, a share URL looks like `https://{sandbox-id}.sandboxes.miosa.ai?ms=ms_<token>`, and it can be created with an expiry.

Related: Share token, Preview, Sandbox

### Share token

A Share token is a short-lived, scoped credential that lets a specific browser open a private Preview without exposing your API key.

Use it to embed a Preview in your product or send a time-limited link. It is returned once, so store it or mint a new one.

In MIOSA, `miosa preview share my-box <id> --ttl 1h` produces a link valid for an hour. The proxy strips the token before the request reaches your app.

Related: Preview, Browser token, API key

### TLS certificate

A TLS certificate (often called SSL) is what makes the padlock and `https://` work for a hostname. It encrypts traffic and shows the site is the one it claims to be.

You do not buy or install one. MIOSA issues it automatically after your hostname's DNS is verified.

In MIOSA, once `portal.example.net` verifies, HTTPS starts working. The certificate does not prove you own the domain registration; DNS verification does.

Related: DNS verification, Custom domain

### Tunnel

A Tunnel publishes one port of a connected host at a MIOSA-managed address. MIOSA routes requests back down the host's outbound connection to your service.

Use it to reach something on a machine behind a firewall, like a dev server on your lab machine. Connecting a host exposes nothing; a tunnel is a separate, explicit exposure with its own access policy.

In MIOSA, `miosa host tunnel new <host> --port 3000 --slug my-app-dev --access organization` creates one reachable by your organization.

Related: OpenComputer, Preview, Host

## Data

Configuration, secrets, and durable storage.

### Backup

A Backup is a saved copy of a database taken at a point in time, so you can restore it after a mistake or failure.

Take one before risky changes. Remember that rolling back a Deployment does not roll back your data; a restore does.

In MIOSA, `miosa db backup <database>` takes one now and `miosa db backups <database>` lists them. Backups are stored outside the database runtime.

Related: Managed database, Postgres, Snapshot, Rollback

### Binding

A Binding is a lasting link between an app and something it depends on: a database, a volume, a secret, a storage bucket, or a service.

Use bindings so the app gets the right connection details for each environment without copying data into the app. The data keeps its own life, separate from deploys.

In MIOSA, you attach Postgres to a Project and `DATABASE_URL` is injected at boot. Staging gets the staging database, production gets the production one, with the same variable name.

Related: Deployment, Managed database, Volume, Secret, Object storage, Connection string

### Bucket

A Bucket is a named container in object storage. You put files in it by key, such as `uploads/foo.png`, and get them back later.

Use one for user uploads and generated assets, kept apart from your app's code and database.

In MIOSA, each Project's bucket is reached through an S3-compatible API, and `STORAGE_BUCKET` is injected into your app.

Related: Object storage, Presigned URL, Binding

### Connection string

A Connection string is a single line, like `postgresql://user:pass@host:port/db`, that holds the address, name, and credentials needed to connect to a database.

Treat it like a password. Your app reads it from an environment variable, usually `DATABASE_URL`, so you never hard-code it.

In MIOSA, you attach a database to an environment and MIOSA injects the right connection string at boot. `miosa db connection <database>` prints it.

Related: Managed database, Binding, Secret, Environment variable

### Data residency

Data residency is where your data is stored and processed across a whole workflow: the files, the outputs, the artifacts, and any requests sent to an AI model.

Check it when rules or customers require data to stay in a place. Running a command on your own machine does not by itself keep the result or model requests there.

In MIOSA, a resource lives in one region, such as `us-west`, and data services live in that region. For OpenComputers, review the [data boundaries](/docs/opencomputers/security#data-boundaries).

Related: Data plane, On-premises, OpenComputers

### Environment variable

An environment variable is a named setting handed to a program when it starts, such as `DATABASE_URL` or `API_BASE_URL`. It keeps configuration out of your code.

Use them for values that differ by place: staging versus production, or one customer versus another. Names must be uppercase, like `NEXT_PUBLIC_API_URL`.

In MIOSA, a value can come from several layers. When names conflict, the more specific layer wins: Organization, then Workspace, Project, the resource itself, and finally a single run.

Related: Secret, Environment, Binding, Connection string, Shared variable, Variable reference

### Log drain

A Log drain sends your logs to a place you choose, such as your logging service, as they happen.

Use it to keep logs longer, search them with your own tools, or alert on them. A drain can be HTTPS, with one JSON object per line and a signing secret, or syslog over TLS.

In MIOSA, `miosa drains add --name ops --url https://logs.example.test/ingest` creates one, and `miosa drains test ops` sends one signed line to check it.

Related: Logs, Service, Webhook

### Managed database

A Managed database is a database that MIOSA creates, runs, backs up, and connects for you. You choose the engine; MIOSA handles the server.

Use one when your app needs to remember things: users, orders, search. It is separate from your compute, so replacing a Sandbox or publishing a new Version does not delete it.

In MIOSA, you can create Postgres, MySQL, Redis, or Qdrant. `miosa db list` shows them, and their connection string is handed to your app as an environment variable.

Related: Binding, Volume, Object storage, Postgres, Redis, Vector database, Backup, Connection string

### Object storage

Object storage keeps files in buckets and hands them back by name: images, uploads, exports, generated assets. It is built for many files and large ones.

Use it for anything users upload or an agent generates that should outlive any one machine. It is not a filesystem you mount; you reach it through an S3-compatible API or signed links.

In MIOSA, each Project gets a bucket with `STORAGE_BUCKET`, `STORAGE_ENDPOINT`, and key variables injected, and standard S3 libraries just work.

Related: Volume, Binding, Managed database, Bucket, Presigned URL

### Postgres

Postgres is a relational database for structured data with tables, joins, and transactions. MIOSA can run it for you per Project.

Choose it for users, orders, and anything you query across. It lives separately from your compute, so replacing a Sandbox or publishing a new Version does not delete it.

In MIOSA, create one with `miosa api POST /databases` and read `DATABASE_URL` at runtime. `miosa db backup <database>` takes an on-demand backup.

Related: Managed database, Connection string, Backup, Binding

### Presigned URL

A Presigned URL is a temporary link, signed by MIOSA, that allows one upload or download of one file without needing an API key.

Use it to let a browser send a large file straight to storage, skipping your server. Large direct uploads over 5 MB should use this path.

In MIOSA, `POST /storage/buckets/{id}/presign` with a `key` and `method` returns one.

Related: Bucket, Object storage, Share token

### Redis

Redis is a very fast key-value store that keeps data in memory. It is typically used for caching, sessions, rate limiting, and pub/sub.

Use it to make your app faster or to hold short-lived data. Treat it as cache-class, not the home of your most important records; use Postgres for those.

In MIOSA, a managed Redis instance is created per Project and exposed to your app as `REDIS_URL`. It is single-node.

Related: Managed database, Connection string, Binding, Postgres

### Secret

A Secret is a sensitive value, like a payment or AI provider key, that MIOSA stores protected, with restricted and audited access to the plaintext.

Use it instead of plain configuration for anything that would be harmful if leaked. A secret is bound to the machine as an opaque placeholder, so your real key never lives inside the machine.

In MIOSA, an agent's code reads `OPENAI_API_KEY` and gets a placeholder; the real key is swapped in only on outbound calls. You rotate it in one place and every machine picks up the change.

Related: Environment variable, Binding, API key

### Shared variable

A Shared variable is set once on an Environment and every Service in that environment can read it, such as an API base URL used by several services.

Use it instead of copying the same value into each service. A service reads it through a reference, and changing it marks every service that reads it as needing a redeploy.

In MIOSA, `miosa project vars set shop API_BASE=https://api.acme.dev` sets one, and a service reads it as `${{shared.API_BASE}}`.

Related: Variable reference, Environment variable, Service, Environment

### Variable reference

A Variable reference points at a value instead of copying it. MIOSA resolves it when the release starts, so the service always gets the current value.

- `${{Postgres.DATABASE_URL}}`: a variable of another service in the same environment
- `${{shared.API_BASE}}`: a shared variable
- `${{NAME}}`: another variable of the same service
- `${{secret(32)}}`: a generated random secret of 32 characters

In MIOSA, `miosa service vars effective shop API` shows every resolved variable, still masked, and any problems that would block a deploy.

Related: Shared variable, Environment variable, Secret, Service

### Vector database

A Vector database stores embeddings, which are lists of numbers that capture meaning, and finds the closest matches. It is how search by similarity works.

Use it for retrieval-augmented generation (RAG), document search, recommendations, and long-term agent memory.

In MIOSA, managed Qdrant gives each Project a dedicated vector database with `QDRANT_URL` injected. Keep raw files in object storage and the vectors in Qdrant.

Related: Managed database, Object storage, Connection string

### Volume

A Volume is a persistent disk you can attach to a Computer. It keeps what you write even if the Computer restarts or is replaced.

Use it for things that must survive and live on the filesystem: a local SQLite database, a cache you cannot lose, or large model weights downloaded once. For a real database use Postgres, and for user uploads use object storage.

In MIOSA, `miosa volume create model-weights --size-gb 50` makes a 50 GB volume. Volumes live on one host, so they do not move automatically across hosts.

Related: Binding, Object storage, Release

## Billing and usage

Credits, metering, limits, and plans.

### Auto-reload

Auto-reload adds credits from your card when your balance falls under an amount you choose, never more than a monthly maximum. The organization's owner or a billing manager turns it on in Billing, under Budgets & limits: a secure checkout adds the first reload now and saves the card with your approval for these charges.

With auto-reload off, or when a charge is declined, a balance of $0 pauses running sandboxes and computers until you add credits, and databases and deployments stop. Nothing is deleted.

Related: Credit, Wallet, Credit pack, Spend cap

### Credit

A Credit is the unit MIOSA measures and bills usage in. One credit is one cent. Running machines, storage, builds, and AI calls draw credits from your wallet balance.

Credits come from your plan's included usage each month and from credit packs you add. Credits you add never expire; monthly plan credits expire a month after they are granted.

In MIOSA, `miosa billing balance` shows your balance, and `miosa limits` shows which wallet the next machine would bill and whether it can start.

Related: Usage, Plan, Spend cap, Wallet, Auto-reload, Credit pack, Included usage

### Credit pack

A Credit pack is a set amount of credits you add to your wallet in one payment.

Add credits when you want more than your plan includes, or to keep a Free account running past its $1. Credits you add never expire. On Free, any added balance above $0 raises your limits to "Free with added credits".

In MIOSA, add credits from **Billing** in the console. The packs offered are $5, $25, $50, and $100.

Related: Credit, Auto-reload, Wallet, Included usage

### Included usage

Included usage is the amount of usage your plan gives you each month. A paid plan's price comes back as usage: Business is $500 a month for $500 of usage, spent at the published prices.

Free includes $1 of usage every month and stops there unless you add credits. Paid plans keep going past the included amount, pay as you go.

In MIOSA, recurring plan credits expire a month after they are granted; credits you add never expire.

Related: Plan, Credit, Credit pack, Auto-reload

### Plan

A Plan is your organization's subscription tier. There are four: Free ($0, $1 of usage a month), Developer ($20 a month), Business ($500 a month), and Enterprise ($2,000 a month). A paid plan's price comes back as the same amount of usage.

Every plan pays the same price for a size. Plans differ in included usage and in how much you can run: replicas, sizes, and how many sandboxes, computers, databases, and projects at once.

In MIOSA, `miosa billing plan` shows your plan and usage against its limits. Change plan in **Billing** in the console.

Related: Organization, Credit, Quota, Included usage, Credit pack

### Price book

The Price book is the list of rates MIOSA charges: each size per hour and per month, disk, egress, builds, snapshots, and model tokens.

Each price book has a version and an effective date, so you can tell which prices applied when. Plans never change the price of a size, only the limits.

In MIOSA, `GET /api/v1/pricing` returns the current price book; the pricing page is built from it.

Related: Plan, Credit, Usage

### Quota

A Quota is a limit you set on how much of something can be used: how many Sandboxes, how many at once, how much storage, or how much spend.

Use it to stop one user from using everything. A Quota is the cap; usage is the current count against it.

In MIOSA, a Lumen platform caps each end user at 3 concurrent Sandboxes. Quotas are checked when a Sandbox is created, so they do not affect ones already running.

Related: Usage, Spend cap, Plan, Rate limit

### Rate limit

A Rate limit caps how fast you can call the API. Go over it and you get an HTTP `429` with a `Retry-After` header telling you how long to wait.

Back off for that time and try again. Do not retry in a tight loop. Read the `X-RateLimit-*` headers rather than hard-coding numbers.

In MIOSA, the documented limits are 600 requests per minute per workspace and 60 per minute per endpoint group, and they can vary by key and policy. Contact support to raise one for a heavy integration.

Related: Quota, API key, Plan

### Spend cap

A Spend cap limits how much can be spent in a month. Set one on the organization, on a workspace, or on one member.

When a cap is reached it either pauses, so new spend is refused and running workloads are stopped but never destroyed, or it only alerts. Alerts go out at 50, 80, 90, and 100 percent by default, by email and webhook. Nothing resumes by itself after a pause.

In MIOSA, `miosa billing cap show` lists the caps, and `miosa org caps set dana --spend 25` limits one member to 25 dollars a month.

Related: Quota, Credit, Usage, Auto-reload, Wallet

### Usage

Usage is the measured amount of resources you consumed, attributed to an Organization, Workspace, or resource: compute seconds, storage, bandwidth.

Use it to see where credits go, and, if you resell MIOSA, to bill your own customers. Usage is not an invoice; it is the underlying measurement.

In MIOSA, `miosa usage my-box` shows a Sandbox's runtime, time left, and estimated cost. Platform builders group usage by `external_workspace_id` to charge each customer.

Related: Credit, Quota, external_workspace_id, Wallet, Included usage

### Wallet

A Wallet is where the money for your usage comes from: a balance of credits and, optionally, a card.

Each person has a personal wallet and each organization has one. A machine bills the wallet it was created in, even if you switch organizations later.

In MIOSA, `miosa org` shows who pays for new machines right now, and `miosa org bill acme` makes new machines bill the acme organization.

Related: Credit, Auto-reload, Credit pack, Organization

## Platform builders

Words for building a product on top of MIOSA.

### API

The MIOSA API is a REST interface: your code sends HTTPS requests, and MIOSA creates, changes, or reports on resources.

Everything in the console and CLI is built on it. Use it directly when you want full control from your own backend.

In MIOSA, the base URL is `https://api.miosa.ai/api/v1`, and every request carries `Authorization: Bearer <token>`. SDKs and the CLI are friendlier wrappers.

Related: CLI, SDK, API key, Idempotency key, Rate limit

### Browser token

A Browser token is a short-lived credential your server creates so that one customer's browser can reach one MIOSA resource, such as a Preview or a terminal.

Use it so your real API key never reaches the browser. The token expires, cannot be renewed, is tied to a single resource, and cannot create new resources.

In MIOSA, Lumen's backend mints a token for a Sandbox preview and hands only that to the customer's browser, in an iframe URL.

Related: API key, White-label, Preview

### CLI

The CLI is the `miosa` command-line program. You type commands to create Sandboxes, run code, publish apps, and manage your account.

Use it for quick work and scripts. It exits with the remote command's exit code, so it works in CI.

In MIOSA, install with `npm i -g @miosa/cli`, then sign in with `miosa login`.

Related: SDK, API, API key, MCP

### external_project_id

`external_project_id` is the project or app ID from your own database. MIOSA stores it as an opaque string, accepts it as a list filter, and copies it onto usage, audit, and child resources.

Use it to map a MIOSA Project back to a row in your system without extra lookups.

In MIOSA, passing `external_project_id: "project_789"` without a canonical ID or slug resolves or creates the matching Project in your Organization.

Related: Project, external_workspace_id, external_user_id

### external_user_id

`external_user_id` is the ID of one end user inside your customer's account, as you identify them. MIOSA stores it and copies it onto usage, audit, and child resources.

Use it to enforce per-person limits, for example counting a person's active Sandboxes before creating another, and to pull a single customer-specific audit trail.

In MIOSA, creating a Sandbox with `external_user_id: "alice"` makes it easy to find all of Alice's work and to cap it with a Quota.

Related: external_workspace_id, external_project_id, Quota, User

### external_workspace_id

`external_workspace_id` is your customer's account ID from your own system. MIOSA stores it as an opaque string and copies it onto usage, audit, and child resources so you can match things up later.

Use it to bill each customer. It is the recommended key for rolling up usage when you resell MIOSA compute.

In MIOSA, passing `external_workspace_id: "acct_123"` on create links the new Sandbox to that customer. If no canonical ID or slug is given, MIOSA resolves or creates the matching Workspace.

Related: Workspace, external_user_id, external_project_id, Usage, Tenant

### HMAC signature

An HMAC signature is a checksum made with a secret that only you and MIOSA know. It proves a message is genuine and unchanged.

Always check it before trusting a Webhook. Compute the signature of the raw request body and compare.

In MIOSA, the `x-miosa-signature` header is `sha256=` plus the hex HMAC-SHA256 of the raw body under your webhook's secret, which is shown once at creation.

Related: Webhook, Secret, API key

### Idempotency key

An Idempotency key is a unique string you send with a request. If you retry with the same key, MIOSA returns the original result instead of doing the work again.

Use it on every create so a network hiccup never makes a duplicate Sandbox or double billing.

In MIOSA, send `Idempotency-Key: create-sandbox-for-alice-2026-05-17`. Official SDKs generate one automatically on create.

Related: API, Rate limit, SDK

### Metadata

Metadata is a small free-form note you attach to a MIOSA resource: up to 16 key-value text pairs. MIOSA stores it and gives it back.

Use it for your own context, like plan tier, feature flags, or a customer label, so you do not need a separate table just to remember which resource is which.

In MIOSA, `miosa create my-box --meta team=infra` tags a Sandbox, and webhook payloads include the metadata you set.

Related: external_workspace_id, external_user_id

### SDK

An SDK is a software development kit: a library for your programming language that wraps the API. You write `miosa.sandboxes.create(...)` instead of crafting HTTP requests.

Use one in your backend. It handles authentication, retries, and typed errors.

In MIOSA, official SDKs exist for TypeScript, Python, Go, Elixir, Java, and Rust. Coverage varies by language.

Related: API, CLI, API key

### Slug

A Slug is a short lowercase name that appears in addresses and APIs, like `acme-corp` or `lead-magnet`. It is easier to read than an ID.

Use slugs for Workspaces and Projects, and expect them in URLs. Your Organization slug is permanent once set.

In MIOSA, `workspace_slug: "acme-corp"` selects or creates a Workspace, and a deployment's managed URL starts with its slug.

Related: Workspace, Project, Organization, Domain

### Tenant

"Tenant" is an older word for an Organization. You will still see it in some API and docs names, such as tenant branding, tenant events, and the tenant preview domain.

It is the top level of the hierarchy from a platform builder's view: your Organization holds the server key, and your customers sit below it as Workspaces. Prefer **Organization** in new writing.

In MIOSA, resources are scoped under your Organization. Another Organization cannot read or change yours, even if it uses the same slug or external ID values.

Related: Organization, Workspace, White-label, external_workspace_id, Preview domain, Slug

### White-label

White-label means running MIOSA behind your own product, so your customers see your brand and your domains and never learn MIOSA is underneath.

Use it when you are building a product, such as an AI app builder, and want MIOSA to supply the compute while you own the user experience, pricing, and data.

In MIOSA, the order matters: set the Organization identity first, create Workspaces and Projects for downstream customers, configure preview and deployment domains separately, then attach exact custom domains where a customer wants their own hostname.

Related: Tenant, Preview domain, Deployment domain, Custom domain, Browser token

## Runtimes

Engines, hosts, and where work actually runs.

### App Engine

App Engine is MIOSA's model for running many deployments for one Workspace on a single always-on host, each as its own container.

Use it for agencies and platforms with many small apps, funnels, APIs, or client sites. MIOSA keeps routing, domains, and billing attribution per Deployment.

In MIOSA, publish with `miosa deploy create --from-sandbox my-box --name team-portal --type docker_deploy --wait`. A publish is not marked ready until the host is healthy.

Related: App Engine Host, Deployment, Container, Workspace, Service

### Bring Your Own Cloud (BYOC)

Bring Your Own Cloud (BYOC) means running MIOSA worker capacity inside your own AWS account or Google Cloud project, while MIOSA remains the control plane.

Use it when you must keep compute in your own cloud account for governance, cost, or policy reasons, but still want MIOSA's product and orchestration.

In MIOSA, BYOC is governed worker capacity. It is different from OpenComputers, which connects individual machines. AWS BYOC sandbox capacity is a gated private preview.

Related: Worker pool, Customer-cloud region, OpenComputers, Control plane, Hybrid infrastructure

### Capability

A Capability is something a resource says it can do, such as running commands or streaming a desktop.

Check capabilities before you rely on a target. Two machines can share the same interface yet support different operations.

In MIOSA, a headless server can report command execution without desktop streaming. Use the infrastructure capability comparison before choosing where to run.

Related: Host, Readiness, Computer, Placement

### Container

A Container is an app packaged with everything it needs to run, and run in its own bounded space on a machine. Many containers can share one machine.

On MIOSA you meet containers in two places: Docker works inside most Sandbox templates, and App Engine runs your published apps as containers on a workspace host.

In MIOSA, `docker run --rm alpine echo hello` works in a Sandbox. A container is not a Sandbox: the Sandbox is a whole VM that can run containers.

Related: Dockerfile, App Engine, microVM, Sandbox

### Control plane

The Control plane is the MIOSA service that decides and remembers: what should exist, who may do what, where work is placed, and what state each resource should be in.

It is the brain, not the muscle. The machines that actually run your workloads belong to the data plane.

In MIOSA, when you create a Sandbox, the control plane records it, checks policy, and picks a host. With BYOC, MIOSA stays the control plane while the workers sit in your own cloud account.

Related: Data plane, Desired state, Placement, Bring Your Own Cloud (BYOC)

### Customer-cloud region

A Customer-cloud region is an approved worker region running in your own cloud account or project. It is capacity you own, controlled through MIOSA.

Use it when policy says a workload must run in a specific cloud location you control.

In MIOSA, you create one with the `miosa cloud region` commands after preflight checks, and placement can target it. It is not one of MIOSA's own regions like `us-west`.

Related: Bring Your Own Cloud (BYOC), Worker pool, Placement

### Data plane

The Data plane is where the actual work and data handling happen: the hosts, machines, and runtimes that execute your code and touch your files.

It matters for questions like "where does my data go?" The control plane authorizes and orchestrates; the data plane does the processing.

In MIOSA, a command runs on a Sandbox host in the data plane. With OpenComputers, that can be hardware you own.

Related: Control plane, Data residency, Host

### Dedicated vCPU

A Dedicated vCPU size gives a Service CPU it owns outright. Each vCPU is one hardware thread of the host, pinned to your service and never shared with another workload.

Use it when you need steady CPU with no noisy neighbors. Memory is 4 GiB per vCPU, and dedicated sizes are available on Business and Enterprise.

In MIOSA, sizes are named like `dedicated-2` (2 vCPU, 8 GiB).

Related: Service, Replica, Plan, Host

### Firecracker

Firecracker is the open-source microVM engine that MIOSA Sandboxes run on by default. A microVM is a very small, fast-starting virtual machine.

It is why a Sandbox feels like a real computer but starts in moments, and why one customer's workload cannot reach another's.

In MIOSA, Firecracker is the default sandbox engine: if you never set `runtime_profile`, you get `standard`, which is Firecracker. SOMA, MIOSA's own engine, is in early access.

Related: SOMA, Sandbox, microVM, runtime_profile (sandbox field)

### Heartbeat

A Heartbeat is a regular small message from a connected host saying "I am still here and talking to MIOSA".

It tells you the agent is alive. It does not prove an app, a desktop, or a particular command works.

In MIOSA, treat a fresh heartbeat plus a completed command as evidence a host is usable.

Related: Host, Readiness, OpenComputer

### Host

A Host is the physical or virtual machine that work actually runs on. Its runtime and the capabilities it reports decide which operations it can do.

You rarely pick one on MIOSA's managed fleet. With OpenComputers, you enroll your own machine as a host and address it by host ID.

In MIOSA, a Sandbox, a desktop session, or a deployment runs on a host but is not the host. `miosa host get <host>` shows its state, platform, region, tags, and capabilities.

Related: Capability, Worker pool, Heartbeat, Host key, OpenComputer

### Hybrid infrastructure

Hybrid infrastructure means using more than one kind of location together, such as MIOSA-managed compute and your own servers, with each workload placed by policy.

Use it to keep sensitive or governed work on your infrastructure and run the rest on MIOSA.

In MIOSA, hybrid does not mean automatic overflow or migration. You choose explicit placement and check each target's capabilities.

Related: Placement, Bring Your Own Cloud (BYOC), MIOSA-managed infrastructure, OpenComputers

### Isolation

Isolation is the wall between a workload and everything else: other customers, the host, and other workloads. It stops one thing from reading or breaking another.

It is what lets you run untrusted or AI-written code safely. Different resource types use different boundaries, so do not assume they are all the same.

In MIOSA, each Sandbox is its own microVM with its own kernel, and production runtimes are isolated from other tenants and from the build Sandbox.

Related: SOMA, Firecracker, Network policy, Sandbox

### microVM

A microVM is a stripped-down virtual machine. It behaves like a full computer, with its own operating system kernel, but boots in moments and uses little overhead.

It is why MIOSA can give every Sandbox and Computer a real, separate machine instead of sharing one. A workload cannot see another workload's files or processes.

In MIOSA, Sandboxes run on Firecracker microVMs by default, and SOMA is MIOSA's own engine.

Related: Firecracker, SOMA, Isolation, Sandbox

### MIOSA-managed Computer

A MIOSA-managed Computer is a Computer whose underlying capacity and lifecycle MIOSA operates. It is the default kind when you create one.

Use it for interactive desktop work without maintaining hardware. MIOSA keeps ready desktop capacity so a new one can be assigned quickly.

In MIOSA, `miosa computer create my-desktop` makes one. An OpenComputer, by contrast, is a machine you own.

Related: Computer, OpenComputer, MIOSA-managed infrastructure, Warm capacity

### MIOSA-managed infrastructure

MIOSA-managed infrastructure is compute capacity and hosts that MIOSA operates for you: hardware, networking, host lifecycle, capacity, and placement.

Use it to start without owning servers. You create resources and MIOSA keeps the machines running.

In MIOSA, every default Sandbox and Computer runs here. You are responsible for your application and data; MIOSA handles the machines.

Related: MIOSA-managed Computer, Shared responsibility, Hybrid infrastructure

### On-premises

On-premises means hardware that runs at your own site: an office, a lab, or a data center. It names a location, not a product.

Use it when a machine physically lives with you, perhaps for data rules or because you already own it.

In MIOSA, you can connect an on-premises server through OpenComputers. The word does not mean a self-hosted copy of MIOSA.

Related: On-premises server, OpenComputers, Data residency

### On-premises server

An on-premises server is a server you operate at your own site that can connect to MIOSA through OpenComputers.

A single one can therefore also be an OpenComputer. You keep responsibility for its hardware, power, operating system, and network.

In MIOSA, you install the host agent, register the machine, and verify with a `hostname` command. See [connect your first server](/docs/opencomputers/connect).

Related: On-premises, OpenComputers, OpenComputer

### OpenComputer

An OpenComputer is a machine you control that you have connected to MIOSA through OpenComputers: a server, workstation, GPU box, or existing hosted server.

It can run anywhere. The connection model does not define its physical location.

In MIOSA, your lab server, once registered, appears as an OpenComputer you can run commands on. It is different from BYOC worker capacity.

Related: OpenComputers, Computer, MIOSA-managed Computer, Host key, Host fingerprint, On-premises server, Tunnel

### OpenComputers

OpenComputers is the system for connecting machines you already own to MIOSA and operating them remotely: commands, containers, tunnels, CI runners, backups, and agent work.

Use it when work must run on hardware you control. You install a host agent, it opens an outbound connection, and MIOSA manages the machine as a host.

In MIOSA, connecting a machine does not install the full MIOSA control plane on it. You keep the hardware, OS, and the decision about what the agent may reach.

Related: OpenComputer, On-premises server, Host key, Bring Your Own Cloud (BYOC), Data residency

### Placement

Placement is the decision of where an eligible workload will run: which host or pool among those allowed.

It follows policy and capability, such as region, size, and whether a host supports what you need. You normally do not choose a host directly.

In MIOSA, the scheduler matches a workload's CPU, RAM, and region needs against the free capacity of the fleet. In hybrid setups, policy can keep governed workloads on your own infrastructure.

Related: Worker pool, Hybrid infrastructure, Capability, Control plane

### Region

A Region is the place where a Sandbox or Computer runs. Each resource is created in exactly one region and stays there for its lifetime.

Pick by data rules first, then by closeness to your users or CI so connections are quicker. The three US regions perform about the same for most people.

In MIOSA, regions are `us-west` (Los Angeles, the default), `us-east` (New York), and `us-mia` (Miami). `miosa regions` shows what is available and whether there is capacity.

Related: Placement, Data residency, Sandbox, Computer, Customer-cloud region

### Replica

A Replica is one of several identical copies of a dynamic app running at the same time. More replicas handle more visitors and keep the app up if one fails.

By default a dynamic service has one. Raise it when you need more capacity or resilience.

In MIOSA, you set `desired_replicas`, and the reconciler spreads copies across hosts, and across zones where possible. Traffic is shared among healthy replicas.

Related: Runtime Instance, Dynamic deployment, Reconciliation

### runtime_profile (sandbox field)

A field on the create-sandbox request body that selects the engine. `standard` is Firecracker (the default, no extra approval), `soma` is SOMA (needs an enabled policy and capacity), and `auto` lets policy decide and may pick Firecracker. Omitting the field is treated exactly as `standard`.

Related: Firecracker, SOMA, Sandbox, Runtime Profile

### Shared responsibility

Shared responsibility is the written split of who looks after what: MIOSA, you, or a cloud provider, depending on where your workload runs.

Read it before choosing infrastructure. With MIOSA-managed compute, MIOSA runs the machines. With OpenComputers, you own the hardware and OS. With BYOC, you own the cloud account and MIOSA runs the control plane.

In every model, the application owner owns application data and backup policy.

Related: MIOSA-managed infrastructure, Bring Your Own Cloud (BYOC), OpenComputers

### SOMA

SOMA, the Secure Optimized Machine Architecture, is MIOSA's own open-source virtual machine monitor, written in Rust under Apache 2.0. It gives a Sandbox its own hardware-isolated Linux machine.

You do not run it. MIOSA operates SOMA for you, and it is in early access, enabled per Organization. It exists so MIOSA owns the exact moment a machine becomes ready to run commands.

In MIOSA, SOMA starts a sandbox by restoring a certified snapshot, then proves the guest is alive before calling it ready.

Related: Firecracker, Isolation, Sandbox, Snapshot, microVM, runtime_profile (sandbox field)

### Warm capacity

Warm capacity is compute kept ready in advance so a new Computer or Sandbox can start faster instead of cold booting.

It is why a managed Computer can be handed over in under a second on the warm path. It is not a guarantee that every size is available instantly.

In MIOSA, the managed fleet keeps ready desktop and sandbox slots on every host.

Related: Computer, Sandbox, MIOSA-managed Computer

### Worker pool

A Worker pool is a group of compatible hosts that may run a certain kind of workload under placement policy.

It gives MIOSA a set of machines to schedule onto. Mere registration is not enough: members must be compatible and able to do the job.

In MIOSA, a Bring Your Own Cloud region is a pool of worker capacity inside your AWS account or Google Cloud project.

Related: Host, Placement, Bring Your Own Cloud (BYOC), Capability

## Keep learning

- [Core concepts](/docs/concepts): Understand the complete resource hierarchy and immutable production model.

- [Quickstart](/docs/quickstart): Create, preview, publish, verify, and promote an application.

- [Deployment domains](/docs/docker-host/domains): Choose between platform URLs, deployment domains, and custom domains.

- [Customer infrastructure](/docs/infrastructure/overview): Compare MIOSA-managed compute, on-premises servers, OpenComputers, BYOC, and hybrid placement.
