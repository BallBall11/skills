# Ticket trackers

Surveyed 2026-10 from vendor docs, changelogs and pricing pages. Agent features move monthly: treat every "preview" or "beta" below as a prompt to re-check the linked page.

The ticket tracker is the factory's human interface, so judge each option by six **seams**:

- **Triggers**: which ticket changes emit a webhook or event the orchestrator can turn into a run.
- **Routing**: labels, statuses or fields a run reads to know its kind (explore, implementer, rework).
- **Thread**: comments where a human answers an agent, which triggers the next run.
- **Identity**: an API plus a bot identity, so runs act as themselves and humans can tell agent from colleague.
- **Native agents**: the tracker's own assign-to-agent feature. Each one is a ready-made run, and it partly fills the orchestrator and runner slots as well.
- **Outside filing**: how easily customers and ticket sources create tickets; plus whether it self-hosts.

## 1. Git-host tracker

**What**: the tracker built into the git host, so tickets, branches and PRs share one product, one permission model and one identity.

**Picks it when**: the factory is small, the team is on the git host already, or the repo is public. This is the "All on GitHub" design: the git host's CI fills the orchestrator slot with zero glue.

- **Triggers**: rich. GitHub Actions fires on `issues` (opened, labeled, assigned) and `issue_comment`. Gotcha: changes made with the default `GITHUB_TOKEN` start no new workflow runs, so a run that labels a ticket to trigger the next run acts through a GitHub App token.
- **Routing**: labels, plus issue types and sub-issues on GitHub; scoped labels on GitLab. Statuses live in a separate Projects board, so labels are the dependable router.
- **Thread**: the issue thread, with PR review one click away.
- **Identity**: GitHub Apps and GitLab service accounts appear as bots.
- **Native agents**: GitHub assigns an issue to Copilot, Claude or Codex (third-party agents in public preview, paid Copilot plans); each opens a draft PR. Copilot reads the issue at assignment only, so later context goes on the PR. GitLab Duo flows start on mention, assign, assign-reviewer, and since 2026-07 on "work item created".
- **Outside filing**: the best of any type for developers: anyone with an account files on a public repo, and issue forms shape the report. Non-developers need an account, so a feedback button files through the API.
- **Hosting**: GitHub Enterprise Server, GitLab self-managed, Forgejo and Gitea all self-host.

