# Observability platforms

Surveyed 2026-10. Prices and beta flags move fast; confirm on the linked page before quoting a number.

Reference for filling the observability platform slot. The options fall into four types, picked by **what the human needs to see inside a run**: the run's log, its agent trace (each model call and tool call as a span), or its trace joined to the rest of the user's system. Self-hosted vs SaaS cuts across the last three types, so it sits under each type's trade-offs.

## How runners hook in

Every type below receives data through one of five hooks. Name the hook alongside the product when filling the slot.

- **Harness OTel**: the coding agent exports OpenTelemetry itself. Claude Code and the Claude Agent SDK export metrics and log events with `CLAUDE_CODE_ENABLE_TELEMETRY=1` plus `OTEL_*` variables; traces (spans per interaction, model request and tool call) need `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1` and are beta. Prompts and tool output stay out until opted in (`OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_TOOL_CONTENT`). Codex CLI takes an `[otel]` block in `config.toml`, with separate log and trace exporters. ([Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/observability), [Claude Code](https://code.claude.com/docs/en/monitoring-usage), [Codex config](https://developers.openai.com/codex/config-advanced))
- **Platform push**: the model platform sends traces for you. Claude Managed Agents export to an OTLP destination an admin adds in the Claude Console (public beta). ([Claude docs](https://platform.claude.com/docs/en/manage-claude/opentelemetry/api))
- **SDK wrapper**: one call in the runner's code, such as `configure_claude_agent_sdk()` (LangSmith) or `logfire.instrument_claude_agent_sdk()`.
- **Hook plugin**: a Claude Code hook reads the session transcript at each Stop and ships it, as in Langfuse's Claude Code plugin.
- **Gateway**: the runner's model base URL points at a proxy that logs every request; zero runner code, model calls only (tool calls inside the harness stay invisible).

OpenTelemetry is the common wire: harness OTel and platform push land in any OTLP backend, and the GenAI semantic conventions (`invoke_agent`, `chat`, `execute_tool` spans) are still at Development stability, so span names drift between releases. ([OTel GenAI agent spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans))

## Type 1: None (the runner or orchestrator product shows enough)

**What it is**: a deliberate none. The human reads each run in the product already filling the runner or orchestrator slot: the CI run page, the sandbox dashboard, the agent's own output saved as an artifact.

**When a factory picks it**: few runs a day, one person watching, and the question is "did it fail, and what did it print?" Most factories start here.

**Trade-offs**:
- Hook: none to build. The agent's final message and transcript go to stdout or an uploaded artifact.
- Self-hostable: as self-hosted as the runner itself.
- Cost: free, bundled with runner minutes.
- Limits: one run per page, no search across runs, no token or cost rollup, retention capped by the runner (GitHub Actions: 90 days default; from 2026-10-01 workflow runs themselves follow that setting; Daytona sandbox telemetry: 3 days).

**Examples**:
- [GitHub Actions run logs](https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization): per-step logs and artifacts on each workflow run; 90-day default retention.
- [E2B sandbox logs and metrics](https://e2b.dev/docs): dashboard and `e2b sandbox logs|metrics` CLI per sandbox; OTLP export of the same data to grow out of it.
- [Daytona sandbox telemetry](https://www.daytona.io/docs/en/observability/otel-collection/): Logs, Traces and Metrics tabs per sandbox, 3-day retention, OTLP forward.
- [OpenAI Agents SDK traces](https://openai.github.io/openai-agents-python/tracing/): tracing on by default, viewed under Logs in the OpenAI platform; unavailable under Zero Data Retention.

## Type 2: LLM and agent tracing platform

**What it is**: a backend built around the agent trace: each run as a tree of model calls and tool calls, with token cost, prompts, sessions, and usually evals and datasets on top.

**When a factory picks it**: the human asks "why did the agent do that?" across many runs; a retro run or a human compares runs; token spend needs a per-run rollup.

**Trade-offs**:
- Hook: harness OTel, SDK wrapper or hook plugin; most also accept the Claude Console platform push.
- Self-hostable: split. Langfuse (MIT, owned by ClickHouse since 2026-01) and Phoenix self-host free; LangSmith self-hosts on Enterprise only; Braintrust is SaaS.
- Cost: metered per trace, span, unit or GB, plus seats on some. Agent runs are span-heavy (one span per tool call), so budget per run, not per request.

**Examples**:
- [Langfuse](https://langfuse.com/pricing): open source, Claude Code plugin and OTLP ingest; cloud free to 50k units/month, then $8 per 100k.
- [LangSmith](https://www.langchain.com/pricing-langsmith): native Claude Agent SDK wrapper; free 5k traces/month, Plus $39/seat; self-host on Enterprise.
- [Arize Phoenix](https://phoenix.arize.com): open source, single Docker container, OTLP on 6006/4317; Arize AX is the paid SaaS sibling.
- [Braintrust](https://www.braintrust.dev/pricing): tracing plus evals; billed by GB processed since 2026-03, free Starter tier.
- [Pydantic Logfire](https://pydantic.dev/pricing): OTel-native, `instrument_claude_agent_sdk()`; free 10M records/month.

## Type 3: General APM and logging backend

**What it is**: the user's existing observability stack (logs, metrics, distributed traces for every service), now with an agent or LLM view added.

**When a factory picks it**: the team already pays for one; runs need joining to the app's own traces (the Agent SDK propagates `TRACEPARENT`, so a run nests inside the caller's trace); or a monitor ticket source already reads from it.

**Trade-offs**:
- Hook: harness OTel straight to the vendor's OTLP endpoint; several also take the Claude Console platform push.
- Self-hostable: SigNoz and Grafana's stack (Tempo, Loki) self-host free; Datadog, Honeycomb and Sentry are SaaS.
- Cost: priced per GB, event or host, rarely per agent span, so a factory's volume is a rounding error on an existing bill and a surprise on a new one. Datadog's agent product bills only LLM spans.

**Examples**:
- [Datadog Agent Observability](https://www.datadoghq.com/product/llm-observability/): Claude Code console in preview; free 40k LLM spans/month, Pro from $160.
- [Grafana Cloud](https://grafana.com/pricing/): free 50 GB logs and 50 GB traces/month; Claude Console provider preset.
- [Honeycomb](https://www.honeycomb.io/pricing): event-based, strong for high-cardinality trace queries; free tier 20M events/month.
- [SigNoz](https://signoz.io/pricing/): open source on ClickHouse; Claude Code and Codex guides; cloud about $0.30/GB.
- [Sentry](https://docs.sentry.io/platforms/python/agent-tracing/openai-agents/): agent spans beside errors; Claude Console destination with only a DSN.

## Type 4: LLM gateway with request logging

**What it is**: a proxy between the runner and the model provider that logs every request and response, with routing, keys, budgets and caching alongside.

**When a factory picks it**: the user wants per-run spend caps or one key vault for many runners, and logging comes free with it; or the harness offers no export of its own.

**Trade-offs**:
- Hook: gateway (base URL swap). Sees model calls only; pair with Type 2 or 3 for tool calls.
- Self-hostable: LiteLLM and the Portkey gateway are open source.
- Cost: per logged request on SaaS; free self-hosted plus the server.

**Examples**:
- [LiteLLM Proxy](https://docs.litellm.ai/docs/proxy/logging): open source; forwards logs to Langfuse, OTel or a bucket.
- [Portkey](https://portkey.ai/pricing): gateway plus log viewer; free 10k logs/month, paid per 100k logs.
- [Helicone](https://www.helicone.ai): in maintenance mode since Mintlify acquired it (2026-03); fine for an existing install, a poor pick for a new factory.

## Choosing

- **Few runs, one human, CI as orchestrator**: Type 1. Save the agent's transcript as a run artifact and stop there.
- **Sandbox runners, runs failing in ways the log hides**: Type 1 first (the sandbox dashboard), then OTLP-forward to Type 2 when retention runs out.
- **Many runs, "why did the agent do that?"**: Type 2 via harness OTel. Self-host Langfuse or Phoenix when transcripts must stay in-house; take a SaaS free tier otherwise.
- **The team already runs Datadog, Grafana, Honeycomb or Sentry**: Type 3, pointing harness OTel at it.
- **Spend control is the real worry**: Type 4, logging into a Type 2 backend.
- **A run should act on what the platform sees**: make the platform a ticket source. A monitor files the ticket, and the run reads the ticket.
