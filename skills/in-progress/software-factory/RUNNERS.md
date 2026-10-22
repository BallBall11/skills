# Runners

Surveyed 2026-10 from vendor docs and pricing pages. Prices and limits drift; re-check the linked page before quoting a number to the user.

A runner is **disposable**: every run starts on a fresh machine and leaves nothing behind. Five types fill the slot. They differ mostly in who owns the agent loop and who owns the machine:

| Type | Agent loop | Machine | Also fills |
| --- | --- | --- | --- |
| CI runner | yours, inside the CI job | CI vendor's, or yours | orchestrator, observability platform |
| MicroVM API | yours, in the orchestrator or on the runner | provider's | nothing |
| Bundled agent platform | vendor's | vendor's | part of orchestrator, observability platform |
| Bring-your-own worker | vendor's | yours | part of orchestrator, observability platform |
| Own VPS or containers | yours | yours | nothing |

## CI runners

A fresh VM (or pod) per CI job, started by the git host's own workflow engine. The agent CLI is one step in the job.

**Pick it when** the git host is GitHub or GitLab and every trigger is a git host event or a clock: the workflow file is the orchestrator and the run page is the observability platform, so one product fills three slots.

- **Startup**: queue plus VM boot, typically tens of seconds.
- **Max run length**: GitHub-hosted 6 h per job, self-hosted 5 days; GitLab defaults to 60 min, configurable up to a month.
- **Cost shape**: per minute. GitHub Linux 2-core is $0.006/min; self-hosted runners carry no GitHub fee (a $0.002/min charge was announced, then postponed).
- **Disposability**: built in on hosted runners (fresh VM per job). Self-hosted needs ephemeral mode: ARC creates one pod per job and deletes it after.
- **Orchestrator starts one**: a workflow trigger (`issues`, `pull_request`, `schedule`, `workflow_dispatch`); concurrency groups stop two runs taking the same ticket.