Examples:
- [GitHub Issues](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents): the default; assign-to-agent for Copilot, Claude, Codex.
- [GitLab Issues](https://about.gitlab.com/blog/how-to-use-a-work-item-created-trigger/): Duo Agent Platform flows triggered from work items.
- [Forgejo](https://forgejo.org/2026-10-release-v17-0/): self-hosted, Actions-compatible; v17.0 (2026-10) added workflow and job webhooks.

## 2. Agent-native dev tracker

**What**: a standalone SaaS tracker for software teams that now treats agents as first-class assignees, with an agent API and a native coding agent.

**Picks it when**: humans outside engineering (product, support, design) schedule work, the team already lives there, or the factory wants polished triage and planning views. The "Linear tickets" design.

- **Triggers**: webhooks on every ticket change, plus a dedicated agent event. Linear sends an AgentSession webhook when a human delegates or mentions the agent, and expects an acknowledgement within 10 seconds. Jira fires an agent on a workflow transition into a chosen status.
- **Routing**: statuses and workflows are first-class, so "moved to Ready" is a clean trigger; labels and custom fields add the rest.
- **Thread**: comments, plus agent activity streams (Linear's thoughts and actions) that show a run's progress inline.
- **Identity**: app users. Linear and Shortcut agents are OAuth apps a human delegates to while staying assignee; Linear agents are free seats.
- **Native agents**: deepest of any type. Linear Agent runs coding sessions with Claude Code or Codex since 2026-06 (AI credits, $0.25 per 20-minute sandbox block). Jira assigns to Claude, Cursor, Codex, Copilot or its own Jira Coding Agent, metered in Rovo credits from 2026-12-03. Shortcut takes Claude Code, Cursor and Devin.
- **Outside filing**: Linear Asks files from Slack (shared channels admit customers) and email; web forms are Enterprise-only. Jira pairs with Jira Service Management portals for customers.
- **Hosting**: SaaS. Jira Data Center reaches end of life on 2029-03-28; YouTrack still self-hosts.

Examples:
- [Linear](https://linear.app/developers/agents): Agent API in developer preview; the most agent-shaped tracker.
- [Jira](https://support.atlassian.com/jira-software-cloud/docs/collaborate-on-work-items-with-ai-agents/): status-triggered Rovo and third-party agents; heavy enterprise workflows.
- [Shortcut](https://www.shortcut.com/help/integrations/using-ai-agents): assign a story or mention an agent in a comment.
- [YouTrack](https://www.jetbrains.com/youtrack/): JetBrains tracker with workflows-as-code; agent assignment still community-built.

## 3. Open-source dev tracker

**What**: a Linear-style tracker you can run on your own infrastructure, with a cloud edition beside it.

**Picks it when**: data must stay in-house (air-gapped, regulated) or the user wants to read and patch the tracker's source, while still buying the service rather than building it.

- **Triggers**: REST webhooks on work-item events. Plane creates an Agent Run when a human mentions an agent bot, and re-fires on each reply.
- **Routing**: states, labels and custom properties, close to Linear.
- **Thread**: comments; Plane's agents reply as comments or ask a question back (an "elicitation").
- **Identity**: Plane installs each OAuth app as a bot user, free of seat billing.
- **Native agents**: younger. Plane Agents are in beta; Huly and OpenProject lean on integrations.
- **Outside filing**: guest roles and intake forms vary by product and edition; check the edition's feature table.
- **Hosting**: self-hosts by design (Plane is AGPL-3.0, Huly EPL-2.0, OpenProject GPL-3.0). Huly's stack (CockroachDB, Elasticsearch, Redpanda) is heavy to operate.

Examples:
- [Plane](https://developers.plane.so/dev-tools/agents/overview): Linear-like, agents in beta, cloud or self-hosted.
- [Huly](https://github.com/hcengineering/huly-selfhost): all-in-one workspace with two-way GitHub sync.
- [OpenProject](https://www.openproject.org/): mature, classic project management, GitHub and GitLab integrations.

## 4. General work tool

**What**: a work-management or docs tool whose database or task list doubles as the tracker.

**Picks it when**: the factory serves a non-engineering team that already lives there, or tickets are mostly content, ops or research work rather than code.

- **Triggers**: webhooks exist but are coarse. Notion batches `page.content_updated`, so events arrive a minute or two late; Asana, Trello and ClickUp send per-task events.
- **Routing**: database properties, sections or lists; flexible, and every factory invents its own schema.
- **Thread**: comments, though developer context (diffs, PR links) sits a click away on the git host.
- **Identity**: integrations act through API tokens; bot presence varies.
- **Native agents**: workspace agents aimed at office work. Notion Custom Agents fire on database changes (Business and Enterprise); Asana AI Teammates take assigned tasks; ClickUp Cloud Agents turn a task into a PR.
- **Outside filing**: forms are a strength: Notion, Asana and ClickUp forms file tickets from anyone with a link.
- **Hosting**: SaaS only.

Examples:
- [Notion](https://www.notion.com/help/custom-agents): database-as-tracker; Custom Agents with database triggers.
- [Asana](https://asana.com/product/ai/ai-teammates): AI Teammates you assign tasks to.
- [ClickUp](https://clickup.com/brain/cloud-agents): Cloud Agents return a PR in the task thread.
- [Trello](https://developer.atlassian.com/cloud/trello/guides/rest-api/webhooks/): boards and lists; webhooks plus Butler automation.

## 5. Repo-native tracker

**What**: tickets stored as files in the repo (JSONL, Markdown), edited by agents through a CLI or MCP server, versioned and merged like code.

**Picks it when**: one developer drives agents locally and wants their plan in git, or as a run's working memory beside a real tracker.

- **Triggers**: a push to the git host is the only event; the orchestrator diffs the files to find what changed.
- **Routing**: fields in each file; Beads tracks dependencies, so a run picks the next unblocked ticket.
- **Thread**: commits and PR reviews; a human talks to agents by editing files.
- **Identity**: the git host's identity.
- **Native agents**: the agent is the primary user by design.
- **Outside filing**: weakest of any type: filing means a commit or PR.
- **Hosting**: lives wherever the repo lives.

Examples:
- [Beads](https://github.com/steveyegge/beads): `bd` CLI, JSONL in `.beads/`, hash IDs to keep branches from colliding.
- [Backlog.md](https://github.com/MrLesk/Backlog.md): Markdown tasks with a terminal and web Kanban, MCP server.

## 6. Build-your-own

**What**: the factory's own app holds tickets in its own database, usually beside the orchestrator. The "Own tracker" design.

**Picks it when**: the tracker *is* the product (an internal platform, a customer-facing support factory) or no off-the-shelf model fits. The skill favours buying: every seam above becomes code to write and maintain.

- **Triggers, routing, identity**: exactly what you build; triggers become function calls instead of webhooks.
- **Thread**: a comment UI you build, or email and Slack replies you parse.
- **Outside filing**: fully in your hands, from public forms to in-app buttons.
- **Hosting**: anywhere.

Examples: a Postgres table plus an admin UI; an [Airtable](https://airtable.com/developers/web/api/webhooks-overview) base with webhooks as a half-built middle.

## Choosing

- **Code-only team, one repo, wants fewest products**: git-host tracker; route by label, trigger through a GitHub App.
- **Public open-source project**: git-host tracker; outside developers file where the code is.
- **Product, support or design staff schedule work**: agent-native dev tracker; Linear for agent depth, Jira where the org already runs Atlassian.
- **Customers report bugs directly**: whichever tracker files from forms or email; a feedback button as a ticket source works with all of them.
- **Data stays on-premises**: open-source dev tracker (Plane), or a self-hosted git host with its tracker.
- **Factory serves a non-engineering team**: general work tool they already use.
- **Solo developer, local agents**: repo-native tracker, graduating to a git-host tracker once others file tickets.
- **The tracker is the product**: build-your-own.
