# End-to-End AI Software Factory Platforms, Reference Architectures, and Governance (as of 7 Nov 2026)

Evidence labels used throughout:
- **[Docs]** vendor's own docs or pricing page, read directly. Reliable for price and feature, not for outcomes.
- **[Self]** self-reported outcome from the company that ran the work.
- **[Vendor-research]** research by a company selling into this market.
- **[Secondary]** press, blog, or aggregator. Not checked against a primary source.
- **[Measured]** telemetry or controlled study. Says so if a vendor produced it.

Most pages were read through a summarising fetch tool, so check exact figures against the page before quoting them in a decision document. Some [Secondary] items came from search summaries that did not tie each claim to one URL. In those cases the link is the best-matching result and was not opened: Factory's SOC 2/ISO 42001 claim, 8090's funding, Harness tiers, Atlassian's training-data change, Kuwa/TAIDE on-premises, and "no penalties" for the Taiwan Act.

## Commercial platforms: SDLC coverage, governance, pricing, deployment

### Takeaway
No vendor credibly covers requirements through operations as one governed product; according to Port's account of Gartner's September 2026 note, no vendor yet offers a complete plug-and-play factory. The market has converged on usage-metered pricing (seat plus credits or tokens), and enterprise prices, VPC terms, and on-premises terms are almost always sales-quoted.

### Cited Findings

