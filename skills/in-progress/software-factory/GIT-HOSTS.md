# Git hosts

Surveyed 2026-10. Options for the git host slot, grouped by type. Each type names what it is, when a factory picks it, the trade-offs a factory feels, and example products. Prices, limits and beta status move fast: confirm them on the vendor page before the user commits.

## What every git host is judged on

- **Triggers**: which git host events (PR opened, push, review submitted, label added, check finished) reach the orchestrator as webhooks.
- **Identity**: what a run acts as. A dedicated bot identity keeps runs attributable, lets an automated review run approve a PR another identity authored, and decides whether a run's own pushes trigger the next run.
- **Gates**: branch protection, required checks and required approvals: the line between what runs may merge and what waits for a human.
- **Rate limits**: the API budget every run on every runner draws from at once.
- **Slots filled**: which other slots the same product fills.

## All-in-one platforms

The git host bundled with a ticket tracker, a CI system that serves as orchestrator, and hosted runners, all sharing one event stream and one permission model.

**Pick it when** the factory starts small, the code already lives there, or the user wants one bill and one login. This is the "All on GitHub" design in [SKILL.md](SKILL.md).

- **Triggers**: the richest event catalogue of any type; every PR, review, label and check event is both a webhook and a native CI trigger.
- **Identity**: GitHub Apps give runs a first-class bot identity with short-lived installation tokens. The built-in Actions `GITHUB_TOKEN` is a trap for chaining: events it causes start no new workflow runs (except `workflow_dispatch` and `repository_dispatch`), so a run pushing with it silently ends the chain; mint an App token instead. GitLab offers service accounts and project or group access tokens, each a bot user.
- **Gates**: GitHub rulesets give required status checks, required approvals and bypass lists (Copilot's agent can sit on one); on private repositories they need Pro, Team or Enterprise. GitLab approval rules need Premium.
- **Rate limits**: GitHub Apps get 5,000 requests/hour per installation, scaling to 12,500 (15,000 on Enterprise Cloud); `GITHUB_TOKEN` gets 1,000/hour per repository. GitLab.com allows 2,000 authenticated requests/minute per user today, with lower per-plan hourly caps proposed.
- **Slots filled**: git host, ticket tracker, orchestrator, runner; the CI run pages often cover the observability platform too. Both vendors also ship their own agents (Copilot coding agent and Agent HQ; GitLab Duo Agent Platform), which compete with the user's runs or slot in as one of them.

Examples:
- [GitHub](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api): the default; Issues, Actions, Apps and the largest integration ecosystem.
- [GitLab](https://about.gitlab.com/pricing/): issues, merge requests, CI/CD and runners in one product, SaaS or self-managed.

## Suite git hosts

A git host sold as one module of a wider suite, where the ticket tracker is a sibling product (Jira, Azure Boards) joined by links and webhooks.

**Pick it when** the organisation already runs the suite and its humans live in Jira or Azure Boards. The factory inherits that tracker as its human interface.

- **Triggers**: webhooks (Bitbucket) or service hooks (Azure DevOps) cover push, PR and comment events; tracker events arrive on a second stream, so the orchestrator joins two event sources.
- **Identity**: Bitbucket repository, project and workspace access tokens each act as a bot user; Forge apps act as an app. Azure DevOps supports Entra service principals and managed identities as first-class users.
- **Gates**: Bitbucket merge checks are pluggable through Forge, and enforcing them needs Premium. Azure DevOps branch policies combine build validation, status checks and required reviewers.
- **Rate limits**: Bitbucket gives access tokens 1,000 requests/hour, scaled up to 10,000 for workspaces over 100 paid seats; user tokens and OAuth stay at the base rate. Azure DevOps meters load in TSTUs, 200 per sliding five minutes per identity; assigning the Basic + Test Plans access level raises it for a bot identity.
- **Slots filled**: git host, orchestrator and runner (Bitbucket Pipelines, Azure Pipelines); the ticket tracker is the sibling product. Azure DevOps covers all four inside one organisation.

Examples:
- [Bitbucket Cloud](https://support.atlassian.com/bitbucket-cloud/docs/api-request-limits/): Atlassian's git host, paired with Jira and Rovo Dev's code reviewer.
- [Azure DevOps (Azure Repos)](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits): Repos, Boards, Pipelines and Entra identities in one organisation.
- [AWS CodeCommit](https://aws.amazon.com/blogs/devops/aws-codecommit-returns-to-general-availability/): bare git host with IAM identities; reopened to new customers in November 2025; pair with an external tracker and CI.

## Self-hosted git hosts

A git host the user runs on their own infrastructure, often beside the orchestrator and runners on the same network.

**Pick it when** code must stay on-premises or air-gapped, the user wants zero per-seat cost, or API limits on a SaaS throttle a busy factory.

- **Triggers**: webhooks for push, PR, review and CI events; Forgejo v17 replaced action-run webhooks with workflow-run events.
- **Identity**: a bot user account with a scoped token; GitHub Enterprise Server and GitLab self-managed keep their Apps and service accounts.
- **Gates**: protected branches with required status checks and approvals in all of them.
- **Rate limits**: set by the operator; the ceiling is the hardware.
- **Slots filled**: Gitea and Forgejo bundle issues and GitHub-Actions-compatible CI with self-hosted runners, so they fill ticket tracker, orchestrator and runner too. The user owns upgrades, backups and uptime for every slot it fills.

Examples:
- [Forgejo](https://forgejo.org/2026-10-release-v17-0/): community-governed, lightweight, with Forgejo Actions; Codeberg runs it for public free software.
- [Gitea](https://about.gitea.com/): lightweight with Gitea Actions; also sold as managed Gitea Cloud.
- [GitLab self-managed](https://about.gitlab.com/install/): the full GitLab platform on the user's own infrastructure, CE or EE.
- [GitHub Enterprise Server](https://docs.github.com/en/enterprise-server@latest): GitHub's feature set on-premises, Apps included.
- [Bitbucket Data Center](https://www.atlassian.com/licensing/data-center-end-of-life): exempt from Atlassian's 2029 Data Center end of life; sold now as a hybrid licence with Cloud.

## Review-first and decentralised git hosts

Git hosts whose review model departs from the branch-and-PR shape: per-commit patchsets with voted labels, or repositories federated across user-run nodes.

**Pick it when** the user's team already reviews this way, or values federation over ecosystem. Most factory tooling assumes PRs, so expect to build more of the glue.

- **Triggers**: Gerrit streams events over SSH and webhook plugins; Tangled emits pipeline events from its nodes.
- **Gates**: Gerrit submit requirements gate on labels; an automated check run votes `Verified` from a bot account, a human votes `Code-Review`.
- **Slots filled**: mostly the git host alone; Tangled adds its own CI runner (Spindle).

Examples:
- [Gerrit](https://www.gerritcodereview.com/): patchset review with label votes; the Android and Chromium model.
- [Tangled](https://blog.tangled.org/ci/): git hosting on the AT Protocol with stacked PRs and Nix-based CI.

## Agent-native git hosts

New git hosts built around agents as the main authors: high commit throughput, server-side reads in place of clones, and per-agent attribution. All are early.

**Pick it when** the factory runs many concurrent runs against one large repository and clone time or API limits dominate. Pilot them by mirroring one repository; each one syncs with GitHub or keeps git compatibility so the move stays reversible.

- **Triggers**: Cursor Origin publishes PR, review, reviewer and push webhooks with signed deliveries; RWX already triggers CI from them.
- **Identity**: per-agent attribution is the selling point, and still landing.
- **Gates and rate limits**: unpublished as of this survey; ask the vendor.
- **Slots filled**: git host; Cursor pairs Origin with its Cloud Agents and Bugbot review, so it edges toward runner and automated review run.

Examples:
- [Cursor Origin](https://cursor.com/origin): git host from the Graphite team, early beta on paid Cursor plans since August 2026, with GitHub sync.
- [GitLab next-gen SCM](https://about.gitlab.com/blog/gitlab-next-gen-scm/): rebuilt GitLab backend with clone-free agent APIs; private beta.

## Git storage for runs

Headless, API-first git repositories created by the thousand: branches, commits and diffs, with no PRs, reviews or gates. They fill half the slot.

**Pick it when** runs need cheap per-run scratch repositories or forks (a fork per ticket, an agent's working state) and the PR still lands on a full git host. Pair one with a git host from the types above.

- **Triggers**: Worker bindings (Cloudflare); otherwise the orchestrator polls.
- **Identity**: short-lived, per-repository read or write tokens, minted per run.
- **Slots filled**: part of the git host; they sync to GitHub or hand off a git URL.

Examples:
- [Cloudflare Artifacts](https://www.cloudflare.com/product/artifacts/): git-compatible storage on Workers; fork a baseline per task, in beta.
- [Pierre code.storage](https://code.storage/docs): white-label git API with ephemeral branches and grep.
## Choosing

- **Starting out, code already on GitHub**: all-in-one GitHub. Give runs a GitHub App identity from day one so chains keep triggering.
- **Humans live in Jira or Azure Boards**: the suite git host beside it, or GitHub with the suite's tracker as the ticket tracker.
- **Code must stay on-premises, or the API budget throttles runs**: self-hosted; Forgejo or Gitea for small teams, GitLab self-managed or GitHub Enterprise Server for organisations.
- **Team already reviews in Gerrit**: keep it, and have check runs vote `Verified`.
- **Hundreds of concurrent runs on one big repository**: pilot an agent-native git host on a mirror, keeping GitHub as the fallback.
- **Runs need throwaway repositories at scale**: add git storage for runs behind the main git host.
