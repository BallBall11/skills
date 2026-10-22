---
name: software-factory
description: Shared language for designing a software factory: the runs it executes and the services it plugs into.
disable-model-invocation: true
---

A **software factory** is a system of agents that turns tickets into merged code, deployed on remote infrastructure the user chooses. This skill is the language for designing one. Use these terms, and only these, when discussing or whiteboarding a factory; when the user reaches for a word under _Avoid_, name the canonical term.

## Runs

**Trigger**:
The event that starts a run: a change in the ticket tracker or git host, a clock in the orchestrator, or another run.

**Run**:
One piece of work executed on one runner, from its trigger to its finish, treated as a black box. A run triggers further runs either directly, through logic in the orchestrator, or by changing the ticket tracker or git host. A run never waits for a human: it ends, and the human's later change to the ticket tracker or git host triggers a new run.
_Avoid_: session, job

**Runner**:
The disposable machine a run executes on. Nothing on it survives the run, even when the machine itself is long-lived: a run's only memory of earlier runs is the ticket tracker, the git host and the orchestrator.
_Avoid_: server, host, box, compute, sandbox

## Services

**Ticket tracker**:
Where tickets live, and the factory's human interface: humans schedule work by filing tickets, talk to agents in their comments, and their changes to tickets trigger runs.
_Avoid_: issue tracker, ticket manager, project board

**Ticket source**:
Anything outside the factory that creates tickets in the ticket tracker: a feedback button, a monitor, an error tracker, an inbox.
_Avoid_: data source, trigger source, intake

**Git host**:
Where branches, PRs and reviews live.
_Avoid_: code host, forge, repo

**Orchestrator**:
The component that receives triggers, starts runs on runners, and records which runs are queued, in flight or finished.
_Avoid_: dispatcher, scheduler, control plane

**Observability platform**:
Where runners send logs, traces and status, so a human can see what happened inside each run. A run never decides on it until something writes it into the ticket tracker or git host.
_Avoid_: observability, logs, monitoring, dashboard

## Designing a factory

A design has two halves: the **services**, each a slot filled with a product, and the **runs** that move tickets through them.

Start from the user's problem, not the factory: untriaged tickets, PRs waiting for automated review, a stricter automated check to migrate to, bugs nobody reads. When the user names a problem or doesn't know what to build, read [EXAMPLE-RUNS.md](EXAMPLE-RUNS.md) for the runs that solve common ones.

### Fill every slot

Each service is a slot. One product can fill several slots, and one server can host several services. Three designs, side by side:

| Service                | All on GitHub     | Linear tickets                 | Own tracker               |
| ---------------------- | ----------------- | ------------------------------ | ------------------------- |
| Ticket tracker         | GitHub Issues     | Linear                         | own app                   |
| Ticket source          | feedback button   | feedback button, monitor       | feedback button, monitor  |
| Git host               | GitHub            | GitHub                         | GitHub                    |
| Orchestrator           | GitHub Actions    | own server + database          | own app                   |
| Runner                 | Actions runner    | Actions, E2B, Daytona or a VPS | own app, or sandboxes     |
| Observability platform | Actions run pages | Langfuse                       | Langfuse, or own app      |

Buy the services and build the runs: the runs carry the user's design, the services are generic. Favour an off-the-shelf ticket tracker: it is robust, people already know it, and outside users are easy to let in.

### Question every run

For each run, answer:

1. What triggers it?
2. What does it do, and which services does it read and write?
3. What does it trigger next?
4. Where does a human re-enter?
5. What stops two runs doing the same ticket?

For each ticket source, answer: what stops it filing the same ticket twice?

A design is finished when every slot names a product and every run and ticket source has an answer to every question.