**Market framing**
- Gartner published *Innovation Insight: AI Software Factories* (abstract: without a factory, agents are trapped inside a slow, human-coordinated SDLC; the factory is the "North Star") — [Gartner, abstract only, paywalled; direct fetch returned 403](https://www.gartner.com/en/documents/8397881).
- Per Port's blog: authors Nabeeha Ahmed and Tigran Egiazarov, published 17 Sep 2026. It describes three paths: DIY, Buy, and a Horizontal platform. Port reports that Gartner predicts at least 45% of large software engineering organisations will operate one or more AI software factories by 2028, and finds 71% of teams use agents in the SDLC but most do not ship faster because humans still move work between stages. **Vendor framing: Port is named in the report and positions itself as the "horizontal" option** — [Port](https://www.port.io/blog/gartner-names-port-ai-software-factory-platform).
- Comparison articles are mostly written by vendors that rank themselves. Warp's article ranks Warp Factories first and gives no pricing for any of the six products — [Warp](https://www.warp.dev/articles/best-software-factory-platforms-2026). Use these for discovering names, not for rankings.

**Comparison table** (retrieved Nov 2026)

| Product | SDLC scope (claimed) | Published pricing | Deployment | Governance features | Evidence |
|---|---|---|---|---|---|
| **Factory** (Droids; domain moved from factory.ai to factory.com) | Described by third parties as triage through incident response | Pro $20, Plus $100, Max $200/mo; Teams $60/team + $40/seat (up to 10 seats); Business and Enterprise contact sales | Enterprise: "on-premise deployment options", data residency, dedicated compute. VPC and air-gap are not stated on the pricing page | Business: SSO/SAML/SCIM, ZDR, audit logs, autonomy-level and model controls, deny lists, network policy. Enterprise: sub-organisations, full admin controls | [Docs](https://factory.com/pricing). VPC/air-gap "Factory Private" comes from one [Secondary](https://hermes-ai.net/news/factory-private-brings-autonomous-coding-droids-inside-enterprise-networks/) write-up. SOC 2/ISO 42001 claim is [Secondary](https://www.agentastic.dev/compare/best-ai-software-factory-tools) only |
| **GitHub Copilot / Agent HQ** | Issue to PR via cloud agent; Claude and Codex as third-party agents; no native requirements or ops product confirmed here | Business 1,900 and Enterprise 3,900 AI credits per user/mo (1 credit = $0.01), pooled; overage by policy. Agent sessions also consume Actions minutes | GitHub-hosted. Enterprise Cloud residency regions: EU, Australia, US, Japan | User, cost-centre, org, and enterprise budgets; AI Controls; audit log entries for partner apps; CodeQL and secret scanning on agent output without needing GHAS | [Billing docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises); [third-party agents docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents); [residency docs](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency) |
| **GitLab Duo Agent Platform** | DevSecOps platform flows, including code review and SAST remediation | $1 per credit on demand at list; commitment pools discounted; included credits per Premium/Ultimate user. Promotional $12/$24 per user is [Secondary](https://www.usagepricing.com/blueprint/activity/gitlab-2026-08-28-duo-agent-credits-all-plans) | GitLab.com, Self-Managed, Dedicated; self-hosted models supported | User usage caps; credits dashboard. Self-hosted models still consume credits (8 requests/credit; 20% discount on flat-priced features) | [Docs](https://docs.gitlab.com/subscriptions/gitlab_credits); [GA 15 Jan 2026](https://www.businesswire.com/news/home/20260115646701/en/) |
| **Atlassian Rovo Dev** | Code review and CLI, attached to Jira context | Standalone being sunset. Code Reviewer moves into Jira/Teamwork Collection and is metered in Rovo Credits from 3 Dec 2026; $20 Standard still listed with 2,000 credits | Not verified | Not verified | [Secondary tracker citing Atlassian pages, 15 Oct 2026](https://www.usagepricing.com/blueprint/activity/atlassian-2026-10-15-rovo-dev-standalone-sunset). **Check before committing: the product is being restructured** |
| **Cognition Devin** | Marketed for planning, migrations, on-call | Pro $20, Max $200, Teams $80 + $40/seat (up to 200 users), Enterprise custom | Enterprise VPC and dedicated option | SAML/OIDC SSO on Enterprise; centralised multi-org admin; per-org ACU cap stops work when reached | [Pricing](https://devin.ai/pricing); [billing docs](https://docs.devin.ai/admin/billing). VPC descriptions conflict between customer-cloud and Cognition-hosted single-tenant, per [Secondary](https://fast.io/resources/devin-enterprise-guide/), so confirm in the order form |
| **AWS Kiro** | Spec-driven IDE/web; hooks and workflows | Pro $20 to Power $200; Teams/Enterprise $20/seat with 500 pooled credits; extra credits $0.04; GovCloud about 20% higher | SaaS billed through AWS; location follows the Kiro profile's AWS Region | Groups, credit pools, limits, alerts at 80%/100%, SSO | [Docs](https://kiro.dev/pricing/) |
| **AWS frontier agents** | Kiro autonomous agent (build), Security Agent, DevOps Agent | DevOps Agent $0.0083 per agent-second; 2-month trial; support-plan credits of 30/75/100%. Release management free in preview. Security Agent price not found | AWS service | Not reviewed in this pass | [DevOps Agent pricing](https://aws.amazon.com/devops-agent/pricing/). Security and DevOps GA by 31 Mar 2026, Kiro autonomous undated at that time: [Secondary](https://thelettertwo.com/2026/03/31/aws-frontier-agents-ga-devops-security/) |
| **AWS Transform** | Modernisation: Windows, mainframe, VMware, custom | Listed agents free; custom agent $0.035 per agent-minute; managed .NET free up to a cap | AWS service | Not reviewed | [Pricing](https://aws.amazon.com/transform/pricing) |
| **Google Antigravity + Gemini Enterprise** | Agent IDE/CLI on a governed Google Cloud base | Consumption, or Standard/Plus seats with $10/$15 pooled seat credits; third-party models by tokens. Seat prices of $21 to $60+ are [Secondary from a competitor](https://www.gosearch.ai/faqs/gemini-enterprise-pricing/) | Endpoints `global`, `us`, `eu`; VPC-SC supported; standalone IDE not supported for enterprise | IAM roles, per-region enablement, third-party model opt-in, request/response logging | [Docs](https://antigravity.google/docs/enterprise) |
| **Azure SRE Agent** | Operations only | 4 AAU per agent-hour always-on plus token-based active flow; docs' estimator example uses $0.1/AAU; no list price on that page | Azure | Monthly active-flow AAU limit (min 500, max 1,000,000). An older result states 200,000, so the limit has changed or sources conflict | [Docs](https://sre.azure.com/docs/reference/pricing-billing). GA status not stated |
| **Microsoft Agent 365 / Foundry** | Agent governance (Agent 365), agent building (Foundry); not an SDLC product | $15/user/mo, GA 1 May 2026; runtime billed separately | Microsoft cloud | Entra Agent ID, registry, Conditional Access; multicloud registry sync in preview; runtime enforcement reportedly incomplete at launch | All [Secondary](https://redresscompliance.com/microsoft-agent-365-what-it-is-what-it-isnt.html); Microsoft's own pages not read |
| **Harness** | CI/CD, testing, security, FinOps; Autonomous Worker Agents (30 Jun 2026); Agent DLC (21 Jul 2026) | Not public; [Secondary](https://codewave.com/feeds/blog/best-ai-native-sdlc-platform-2026) says modular enterprise tiers | Not confirmed for agents | Pipeline-embedded agents; AI governance claims in the press release | [Worker Agents press](https://www.harness.io/press-and-news/harness-launches-autonomous-worker-agents-for-software-delivery); [Agent DLC press](https://www.harness.io/press-and-news/introducing-harness-agent-dlc). A [Secondary report](https://letsdatascience.com/news/harness-adds-autonomous-ai-agents-for-devops-automation-e27b029c) notes conflict between "GA" and "Limited Preview" for the Marketplace |
| **Tessl** | Skills registry, evaluation, governance. Pivoted away from its spec framework. Its homepage title now uses "Software factory" | Registry free; enterprise custom; credits unverified | SaaS (not verified further) | Eval and security scores | [Site](https://tessl.io/); [Secondary](https://codemyspec.com/blog/tessl-review) on pivot. A component, not a factory |
| **Augment** | Context engine, CLI, agents | Standard $20, Business $100 flat with equal usage; LLM at provider list **plus 40% fee**; Enterprise custom | Multi-region compute; residency option; no VPC/on-prem detail on the page | SSO/OIDC/SCIM, CMEK, ISO 42001, audit trails, SIEM | [Pricing](https://www.augmentcode.com/pricing); [ISO 42001 announcement](https://www.augmentcode.com/blog/augment-code-is-the-first-ai-coding-assistant-to-be-iso-iec-42001-certified) |
| **Amp** | Coding agent; per-thread "orbs" machines | Not captured | Web, desktop, mobile, CLI; a self-hosted orbs page is listed | Enterprise pages: MCP allowlist, thread visibility, entitlements, allowed models, minimal retention | [Manual](https://ampcode.com/manual). Sourcegraph relationship not stated there |
| **Port** | Horizontal control plane: context lake, workflows, agent registry | Not found; [Secondary](https://toolradar.com/tools/port) says seats plus entities | SaaS assumed; not verified | RBAC, SSO, teams, audit trails, scorecards. No budget controls mentioned | [Docs](https://docs.port.io/getting-started/agentic-sdlc-pillars/) |
| **Warp Factories, 8090** | New "software factory" entrants | Not found | Warp: customer-hosted or managed (its own claim) | SSO/SCIM, audit export, ZDR (Warp's own claim) | [Warp](https://www.warp.dev/articles/best-software-factory-platforms-2026). 8090's reported $135M Series A is one [Secondary](https://www.agentastic.dev/compare/best-ai-software-factory-tools) item, unverified |

**Details that affect the decision**
- GitHub organisation and enterprise policies **do not apply to local agents in VS Code**, which have separate settings. Partner agent apps appear in the audit log but are not listed among installed GitHub Apps — [GitHub](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents).
- GitHub budgets enforce only when "Stop usage when budget limit is reached" is on, paid usage is enabled by default on Business and Enterprise, and individual GitHub Apps cannot have their own budgets — [GitHub](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises).
- GitLab charges credits for self-hosted models too, failed self-hosted flows may still consume credits, and flat-priced features are charged in full even on failure — [GitLab](https://docs.gitlab.com/subscriptions/gitlab_credits). Purchase requires subscription sync, and the doc does not address air-gapped use.
- Azure SRE Agent's always-on charge continues after the active-flow limit is hit, and until the agent is deleted — [Microsoft](https://sre.azure.com/docs/reference/pricing-billing).
- AWS DevOps Agent's connected services, such as CloudWatch Logs Insights, are billed separately — [AWS](https://aws.amazon.com/devops-agent/pricing/).
- **Correction to a likely error in circulation:** one search synthesis attributed "Autonomous Worker Agents, June 2026" to GitLab. Harness's own press release shows this is a Harness product — [Harness](https://www.harness.io/press-and-news/harness-launches-autonomous-worker-agents-for-software-delivery).
- Atlassian reportedly changed default AI-training data collection in August 2026, with no opt-out below Enterprise. This is single-source and needs checking — [UsagePricing](https://usagepricing.com/blueprint/atlassian).

### Inferences
- Calculated from the list prices above: DevOps Agent about $29.88 per active hour; Transform custom about $2.10 per agent-hour; Azure SRE always-on about $290 per agent-month if $0.1/AAU applied (the page only uses that as an example). Metering units differ too much to compare without a pilot workload.
- Three vendors are the plausible anchors, depending on where the customer's source and pipelines already live: GitHub, GitLab, and Harness. Factory and Devin are agent-first buys. Port, Coder, and Backstage are control-plane or environment layers, not agents.
- GitLab Self-Managed with self-hosted models is the most clearly documented route to on-premises inference among the SCM-native options reviewed, but it needs an air-gap check with the account team.
- Rovo Dev's restructuring and Tessl's pivot show that product shape can change within a contract term. That favours short terms and portable assets such as specs, skills, and MCP servers.

### Gaps
- Enterprise prices for Factory, Devin, Harness, Port, Coder, Augment, and Amp are not public. Budget figures need quotes.
- Could not confirm Agent HQ as a current product name: the expected docs URL returned 404, and the [press coverage](https://venturebeat.com/ai/githubs-agent-hq-aims-to-solve-enterprises-biggest-ai-coding-problem-too) dates from the Oct 2025 announcement. The third-party agents docs are live.
- Kiro autonomous agent's current GA status, and AWS Security Agent's price, were not found.
- Factory's certifications were not verified from its Trust Center.
- Atlassian's own pages on Rovo Dev residency and governance were not reached.

## Open-source platforms and projects

### Takeaway
All the named projects exist, but they are not equivalent. The healthy, widely used ones are building blocks (agent runtime, spec method, portal, environments). The projects literally calling themselves "AI software factory" are mostly very small, and StrongDM Attractor is verifiably a spec-only repository with no commits since March 2026.

### Cited Findings

Stars, license, and activity from the GitHub API on 2026-11-07.

| Repo | Stars | License | Last commit | Latest release | Health |
|---|---|---|---|---|---|
| [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | 92,839 | MIT (as detected) | 2026-11-07 | v1.37.0, 2026-11-06 | Very active |
| [FoundationAgents/MetaGPT](https://github.com/FoundationAgents/MetaGPT) | 71,204 | MIT | 2026-01-21 | v0.8.1, 2024-04-22 | **Stalled: about 9.5 months without a commit** |
| [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 55,085 | MIT per [LICENSE file](https://raw.githubusercontent.com/bmad-code-org/BMAD-METHOD/main/LICENSE) (API shows NOASSERTION) | 2026-11-06 | v6.12.4, 2026-10-14 | Active |
| [OpenBMB/ChatDev](https://github.com/OpenBMB/ChatDev) | 34,700 | Apache-2.0 | 2026-10-30 | v2.2.0, 2026-03-23 | Active |
| [backstage/backstage](https://github.com/backstage/backstage) | 34,793 | Apache-2.0 | 2026-11-07 (push) | Stable docs at v1.56.0 | Very active |
| [onyx-dot-app/onyx](https://github.com/onyx-dot-app/onyx) | 33,260 | MIT except `ee` directories under the Onyx Enterprise License, per [LICENSE](https://raw.githubusercontent.com/onyx-dot-app/onyx/main/LICENSE) | 2026-11-06 | v5.1.0, 2026-11-06 | Very active; open-core |
| [coder/coder](https://github.com/coder/coder) | 17,311 | AGPL-3.0 | 2026-11-07 (push) | Not checked | Very active; AGPL needs legal review; AI governance is a paid add-on |
| [github/spec-kit](https://github.com/github/spec-kit) | 148,212 | MIT | 2026-11-07 (push) | Not checked | Very active |
| [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) | 74,176 | MIT | 2026-11-07 (push) | Not checked | Very active; Radar: Trial |
| [gastownhall/gastown](https://github.com/gastownhall/gastown) | 18,719 | MIT | 2026-10-28 | v1.2.1, 2026-06-06 | Active |
| [strongdm/attractor](https://github.com/strongdm/attractor) | 1,341 | Apache-2.0 | 2026-03-17 | None | **Exists, but contains specs only; dormant** |
| [strongdm/cxdb](https://github.com/strongdm/cxdb) | Not checked | Not checked | 2026-08-28 | None | Exists |
| [danshapiro/kilroy](https://github.com/danshapiro/kilroy) | Not checked | Not checked | 2026-04-27 | v0.1.0, 2026-04-16 | Early; quiet for 6 months |

Repos matching "software factory" in GitHub search:

| Repo | Stars | Note |
|---|---|---|
| [dzhng/skills](https://github.com/dzhng/skills) | 1,404 | Skills for factories, not a platform |
| [alexgreensh/outsourcerer](https://github.com/alexgreensh/outsourcerer) | 192 | Multi-model orchestration |
| [mrinalwadhwa/fluent](https://github.com/mrinalwadhwa/fluent) | 125 | Self-described self-improving factory |
| [preloop/preloop](https://github.com/preloop/preloop) | 88 | Agent control plane; governs tool and model calls |
| [glassflow/tares](https://github.com/glassflow/tares) | 44 | Always-on agent platform for Claude Code |
| [holgerleichsenring/agent-smith](https://github.com/holgerleichsenring/agent-smith) | 41 | Ticket to verified PR; self-hosted |
| [OmegaMc1331/agentic-software-factory](https://github.com/OmegaMc1331/agentic-software-factory) | 25 | Self-described platform |
| [dewantrie/ai-software-factory](https://github.com/dewantrie/ai-software-factory) | 11 | Shared prompt library; no license detected |
| [8090-inc/software-factory-harness](https://github.com/8090-inc/software-factory-harness) | 9 | Harness for 8090's commercial product; no license detected |

- A curated index, [awesome-software-factories](https://github.com/varun1505/awesome-software-factories) (79 stars, CC0), lists dozens more, including corporate-origin templates [doordash-oss/agentic-orchestrator](https://github.com/doordash-oss/agentic-orchestrator), [vercel-labs/eve-software-factory-template](https://github.com/vercel-labs/eve-software-factory-template), and [addyosmani/factory](https://github.com/addyosmani/factory). These three were not independently health-checked.
- The Attractor README instructs readers to give the NLSpecs to a coding agent to build their own version; the repo holds three specs: Attractor, coding agent loop, unified LLM client — [README](https://raw.githubusercontent.com/strongdm/attractor/main/README.md). The curated list flags it as unmaintained — [awesome list](https://github.com/varun1505/awesome-software-factories).
- Stripe's Minions is built on a fork of Block's open-source goose agent, which is evidence that serious builds fork a runtime rather than adopt a "factory" product — [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents).

### Inferences
- Stars measure attention, not production fitness. MetaGPT's 71k stars with a stalled repo is the clearest case. I would not base a new factory on it, nor on ChatDev without a separate evaluation.
- The realistic open-source stack is layered: runtime (OpenHands or goose), spec method (Spec Kit, OpenSpec, or BMAD), portal (Backstage), governed environments (Coder), knowledge (Onyx). None of these alone is a factory.
- Self-described factory repos with fewer than a few hundred stars are useful as design references, not as dependencies.

### Gaps
- No independent production-readiness assessment or enterprise deployment evidence was found for any open-source factory.
- OpenHands' enterprise licensing boundary and commercial pricing were not checked. The MIT label is what the GitHub API detects at repository level.
- Onyx's fit as an SDLC component was not evaluated. MetaGPT and ChatDev current feature sets were not reviewed beyond repo metadata.

## Reference architectures and enterprise case studies

### Takeaway
The strongest case studies (Stripe, Spotify) are self-reported and rest on years of prior platform investment. Telemetry and survey data show a much weaker average outcome. Across the cases, the recurring pattern is deterministic rails plus isolated environments plus fast automated feedback, with the bottleneck moving to review and verification.

### Cited Findings

**Case studies (all self-reported)**

| Case | What they did | Metrics | What failed or was limited | Evidence |
|---|---|---|---|---|
| **Stripe Minions** | Fork of goose interleaving agent loops with deterministic steps; pre-warmed devboxes in about 10s, isolated from production and internet; Toolshed MCP server with 400+ tools, curated per agent; rules scoped by subdirectory; started from Slack, CLI, web, and internal apps | 1,000+ PRs merged weekly, fully minion-produced; later [1,300+ reported secondarily](https://software-factories.port.io/stripe) | **Humans review every change.** CI capped at two rounds for cost and diminishing returns, so some runs end unresolved. Engineers sometimes finish the work. Depends on years of developer tooling investment | [Self](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) |
| **Spotify Honk** | Fleetshift orchestration + Honk (Claude Agent SDK in own harness on Kubernetes) + Backstage exposed via MCP/CLI + Soundcheck golden-state standards + Slack | 99% weekly use; 94% report higher productivity; PR frequency +76%; 2.5M automated maintenance PRs merged, most auto-merged; Java migration in 3 days | 76% more PRs to review. Agents measurably worse in fragmented codebases. Bottleneck moved to human decisions. Specific failures not described | [Self](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint) |
| **StrongDM** | Rule that code is neither written nor reviewed by humans; scenarios held outside the codebase as holdouts; probabilistic "satisfaction"; Digital Twin Universe of Okta, Jira, Slack, Google apps | No delivery or quality metrics published. Guideline: at least $1,000 tokens per engineer per day | No failures disclosed. Small team founded July 2025. The most extreme operating model of those reviewed | [Self](https://factory.strongdm.ai/); [Simon Willison commentary](https://simonwillison.net/2026/feb/7/software-factory/) |
| **OpenAI harness engineering** | Five-month internal beta built by agents with no manually written source; harness of constraints and feedback | About 1M lines; team of three and about 1,500 PRs reported secondarily; a later account says seven engineers | Greenfield internal product, built by the model vendor. **Primary page returned 403; only secondary reading** | [InfoQ](https://www.infoq.com/news/2026/02/openai-harness-engineering-codex); [Secondary copy](https://zby.github.io/commonplace/sources/harness-engineering-leveraging-codex-agent-first-world) |
| **Anthropic Trends Report** | Case studies of Rakuten, CRED, TELUS, Zapier | TELUS 500k+ hours is third-party reporting; developers reportedly use AI in about 60% of work but can fully delegate only 0 to 20% | Vendor marketing of its own product | [Vendor](https://resources.anthropic.com/2026-agentic-coding-trends-report); [Secondary](https://dev.to/amitba/i-read-anthropics-2026-agentic-coding-trends-report-heres-what-it-actually-means-for-engineering-o6n) |
| **AWS Kiro incident** | Reported agent deletion and recreation of a live environment, causing a 13-hour Cost Explorer outage in Dec 2025 | n/a | FT-based reports blame the agent; Amazon blames misconfigured access and user error. Unresolved. Either reading supports least privilege | [The Register](https://www.theregister.com/2026/02/20/amazon_denies_kiro_agentic_ai_behind_outage/); [PC Gamer](https://www.pcgamer.com/software/ai/reports-claim-an-aws-outage-last-year-was-caused-by-an-ai-coding-tool-deciding-to-delete-and-recreate-the-environment-from-scratch-while-amazon-says-misconfigured-access-controls-were-to-blame) |
| **Dex Horthy** | Reportedly ran a lights-off factory for three months in 2025 and abandoned it as the codebase degraded | n/a | Single secondary line in a curated list; primary talk not reviewed | [awesome list](https://github.com/varun1505/awesome-software-factories); [Addy Osmani on the talk](https://addyosmani.com/blog/software-factories/) |

**Aggregate evidence**

| Source | Type | Finding | Caveat |
|---|---|---|---|
| [METR, Feb 2026](https://metr.org/blog/2026-02-24-uplift-update/) | [Measured] RCT, independent | Early 2025: tasks about 19% slower (CI +2% to +39%). Late 2025: returning developers 18% faster (CI -38% to +9%, n=10), new developers 4% faster (CI -15% to +9%) | METR calls the latest data unreliable because of selection effects, thinks true speedup is likely larger, and is redesigning the study |
| [Faros, Q2 2026](https://faros.ai/research/ai-acceleration-whiplash) | [Measured] telemetry, [Vendor-research] | 22K developers, 4K teams, high vs. low adoption: epics +66.2%, tasks +33.7%, PR merge rate +16.2%; review time 5x; incidents per PR 3x; monthly incidents +57.9%; bugs per developer +54%; churn about 10x; **deployments per week -11.7%** | Observational; Faros sells this category; tests unnamed. [Older Faros pages](https://www.faros.ai/research) and third parties carry different sample and review figures (91%, 441.5%, 10K developers) |
| [McKinsey via CIO Dive, Sep 2026](https://www.ciodive.com/news/enterprises-bet-coding-agents-despite-ROI/830943/) | Consultancy | One quarter saw meaningful acceleration; in 30%, productivity **fell**; most exceed AI budgets; nearly half of developers distrust accuracy | Sample not reported in the article; primary not read; McKinsey sells transformation |
| [DORA ROI 2026.01](https://services.google.com/fh/files/misc/dora-roi-of-ai-assisted-software-development-2026.pdf) | Model, Google-backed | J-curve caused by learning curve, verification tax, downstream bottlenecks. Illustrative 500-person org: about $11.6M return on $8.4M, about 8-month payback ([InfoQ](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report)) | **Modelled, not measured.** Authors call it a high-uncertainty estimate |
| [Gartner, Jun 2025](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027) | Analyst forecast | Over 40% of agentic AI projects canceled by end of 2027 due to cost, unclear value, or inadequate risk controls | Forecast across all agentic AI, not SDLC-specific |
| [Thoughtworks Radar Vol. 35, Nov 2026](https://www.thoughtworks.com/radar) | Practitioner opinion | No blip named "software factory". Adopt: agent sandboxing. Trial: risk-tiered human oversight, OpenSpec. Assess: enterprise harness. Caution: human-in-the-loop theater, coding throughput as a productivity measure, blanket frontier model usage, agent instruction bloat | Thoughtworks sells consulting. Ring placement read via summarising fetch |

- Martin Fowler calls harness engineering "a valuable framing" — [InfoQ](https://www.infoq.com/news/2026/02/openai-harness-engineering-codex). Thoughtworks' Radar discussion warns about maintenance drift from copying harness assets between teams, and "be careful of mistaking activity for progress" — [podcast](https://www.thoughtworks.com/insights/podcasts/technology-podcasts/themes-technology-radar-vol-35).
- Published architectures from vendors exist but promote the publisher: [Factory articles](https://factory.ai/articles), [Unblocked buyer's guide](https://getunblocked.com/blog/what-is-an-ai-software-factory-buyers-guide/index.md). One academic general agent reference architecture exists, not factory-specific: [arXiv 2602.10479](https://arxiv.org/pdf/2602.10479.pdf).
- Dan Shapiro's five levels are a widely used scale, with "dark factory" at the top — [Simon Willison](https://simonwillison.net/2026/Jan/28/the-five-levels/). The light and dark factory contrast is in [Addy Osmani](https://addyosmani.com/blog/software-factories/).

### Inferences
- **Reference architecture common to Stripe and Spotify:** trigger surface (Slack, tickets), isolated pre-warmed sandboxes, a central tool gateway with curated subsets, scoped rules, deterministic steps around the agent, bounded CI retries, machine-checkable standards, and a telemetry dashboard. These are the components I would treat as required.
- Spotify's auto-merge applies to its maintenance fleet, not to all code, and Stripe keeps human review on everything. StrongDM's no-review rule is an outlier and should not be a target for a first rollout.
- Faros showing tasks up and deployments down is consistent with McKinsey's weak averages and with Spotify's review load, though these are different populations. The shared reading: generation gains are not delivery gains unless downstream stages are also changed.
- None of the headline cases supplies a defect-escape rate or a cost per merged PR, so they do not prove ROI.

### Gaps
- **BCG:** no BCG software factory report or reference architecture was found. The only hit was a [BCG Platinion job posting](https://www.ziprecruiter.de/jobs/554562197-ai-associate-all-genders-bcg-platinion-bei-boston-consulting-group) that mentions the term. Treat claims of a BCG framework as unverified.
- **McKinsey's** 20x productivity claim appears only in [secondary summaries](https://www.besthub.dev/articles/how-ai-can-boost-software-development-productivity-by-20-insights-from-mckinsey-3f97e9871384). It is not a verified finding.
- No independent audit of any case study was found.
- The full Gartner note is paywalled. Everything beyond its abstract is via Port.
- No 2026 annual DORA State of report was confirmed.

## Internal Developer Platform integration

### Takeaway
Backstage, Port, and Coder occupy different layers, and all three are complementary to agent vendors. Backstage and Port act as catalog and context for agents; Coder acts as the governed runtime. Spotify is the best evidence for the portal-as-agent-context pattern, but it is self-reported from Backstage's originator.

### Cited Findings
- **Backstage:** the MCP Actions Backend exposes actions to MCP clients such as Claude, Cursor, and Copilot, so they can query the catalog, run templates, and search. It uses Streamable HTTP and OAuth via CIMD. `AiResource` entities model skills and governance rules with ownership and lifecycle; MCP servers can be `API` entities with `type: mcp-server` — [Backstage docs, "Next" version](https://backstage.io/docs/next/ai/). **These are unreleased docs, so confirm against stable v1.56.0.**
- **Backstage caveat:** the Actions Registry does not enforce permissions automatically; each tool must check authorisation in its own handler — [Spotify Portal docs](https://backstage.spotify.com/docs/portal/core-features-and-plugins/mcp/registering-tools).
- **Spotify in production:** agents use Backstage via MCP and CLI to look up owners, read docs, and contact teams; Soundcheck gives corrective feedback — [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint).
- **Port:** six pillars including context lake and an AI registry for skills, prompts, and MCP server access, with RBAC, SSO, audit trails, and scorecards. The page doesn't mention budgets — [Port docs](https://docs.port.io/getting-started/agentic-sdlc-pillars/). Agent registry records owners, capabilities, and usage policies, and Port does not supply the agents — [Port docs](https://docs.port.io/solutions/agentic-work-management/overview).
- **Coder:** AI Bridge (LLM gateway, session audit, central MCP management, policy) and Agent Boundaries (domain restriction) are GA from v2.30, Feb 2026, and require the paid AI Governance add-on. Community and Premium include 1,000 Agent Workspace Builds for proof of concept. Community has no AI Bridge or Boundaries. Price by contact — [Coder docs](https://coder.com/docs/ai-coder/ai-governance).
- **Cross-tool packaging:** Radar Vol. 35 places agent plugin marketplace and Agent Package Manager at Trial, and Agent Plugins, an emerging standard for packaging MCP servers and skills across tools, at Assess — [Radar](https://www.thoughtworks.com/radar); [podcast](https://www.thoughtworks.com/insights/podcasts/technology-podcasts/themes-technology-radar-vol-35).

### Inferences
- A workable multi-department pattern: the catalog (Backstage or Port) is the system of record for who owns which agent, skill, and MCP server; a gateway (Coder AI Bridge, LiteLLM, or a cloud gateway) enforces and meters; sandboxes contain execution. Registries do not enforce anything on their own.
- Backstage is the lower-licence-cost, higher-engineering-cost option, and its permission model puts responsibility on each action author. That is a real risk with many departments contributing actions. Port trades licence cost for less building.
- Coder's AGPL core plus paid governance means the free tier does not provide the governance feature most enterprises would want it for.

### Gaps
- No independent comparison of Backstage vs. Port for agent use was found; [Port's comparison](https://www.port.io/blog/port-vs-factory) is vendor content.
- Multi-department skill marketplace case studies outside Spotify and Stripe were not found.
- Port's list prices were not found.

## Governance and the multi-department operating model

### Takeaway
Regulation sets general obligations but rarely addresses coding agents directly, so internal governance has to be risk-tiered by autonomy level. Vendor controls are uneven, especially for budgets and local agents, so a neutral gateway and sandbox layer is what makes controls consistent across departments.

### Cited Findings

**Autonomy and human gates**
- Gartner proposes four levels (observe, advise, act with approval, act autonomously) with controls scaled to each; predicts that by 2027, 40% of enterprises will demote or decommission autonomous agents due to governance gaps found only after production incidents — [Gartner](https://gartner.com/en/newsroom/press-releases/2026-05-26-gartner-says-applying-uniform-governance-across-ai-agents-will-lead-to-enterprise-ai-agent-failure).
- Radar Vol. 35: risk-tiered human oversight is Trial and human-in-the-loop theater is Caution. Discussion says sandboxing and least privilege are more reliable than human judgment, vendor safeguards are a supplement, and agents cannot be held accountable so humans must own consequences — [Radar](https://www.thoughtworks.com/radar); [podcast](https://www.thoughtworks.com/insights/podcasts/technology-podcasts/themes-technology-radar-vol-35).
- Practice ranges from all human review (Stripe), to auto-merge for fleet maintenance (Spotify), to no human review (StrongDM) — [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents); [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint); [StrongDM](https://factory.strongdm.ai/).

**Identity, RBAC, budgets, audit: what products actually provide**

| Control | Documented support |
|---|---|
| Per-agent identity | Microsoft Entra Agent ID ([Secondary](https://www.reply.com/valorem-reply/en/resources/insights/blog/azure/what-is-microsoft-agent-365-the-control-plane-for-enterprise-ai-agents-explained)); GitHub partner apps ([Docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)) |
| Departmental budgets | GitHub cost centres ([Docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)); Kiro groups and pools ([Docs](https://kiro.dev/pricing/)); Devin per-org ACU limits ([Docs](https://docs.devin.ai/admin/billing)); GitLab per-user caps ([Docs](https://docs.gitlab.com/subscriptions/gitlab_credits)) |
| Sub-organisation structure | Factory Enterprise ([Docs](https://factory.com/pricing)); Devin multi-org ([Docs](https://devin.ai/pricing)) |
| Audit | GitHub, Factory, Port, Augment + SIEM, Coder AI Bridge, Google request/response logging ([Google docs](https://antigravity.google/docs/enterprise)) |
| Network containment | Coder Agent Boundaries ([Docs](https://coder.com/docs/ai-coder/ai-governance)); Factory network policy; Google VPC-SC; Stripe devboxes with no internet ([Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents)) |

**Frameworks**
- **EU AI Act:** Digital Omnibus reported as Regulation (EU) 2026/1744, published in the Official Journal 24 Jul 2026, in force 27 Jul 2026 — [FASI](https://fasi.eu/en/articles/news/29051-digital-omnibus-package.html); high-risk dates moved to 2 Dec 2027 (Annex III) and 2 Aug 2028 (Annex I) — [Winston Taylor](https://www.winstontaylor.com/insights/ai-act-rules-on-high-risk-ai-delayed-as-ai-digital-omnibus-agreed). Check EUR-Lex; Article 50 transparency is reported as not delayed — [Secondary](https://pranoti.thesciencetalk.com/perspectives/eu-ai-act-omnibus-delay-relief-or-trap/).
- **NIST:** CAISI launched the AI Agent Standards Initiative on 17 Feb 2026, covering security, identity, and authorisation; no finalised standard as of the sources reviewed; NCCoE identity concept paper comment period closed 2 Apr 2026 — [CSA research note](https://labs.cloudsecurityalliance.org/research/csa-research-note-nist-ai-agent-standards-initiative-2026040/); [WorkOS](https://workos.com/blog/nist-ai-agent-standards-initiative-explained). A Q4 2026 interoperability profile is secondary and unconfirmed.
- **ISO/IEC 42001:** certifies an organisation's AI management system, not an agent's live behaviour — [Aguardic](https://dev.to/aguardic/iso-42001-in-the-wild-what-certification-actually-proves-4lnf). Augment certified via Coalfire — [Coalfire](https://coalfire.com/insights/news-and-events/press-releases/augment-ai-achieves-iso-iec-42001-certification-from-coalfire-certification-marking-a-milestone-in-responsible-ai-governance). "First" claims conflict between vendors such as Augment and [Cycode](https://cycode.com/blog/iso-iec-42001-certification/).
- **Taiwan AI Basic Act:** passed 23 Dec 2025, with NSTC as competent authority and seven principles. No immediate private-sector operational duties; sector regulators will issue rules from a MODA risk framework (publicly targeted Q1 2026); government must complete risk assessments within 6 months and review laws within 24 months. Baker McKenzie advises enterprises to inventory and classify AI, document and test, apply privacy by design, and allocate vendor responsibilities by contract — [Baker McKenzie](https://www.bakermckenzie.com/en/insight/publications/2026/01/taiwan-ai-basic-act). Promulgated and effective 14 Jan 2026 per secondary — [GPA](https://gpa.net/blogs/emea/taiwan-basic-law-on-ai-now-in-effect). No penalties or risk tiers yet per one analysis — [meta-intelligence](https://meta-intelligence.tech/en/insight-taiwan-ai-act).

### Inferences
- A four-tier policy mapped to Gartner's levels is the defensible starting point:
  - Observe or advise: open to all departments.
  - Act with approval: normal PR flow, with human review of specs and high-risk paths.
  - Act autonomously: limited to dependency upgrades, flaky test fixes, and lint or migration fleets with strong tests.
  - Payments, auth, regulated data, and production infrastructure stay gated.
- Coding agents are probably not themselves "high-risk" under the EU AI Act, but products built with them may be. Counsel should confirm this. The practical compliance burden is more likely to come from vendor management, data handling, and sector rules than from the Act itself.
- ISO 42001 on a vendor's datasheet should be treated as a procurement checkbox, not an assurance about agent behaviour. Ask for the certificate's scope.
- Least privilege is the non-negotiable lesson of the Kiro dispute: both parties' explanations imply agent permissions were too broad.

### Gaps
- A published multi-department chargeback or RACI model from a real enterprise was not found.
- MODA's risk classification framework status and sector rules from Taiwan's FSC or others were not found.
- NIST AI RMF mapping specific to coding agents was not found from a primary source.

## Taiwan and Asia availability

### Takeaway
In-Taiwan processing of frontier coding models is currently the hard constraint. AWS has a Taipei Region, but Claude there is documented as global cross-Region only, and Antigravity offers no in-country path for third-party models. Organisations requiring in-country inference should plan on self-hosted or open-weight models for sensitive repos, with frontier models reserved for lower-sensitivity work.

### Cited Findings
- AWS Asia Pacific (Taipei) `ap-east-2` launched June 2025 with three AZs — [AWS](https://aws.amazon.com/de/blogs/aws/now-open-aws-asia-pacific-taipei-region/).
- Bedrock's compatibility table shows Claude Haiku 5.5, Sonnet 5.5, and Opus 5.5 in `ap-east-2` through **Global cross-Region inference only**, not in-Region or Geo. Nova Pro shows Geo; Nova 2 Lite shows Global. **Only the first 100k of 706k characters were read, so other models are unknown** — [AWS Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/models-region-compatibility.html).
- Vertex AI residency page: no Gemini row confirms ML processing in Taiwan (`asia-east1`); the table reading shows Claude Haiku 4.5 as the only partner model with Taiwan processing — [Google](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency). **Conflict:** search snippets from other doc versions listed Opus 4.5, DeepSeek, Llama, and Qwen in a Taiwan column. Check the live table. Google also says model endpoints do not by themselves guarantee region availability — [Google](https://docs.cloud.google.com/vertex-ai/docs/general/locations).
- Antigravity enterprise: endpoints `global`, `us`, `eu`; **in-country regions not supported for third-party inference**; global endpoints must remain reachable even when using `us` or `eu` — [Google](https://antigravity.google/docs/enterprise).
- GitHub Enterprise Cloud residency: EU, Australia, US, Japan; **no Taiwan**, and some Copilot features are unavailable on GHE.com — [GitHub](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency).
- Kiro enterprise location follows the AWS Region of the Kiro profile, and model availability varies by region; paid individual billing is by listed country — [Kiro](https://kiro.dev/pricing/). Taiwan's status was not confirmed.
- **Local models:** TAIDE commercial models based on Llama: 7B (2024), 8B on Llama 3 (2024), 8B on Llama 3.1 (Feb 2025) — [CNA](https://www.cna.com.tw/news/ait/202404290314.aspx); [TechNews](https://technews.tw/2025/02/13/taide-8b-launched/). These are general language models, with no coding-agent evidence found. Kuwa GenAI OS runs TAIDE on-premises — [iThome](https://www.ithome.com.tw/news/162663). "TAIDE 2.0, 70B" is unofficial and unconfirmed — [blog](https://cloudinsight.cc/zh/blog/taiwan-llm).
- Radar Vol. 35 places open-weight models and small language models at Trial and local coding models at Assess, with LiteLLM at Trial for routing — [Radar](https://www.thoughtworks.com/radar).

### Inferences
- Global cross-Region inference means prompts and code may be processed outside Taiwan. A Taipei Region alone does not satisfy an in-country processing requirement. Confirm with AWS whether data at rest and logs remain in `ap-east-2`.
- Data-sensitivity tiers should drive routing: public or low-sensitivity code to frontier models; sensitive code to self-hosted open-weight models behind a gateway. GitLab Self-Managed or Coder plus a self-hosted model gateway are the documented building blocks.
- TAIDE should not be assumed to be a viable coding model without a benchmark. Its value is more likely in Traditional Chinese document and requirements work, which is my inference and has not been tested.

### Gaps
- Taiwan-specific rules for finance, government, or healthcare cloud and cross-border AI use were not researched and need local counsel.
- Factory, Devin, and Augment residency regions for Asia were not published on the pages read.
- TAIDE's current license terms and any 2026 release could not be confirmed from official sources.
- Coding benchmarks in Traditional Chinese contexts were not found.

## Adoption roadmap: pilot scope, KPIs, causes of failure

### Takeaway
Evidence supports starting with bounded, well-tested, repetitive work and investing in verification before scaling, while expecting and budgeting for an initial dip. Measure delivery and quality rather than generation volume. No public benchmark for token cost per merged PR was found, so that KPI must be baselined internally.

### Cited Findings

**Pilot scope**
- Stripe triggers minions from flaky-test tickets; Spotify's strongest numbers come from automated maintenance and migrations — [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents); [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint).
- Well-scoped tasks with clear acceptance criteria and automated feedback let agents work with less supervision. Prioritise agents that can run longer unattended over more parallel sessions — [Thoughtworks podcast](https://www.thoughtworks.com/insights/podcasts/technology-podcasts/themes-technology-radar-vol-35).
- External Stanford data cited by DORA's coverage: 35 to 40% gain on simple greenfield tasks vs. 10% or less on complex legacy code — [InfoQ](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report).
- Spotify reports worse agent performance in fragmented codebases and recommends standardising — [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint).

**Timing and budget**
- DORA expects a J-curve dip and warns that leaders may withdraw funding right before gains arrive. It recommends scenario ranges with finance and strongly discourages headcount reduction as the ROI strategy — [InfoQ on DORA](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report). A "15% over three months" assumption is secondary — [ecorpit](https://ecorpit.com/ecorpit-ai-coding-agent-rollout-governance-service-india-2026/).

**KPIs**
- DORA value chain: AI adoption, enabling capabilities, delivery metrics, then non-financial and financial outcomes — [InfoQ](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report).
- Faros's metric set shows what to pair: throughput with review time, incidents per PR, bugs per developer, churn, and deployments — [Faros](https://faros.ai/research/ai-acceleration-whiplash).
- Coding throughput as a productivity measure is at Caution — [Radar](https://www.thoughtworks.com/radar). Self-report is unreliable: METR's 2025 trial is commonly cited as showing a large perception gap, [per secondary](https://leaddev.com/velocity/ai-doesnt-make-devs-as-productive-as-they-think-study-finds).

**Observed causes of failure**

| Cause | Evidence |
|---|---|
| Verification and review overload | [Faros](https://faros.ai/research/ai-acceleration-whiplash); [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint); [DORA via InfoQ](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report) |
| Downstream stages unchanged (Gartner, via Port) | [Port](https://www.port.io/blog/gartner-names-port-ai-software-factory-platform) |
| Cost overruns; unclear value; weak risk controls | [McKinsey via CIO Dive](https://www.ciodive.com/news/enterprises-bet-coding-agents-despite-ROI/830943/); [Gartner](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027) |
| Uniform or binary governance | [Gartner](https://gartner.com/en/newsroom/press-releases/2026-05-26-gartner-says-applying-uniform-governance-across-ai-agents-will-lead-to-enterprise-ai-agent-failure) |
| Over-permissioned agents | [The Register](https://www.theregister.com/2026/02/20/amazon_denies_kiro_agentic_ai_behind_outage/) (disputed cause) |
| Rubber-stamp approvals and copied harness assets without understanding | [Thoughtworks podcast](https://www.thoughtworks.com/insights/podcasts/technology-podcasts/themes-technology-radar-vol-35) |
| Weak prerequisite foundations: tests, CI, consistent code | [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents); [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint) |
| Lights-off operation degrading the codebase (single secondary report) | [awesome list](https://github.com/varun1505/awesome-software-factories) |

### Inferences

**Suggested roadmap.** This is my synthesis; the phase lengths and gates are judgement, not measured evidence.
1. **Weeks 0 to 6, baseline and foundations:** capture current DORA metrics, cycle time, review time, and escape rate; turn on sandboxing, gateway logging, and budgets; choose 2 or 3 repos with good tests.
2. **Weeks 6 to 16, bounded pilot:** flaky tests, dependency upgrades, small backlog fixes; 1 or 2 teams; every PR human-reviewed; cap CI retries as Stripe does.
3. **Months 4 to 9, spec-to-PR:** add specs and acceptance criteria upstream, AI-assisted review, a second department, and a catalog for skills.
4. **Months 9 and later, tiered autonomy:** allow auto-merge only for classes of change with a proven low failure rate, with explicit demotion rules.

**Suggested KPI set**

| KPI | Purpose |
|---|---|
| Lead time, deployment frequency, change failure rate, recovery time | Outcome: delivery |
| Defect escape rate; incidents per merged PR | Outcome: quality |
| PR review time and queue age; no-review merge rate | Bottleneck detection |
| Agent PR merge rate without human edits; CI rounds per PR; 30-day churn | Agent quality |
| Fully loaded token and compute cost per merged PR, including failed runs | Cost |
| Budget adherence by cost centre; policy violations | Governance |

Set the gate so that scaling requires change failure rate and escape rate to hold steady while lead time improves. Do not use PR count alone.

**Buy, build, or assemble**

| Option | When it fits | Main risk |
|---|---|---|
| **Buy** (Factory, Devin) | Need speed, single vendor acceptable, data can sit in vendor or VPC | Agent and model lock-in; Taiwan processing terms unclear |
| **Extend the existing SCM platform** (GitHub, GitLab, Harness) | Already standardised; want least integration | Gaps at requirements and across departments; usage bills |
| **Assemble** (Backstage or Port + Coder + gateway + agent runtime + spec method) | Multiple departments, multi-model, data sensitivity tiers, strong platform team | Integration ownership; needs a team of its own |
| **Full build** (Stripe style) | Very large scale and uncommon stack | Justified only by years of platform investment and scale |

**Default recommendation from this evidence:** assemble around the existing SCM, buying the agent(s) and keeping specs, skills, MCP tools, evaluation scenarios, and gateway logs under own control. Those assets are what stay portable when products change. A full build or a dark factory is not supported by available evidence for a typical enterprise.

### Gaps
- **Token cost per merged PR:** no public benchmark found. The only published marker is StrongDM's $1,000/engineer/day aspiration, a guideline rather than a result, which at about 21 working days would be about $21,000 per engineer per month (my arithmetic).
- **Defect escape rate:** no case study reports it.
- A controlled study of a full requirements-to-testing factory rollout was not found. Available evidence covers coding and adjacent stages; **requirements-stage automation has essentially no outcome evidence in the sources reviewed.**
- Faros's Q3 2026 "The Speed Trap" report exists, linked from its [research page](https://faros.ai/research/ai-acceleration-whiplash), but was not read.