Examples:
- [GitHub Actions](https://docs.github.com/en/actions/reference/limits): hosted runners, the default for an all-GitHub factory; [pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing).
- [Claude Code GitHub Action](https://code.claude.com/docs/en/github-actions): Claude Code as a workflow step; bills Actions minutes plus tokens or a subscription.
- [GitLab hosted runners](https://docs.gitlab.com/ci/runners/hosted_runners/linux/): an isolated, ephemeral VM per job.
- [Actions Runner Controller](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller): self-hosted, autoscaled, one ephemeral pod per job on the user's Kubernetes.
- [Blacksmith](https://docs.blacksmith.sh/introduction/quickstart): drop-in faster hosted runners; swap the `runs-on` label.

## MicroVM APIs

An SDK call creates an isolated microVM (vendors say "sandbox") in under a second; the orchestrator drives commands into it, or starts an agent CLI inside it.

**Pick it when** the factory has its own orchestrator (a server, a queue) and wants many short-lived, parallel runs with no machines to manage.

- **Startup**: sub-second from a snapshot or template (Daytona claims under 90 ms).
- **Max run length**: 24 h on paid plans is the norm (E2B, Modal, Vercel); hobby tiers cap at 45–60 min.
- **Cost shape**: per second of vCPU and RAM. E2B ~$0.05/vCPU-h; Daytona $0.0504/vCPU-h; Vercel bills only *active* CPU ($0.128/h), so time spent waiting on the model is nearly free.
- **Disposability**: create per run, kill at the end, and set a timeout as the backstop. Each provider also sells persistence (pause/resume, snapshots, drives); a factory reaches for it only as a cache, never as memory between runs.
- **Orchestrator starts one**: `Sandbox.create()` from the SDK, then exec.

Examples:
- [E2B](https://e2b.dev/pricing): Firecracker microVMs; 1 h on Hobby, 24 h on Pro; BYOC and a self-hostable single node.
- [Daytona](https://www.daytona.io/pricing): fastest creation; auto-stops after 15 min idle by default, and an ephemeral flag deletes on stop.
- [Modal Sandboxes](https://modal.com/docs/guide/sandboxes): 5 min default, 24 h max, GPU available; snapshot and restore to go longer.
- [Vercel Sandbox](https://vercel.com/docs/vercel-sandbox/pricing): Firecracker; 10,000 concurrent on Pro; 5 min default timeout, extendable to 24 h.
- [Cloudflare Sandbox](https://developers.cloudflare.com/sandbox/): a container microVM started from a Durable Object in a Worker; up to 4 vCPU / 12 GiB.
- [Fly Sprites](https://fly.io/learn/sprites-for-ai-agents): Firecracker with a persistent 100 GB disk and free idle; disposability rests on the orchestrator wiping it.

## Bundled agent platforms

A hosted coding agent whose runner comes with it: the vendor provisions a fresh VM or container per run, runs its own agent loop there, and pushes a branch or PR.

**Pick it when** the user wants the fewest moving parts and is happy with that vendor's agent. Each platform reads triggers from the git host, ticket tracker or chat, so it fills much of the orchestrator; its transcript view fills the observability platform.

- **Startup**: tens of seconds to minutes; setup scripts are cached between runs.
- **Max run length**: varies; Copilot caps a run at 59 min, others end on inactivity.
- **Cost shape**: folded into a seat or subscription, or per run (Copilot: one premium request per run × model rate; Managed Agents: $0.08 per running hour plus tokens).
- **Disposability**: the vendor's; each run gets its own VM or container, and only the branch survives.
- **Orchestrator starts one**: an @-mention or assignment in the git host or ticket tracker, a schedule, or an HTTP call.

Examples:
- [Claude Code cloud](https://code.claude.com/docs/en/claude-code-on-the-web): a VM per run, no separate VM charge; [scheduled tasks](https://code.claude.com/docs/en/routines) fire on cron, API POST, or GitHub PR/release events.
- [Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview): API-first (beta): create a run against an environment and stream events; cron via scheduled deployments.
- [Codex cloud](https://developers.openai.com/codex/cloud): a container per task; started from the web, CLI, Slack, Linear or `@codex` on GitHub.
- [Copilot coding agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent): assign an issue to Copilot; its runner *is* a GitHub Actions runner.
- [Cursor cloud agents](https://cursor.com/docs/cloud-agent): an isolated VM per agent; started from Slack, GitHub, Linear or the API; billed at model API rates.
- [Devin](https://docs.devin.ai/): started from Slack, Teams, Linear, Jira or the API; plan quota plus credits.

## Bring-your-own worker

The bundled platform keeps its agent loop and its queue of runs, but tool calls execute on a worker the user hosts. The worker polls the vendor outbound, claims a run, executes it, and exits. Same shape as a self-hosted CI runner.

**Pick it when** the user wants a bundled agent but runs must reach a private network, hold data on their own machines, or use custom hardware.

- **Startup and max run length**: whatever the user's machines give.
- **Cost shape**: the vendor's seat or usage, plus the user's own machines.
- **Disposability**: the user's job. Run the worker under Kubernetes or a microVM API so each run gets a fresh disk (Claude's runner exits after its runs by default for exactly this).
- **Orchestrator starts one**: the vendor queues the run; an autoscaler the user hosts (or the vendor ships) starts workers.

Examples:
- [Claude Code self-hosted environments](https://code.claude.com/docs/en/self-hosted-environments): public beta, Team and Enterprise; ships an autoscaling orchestrator and Kubernetes recipes.
- [Cursor self-hosted pool](https://cursor.com/docs/cloud-agent/self-hosted-pool): outbound workers; guides for Modal, Daytona, E2B, Vercel and Cloudflare.
- [Devin Outposts](https://docs.devin.ai/cloud/outposts/overview): early access; a Kubernetes reference, with E2B, Modal and Namespace as hosts.
- [Copilot coding agent on ARC](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment): point `runs-on` at an ephemeral ARC scale set; Copilot's firewall switches off.

## Own VPS or containers

A machine the user rents or runs, with a fresh container (or machine) per run. The machine may live for months; the container lives for one run.

**Pick it when** runs are long, steady or heavy, cost per hour dominates, or the factory already runs on a server that hosts the orchestrator.

- **Startup**: seconds for a container on a warm host; Fly creates a new Machine in low double-digit seconds and restarts a stopped one in under one.
- **Max run length**: unbounded; the orchestrator sets a deadline (`activeDeadlineSeconds` on a Kubernetes Job).
- **Cost shape**: flat monthly (Hetzner from €5.99/mo) or per second while running (Fly: stopped Machines pay only for rootfs).
- **Disposability**: `docker run --rm` per run, a Kubernetes Job with `ttlSecondsAfterFinished: 0`, or a Fly Machine destroyed at the end. Keep every cache on a path the next run treats as optional.
- **Orchestrator starts one**: a Docker, Kubernetes or Machines API call from the orchestrator, often on the same server.

Examples:
- [Fly Machines](https://docs.fly.io/machines/overview): Firecracker VMs from an API; [per-second pricing](https://docs.fly.io/about/pricing).
- [Hetzner Cloud](https://www.hetzner.com/cloud/) plus Docker: the cheapest steady capacity; one container per run.
- [Kubernetes Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/): a Job per run with a deadline and a TTL.

## Choosing

- **All on GitHub, triggers are issues, PRs and clocks**: GitHub Actions. One product fills orchestrator, runner and observability platform.
- **Wants a named vendor's agent, minimal infrastructure**: a bundled agent platform; check its triggers cover the user's ticket tracker.
- **Own orchestrator, many parallel short runs**: a microVM API. Choose on max run length, concurrency cap, and whether model-wait time is billed.
- **Runs must reach a private network**: bring-your-own worker, or self-hosted CI runners on ARC.
- **Long or heavy runs on a tight budget**: own VPS or containers, with a container per run.
- **Mixed**: runner choice is per run. A cheap Actions runner for review runs and a microVM for implementer runs is a normal design.
