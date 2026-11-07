# Orchestrator, Multi-Agent Coordination, Protocol and Agent-Infrastructure Options for a Multi-Department AI Software Factory (as of 2026-11-07)

Reading guide for the report writer:
- Stars, licence (SPDX), archive flag, last push, latest release and 30-day commit counts come from my direct GitHub GraphQL/REST API query on 2026-11-07. The repo link in each row is the source. "Licence" is GitHub's SPDX detection, except where I read the LICENSE file (`NOASSERTION` cases, stated explicitly).
- Security advisory items come from each repo's published GitHub advisories (`/security/advisories`). I read severity, date, CVE and summary fields only, not the full write-ups.
- Source quality is flagged inline: **[primary]** is vendor docs, spec, repo or paper. **[secondary]** is a blog or aggregator that I could not confirm against a primary source.

## Agent frameworks: status, licence, maturity and SDLC fit

### Takeaway
LangGraph, Microsoft Agent Framework, Google ADK, Pydantic AI and Mastra are the healthy model-portable options with explicit workflow or graph control. OpenAI Agents SDK and Claude Agent SDK are strong but vendor-centred. AutoGen is end-of-life and MetaGPT is dormant, despite very high star counts. No framework on its own replaces a durable engine for multi-day SDLC runs.

### Cited Findings

| Framework | Repo | Stars | Licence (verified) | Latest release | Commits 30d | Status flag |
|---|---|---|---|---|---|---|
| LangGraph | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | 44,588 | MIT | 1.2.20 (2026-11-05) | 151 | Active, 1.x |
| CrewAI | [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) | 60,591 | MIT | 1.15.37 (2026-11-07) | 222 | Active |
| Microsoft Agent Framework | [microsoft/agent-framework](https://github.com/microsoft/agent-framework) | 14,568 | MIT | dotnet-1.28.0 (2026-11-06); tag python-1.25.0 | 439 | Active; Go in public preview |
| AutoGen | [microsoft/autogen](https://github.com/microsoft/autogen) | 61,736 | CC-BY-4.0 detected (GitHub picks up the docs licence; code licence not re-verified) | python-v0.7.5 (2025-09-30) | 2 | **README says archived and no longer maintained**, while GitHub `isArchived=false` |
| Semantic Kernel | [microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel) | 28,740 | MIT | dotnet-1.82.0 (2026-11-07) | n/c | Still releasing; README steers to Agent Framework |
| Google ADK (Python) | [google/adk-python](https://github.com/google/adk-python) | 22,127 | Apache-2.0 | v2.14.0 (2026-11-07) | 589 | Active, 2.x |
| OpenAI Agents SDK | [openai/openai-agents-python](https://github.com/openai/openai-agents-python) | 30,535 | MIT | v0.24.0 (2026-10-15) | 83 | Active, **still 0.x** |
| Claude Agent SDK (Py) | [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python) | 8,444 | GitHub says MIT; **README says Anthropic Commercial Terms** | v0.2.179 (2026-11-06) | n/c | Active, 0.x; licence signal conflict |
| Claude Agent SDK (TS) | [anthropics/claude-agent-sdk-typescript](https://github.com/anthropics/claude-agent-sdk-typescript) | 1,869 | No licence file; Commercial Terms | v0.3.319 (2026-11-06) | n/c | Active, 0.x, not OSS |
| Pydantic AI | [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) | 21,064 | MIT | v2.66.0 (2026-11-04) | 1,353 | Active, 2.x |
| Mastra | [mastra-ai/mastra](https://github.com/mastra-ai/mastra) | 29,384 | Apache-2.0 outside `ee/`; `ee/` dirs under separate licence ([LICENSE.md](https://github.com/mastra-ai/mastra/blob/main/LICENSE.md)) | @mastra/core 1.82.0 (2026-11-04) | 2,131 | Active |
| MetaGPT | [FoundationAgents/MetaGPT](https://github.com/FoundationAgents/MetaGPT) | 71,204 | MIT | v0.8.1 (**2024-04-22**); last push 2026-01-21 | **0** | **Effectively dormant** |
| Agno | [agno-agi/agno](https://github.com/agno-agi/agno) | 43,086 | Apache-2.0 | v3.1.12 (2026-10-29) | n/c | Active |
| Strands | [strands-agents/harness-sdk](https://github.com/strands-agents/harness-sdk) | 9,066 | Apache-2.0 | typescript/v1.24.0 (2026-11-04) | n/c | **Renamed** from strands-agents/sdk-python (redirect) |
| smolagents | [huggingface/smolagents](https://github.com/huggingface/smolagents) | 30,241 | Apache-2.0 | v1.26.0 (2026-05-29) | n/c | Release cadence slowed |
| VoltAgent | [VoltAgent/voltagent](https://github.com/VoltAgent/voltagent) | 10,925 | MIT | core 2.11.1 (2026-11-04) | n/c | Active |
| CAMEL | [camel-ai/camel](https://github.com/camel-ai/camel) | 18,016 | Apache-2.0 | v0.2.90 (2026-03-22) | n/c | Pre-1.0 |

(n/c = not collected.)

**Control and human-gate capabilities**
- LangGraph `interrupt()` suspends the graph, saves state and waits indefinitely, then resumes through `Command(resume=...)`. It requires a checkpointer (database-backed in production) and a reused `thread_id`. A node **re-runs from its beginning on resume**, so side effects need to be idempotent — [LangGraph docs](https://docs.langchain.com/oss/python/langgraph/interrupts) [primary]
- LangGraph checkpoint layer CVEs in 2026, all deserialization of persisted state: CVE-2026-27794 (RCE via cache pickle fallback, fixed in checkpoint 4.0.0) — [NVD](https://nvd.nist.gov/vuln/detail/CVE-2026-27794); CVE-2026-28277 (msgpack) — [GitLab advisory](https://advisories.gitlab.com/pypi/langgraph/CVE-2026-28277/); CVE-2026-48775 (SQLite JSON, CVSS 6.8, needs write access to the checkpoint store) — [cvefeed](https://cvefeed.io/vuln/detail/CVE-2026-48775)
- Microsoft Agent Framework is the "direct successor" of AutoGen and Semantic Kernel. It has an agent harness (planning, compaction, tool approval), graph and functional workflows, state management for long-running and human-in-the-loop scenarios, and providers including Foundry, Anthropic, OpenAI and Ollama. Microsoft's own rule: "If you can write a function to handle the task, do that instead of using an AI agent" — [Microsoft Learn, updated 2026-10-28](https://learn.microsoft.com/en-us/agent-framework/overview/) [primary]
- Agent Framework 1.0 GA was 2026-04-03, with Semantic Kernel in maintenance mode (fixes through at least April 2027) — [secondary guide](https://www.developersdigest.tech/blog/microsoft-agent-framework-developer-guide-2026), [secondary](https://www.managed-code.com/blog-post/semantic-kernel-is-done). The exact date and support window were not confirmed on a Microsoft page.
- AutoGen README: "Project archived and no longer maintained"; new users should go to Agent Framework — [repo](https://github.com/microsoft/autogen) [primary]. Secondary sources still say "maintenance mode", which is now out of date.
- ADK 2.0 adds graph workflows mixing deterministic code and AI reasoning (routes, human input, retries and timeouts, dynamic workflows). It offers Python, TypeScript, Go, Java and Kotlin; adapters for Claude, OpenAI, Ollama and LiteLLM; and A2A quickstarts. "ADK TypeScript 2.0 GA" is announced — [adk.dev](https://adk.dev/) [primary]
- OpenAI Agents SDK has handoffs, guardrails, human-in-the-loop with interruptions and resumable state, sessions (SQLAlchemy, Redis, MongoDB, Dapr backends), sandbox agents, and tracing. It defaults to the Responses API, with LiteLLM and Any-LLM adapters for other vendors — [docs](https://openai.github.io/openai-agents-python/) [primary]
- Claude Agent SDK is "a library that runs the Claude Code binary", with hooks, subagents, MCP, permissions, sessions and skills. It is governed by Commercial Terms, and third parties may not offer claude.ai login without approval. Anthropic also offers Managed Agents, with Anthropic-hosted or self-hosted sandboxes — [Claude docs](https://code.claude.com/docs/en/agent-sdk/overview) [primary]
- Agent Teams cannot be spawned from the Agent SDK or `-p` non-interactive mode — [Claude docs](https://code.claude.com/docs/en/agent-teams) [primary]
- Secondary framing of lock-in: choose vendor SDKs for single-provider builds and LangGraph for stateful multi-provider workflows; a Claude SDK plus LangGraph hybrid is common — [comparison](https://web-9ja.pages.dev/blog/langgraph-vs-openai-claude-agent-sdk-2026) [secondary, low weight]
- Advisory signals: Pydantic AI published 29 advisories, 7 high, most recently 2026-11-04 ([advisories](https://github.com/pydantic/pydantic-ai/security/advisories)); Mastra published a high-severity unauthenticated prototype pollution with cross-user impact, 2026-10-12 ([advisories](https://github.com/mastra-ai/mastra/security/advisories)); CrewAI and ADK-Python show no published GitHub advisories ([CrewAI](https://github.com/crewAIInc/crewAI/security/advisories), [ADK](https://github.com/google/adk-python/security/advisories)).

### Inferences
- Vendor lock-in ranking, least to most: Pydantic AI, LangGraph, Mastra, ADK and Agent Framework (portable but with gravity toward Google or Azure), OpenAI Agents SDK (Responses-first), Claude Agent SDK (Claude-model harness and commercial terms).
- For SDLC orchestration, the framework should be the **inner loop inside one stage**, not the factory control plane. LangGraph or Agent Framework are the best candidates when a graph is required; the Python 2.x cadence of ADK makes it credible for Google-leaning shops.
- Zero published advisories is not evidence of safety. Pydantic AI's many advisories may equally reflect better disclosure practice.
- Stars are a poor maturity signal here: MetaGPT (71k, dormant) and AutoGen (62k, ended) rank at the top.
- For .NET-heavy departments, Agent Framework is the natural pick.

### Gaps
- No independent, reproducible benchmark comparing these frameworks on SDLC tasks was found. Comparison articles were SEO or vendor material.
- CrewAI Flows' human-gate and durability behaviour was not verified against CrewAI docs.
- Production adoption claims per framework were not independently verified.

## Durable workflow engines, and when deterministic beats an LLM orchestrator

### Takeaway
Deterministic workflows should own the SDLC skeleton (stages, gates, retries, approvals, budgets), with agents as bounded nodes. Temporal has the strongest agent-integration evidence and a permissive licence. Licence terms disqualify or constrain several popular options for a multi-department deployment (Dify, n8n, Inngest, Restate).

### Cited Findings

| Engine | Repo | Stars | Licence (read from file where noted) | Latest release | Note |
|---|---|---|---|---|---|
| Temporal | [temporalio/temporal](https://github.com/temporalio/temporal) | 24,238 | MIT | v1.32.2 (2026-11-02) | Advisory list empty on GitHub |
| Inngest | [inngest/inngest](https://github.com/inngest/inngest) | 6,022 | **SSPL-1.0 with Apache-2.0 future licence** ([LICENSE.md](https://github.com/inngest/inngest/blob/main/LICENSE.md)) | v1.51.0 (2026-11-06) | Source-available, not OSI |
| Restate | [restatedev/restate](https://github.com/restatedev/restate) | 4,671 | **BUSL-1.1**, cannot offer a "Public Restate Platform Service" ([LICENSE](https://github.com/restatedev/restate/blob/main/LICENSE)) | v1.8.1 (2026-10-21) | Internal use allowed by grant |
| Dapr / Dapr Agents | [dapr/dapr](https://github.com/dapr/dapr), [dapr/dapr-agents](https://github.com/dapr/dapr-agents) | 26,176 / 771 | Apache-2.0 | v1.19.1 / v1.1.0 (2026-10-27) | CNCF; Python agents only |
| Argo Workflows | [argoproj/argo-workflows](https://github.com/argoproj/argo-workflows) | 17,083 | Apache-2.0 | v4.1.6 (2026-10-22) | Kubernetes-native |
| Prefect | [PrefectHQ/prefect](https://github.com/PrefectHQ/prefect) | 24,126 | Apache-2.0 | 3.9.2 (2026-11-06) | Data-pipeline heritage |
| n8n | [n8n-io/n8n](https://github.com/n8n-io/n8n) | 209,719 | **Sustainable Use License, `.ee` files under Enterprise licence** ([LICENSE.md](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)) | n8n@2.46.6 (2026-11-04) | Fair-code; heavy CVE history |
| Dify | [langgenius/dify](https://github.com/langgenius/dify) | 160,581 | **Modified Apache-2.0: no multi-tenant environment without written authorization; one workspace = one tenant** ([LICENSE](https://github.com/langgenius/dify/blob/main/LICENSE)) | 1.19.0 (2026-11-04) | Direct blocker for per-department workspaces |
| Hatchet | [hatchet-dev/hatchet](https://github.com/hatchet-dev/hatchet) | 8,236 | MIT | v0.114.2 (2026-10-26) | Pre-1.0 |
| DBOS | [dbos-inc/dbos-transact-py](https://github.com/dbos-inc/dbos-transact-py) | 1,667 | MIT | 3.4.0 (2026-10-28) | Database-backed library |
| Trigger.dev | [triggerdotdev/trigger.dev](https://github.com/triggerdotdev/trigger.dev) | 16,725 | Apache-2.0 | v4.8.3 (2026-10-30) | |
| Conductor OSS | [conductor-oss/conductor](https://github.com/conductor-oss/conductor) | 32,438 | Apache-2.0 | v3.33.0 (2026-10-26) | **Not** the conductor.build Mac app |

**Agent integration evidence**
- Temporal: a Workflow "resumes automatically after a crash, a network timeout, or a multi-day wait for a human to approve a step" — [Temporal docs](https://docs.temporal.io/ai) [primary]
- OpenAI Agents SDK plus Temporal Python SDK reached GA 2026-03-23 — [Temporal blog](https://temporal.io/blog/announcing-openai-agents-sdk-partnership) [primary]. Pydantic AI has a `TemporalAgent` wrapper — [Temporal blog](https://temporal.io/blog/build-durable-ai-agents-pydantic-ai-and-temporal) [primary]. An early-stage open-source Temporal Agent Harness wraps existing SDKs — [summary](https://www.plushcap.com/content/temporal/blog/temporal-temporal-agent-harness-an-early-look-at-durable-agent-infrastructure) [secondary]
- Dapr Agents v1.0 GA was announced by CNCF on 2026-03-23 — [CNCF](https://www.cncf.io/announcements/2026/03/23/general-availability-of-dapr-agents-delivers-production-reliability-for-enterprise-ai/) [primary]. Each agent invocation is a durable workflow — [Diagrid](https://www.diagrid.io/blog/dapr-agents-1-0-durable-cloud-native-production-ready) [vendor]. It uses SPIFFE identity and 30+ state stores — [Cloud Native Now](https://cloudnativenow.com/kubecon-cloudnativecon-europe-2026/cncf-announces-general-availability-of-dapr-agents-v1-0-for-production-ai-workloads/) [secondary]

**Deterministic versus LLM orchestration evidence**
- Stripe "blueprints" are code-defined state machines that intermix deterministic nodes (linters, push) with agent nodes (implement, fix CI). They allow at most two CI runs before returning to a human. Over 1,300 merged PRs per week are minion-produced and human-reviewed. Devboxes are isolated from production and arbitrary egress. Toolshed holds nearly 500 MCP tools, with a small default subset per agent — [Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2) [primary]
- Anthropic: workflows suit well-defined tasks needing predictability; agents suit flexibility at scale; start with the simplest solution — [Building Effective Agents, Dec 2024](https://www.anthropic.com/engineering/building-effective-agents) [primary]
- Microsoft: use a workflow for well-defined steps and explicit control over execution order — [Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/) [primary]
- On sequential tasks, every multi-agent variant degraded performance by 39-70% — [Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/) [primary]

**Security**
- n8n: CVE-2026-21858 "Ni8mare", CVSS 10.0 unauthenticated RCE, fixed in 1.121.0 — [Indusface](https://www.indusface.com/blog/cve-2026-21858-ni8mare-n8n-remote-code-execution/) and [SOC Prime](https://socprime.com/blog/cve-2026-21858-vulnerability/) [secondary]. Exposure estimates conflict (26,500 per Censys versus about 100,000 per Cyera), as do exploitation-in-the-wild reports — [NCC CSIRT](https://csirt.ncc.gov.ng/index.php/resources/security-advisories/392-tlp-clear-ngcert-security-advisory-on-critical-ni8mare-vulnerability-affecting-n8n-workflow-automation-platform)
- n8n published at least 100 advisories (my API page was capped at 100), 50 of them high or critical, with new high-severity expression-isolation items on 2026-10-28 — [advisories](https://github.com/n8n-io/n8n/security/advisories) [primary]
- Dify: **cross-tenant** file preview and API detail disclosure (2026-07-15), unauthenticated SSRF CVE-2026-105762, MCP server IDOR CVE-2026-105761 — [advisories](https://github.com/langgenius/dify/security/advisories) [primary]
- Langflow (adjacent low-code platform, excluded from the shortlist): CVE-2026-33017 unauthenticated RCE exploited within about 20 hours and added to CISA KEV on 2026-03-25 — [CSA note](https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/03/CSA_research_note_langflow-CVE-2026-33017-active-exploitation_20260323-csa-styled.pdf). VulnCheck reports 11 further exploited Langflow flaws — [VulnCheck](https://www.vulncheck.com/blog/pwning-the-ai-stack)

### Inferences
- Prefer a deterministic engine when any of these hold: the step order is known; compliance needs replayable audit; a stage can wait hours or days for a human; side effects (merge, deploy, ticket) must be exactly-once or idempotent; the budget must be enforced outside the model; or a failure must not cost re-running completed work.
- Reserve LLM orchestration for discovery-shaped work inside a stage: decomposition, triage, research, debugging hypotheses.
- Suggested default: Temporal for the factory control plane, because of its MIT licence and vendor-neutral integrations. Dapr is the alternative when a CNCF-only, Kubernetes-first posture matters. Argo fits batch or CI-like fan-out but not long conversational human gates. Prefect is a less natural fit unless the data team already owns it.
- Dify and n8n are suitable at most as departmental low-code front ends behind network isolation, not as the factory core. Each department workspace in one Dify instance would likely require a commercial licence; legal should confirm.
- Inngest's SSPL and Restate's BUSL permit most internal use but warrant legal review if a platform team "offers" the service to departments that are separate legal entities.

### Gaps
- No controlled comparison of durable engines under agent workloads (latency, history size, cost) was found.
- Argo, Prefect, Hatchet and Trigger.dev agent-specific features were not checked against their docs; only repo metadata is verified.
- Temporal's self-hosted operating burden, and pricing for each engine, were not researched.

## Coding-agent orchestrators and swarms: what exists and how healthy

### Takeaway
Almost all named projects exist, but several were renamed, sunset or slowed. None is proven enterprise control-plane software. Paperclip and Multica fit the "board of agents" model most closely; Paperclip's disclosed cross-tenant flaws are a serious warning for multi-department use. Vendor-run surfaces (GitHub Agent HQ, Claude Code teams) remain preview or experimental.

### Cited Findings

| Project | Repo | Stars | Licence | Last release | Commits 30d | Open issues / PRs | Verdict |
|---|---|---|---|---|---|---|---|
| Ruflo (ex Claude Flow) | [ruvnet/ruflo](https://github.com/ruvnet/ruflo) | 81,089 | MIT | v3.72.2 (2026-11-06) | 1,547 | 316 / 326 | Exists, **renamed**; credibility and security concerns |
| Vibe Kanban | [BloopAI/vibe-kanban](https://github.com/BloopAI/vibe-kanban) | 28,610 | Apache-2.0 | v0.1.44 (2026-04-24); later tag 2026-09-19 | 4 | 388 / 155 | **Sunset**; company closed |
| Gas Town | [gastownhall/gastown](https://github.com/gastownhall/gastown) | 18,719 | MIT | v1.2.1 (2026-06-06) | 8 | 379 / 125 | **Moved** from steveyegge; activity slowed sharply |
| Beads | [gastownhall/beads](https://github.com/gastownhall/beads) | 28,564 | MIT | v1.3.4 (2026-11-04) | n/c | n/c | Active; agent memory and issue tracking |
| Agent Orchestrator | [OrchestratorInc/agent-orchestrator](https://github.com/OrchestratorInc/agent-orchestrator) | 14,185 | Apache-2.0 | v0.14.10 (2026-11-06) | 714 | 446 / 461 | **Moved** from ComposioHQ; pre-1.0 |
| OpenHands | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | 92,839 | MIT (per GitHub detection) | v1.37.0 (2026-11-06) | 502 | 611 / 539 | Healthy; most mature OSS agent platform here |
| OpenHands SDK | [OpenHands/software-agent-sdk](https://github.com/OpenHands/software-agent-sdk) | 1,357 | MIT | v1.60.1 (2026-11-05) | n/c | n/c | Active |
| Multica | [multica-ai/multica](https://github.com/multica-ai/multica) | 55,173 | **"Multica License": Apache-2.0 plus extra conditions** | v0.8.1 (2026-11-06) | 468 | 1,053 / 854 | Exists; pre-1.0; source-available in effect |
| Paperclip | [paperclipai/paperclip](https://github.com/paperclipai/paperclip) | 111,765 | MIT | v2026.1106.0 | 1,964 | **3,545 / 2,927** | Exists; created 2026-03; very large backlog |
| Symphony (OpenAI) | [openai/symphony](https://github.com/openai/symphony) | 27,958 | Apache-2.0 | v0.0.3 (2026-09-15) | **0** | 0 / 0 | "Low-key engineering preview"; a spec, not a product |
| GitHub Agentic Workflows | [github/gh-aw](https://github.com/github/gh-aw) | 5,585 | MIT | v1.0.1 (2026-10-27) | 1,380 | 587 / 19 | Active, 1.x |
| Claude Squad | [smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad) | 8,743 | AGPL-3.0 | v1.0.20 (2026-08-20) | 0 | 16 / 34 | Quiet |
| Emdash | [generalaction/emdash](https://github.com/generalaction/emdash) | 6,209 | Apache-2.0 | v1.2.11 (2026-11-06) | 235 | 47 / 39 | Active; worktree-runner style |
| Paradev (ex Superset) | [ParadevAI/Paradev](https://github.com/ParadevAI/Paradev) | 16,038 | MIT | desktop-v2.0.5 (2026-11-05) | 706 | 538 / 649 | **Renamed** from superset-sh/superset |
| xum (ex mux) | [coder/xum](https://github.com/coder/xum) | 2,069 | AGPL-3.0 | v0.32.1 (2026-10-27) | n/c | n/c | **Renamed** from coder/mux |
| Crystal | [stravu/crystal](https://github.com/stravu/crystal) | 3,128 | MIT | v0.3.5 (2026-02-26) | n/c | n/c | **Deprecated**, replaced by Nimbalyst |
| container-use | [dagger/container-use](https://github.com/dagger/container-use) | 4,087 | Apache-2.0 | v0.4.2 (2025-08-19) | n/c | n/c | Release stale |
| BMAD-METHOD | [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 55,085 | MIT (LICENSE file read; GitHub reports NOASSERTION) | v6.12.4 (2026-10-14) | n/c | n/c | Method and prompts, not a runtime |
| Spec Kit | [github/spec-kit](https://github.com/github/spec-kit) | 148,212 | MIT | v1.3.2 (2026-11-06) | n/c | n/c | Spec-driven toolkit, not an orchestrator |
| Claude Code | [anthropics/claude-code](https://github.com/anthropics/claude-code) | 154,810 | No OSS licence | v2.1.319 (2026-11-06) | n/c | n/c | Proprietary harness |
| OpenAI Codex CLI | [openai/codex](https://github.com/openai/codex) | 133,494 | Apache-2.0 | rust-v0.167.0 (2026-11-06) | n/c | n/c | OSS worker harness |
| OpenCode | [anomalyco/opencode](https://github.com/anomalyco/opencode) | 225,081 | MIT | v1.19.5 (2026-10-28) | n/c | n/c | Vendor-neutral worker harness |
| goose | [aaif-goose/goose](https://github.com/aaif-goose/goose) | 56,314 | Apache-2.0 | v1.57.0 (2026-10-28) | n/c | n/c | **Moved** from block/goose; AAIF project |
| Conductor (Melty) | No public repo (`melty-labs/conductor` does not resolve) | n/a | Closed source | n/a | n/a | n/a | Real, closed, Mac-only |

**Details**
- Claude Code **agent teams are experimental and off by default**, with limitations: no in-process session resumption, one team per session, no nested teams, no per-teammate permission mode at spawn, and task status that can lag. A teammate's plan is auto-approved in the default spawn path rather than reviewed by a human. Guidance: start with 3-5 teammates; token cost scales linearly; same-file edits cause overwrites — [Claude docs](https://code.claude.com/docs/en/agent-teams) [primary]
- Agent-to-agent messages cannot supply user approval. Quality gates exist through `TaskCompleted`, `TaskCreated` and `TeammateIdle` hooks — [Claude docs](https://code.claude.com/docs/en/agent-teams) [primary]
- GitHub third-party agents (Claude, Codex) are in **public preview, not GA**, as read on 2026-11-07. Enablement is by policy; generated code gets CodeQL, secret scanning and advisory checks; partner agent actions appear in the audit log; sessions consume Actions minutes and AI credits — [GitHub docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents) [primary]. Announced 2026-02-04 — [GitHub blog](https://github.blog/news-insights/product-news/pick-your-agent-use-claude-and-codex-on-agent-hq/) [primary]
- gh-aw supports Copilot, Claude Code, Codex, Gemini and Pi. Agent jobs are read-only and sandboxed by default, with writes through validated `safe-outputs` jobs — [repo](https://github.com/github/gh-aw) [primary]
- Paperclip offers org charts, budgets at company, agent and project level with pause at limits, approvals, and scoped secrets — [repo](https://github.com/paperclipai/paperclip) [primary]. But it published **critical** advisories: cross-tenant API-key IDOR and token minting, OS command injection (2026-04-16), and drive-by RCE on local instances through DNS rebinding (2026-07-22) — [advisories](https://github.com/paperclipai/paperclip/security/advisories) [primary]
- Multica: issue board for Claude Code, Codex, Cursor, Copilot, Kimi and OpenCode with a self-hosted daemon runtime and audit trail — [repo](https://github.com/multica-ai/multica) [primary]. Licence: internal use within one organization, including multiple workspaces, needs no commercial licence; hosted or embedded service for third parties does; the UI branding must not be removed — [LICENSE](https://github.com/multica-ai/multica/blob/main/LICENSE) [primary]. Producer: Index Labs (Hong Kong) Limited (same file). A comparison site claiming its licence "is not publicly specified" is wrong — [openalternative](https://openalternative.co/compare/multica/vs/paperclip)
- Gas Town: Mayor coordinator, git-backed persistence, per-rig Bors-style merge queue with verification gates; tmux required for most roles; Claude Code is the default runtime, with others supported — [repo](https://github.com/gastownhall/gastown) [primary]. Cost estimates conflict: $2,000-5,000 per month ([analysis](https://rywalker.com/research/gastown)) versus about $100 per hour at full capacity ([guide](https://codex.danielvaughan.com/2026/04/08/gas-town-multi-agent-factory/)) [all secondary]. A related MIT SDK, Gas City, is described — [analysis](https://rywalker.com/research/gas-city) [secondary]
- Agent Orchestrator (Orchestrator.inc): each task gets its own agent, branch and worktree, in a local desktop app, with a cloud option — [repo](https://github.com/OrchestratorInc/agent-orchestrator) [primary]
- Symphony: monitors a Linear board and spawns agents that return proof of work (CI, review, walkthrough videos). README warns it is for "testing in trusted environments", and offers a SPEC.md plus an experimental Elixir reference — [repo](https://github.com/openai/symphony) [primary]
- Vibe Kanban: bloop shut down on 2026-04-10; remote services removed after 30 days; project continues under community maintenance with local workspaces — [announcement](https://www.vibekanban.com/blog/shutdown) [primary]. README header: "Vibe Kanban is sunsetting" — [repo](https://github.com/BloopAI/vibe-kanban)
- Conductor: free Mac app using worktrees; Apple Silicon only; Windows waitlist; no Linux — [Morph](https://www.morphllm.com/conductor-ai-coding) [secondary]; hands-on — [The New Stack](https://thenewstack.io/a-hands-on-review-of-conductor-an-ai-parallel-runner-app/). A reported $22M Series A is single-source and unverified.
- Ruflo: rename stated in README — [repo](https://github.com/ruvnet/ruflo) [primary]. Six high-severity advisories were published on 2026-11-05, including SSRF, loopback bridge exposure and arbitrary path read/write — [advisories](https://github.com/ruvnet/ruflo/security/advisories) [primary]
- Ruflo community criticism, **unverified**: an independent audit claim of 4 critical, 8 high and 10 medium findings against v3.5.15 — [discussion #1359](https://github.com/ruvnet/ruflo/discussions/1359); a "99% theater" gist arguing most tools are stubs — [gist mirror](https://gist.icoder.workers.dev/roman-rr/ed603b676af019b8740423d2bb8e4bf6); and a remote behaviour-injection disclosure via IPFS — [disclosure](https://glama.ai/mcp/servers/@8b-is/smart-tree/blob/200a621ad2ce7d8ebfbb5066bab6ae2bf1504ccd/docs/SECURITY_DISCLOSURE_AFFECTING_LLMs.md). I found no neutral, reproducible audit.
- OpenHands: high-severity command injection in git diff handler, CVE-2026-33718 (2026-03-23) — [advisories](https://github.com/OpenHands/OpenHands/security/advisories) [primary]
- Coding-agent harness risk: Claude Code CVE-2025-59536 (repo-controlled settings and hooks led to RCE, CVSS 8.7) and CVE-2026-21852 (API key exposure through `ANTHROPIC_BASE_URL`, CVSS 5.3) — [ITPro](https://www.itpro.com/software/development/claude-code-flaws-left-ai-tool-wide-open-to-hackers-heres-what-developers-need-to-know) [secondary; fix versions need confirmation]

### Inferences
- Treat this layer as **replaceable UX and dispatch**, not as the system of record. Keep state in Git, the issue tracker and the durable engine, so that a sunset like Vibe Kanban's costs little.
- Stars here look unusually detached from maturity: Paperclip reached 111k in about eight months with about 6,500 open issues and PRs, and Ruflo's 1,500+ commits per month suggest heavy automated commits. Neither fact proves low quality, but both argue for code review over popularity.
- Multi-department shortlist for a pilot: gh-aw or Agent HQ if the enterprise is already on GitHub; Multica (internal use allowed) or OpenHands for self-hosting. Paperclip only after a security review of tenancy fixes, and only with each department on a separate instance.
- Avoid in production: Ruflo (until neutrally audited), Vibe Kanban, Crystal, Symphony reference implementation, and Claude Agent Teams for unattended flows.
- Desktop worktree runners (Conductor, Emdash, Paradev, Orchestrator.inc desktop) are developer productivity tools, with no central audit or tenancy evidence found.

### Gaps
- OpenHands' multi-agent orchestration features and any enterprise-licensed parts were not verified; only repo health was.
- No independent throughput or defect comparison across these orchestrators exists that I could find.
- No Agent HQ GA date was found. Multica showed no published advisories, but with 1,053 open issues that absence means little.
- Ruflo's disputed claims cannot be resolved without a code audit.

## Protocols: MCP, A2A, AGENTS.md, Agent Skills, AG-UI, ACP

### Takeaway
The stack has largely consolidated under the Linux Foundation's Agentic AI Foundation: MCP for tools, A2A for agents, AGENTS.md and Skills for instructions. AG-UI (user layer) and Agent Client Protocol (editor layer) complete it outside AAIF as far as its pages show. MCP's July 2026 revision is a breaking change that materially helps gateways and multi-tenant scale.

### Cited Findings
**MCP**
- Latest spec **2026-07-28**, final, released that day — [GitHub release](https://github.com/modelcontextprotocol/modelcontextprotocol/releases), [MCP blog](https://blog.modelcontextprotocol.io/) [primary]
- Breaking changes: removes protocol sessions and `Mcp-Session-Id`; removes the initialize handshake; adds required `server/discover`; replaces server-initiated requests with Multi Round-Trip Requests; removes SSE resumability, so a broken stream loses the in-flight request and the client must re-issue it; moves tasks to the official extension `io.modelcontextprotocol/tasks` — [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) [primary]
- Roots, Sampling and Logging are deprecated, as is Dynamic Client Registration in favour of Client ID Metadata Documents. There is a new lifecycle policy with a minimum 12-month deprecation window. The spec adds `Mcp-Method` and `Mcp-Name` routing headers, `ttlMs` and `cacheScope` cache fields, and OpenTelemetry trace context in `_meta` — [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) [primary]
- Gateway impact: no sticky routing; header-based routing; dual-era backends during migration. Enterprise Managed Authorization (ID-JAG, "Cross App Access") is a formal extension — [agentgateway blog](https://agentgateway.dev/blog/2026-08-03-new-mcp-spec-revision/) [vendor]
- Beta SDKs (Python, TS, Go, C#) arrived 2026-06-29; Ruby SDK Tier 1 on 2026-10-30; MCP joined AAIF on 2025-12-09 — [MCP blog](https://blog.modelcontextprotocol.io/) [primary]
- Licence is transitioning from MIT to Apache-2.0, with docs under CC-BY-4.0 — [LICENSE](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/LICENSE) [primary]. 9,631 stars — [repo](https://github.com/modelcontextprotocol/modelcontextprotocol)
- **Registry**: launched in preview 2025-09-08 — [MCP blog](https://blog.modelcontextprotocol.io/). README still shows the 2025-10-24 API freeze v0.1 note while code is at v1.8.8 (2026-11-06) — [repo](https://github.com/modelcontextprotocol/registry). I found no GA declaration.

**AAIF**
- Hosted projects: Agent Router, Agent Skills, A2A, AGENTS.md, agentgateway, goose, MCP, Open Responses, OpenEnv, OpenSandbox — [AAIF projects](https://aaif.io/projects/) [primary]; the [about page](https://aaif.io/about/) lists only five, so that page may be stale.
- Platinum members: AWS, Anthropic, Block, Bloomberg, Cloudflare, Google, Microsoft, OpenAI — [AAIF about](https://aaif.io/about/) [primary]

**A2A**
- v1.0 is the first stable spec: multi-protocol support, enterprise multi-tenancy, signed Agent Cards. 150+ supporting organizations; SDKs in Python, JS, Java, Go and .NET; integrated in Azure AI Foundry, Copilot Studio and Bedrock AgentCore — [Linux Foundation, 2026-04-09](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year) [primary, foundation-reported]
- Repo: 26,437 stars, Apache-2.0, v1.0.1 (2026-05-28), tag v1.1.0-rc — [a2aproject/A2A](https://github.com/a2aproject/A2A)
- Joined AAIF on 2026-08-17 per a single secondary source — [Latent Space wiki](https://latent.space/wiki/aaif/); membership itself is confirmed by [AAIF](https://aaif.io/projects/).

**ACP (name collision, keep the three apart)**
- IBM **Agent Communication Protocol** merged into A2A in 2025; repo is archived with last release v1.0.3 (2025-08-21) — [i-am-bee/acp](https://github.com/i-am-bee/acp) [primary]; merge context — [Tyk](https://tyk.io/?p=7846) [secondary]
- **Agent Client Protocol** (editor to agent): 4,590 stars, Apache-2.0, schema-v1.29.1 (2026-11-02) — [repo](https://github.com/agentclientprotocol/agent-client-protocol). JetBrains and Zed launched an ACP Registry beta on 2026-01-28 — [gihyo](https://gihyo.jp/article/2026/01/acp-registry); JetBrains lists Junie, Claude Agent and Codex — [JetBrains](https://blog.jetbrains.com/ja/blog/2026/02/19/acp-agent-client-protocol-jetbrains-ide/) [primary]
- An unrelated "Agentic Commerce Protocol" also uses the acronym — [note](https://crystl.dev/blog/agent-communication-protocols/) [secondary]

**Others**
- AGENTS.md: 25,373 stars, MIT, no versioned releases, last push 2026-09-10 — [repo](https://github.com/agentsmd/agents.md)
- Agent Skills: `SKILL.md` folders with progressive disclosure; originally developed by Anthropic, released as open standard — [agentskills.io](https://agentskills.io/home) [primary]. 27,024 stars, Apache-2.0, no tagged releases — [repo](https://github.com/agentskills/agentskills)
- AG-UI: event-based agent-to-frontend protocol. First-party support for Agent Framework, ADK, Strands, Mastra, Pydantic AI, Agno, LlamaIndex, and partners LangChain and CrewAI; community support for Claude Agent SDK and OpenAI Agents SDK — [AG-UI docs](https://docs.ag-ui.com/introduction) [primary]. 17,051 stars, MIT, release/2026-11-06 — [repo](https://github.com/ag-ui-protocol/ag-ui)

**MCP security ecosystem**
- CVE-2026-67531 FrontMCP sandbox escape, CVSS 9.3, a single `tools/call` — [OSV](https://osv.dev/vulnerability/CVE-2026-67531) [primary]
- Trackers report 30-40+ MCP implementation CVEs in early 2026, with exec/shell injection the largest class — [tracker post](https://dev.to/piiiico/mcp-security-vulnerabilities-in-2026-40-cves-and-counting-4pco) [secondary, counts approximate]
- OX Security's April 2026 disclosure of 10 CVEs around unsandboxed commands, disputed by Anthropic as requiring explicit user permission — [summary](https://www.pulse.bot/ai/news/ai-agents-using-anthropic-mcp-could-be-a-vector-for-supply-chain-attacks-claim-researchers-b2a093a0-c1c8-4e15-b1a8-45e6f2846857/) [secondary]

### Inferences
- Adopt: MCP (tools), AGENTS.md and Skills (repo and department know-how), OTel propagation in `_meta`. Build new MCP servers against 2026-07-28, stateless, with explicit handles, and keep a dual-era gateway.
- A2A only at real trust or team boundaries (for example between department-owned agent systems). Within one orchestrated pipeline, a durable engine call is simpler and more auditable.
- SSE resumability removal plus tasks-as-extension means long-running tool calls should be modelled as durable-engine jobs or task handles, not long-lived streams.
- Do not auto-consume the public MCP registry. Run a curated internal registry and allowlist, given the CVE volume.
- Skills and AGENTS.md give the strongest anti-lock-in return for least cost: prompts and processes port across harnesses.

### Gaps
- Independent adoption numbers are weak. A2A's 150+ is supporter count, not production traffic.
- A figure of 9,652 official MCP registry records (May 2026) appeared in one search summary without a verifiable origin; do not cite it without checking.
- I did not find a primary AAIF announcement for A2A's date of joining, or AAIF project stage assignments.
- AG-UI's governance owner was not established; it does not appear on the AAIF list.

## Evidence on multi-agent pattern tradeoffs

### Takeaway
Evidence favours centralized orchestrator-worker for parallelizable read-heavy work, and single-writer or pipeline structures for sequential code changes. Coordination is expensive and error-amplifying; decentralized swarms can work only with strong external test harnesses and large budgets. Most viral failure-rate figures are unreliable.

### Cited Findings
- **Google Research and academic collaborators**: independent multi-agent amplifies errors up to 17.2x; centralized contains it to 4.4x. Centralized gains +80.9% on parallelizable finance tasks; every multi-agent variant loses 39-70% on sequential planning; coordination tax grows disproportionately with tool count (16+) — [Google blog, 2026-01-28](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/) [primary]
- The same paper's later version differs: v3 (2026-04-08) reports 260 configurations, six benchmarks, +80.8%, -70.0%, R² 0.373, and 87% best-architecture identification, versus 180 configurations, four benchmarks and R² 0.513 in the blog — [arXiv 2512.08296](https://arxiv.org/abs/2512.08296) [primary]. **Cite the version explicitly.** Benchmarks are finance, planning and similar, not end-to-end SDLC.
- **Anthropic**: multi-agent research beat single-agent Opus 4 by 90.2% on its internal eval, and parallelism cut research time by up to 90%. Multi-agent systems use about 15x the tokens of chat (agents about 4x); token usage explains about 80% of BrowseComp variance. Anthropic explicitly says tasks needing shared context, "such as most coding work", fit poorly — [Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system) [primary, vendor self-report]
- **Cognition (2026-04-22)**: multi-agents work best when "writes stay single-threaded" and extra agents contribute intelligence. Working patterns: clean-context generator-verifier review (2 bugs per PR, about 58% severe, self-reported), "smart friend" escalation, and manager-child map-reduce. Unstructured swarms are "mostly a distraction". Open problems: over-prescriptive managers, children not reporting back, discovery propagation — [Cognition](https://cognition.com/blog/multi-agents-working) [primary]
- **Anthropic C compiler counter-example**: 16 parallel agents, no orchestrator agent, lock files in git, nearly 2,000 sessions over two weeks, just under $20,000, about 100k lines of Rust. Limits: not a drop-in compiler, still used GCC for some phases, new features often broke existing ones — [Anthropic, 2026-02-05](https://www.anthropic.com/engineering/building-c-compiler) [primary]
- **Cursor**: planner and worker hierarchy with hundreds of concurrent workers; workers do not coordinate with each other. Earlier locking collapsed 20 agents to the effective throughput of 2-3. Fresh starts are still needed to fight drift — [Cursor](https://cursor.com/blog/scaling-agents) [primary, reached through search excerpt]. A cost comparison ($1,339 to $10,565 across model mixes; workers at least 69% of tokens) — [The Decoder](https://the-decoder.com/cursors-agent-swarm-suggests-cheaper-models-can-handle-most-coding-when-frontier-models-plan-the-work/) [secondary; not confirmed against a primary post]
- **Stripe**: blueprint pipeline with bounded retries yields 1,300+ merged PRs per week, human-reviewed — [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2) [primary]
- **Failure research**: MAST defines 14 failure modes in 3 categories (system design, inter-agent misalignment, task verification) from 1,600+ traces across 7 frameworks, kappa 0.88 — [Cemri et al., arXiv 2503.13657](https://arxiv.org/abs/2503.13657) [primary]. A 41-86.7% failure range is attributed to it — [secondary](https://steady.page/software-insights/posts/a18362c8-9807-4d1a-8a7e-fb5604ecac3f); the abstract page does not state it and I did not check the paper body. Those models (GPT-4, Claude 3 era) are also old.
- **Discard**: "68% fail within 72 hours" and "6.25x cost, 23% vs 5% error" come from promotional posts citing no study — [DEV](https://dev.to/kavinkimcreator/68-of-multi-agent-deployments-fail-within-72-hours-the-models-are-fine-the-coordination-layer-is-7bb)
- Claude Code docs: for sequential tasks, same-file edits or many dependencies, a single session or subagents are more effective — [Claude docs](https://code.claude.com/docs/en/agent-teams) [primary]

| Pattern | Best evidence for | Best evidence against | SDLC placement (my inference) |
|---|---|---|---|
| Deterministic pipeline with agent nodes | Stripe volume; sequential multi-agent loss | Less adaptive to novel work | Factory backbone |
| Orchestrator-worker | Google 4.4x containment; Anthropic 90.2% | About 15x tokens; manager over-specification | Research, review, test generation, multi-repo map-reduce |
| Generator-verifier | Cognition bugs per PR | Looping risk without a filter bridge | Every code-writing stage |
| Planner-worker at scale | Cursor | High spend, drift, integration burden | Migrations with strong test oracles |
| Swarm or peer | C compiler result with strong tests | 17.2x (independent topology, nearest analogue); Cognition rejection | Experiments only |

### Inferences
- Default design rule: **one writer per branch or module, many readers and verifiers, deterministic merge gates**.
- Budget multi-agent at about an order of magnitude above single-session work until measured locally. The 15x figure is for research, not coding, and should be treated as a planning prior only.
- A strong external oracle (test suite, compiler) is the common success factor across the C compiler, Cursor and Stripe cases. Departments with weak tests should not expect swarm or long-running results to transfer.
- Google's numbers measure "independent" agents, which is not identical to modern self-coordinating swarms, so the 17.2x figure is an analogue.

### Gaps
- No rigorous public study compares these patterns on real enterprise SDLC work with defect, rework and cost outcomes. Vendor numbers are self-reported.
- Coordination failure rates using current models (2026 generation) were not found.
- An SSRN preprint on coordination architectures exists, but its results were not read — [SSRN 6687518](https://papers.ssrn.com/abstract=6687518).

## Agent infrastructure: sandboxes, gateways, observability, evaluation, identity, cost, audit, and security incidents

### Takeaway
Self-hostable, neutral building blocks exist for each layer, but the gateway layer is the riskiest: LiteLLM has a real supply-chain compromise and ongoing critical advisories, Portkey's OSS gateway has gone quiet since its acquisition, and MCP gateways have had cross-tenant and auth flaws. Daytona's open repo is archived. Plan for least privilege, pinned artifacts, and per-department isolation at the infrastructure level rather than trusting application-level tenancy.

### Cited Findings

| Layer / tool | Repo | Stars | Licence | Latest | Health flag |
|---|---|---|---|---|---|
| K8s agent-sandbox | [kubernetes-sigs/agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) | 4,535 | Apache-2.0 | v1.0.10 (2026-11-06) | Active |
| E2B | [e2b-dev/E2B](https://github.com/e2b-dev/E2B) | 14,752 | Apache-2.0 | python-sdk 2.63.0 (2026-11-04) | Active |
| Daytona | [daytonaio/daytona](https://github.com/daytonaio/daytona) | 71,365 | Root has only README and assets; LICENSE only at the v0.190.0 tag | v0.190.0 (2026-06-23) | **ARCHIVED; core now private** |
| Firecracker | [firecracker-microvm/firecracker](https://github.com/firecracker-microvm/firecracker) | 37,874 | Apache-2.0 | v1.17.0 (2026-09-10) | Active |
| gVisor | [google/gvisor](https://github.com/google/gvisor) | 19,897 | Apache-2.0 | 2026-11-02 | Active |
| Kata Containers | [kata-containers/kata-containers](https://github.com/kata-containers/kata-containers) | 9,093 | Apache-2.0 | 4.3.1 (2026-10-21) | Active |
| microsandbox | [superradcompany/microsandbox](https://github.com/superradcompany/microsandbox) | 8,973 | Apache-2.0 | v0.7.13 (2026-11-01) | **Moved** from zerocore-ai |
| Anthropic sandbox-runtime | [anthropics/sandbox-runtime](https://github.com/anthropics/sandbox-runtime) | 7,012 | Apache-2.0 | v0.0.81 (2026-10-20) | **Moved** from anthropic-experimental; 0.0.x |
| LiteLLM | [BerriAI/litellm](https://github.com/BerriAI/litellm) | 62,848 | MIT outside `enterprise/` ([LICENSE](https://github.com/BerriAI/litellm/blob/main/LICENSE)) | v1.108.2 (2026-11-06) | Very active; 4,042 open PRs; incidents below |
| Portkey gateway | [Portkey-AI/gateway](https://github.com/Portkey-AI/gateway) | 13,335 | MIT | v1.15.2 (2026-01-12) | **Stale**: last push 2026-05-25, 0 commits in 30d |
| Kong | [Kong/kong](https://github.com/Kong/kong) | 44,366 | Apache-2.0 core | 3.9.4 (2026-10-12) | AI/MCP features largely Enterprise |
| Agent Router (ex Envoy AI Gateway) | [theagentrouter/agent-router](https://github.com/theagentrouter/agent-router) | 2,365 | Apache-2.0 | v1.2.1 (2026-11-06) | **Moved**; AAIF project |
| agentgateway | [agentgateway/agentgateway](https://github.com/agentgateway/agentgateway) | 6,000 | Apache-2.0 | v1.7.0 (2026-10-27) | Active; AAIF project |
| IBM ContextForge | [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge) | 4,765 | Apache-2.0 | v1.1.0 (2026-10-20) | Active; advisories |
| Docker MCP Gateway | [docker/mcp-gateway](https://github.com/docker/mcp-gateway) | 1,627 | MIT | tag v0.44.3 | Pre-1.0; advisories |
| Microsoft MCP Gateway | [microsoft/mcp-gateway](https://github.com/microsoft/mcp-gateway) | 909 | MIT | tag v1.0.0 | Active |
| Langfuse | [langfuse/langfuse](https://github.com/langfuse/langfuse) | 37,116 | MIT outside `ee/` dirs; © ClickHouse, Inc. ([LICENSE](https://github.com/langfuse/langfuse/blob/main/LICENSE)) | v4.76.0 (2026-11-05) | Active |
| Arize Phoenix | [Arize-ai/phoenix](https://github.com/Arize-ai/phoenix) | 12,181 | **Elastic License 2.0**, not OSI ([LICENSE](https://github.com/Arize-ai/phoenix/blob/main/LICENSE)) | 2026-11-07 | Active |
| LangSmith SDK | [langchain-ai/langsmith-sdk](https://github.com/langchain-ai/langsmith-sdk) | 1,083 | MIT (SDK only) | v0.14.9 (2026-10-15) | **Archived 2026-10-16; Python SDK moved to langchain-ai/langsmith-python** |
| OTel semconv | [open-telemetry/semantic-conventions](https://github.com/open-telemetry/semantic-conventions) | 684 | Apache-2.0 | v1.45.0 (2026-11-02) | GenAI moved out |
| OpenLLMetry | [traceloop/openllmetry](https://github.com/traceloop/openllmetry) | 7,523 | Apache-2.0 | 0.63.0 (2026-11-05) | Active |
| Helicone | [Helicone/helicone](https://github.com/Helicone/helicone) | 6,272 | Apache-2.0 | release 2025-08-21; deploy tag 2026-10-10 | Formal releases stale |
| MLflow | [mlflow/mlflow](https://github.com/mlflow/mlflow) | 28,706 | Apache-2.0 | v3.18.0 (2026-11-04) | Active |
| promptfoo | [promptfoo/promptfoo](https://github.com/promptfoo/promptfoo) | 26,915 | MIT | 0.125.1 (2026-10-25) | Active |
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | 19,341 | Apache-2.0 | python-v4.2.4 (2026-09-22) | Active |
| SPIRE | [spiffe/spire](https://github.com/spiffe/spire) | 2,608 | Apache-2.0 | v1.16.0 (2026-11-05) | Active |
| OpenBao | [openbao/openbao](https://github.com/openbao/openbao) | 8,910 | MPL-2.0 | v2.7.2 (2026-10-26) | Active |
| OPA | [open-policy-agent/opa](https://github.com/open-policy-agent/opa) | 12,442 | Apache-2.0 | v1.22.2 (2026-11-06) | Active |
| Cedar | [cedar-policy/cedar](https://github.com/cedar-policy/cedar) | 1,849 | Apache-2.0 | v4.14.1 tag (2026-11-02) | Active |

**Sandboxes**
- Agent-sandbox is a *sandbox orchestrator* that delegates isolation to gVisor or Kata through `RuntimeClass`. API is `agents.x-k8s.io/v1beta1`, with `SandboxTemplate` and `SandboxWarmPool` — [repo](https://github.com/kubernetes-sigs/agent-sandbox) [primary]. Secondary sources citing v0.5.2 or `apps.k8s.io/v1alpha1` are stale — [example](https://businesstechnavigator.com/news/kubernetes-agent-sandbox-crd-gke-2026). Cold start under one second with warm pools, per Google — [InfoQ](https://www.infoq.com/news/2026/05/gke-agent-sandbox-hypercluster) [secondary]
- Daytona README: "no longer maintained"; "As of June 2026, Daytona's core development has moved to a private codebase" — [repo](https://github.com/daytonaio/daytona) [primary]. A third party lists it as AGPL-3.0 before this — [listing](https://legost.in/en/open-source/daytona) [secondary]. Its default isolation is disputed (Docker with optional Kata or Sysbox per one source) — [source](https://www.softwareseni.com/ai-agent-sandboxing-explained-why-docker-is-not-enough-and-what-actually-works) [secondary]
- E2B uses Firecracker microVMs per comparison articles — [source](https://fast.io/resources/best-ai-agent-sandboxes/) [secondary]. OpenSandbox is now an AAIF project — [AAIF](https://aaif.io/projects/) [primary]
- Isolation flaw example: a Smithery path traversal let attackers read Docker credentials of other users' MCP deployments — [summary](https://www.heyuan110.com/posts/ai/2026-03-10-mcp-security-2026/) [secondary]

**Gateways and incidents**
- **LiteLLM PyPI compromise, 2026-03-24**: malicious 1.82.7 and 1.82.8; credential stealer exfiltrating to `models.litellm.cloud`; suspected origin in the Trivy dependency in CI, with direct PyPI upload bypassing CI. **Not affected**: official proxy Docker image, LiteLLM Cloud, source installs, and pinned ≤1.82.6. Releases resumed at v1.83.0 through a new pipeline; Docker images are cosign-signed from v1.83.0-nightly — [LiteLLM advisory](https://docs.litellm.ai/blog/security-update-march-2026) [primary]. The window is inconsistent even in the primary (about 40 minutes versus 10:39-16:00 UTC), so assume the wider one. Kubernetes backdoor attempts and TeamPCP attribution — [NetSPI](https://www.netspi.com/blog/executive-blog/ai-ml-pentesting/litellm-supply-chain-compromise/), [Comet](https://www.comet.com/site/blog/litellm-supply-chain-attack/) [secondary]
- **LiteLLM ongoing**: 18 published advisories, 11 high or critical, including *unauthenticated file read leaks master key on Enterprise proxy* (critical, 2026-10-27), proxy-admin privilege escalation CVE-2026-114612 (critical, 2026-09-30), MCP auth bypass CVE-2026-59822 (high, 2026-06-30) — [advisories](https://github.com/BerriAI/litellm/security/advisories) [primary]
- **Portkey**: Palo Alto Networks acquisition announced 2026-04-30 and reported closed about June 2026, to anchor Prisma AIRS; $140M per 10-Q as cited by a competitor — [Tetrate](https://tetrate.io/learn/ai/portkey-palo-alto-acquisition); [Technology Decisions](https://www.technologydecisions.com.au/content/security/news/palo-alto-completes-acquisition-of-portkey-1780733452). Dates and price conflict across sources [secondary]
- **Kong**: MCP and several AI plugins require Enterprise or Konnect; MCP came in 3.12 and per-tool ACLs in 3.13 — [comparison](https://www.getmaxim.ai/articles/best-mcp-gateways-in-2026.md) [competitor-authored]. Whether OSS includes AI Proxy conflicts across sources — [comparison](https://web-9ja.pages.dev/blog/llm-gateway-pricing-comparison-june-2026/) [secondary]
- **MCP gateways**: agentgateway *stateful MCP sessions can cross routes and overwrite the authorization policy* (high, 2026-07-27) — [advisories](https://github.com/agentgateway/agentgateway/security/advisories). IBM ContextForge *cross-team A2A agent takeover* (high, 2026-09-07) and a critical SSRF (2026-08-25) — [advisories](https://github.com/IBM/mcp-context-forge/security/advisories). Docker MCP Gateway CVE-2026-76092 (unauthenticated proxied tools, critical), CVE-2026-76099 (bind mount, critical), CVE-2026-76094 (image signature verification off by default) — [advisories](https://github.com/docker/mcp-gateway/security/advisories) [all primary]
- Docker's gateway reportedly lacks org-wide RBAC and central audit — [comparison](https://www.getmaxim.ai/articles/top-5-enterprise-mcp-gateways-in-2026/) [competitor-authored]

**Observability**
- OTel GenAI conventions moved to a dedicated repo; all `gen_ai.*` remains at **Development** status; agent operations include `invoke_agent`, `invoke_workflow`, `plan`, `execute_tool` — [Dash0, 2026-09-14](https://www.dash0.com/knowledge/opentelemetry-genai-semantic-conventions-explained) [secondary]. No public stabilization timeline as of May — [Greptime](https://greptime.com/blogs/2026-05-09-opentelemetry-genai-semantic-conventions). A claim of June 2026 stabilization looks wrong — [BTN](https://businesstechnavigator.com/news/opentelemetry-genai-semantic-conventions-agents-2026)
- ClickHouse acquired Langfuse in January 2026 with MIT core unchanged per reports — [morphllm](https://www.morphllm.com/comparisons/langfuse-alternatives) [secondary]; LICENSE confirms ClickHouse copyright [primary, table above]. Langfuse Cloud v4-only from 2026-11-16 — [comparison](https://ecorpit.com/llm-observability-langfuse-langsmith-braintrust-phoenix-opik-2026/) [secondary]
- Dynatrace reportedly announced buying Arize on 13 August and closed on 1 October, and says it will keep supporting Phoenix and AX — [OpenObserve](https://openobserve.ai/blog/langfuse-alternatives/) [competitor-authored, unconfirmed]. Phoenix maps `gen_ai.*` to OpenInference from v15.10.0 — same source.

**Cost, audit, tenancy**
- Paperclip has company, agent and project budgets — [repo](https://github.com/paperclipai/paperclip) [primary]. Agent HQ uses audit log entries and usage-based billing — [GitHub docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents) [primary]. MCP `cacheScope` distinguishes public and private caching at intermediaries — [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) [primary]. Enterprise Managed Authorization addresses multi-domain identity — [agentgateway](https://agentgateway.dev/blog/2026-08-03-new-mcp-spec-revision/) [vendor]. Dapr Agents uses SPIFFE — [Cloud Native Now](https://cloudnativenow.com/kubecon-cloudnativecon-europe-2026/cncf-announces-general-availability-of-dapr-agents-v1-0-for-production-ai-workloads/) [secondary]

### Inferences
- **Reference stack (vendor-neutral) for validation**: Temporal plus a thin SDLC service; agent harnesses (Claude Code, Codex, OpenCode) as swappable workers; kubernetes-sigs/agent-sandbox with gVisor or Kata (microVM for untrusted code); agentgateway or Agent Router for LLM and MCP traffic; Langfuse with OTel export kept as raw OTLP; OpenBao plus SPIRE for short-lived credentials; OPA or Cedar for policy.
- **Tenancy**: separate namespaces, gateway keys, credentials, Langfuse projects and budgets per department; treat application-level tenancy as defence in depth only. The Paperclip, Dify, ContextForge and agentgateway advisories all show that cross-tenant bugs are common in this young category.
- **LiteLLM**: remains useful for model breadth, but only as a pinned, signed container, with no outward admin plane, and with a fast patch process. Have an exit path to an Envoy-based gateway.
- **Avoid building new work on**: the Portkey OSS gateway (uncertain future under Palo Alto) and Daytona's archived repo (managed service still possible, at the cost of lock-in).
- **Supply chain**: the LiteLLM case began in a *security scanner*; pin CI actions and tools by digest, restrict CI publish tokens, and hold the upgrade window for new AI packages.
- Keep instrumentation at OTel with an abstraction for `gen_ai.*` names, because they can still change.
- Cost control needs enforcement at the gateway and engine, because self-reported harness spend is advisory. Allocate by department key, workflow ID and stage.
- Vendor lock-in lives in the harness, tracing destination (LangSmith, vendor consoles) and hosted sandboxes, more than in the protocols.

### Gaps
- I found no primary evidence of per-tenant budget capabilities for Portkey or Kong, and LiteLLM's per-team budget capability was not checked against current docs. Most gateway comparison content is competitor-authored.
- Agent identity (SPIFFE per agent, ID-JAG deployment maturity) has little production evidence; search returned only vendor material.
- No independent latency or scale benchmark for gateways was verified.
- E2B self-hosting and its current isolation default need checking against E2B docs.
- A memory-based note that OpenAI may have acquired promptfoo in 2026 was not verified in this research; check before citing promptfoo as neutral.
- Codex CLI CVEs and the "IDEsaster" research were searched for but not found in my results.
- OpenAI's Symphony launch post returned HTTP 403, so only the repo was used.
- Advisory items were read at summary level; exploitability and fix versions need confirmation before being cited in a security recommendation.
