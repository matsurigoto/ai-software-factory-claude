# Commercial Coding Agents and Model Routing for a Hybrid AI Software Factory on Azure DevOps Services (as of 7 Nov 2026)

Evidence labels:
- **[Primary]** official docs, pricing page or repo. **[API]** GitHub REST API read on 2026-11-07. **[Vendor]** a claim by a party with a stake. **[3P]** independent or third party. **[2nd]** secondary blog or aggregator, directional only.
- **[ESTIMATE]** my calculation, with stated inputs. **[Prior]** reused from `onprem_models_compute.md`, `coding_agents_skills.md` or the earlier report, with its original source.

**Disclosure:** this research was produced by an Anthropic model, and Anthropic products (Claude Code, Claude models, Claude in Foundry) are among those compared. Anthropic's docs were easy to read, so coverage depth is not a ranking. The base demand model is derived from an Anthropic cost page; weigh it accordingly.

**Limits:** most pages were read through a summarizing fetch tool, not verbatim. No product was run. Nothing here is a compliance or legal conclusion. Prices are list prices seen on 7 Nov 2026; enterprise discounts were not available.

**Stale or moving items to re-check before use:** Azure Taiwan North status; Foundry tables (updated Sep to Oct 2026); Gemini prices (scheduled doubling on 1 Dec 2026 and 1 Jan 2027); Azure OpenAI list prices (did not render); FrontierCode snapshot (6 Nov).

## Which coding agents and review tools work with Azure Repos on Services, and which can use a self-hosted endpoint?

### Takeaway
Moving to Services opens real native options for review and cloud agents (CodeRabbit, Qodo, Cursor, Devin, OpenHands Enterprise, Copilot code review), but none of the native cloud-agent paths runs on a self-hosted model. The tools that combine headless use on your own pipeline agent with a self-hosted endpoint remain CLI harnesses over plain git and REST: OpenCode, Goose, Codex CLI, Copilot CLI and Factory Droid. Claude Code also runs that way, but is supported on Claude models only.

### Cited Findings

**Coding agents**

| Tool | Azure Repos integration | Headless on self-hosted agent | Self-hosted endpoint | Controls and caveats |
|---|---|---|---|---|
| **Claude Code** | No Azure DevOps-specific Anthropic doc found (one search found nothing, and the docs index fetch returned 404, so this is weak evidence of absence). Plain git and CLI | [`claude -p`](https://code.claude.com/docs/en/plugins/org.md) [Prior]; Foundry config by [environment variables only, with API key, Entra credential chain or bearer token](https://code.claude.com/docs/en/microsoft-foundry) | [Anthropic Messages, Bedrock and Vertex formats through a gateway](https://code.claude.com/docs/en/llm-gateway-protocol). [Anthropic does not support routing it to non-Claude models](https://code.claude.com/docs/en/llm-gateway), though model vendors do benchmark through it ([GLM card](https://huggingface.co/zai-org/GLM-5.3-Flash/raw/main/README.md)) [Prior] | Managed settings; [`allowedProviders` can pin one gateway URL (v2.1.285+)](https://code.claude.com/docs/en/llm-gateway); [self-hosted Claude apps gateway has SSO, per-group model access, spend limits, OTLP](https://code.claude.com/docs/en/claude-apps-gateway). Proprietary. [Hosted Code Review is GitHub only](https://code.claude.com/docs/en/code-review) [Prior] |
| **Codex CLI** | None native found. [Microsoft's Codex guide uses GitHub Actions samples](https://learn.microsoft.com/en-my/Azure/foundry/openai/how-to/codex); an [Azure Pipelines job appears only in a third-party post](https://codex.danielvaughan.com/2026/04/27/codex-cli-azure-openai-foundry-enterprise-deployment-compliance/) [2nd] | `codex exec` [Prior] | [Custom `model_providers` with base URL and `wire_api`; `--oss` for Ollama and LM Studio](https://learn.chatgpt.com/docs/config-file/config-advanced). Chat wire [reported removed](https://github.com/janhq/jan/issues/7413) [2nd], so the server must speak Responses. [vLLM documents this path](https://docs.vllm.ai/en/stable/serving/integrations/codex/), and a [community proxy exists to patch format gaps](https://github.com/florath/vllm-codex-proxy) | Apache-2.0, rust-v0.167.0 [API]. [Issue #31875: Codex against an Azure GPT-5.6 deployment fails](https://github.com/openai/codex/issues/31875), open since July [API]. [Entra reportedly unsupported](https://codex.danielvaughan.com/2026/04/27/codex-cli-azure-openai-foundry-enterprise-deployment-compliance/) [2nd]. Test before committing |
| **Copilot CLI** | None documented for Azure Repos. [ado-aw uses it as engine](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md) [Prior] | [Non-interactive use reported, with `--allow-all-tools`](https://www.developersdigest.tech/reference/copilot-cli/env-vars) [2nd] | [BYOK via `providers.json`: Azure, Anthropic, OpenAI types, Ollama; `COPILOT_OFFLINE=true`](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models). [GitHub login not required with BYOK; `/delegate`, GitHub MCP and code search are lost](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli) | v1.0.107; license reported NOASSERTION [API]. Credentials in that file are plain text, per the BYOK doc |
| **Devin** | [Native, Git only: PRs, branches, comments. No work items, pipelines, test plans, wiki](https://docs.devin.ai/integrations/azure-devops). OAuth with tenant admin consent | Not on your agent: [the brain always runs in Cognition's cloud](https://docs.devin.ai/enterprise/deployment/overview) | **No.** [Third-party LLM keys are not supported](https://docs.devin.ai/enterprise/deployment/overview) | [Customer Dedicated is a single-tenant VPC hosted by Cognition; Assured adds CMK](https://docs.devin.ai/enterprise/deployment/overview). Retention and training terms were not on that page |
| **Factory Droid** | Integration page returned 404; [no native ADO doc verified](https://docs.factory.ai/integrations/azure-devops). Assume git and CLI | [`droid exec` with autonomy tiers](https://docs.factory.com/cli/droid-exec/overview) [Prior] | [`customModels` with `anthropic`, `openai`, `generic-chat-completion-api`, `bedrock-converse`; vLLM listed](https://docs.factory.com/cli/byok/overview). It warns that models under 30B do poorly and only official Anthropic and OpenAI APIs are fully tested | [Air-gap build](https://docs.factory.com/enterprise/airgapped-deployment.md) [Prior] |
| **Cursor** | [Native for Services: Cloud Agents in public beta; Bugbot in limited availability; Server unsupported](https://cursor.com/docs/integrations/azure-devops). Automations, Bugbot autofix and Security Agents are not yet available on ADO | [CLI print mode](https://cursor.com/docs/cli/headless) [Prior]; no Azure Pipelines doc found | Not verified; [planning stays in Cursor's cloud even with self-hosted cloud agents](https://cursor.com/blog/self-hosted-cloud-agents) [Prior] | [SSO, SCIM, audit on Enterprise](https://cursor.com/pricing) [Prior]. [SpaceX-owned](https://x.com/cursor_ai/status/2088249881718919393?lang=en) [Prior] |
| **OpenCode** | None native; git and REST or MCP | [`opencode run`](https://opencode.ai/docs/cli) [Prior] | [OpenAI-compatible and Anthropic-compatible packages with `baseURL`; built-in Azure, Bedrock, Vertex](https://opencode.ai/docs/providers) | MIT, v1.19.5 [API]. [Third-party audit: 17 outbound paths, no master switch](https://particula.tech/blog/opencode-offline-mode-air-gapped) [Prior] |
| **Goose** | None native; git and MCP | [`goose run`](https://goose-docs.ai/docs/getting-started/providers) | [Azure OpenAI with Entra token, key or credential chain; custom OpenAI, Anthropic or Ollama-compatible providers; vLLM via `OPENAI_HOST`](https://goose-docs.ai/docs/getting-started/providers) | Apache-2.0, v1.57.0, Linux Foundation AAIF [API, Prior]. Thin enterprise policy layer |
| **OpenHands** | [Native for Services on Enterprise: `@openhands` in PRs and work items; service hooks to your URL; Server not covered](https://docs.openhands.dev/enterprise/integrations/azure-devops) | Agent Server and SDK [Prior] | [Any LiteLLM-supported model, base URL field; Ollama, vLLM, SGLang named](https://docs.openhands.dev/openhands/usage/llms/llms) | MIT core, v1.37.0 [API]. Needs inbound reachability from Azure DevOps and Entra app registration |
| **Tabnine** | [ADO account connection for repo context, ADO TFVC indexer since v6.5.0](https://docs.tabnine.com/main/administering-tabnine/release-notes); [Azure DevOps MCP listed](https://docs.tabnine.com/main/getting-started/tabnine-agent/mcp-intro-and-setup) | [Headless reported as a paid add-on naming GitHub, GitLab, Bitbucket, not ADO](https://pricingsaas.com/companies/tabnine/diffs/2026W20) [2nd] | [Private endpoints including Azure, framed around Chat](https://docs.tabnine.com/main/administering-tabnine/managing-your-team/settings/models-settings). Agent coverage unconfirmed | [SaaS, VPC, on-prem, air-gapped](https://docs.tabnine.com/main/welcome/readme/architecture/deployment-options.md). [Site now brands it Tricentis AI Workspace](https://tabnine.com/platform), consistent with a single-source acquisition report |

**Review tools**

| Tool | Azure Repos | Deployment | Own model | Caveat |
|---|---|---|---|---|
| **CodeRabbit** | [Native for Services: Entra service principal or PAT, auto-installed webhooks, work item events](https://docs.coderabbit.ai/platforms/azure-devops) | SaaS; [self-host Enterprise, 500-seat floor per earlier report](https://docs.coderabbit.ai/self-hosted/azure-devops) [Prior] | Not stated on the integration page | Its commit status cannot satisfy an ADO status branch policy. Service principals need a CLI-set permission per project, or PRs are silently never reviewed. Retention page unavailable (404) |
| **Qodo** | [Announced Feb 2026](https://stocks.observer-reporter.com/observerreporter/article/gnwcq-2026-2-11-qodo-expands-integration-with-azure-devops-to-scale-ai-code-review-for-the-enterprise) | **Conflict:** [matrix shows Services on multi-tenant and single-tenant, not on-prem](https://docs.qodo.ai/install-and-configure/deployment-model-support); [setup doc requires single-tenant](https://docs.qodo.ai/qodo-documentation/code-review/qodo-merge/getting-started/setup-and-installation/azure-devops); [a separate on-prem guide also exists](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md) | Not found | Get deployment and retention in writing |
| **Greptile** | **Not listed**: [SCM table has GitHub, GitLab, Perforce](https://www.greptile.com/docs/deployment-options) | Self-hosted Docker or Kubernetes | Self-hosted supports OpenAI, Anthropic, Bedrock, Azure OpenAI, Vertex | Excluded unless Greptile confirms ADO in writing |
| **Copilot code review** | [Open preview for all ADO customers; Comment-only; no self-hosted agents; 10 GB and 100-file limits; budgets notify only; cost lags 48 hours](https://infoq.com/news/2026/09/copilot-code-review-azure-repos) [2nd] | Microsoft-hosted or Managed DevOps Pools (Ubuntu) | No | No SLA, and five concurrent reviews per org |
| **PR-Agent** | [Supported, docs show `dev.azure.com`](https://docs.pr-agent.ai/installation/azure/) | Your pipeline | Yes, BYO | MIT, v0.49.0 [API]; not the Qodo free tier [Prior] |

**Microsoft MCP on Services:** [azure-devops-mcp](https://github.com/microsoft/azure-devops-mcp) is MIT, v2.11.0 [API]. The [remote MCP server runs only in Europe and the US and is described as a non-storing pass-through](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location), so an Asia Pacific org's MCP traffic crosses regions. The local server avoids this.

### Inferences
- **Default harness shortlist under each decision:**
  - Cloud inference allowed, Claude acceptable: Claude Code or Codex CLI plus OpenCode as portability check.
  - Mixed routing through one gateway: OpenCode, Goose or Factory (one harness can reach both tiers). Claude Code covers only the Claude tier, supported, so a second harness is needed for local models.
  - No cloud inference: OpenCode and Goose, as in the earlier report.
- **Review tool order to bake off on Services:** PR-Agent (own model, any route), CodeRabbit, Qodo single-tenant. Copilot code review is cheap to trial on low-sensitivity repos, but cannot use self-hosted agents, and I would not make it a gate.
- **Native cloud agents (Cursor, Devin) add a data processor** beyond Microsoft and the model vendor, and they hold repo write credentials. They fit S0 and S1 repos only, and they fall outside on-prem logging.
- Claude Code's single-vendor support is a real architecture constraint for hybrid routing, not just a preference question.

### Gaps
- No vendor-published end-to-end Azure Pipelines recipe was found for Claude Code, Codex, OpenCode, Goose, Copilot CLI or Factory. Feasibility is inferred from CLI behaviour; the agent's own REST or MCP steps must be built.
- Factory's ADO status, Cursor custom endpoints, CodeRabbit and Qodo retention and BYO-model terms, and Tabnine Agent's ADO depth need written vendor confirmation.

## Inference routes and data terms, and whether Azure-hosted inference reduces incremental exposure

### Takeaway
No frontier route verified here offers documented in-Taiwan inference for a current flagship model. The closest documented options are Azure OpenAI's APAC Data Zone (no Taiwan region listed), Bedrock's Japan geo, and Google's Singapore processing. Azure hosting does reduce incremental exposure for Azure OpenAI, but only partly, and the benefit is smaller for Claude in Foundry because Anthropic stays the seller, operator and processor.

### Cited Findings

**Route matrix** (not a compliance finding; each cell needs contract confirmation)

| Route | Training on your data | Retention | Residency and Taiwan or nearby | Private networking | Notable limits |
|---|---|---|---|---|---|
| **Azure OpenAI in Foundry** (sold by Azure) | [Not used; not available to OpenAI; models do not call OpenAI services](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/openai/data-privacy) | Stateless inference, but a flagged-content store exists for human review unless [approved for modified abuse monitoring](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/abuse-monitoring) (Limited Access; stricter for some advanced models). Verifiable through the `ContentLogging` capability. Duration: [third-party and forum answers say about 30 days](https://learn.microsoft.com/en-us/answers/questions/2286499/azure-openai-abuse-monitoring); the Learn pages I read did not state days. Responses API and stored completions persist data | [Global: any Azure region. Data Zone: US, EU or APAC. Standard: chosen geography. Microsoft may add regions to a zone without notice](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types). [APAC columns list Australia East, Japan East, Korea Central, Southeast Asia, South India; **no Taiwan region**. gpt-6.1-sol and terra, gpt-5.6-sol are in APAC Data Zone; regional gpt-5.6-sol only in Australia East](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure-region-availability) | [Foundry resources can join a VNet](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry); verify private endpoints per SKU and deployment type | Global Batch can process anywhere. Preview features may differ |
| **Claude in Foundry, Hosted on Azure** | [Commercial terms: not trained unless opted in](https://code.claude.com/docs/en/data-usage); [Anthropic terms govern](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/claude-models/data-privacy) | [Governed by Anthropic's DPA and commercial terms](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison) | [Global Standard or US Data Zone only](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry). [Data Zone APAC not available; Global resource regions are US and Sweden Central, no Asia](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-from-partners) | [VNet optional on the Foundry resource](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry) | [CSP subscriptions unsupported](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-from-partners). [No Batch, Files, Skills, code execution; Fable not hosted on Azure](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry) |
| **Claude in Foundry, Hosted on Anthropic** | Same | Same | [May process outside Azure and outside the selected geography](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison) | Foundry side only | Required for Fable |
| **Other Foundry partner models** | Per provider | Per provider | [FW-GLM-5.3 is hosted by Fireworks; data leaves Microsoft and is outside Foundry residency commitments](https://ai.azure.com/catalog/models/FW-GLM-5.3) | Not verified | "In Foundry" does not mean "in Azure" |
| **Amazon Bedrock** | [Providers have no access to prompts, completions or logs](https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html) | [Per-Region `data_retention_mode`: none, default, aws_review; enforceable by SCP. Fable requires aws_review, up to 30 days, AWS reviewers only](https://docs.aws.amazon.com/bedrock/latest/userguide/data-retention.html). Opus 4.8 example permits none | [Opus 5.5: Taipei (ap-east-2) Global only; Tokyo and Osaka have JP geo; Singapore Global only; in-Region only in London](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-opus-5-5.html) | [VPC and PrivateLink documented](https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html) | [Regional premium of 10%](https://platform.claude.com/docs/en/about-claude/pricing). Taipei to JP geo needs a Tokyo or Osaka source Region (my reading). Retained data sits in destination Regions |
| **Google Vertex / Agent Platform** | [Prompts not shared with third parties; processed as instructed](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-partner-models). Explicit no-training statement was not on that page | [Zero retention needs caching disabled; docs conflict on caching and ZDR](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/data-governance?authuser=6). [Partner-model prompt logging off by default](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/security-controls?authuser=0) | [Taiwan (asia-east1) ML processing listed only for Claude Haiku 4.5. Singapore covers Opus 5.5, Sonnet 5.5, Sonnet 5, Opus 5, Fable. Gemini has no Taiwan commitment](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency). [An endpoint listing does not imply in-region processing](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/locations) | Not verified in this pass | Most complete near-Taiwan Claude option on paper, at a 10% premium |
| **Anthropic API direct** | [Not trained under commercial terms unless opted in](https://code.claude.com/docs/en/data-usage) | [Standard 30 days; ZDR by per-org approval, flagged sessions up to 2 years; Fable excluded by default; ZDR applies to first-party only](https://code.claude.com/docs/en/zero-data-retention) | [Inference only `global` or `us`; workspace geo only `us`](https://platform.claude.com/docs/en/manage-claude/data-residency). No Asia | Not verified | Worst residency fit for Taiwan among these |
| **OpenAI API direct** | [Not trained by default](https://developers.openai.com/api/docs/guides/your-data) | [30-day abuse logs; ZDR by approval; ineligible endpoints include files, batches; prompt cache tensors up to 24 hours in GPU-local storage; CSAM and safety retention exceptions](https://developers.openai.com/api/docs/guides/your-data) | [Japan, India, Singapore, Korea are storage only; regional processing only US, EU, UAE; Taiwan not mentioned; 10% uplift](https://developers.openai.com/api/docs/guides/your-data) | Not verified | Storage residency is not inference residency |
| **Private VPC, open weights** | None by model vendor if self-served | Yours | Depends on GPU location; [ND H200 v5 reportedly East US 2 and others; Asian availability unconfirmed](https://www.spheron.network/blog/azure-h200-pricing-2026/) [2nd] | Yours | Code and prompts on your tenant, in the provider's cloud |
| **On-prem GPU** | None | Yours | Your site | Yours | Quality, capex, staff; see cost and quality sections |

**Azure Taiwan North:** [operational but in staged rollout, not GA, selective onboarding, no timetable, and may lack capabilities of Singapore, Malaysia or Japan regions (Sept 2026)](https://focustaiwan.tw/business/202609010030). It does not appear in the Foundry tables I read. Treat Taiwan-local Foundry inference as unavailable until your Microsoft team confirms it.

**Conflicts to resolve in writing**
- **Egress wording:** [Microsoft says prompts and outputs never leave Azure for Hosted on Azure](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/claude-models/data-privacy). [Anthropic says usage metadata and safety-flagged content do egress to Anthropic](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry). Assume the Anthropic wording is the operative edge case until contract text says otherwise.
- **Sonnet 5.5 hosting:** [Anthropic says Global only](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry); [Microsoft's overview lists it for US Data Zone](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models); [the region table shows version 1 under Global and version 2 under Data Zone](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-from-partners).
- **Vertex caching and ZDR**, as above.

**Where Azure DevOps data lives:** [an org is pinned to a chosen location (Asia Pacific is an option, Taiwan is not named), with customer data kept there apart from support, troubleshooting and legal needs; tokens and non-US/EU/UK profile data sit in the US; previews may store globally](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location).

**Claude Code telemetry by route:** [on Bedrock, Vertex and Foundry, metrics, error reports and `/feedback` upload default off, while surveys and the WebFetch hostname check still run and have their own opt-outs](https://code.claude.com/docs/en/data-usage). Local transcripts are plaintext under `~/.claude/projects/` for 30 days by default, per the same page, which matters on shared build agents.

### Inferences

**Does Azure-hosted inference reduce incremental exposure? Partly.**

| Dimension | ADO Services today | Azure OpenAI (Data Zone APAC) | Claude Hosted on Azure | Change |
|---|---|---|---|---|
| Parties with processing role | Microsoft | Microsoft | Microsoft plus Anthropic as independent processor | Azure OpenAI adds no vendor; Claude adds one |
| Governing terms | Microsoft DPA | Microsoft DPA | Anthropic DPA and terms; Non-Microsoft Product | Same versus new |
| Location | Chosen geography | Multi-country APAC zone; may be expanded | Global or US | Wider in all cases |
| Data content | Source, work items, test results | Plus prompts, tool output, logs, env values, test data, stack traces | Same | **New content class** |
| Human access path | Support and troubleshooting | Plus abuse-review store unless modified | Per Anthropic terms | New path |
| Network and identity | Entra, existing tenant | Same tenant, Entra, RBAC | Same | Real reduction |

- Real reductions: no new tenant, billing or identity estate, an already-reviewed vendor for Azure OpenAI, and MACC applicability [for Claude CCU too](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison).
- What it does not remove: "code may be in Microsoft's cloud" does not automatically cover agent context. Agents send what they read: secrets in env files, production logs, customer fixtures and internal design text. Permission for repos at rest in a chosen geography is a different question from permission for processing across a multi-country zone.
- **Therefore classification should turn on what prompts can contain, not only on the code.** A repo whose code is fine in Azure may still be S3 if its tests load production-like data.

**Must verify (do not assume):**
1. Whether Azure's APAC Data Zone meets your geography requirement, given no Taiwan region and Microsoft's right to add regions.
2. Whether your purchase channel is CSP; if so, Claude in Foundry is blocked as documented.
3. Eligibility for modified abuse monitoring on the exact models, and retention duration.
4. Anthropic's flagged-content handling for Hosted on Azure, and Marketplace or Anthropic availability for a Taiwan billing account.
5. Legal review on cross-border processing, sector rules, customer contracts, and whether agent context contains personal data.
6. Any Taiwan public-sector-linked obligations on PRC-origin models ([Prior](https://www.taiwannews.com.tw/en/news/6248783)); note GLM on Bedrock is hosted by AWS but is still a PRC-origin model.

**Best-fit shortlist (my judgment, by decision state):**
- Cloud approved with geography preference: Azure OpenAI Data Zone APAC first for least incremental vendors. Vertex Singapore or Bedrock JP geo if Claude's benchmark lead justifies a second cloud contract.
- Cloud approved, Azure consolidation valued over geography: Claude Hosted on Azure, Global. Be explicit that processing is not in Asia by commitment.
- Cloud not approved: on-prem only. A private VPC does not satisfy this.

### Gaps
- Private-link details per Foundry partner model, Vertex private connectivity, first-party Anthropic and OpenAI private connectivity were not verified.
- No Microsoft list of APAC Data Zone member countries beyond the table columns was read; the docs say it can change.
- I did not read the Anthropic DPA or Microsoft Product Terms text.

## Sensitivity-based routing: classification, gateway policy, fail-closed, on-prem logging

### Takeaway
Treat routing as a four-layer control: repo classification, identity, network egress and gateway policy, with the network layer as the real fail-closed guarantee. Gateway failover features are the main hazard, because the documented default behaviour of the products here is to move to another backend. No public enterprise case of per-repo code-sensitivity routing was found; GitLab's per-feature hybrid is the nearest shipped analogue.

### Cited Findings

**Gateway comparison for this job**

| Gateway | Formats and routing | Failure behaviour | Logging on-prem | Risks |
|---|---|---|---|---|
| [agentgateway](https://github.com/agentgateway/agentgateway) (Apache-2.0, v1.7.0, [API]) | [Content-based routing, CEL RBAC, prompt guards](https://agentgateway.dev/docs/kubernetes/latest/llm/) | [Priority groups: when all models in a group are evicted, traffic moves to the next group; end state with none left is undocumented](https://agentgateway.dev/docs/kubernetes/latest/llm/failover/) | Metrics and access logs; prompt logging unspecified on that page | [MCP authorization advisories](https://github.com/agentgateway/agentgateway/security/advisories) [Prior] |
| [Agent Router](https://github.com/theagentrouter/agent-router) (Apache-2.0, v1.2.1, [API]) | Envoy-based; [Global Rate Limit, CEL cost](https://theagentrouter.ai/docs/capabilities/traffic/usage-based-ratelimiting/) [Prior] | [First backend is primary, later ones are fallbacks; final client response when all fail is undocumented there](https://theagentrouter.ai/docs/capabilities/traffic/provider-fallback/) | Via Envoy and OTel (not re-verified) | [Six advisories 6 Nov 2026](https://github.com/theagentrouter/agent-router/security/advisories) [Prior] |
| [LiteLLM](https://github.com/BerriAI/litellm) (v1.108.2, [API]) | Richest budgets [Prior]; [team-specific models are Enterprise-licensed](https://docs.litellm.ai/docs/proxy/team_model_add); [secret detection is Enterprise-only](https://docs.litellm.ai/docs/proxy/guardrails/secret_detection) | [Global budget fails open without Postgres](https://docs.litellm.ai/docs/proxy/users) [Prior] | Self-hosted DB | [March 2026 supply-chain compromise and critical auth CVEs](https://docs.litellm.ai/blog/security-update-march-2026) [Prior]. Weak fit for a security boundary |
| [Kong](https://github.com/Kong/kong) (Apache-2.0, 3.9.4, [API]) | AI plugins | Not reviewed | Self-hosted | [Token limits Enterprise-only](https://developer.konghq.com/plugins/ai-rate-limiting-advanced/) [Prior] |
| **Azure API Management** | [OpenAI, Vertex and Anthropic Messages schemas; Anthropic currently v2 tiers; token limit by arbitrary key; circuit breaker, priority load balancing; unified model API in preview](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities) | [**Policies a self-hosted gateway does not support are skipped during execution**](https://learn.microsoft.com/en-us/azure/api-management/api-management-gateways-overview), a fail-open risk. Control plane loss is [fail static](https://learn.microsoft.com/en-us/azure/api-management/self-hosted-gateway-overview) | [Self-hosted gateway has local logs and OTel; no resource logs to Azure Monitor](https://learn.microsoft.com/en-us/azure/api-management/api-management-gateways-overview). Managed gateway sends all traffic through Azure; [diagnostic bodies capped at 8,192 bytes](https://learn.microsoft.com/en-us/azure/api-management/api-management-gateways-overview) | [Self-hosted needs Developer or Premium and outbound 443 to Azure, with heartbeat and config pulls](https://learn.microsoft.com/en-us/azure/api-management/self-hosted-gateway-overview) |
| **Claude apps gateway** | [Self-hosted; Messages to Bedrock, Claude Platform on AWS, Google, Foundry, Anthropic, with failover; per-group models; spend limits; OTLP](https://code.claude.com/docs/en/claude-apps-gateway) | Upstream failover exists | Your telemetry collector | Claude only, so it can be the frontier branch, not the router |

**Real-world and design references**
- [GitLab Duo hybrid: each feature uses one model, self-hosted or GitLab-managed; if that model is unavailable, the feature stops until another is chosen](https://docs.gitlab.com/administration/gitlab_duo_self_hosted). This is per-feature, not per-repo, and maturity labels differ between doc copies. It is a shipped fail-closed example.
- Architecture posts: [classify at the gateway and deny when unsure](https://skaira-labs.pages.dev/insights/ai-route-segregation-without-data-leakage) [2nd]; [no cloud path when local is down, enforced outside client hints](https://engineered.at/articles/hybrid-cloud-local-llm-the-complete-architecture-guide-2026) [2nd]; [a per-route fail-closed flag](https://www.tdcommons.org/dpubs_series/10592) (disclosure, not a tested system). [Gravitee's dual-homed overflow pattern](https://gravitee.io/corpus/gen-1161/hybrid-security/hybrid-security-patterns-for-llm-gateway-deployment-across-on-prem-and-cloud-environments.html) [Vendor] conflicts with fail-closed unless limited to non-sensitive classes.

**Azure DevOps hooks available**
- [Project properties endpoint exists per third-party module code](https://www.powershellgallery.com/packages/PSDevOps/0.4.1/Content/Set-ADOProject.ps1) (Microsoft reference not read). I found no native per-repo custom metadata feature.
- [Workload identity federation service connections are the recommended pattern; the ADO issuer retires 1 July 2027 in favour of Entra](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure). Per-pipeline authorization is available and all-pipelines access is discouraged, per that page.
- Client pins: [Claude Code `allowedProviders`](https://code.claude.com/docs/en/llm-gateway), [OpenCode `enabled_providers`](https://particula.tech/blog/opencode-offline-mode-air-gapped) [Prior], [`COPILOT_OFFLINE`](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models).

### Inferences

**Proposed design (mine; untested)**

| Class | Criterion (what agent context can contain) | Allowed routes | On failure |
|---|---|---|---|
| S0 Public | Open-source or public content | Any approved | Any |
| S1 Internal | Ordinary proprietary code, no regulated data | On-prem or approved private frontier endpoint | Other approved route |
| S2 Confidential | Sensitive IP, customer-specific logic, no raw regulated data | On-prem; named cloud only by decision (Data Zone or VPC) | Queue or deny |
| S3 Restricted | Regulated data, secrets, or contract-bound material reachable from context | On-prem only | **Deny; never failover** |

1. **Registry, not repo file.** Keep repo-to-class mapping in a protected central repo with CODEOWNERS approval. A file inside the repo can be edited by an agent or a PR. Default for unlisted repos is S3.
2. **Isolation by ADO structure.** Separate agent pools per class, with pool and service-connection authorization limited to projects of that class. Pools are easier to audit than per-request checks.
3. **Credentials carry class.** The gateway issues class-scoped tokens from the pipeline step. Route tables are per class. The S3 route table has only on-prem backends, so failover has nowhere else to go. Do not model tiers as priority groups on one route.
4. **Network is the guarantee.** S3 pool egress allows only ADO and the on-prem gateway. A misconfigured gateway or client then fails with a connection error, not a leak. Gateways and clients are defence in depth.
5. **High-water mark.** A session's class is the maximum of all repos, wiki pages, RAG sources and MCP results read. Retrieval must filter by the caller's class before context assembly. Cross-repo tasks need this most.
6. **Interactive developers.** Laptops are the weakest point. Route through the same gateway with managed client settings, and block direct provider domains at the proxy. BYO accounts defeat all of this.
7. **DLP is secondary.** Run secret scanning on prompts as detection, not as the boundary, given its false negatives.
8. **Logging on-prem.** Gateway writes request metadata (principal, repo, class, route, model, tokens, decision) to on-prem storage, with prompt bodies in restricted storage such as Langfuse OSS on a short retention. Disable cloud diagnostic prompt logging. **Do not let agents print context into pipeline logs: ADO logs live in Microsoft's cloud**, which undoes on-prem inference for S3.
9. **Continuous proof.** Monthly game day: stop on-prem backends and assert S3 returns an error, with canary tokens watched at the egress proxy. Alert on any S3-class principal reaching a cloud route.

**Gateway choice:** agentgateway or Agent Router on-prem as the policy point, as the earlier report recommended. APIM self-hosted is attractive for Azure-centric teams, but the skipped-policy behaviour, outbound dependency, Premium tier, and an apparent mismatch (Anthropic schema on v2 tiers, self-hosted on Developer or Premium) all need testing. A self-hosted gateway calling Foundry with on-prem logs is otherwise a sensible frontier branch.

### Gaps
- No public, verifiable enterprise case study of per-repo LLM routing for code. Most sources are vendor or SEO content.
- Azure Pipelines OIDC token claims usable by a non-Azure gateway were not confirmed. If unavailable, use short-lived tokens issued by a broker step, or per-pool credentials.
- End-state failure responses for agentgateway and Agent Router must be tested.
- APIM Premium cost and whether LLM policies support Anthropic Messages on self-hosted were not established.

## Cost comparison for 20, 100 and 400 developers, and the GPU purchase outcome

### Takeaway
On list prices at average usage, managed cloud routes (B to D below) cost roughly 20 to 45% of the on-prem stack at 100 and 400 developers, and 50 to 75% at 20, and hybrid routes fall between them depending on the restricted share. The GPU decision should follow the data classification result: no S3 or a small S3 share means no 8-GPU purchase at 20 developers and usually not at 100. Results are highly sensitive to cache hit rate and agent intensity, and rework can outweigh token price.

### Cited Findings

**Official token prices, USD per million tokens**

| Model | Input | Cached input | Output | Cache write | Source and note |
|---|---|---|---|---|---|
| Claude Fable 5.1 | 10 | 0.25 | 50 | 12.50 | [Anthropic](https://platform.claude.com/docs/en/about-claude/pricing) |
| Claude Opus 5.5 | 4 | 0.20 | 20 | 5 | Same; [also Foundry CCU at same rates, 1.1x for US Data Zone](https://platform.claude.com/docs/en/about-claude/pricing) |
| Claude Sonnet 5.5 | 2 | 0.10 | 10 | 2.50 | Same |
| Claude Haiku 5.5 | 0.10 / 0.50 over 100k prompt | 0.01 / 0.05 | 0.50 / 2.50 | 0.125 / 0.625 | Same; long agent prompts pay the higher rate |
| GPT-6 Astra | 10 | 1.00 | 50 | n/a | [OpenAI](https://developers.openai.com/api/docs/pricing), up to 272K input |
| GPT-6.1 Sol | 2 | 0.10 | 10 | n/a | Same; 10% residency uplift; batch 50% |
| GPT-6.1 Terra | 1 | 0.05 | 5 | n/a | Same |
| GPT-6 Luna | 0.10 | 0.01 | 0.50 | n/a | Same |
| Gemini 4 Argon | 2 → 4 on 1 Dec | 0.10 → 0.20 | 10 → 20 | implicit | [Google](https://aistudio.google.com/docs/pricing). Introductory price ends 30 Nov |
| Gemini 3.8 Flash | 0.75 → 1.50 on 1 Jan | 0.075 → 0.15 | 3.75 → 7.50 | storage fee per hour | Same |
| GLM-5.3 (Z.ai) | 1.40 | 0.26 | 4.40 | storage free for now | [Z.ai](https://docs.z.ai/guides/overview/pricing), PRC-based operator |
| GLM-5.3-Flash (Z.ai) | 0.15 | 0.03 | 0.50 | same | Same |
| GLM-5.3 (Bedrock Global / US) | 1.68 / 1.848 | 0.312 / 0.343 | 5.28 / 5.81 | 2.10 / 2.31 | [AWS](https://aws.amazon.com/bedrock/pricing/) |
| Kimi 3 (Bedrock Global) | 3 | 0.30 | 15 | 3.75 | Same |
| Kimi K3.1 Pro Highspeed (Bedrock Global) | 4 | 0.20 | 20 | 5 | Same; identical to Opus 5.5 |

Not obtained: [Azure OpenAI list prices did not render](https://azure.microsoft.com/en-us/pricing/details/azure-openai/); Bedrock's Claude rows did not render; Gemini 4 Carbon; DeepSeek official rates. A [CostBench tracker lists Astra and Luna consistently with OpenAI](https://costbench.com/software/llm-api-providers/openai-api/) [2nd]. I assume Azure parity with OpenAI for planning, which must be verified.

**[ESTIMATE] Demand basis.** Scale the [Claude Code docs' illustrative session](https://code.claude.com/docs/en/costs) (1.2k input, 5.3k output, 940k cache read, 50k cache write; $0.55 at Opus 5.5 rates) to the middle of the [$150-250/month enterprise average](https://code.claude.com/docs/en/costs) [Vendor]: **$200/developer/month at Opus 5.5**, or 0.44M input, 1.93M output, 18.2M cache write, 342.6M cache read, about 95% cache hit. This is one vendor example applied to all models, so it ignores differences in tokens per task. Where no separate write price is listed (OpenAI, Z.ai), cache writes are charged at the input rate (assumption).

| Model at same mix | $/dev/month | 20 devs/yr | 100 devs/yr | 400 devs/yr |
|---|---|---|---|---|
| GPT-6 Astra | 626 | $150K | $751K | $3.00M |
| Fable 5.1 | 414 | $99K | $497K | $1.99M |
| Opus 5.5 (US Data Zone 1.1x: $220) | 200 | $48K | $240K | $960K |
| Kimi 3 / K3.1 Pro on Bedrock | 200 | $48K | $240K | $960K |
| GLM-5.3 on Bedrock Global | 156 | $37K | $187K | $749K |
| GLM-5.3 Z.ai | 124 | $30K | $148K | $594K |
| Sonnet 5.5 | 100 | $24K | $120K | $480K |
| GPT-6.1 Sol (with 10% uplift: $100) | 91 | $22K | $109K | $436K |
| GPT-6.1 Terra | 45 | $11K | $55K | $218K |
| Haiku 5.5 (over 100k prompts) | 34 | $8K | $40K | $161K |
| GLM-5.3-Flash Z.ai | 14 | $3K | $17K | $67K |
| GPT-6 Luna | 6 | $2K | $8K | $30K |

**[ESTIMATE] Cache sensitivity.** $/dev/month by hit rate (write cost replaces read for the remainder):

| Hit rate | Opus 5.5 | Sonnet 5.5 | GPT-6.1 Sol | GLM-5.3 Bedrock |
|---|---|---|---|---|
| 95% | 197 | 99 | 90 | 155 |
| 90% | 284 | 142 | 124 | 187 |
| 70% | 631 | 315 | 261 | 316 |
| 50% | 977 | 489 | 399 | 446 |

GPT-6.1 Sol is cheaper than GLM-5.3 on Bedrock at every listed hit rate, and Sonnet 5.5 is cheaper above about 70%. [GLM's cached price is 18.6% of input](https://aws.amazon.com/bedrock/pricing/), versus 5% for those two models, so hosted open weights gain ground mainly when caching is poor.

**GPU rental reference [2nd]:** [Azure ND96isr H200 v5 about $84.80 to $110.24 per hour](https://www.spheron.network/blog/azure-h200-pricing-2026/) (dated 6 Jul 2026, seller-authored, regions vary); [MI355X $2.59/GPU-hr at one provider](https://www.gpuperhour.com/rent/mi355x) [Prior; supply unverified].

### Inferences

**[ESTIMATE] Annual cost by route.** On-prem totals of $105K, $681K and $2.48M are my earlier estimates in `onprem_models_compute.md` (3-year straight line, 10% support, staff, no facility build-out). Cloud routes add gateway and governance staff of 0.25, 0.5 and 1 FTE at $120K (assumption). Hybrid splits use the mid-frontier to Opus range for the cloud share. Sizing assumes about 74 developers per 8-GPU server [Prior], at about $250K per server per year excluding staff (from the prior tier numbers).

| Route | 20 devs | 100 devs | 400 devs | GPU purchase outcome |
|---|---|---|---|---|
| **A. On-prem only** | $105K ($437/dev/mo) | $681K ($568) | $2.48M ($518) | Pilot; 2× 8-GPU; about 8× 8-GPU, after bake-off |
| **B. Frontier premium** via Foundry, Bedrock or Vertex (Opus 5.5 class) | $78K ($325) | $300K ($250) | $1.08M ($225) | None; rent for evals only |
| **C. Frontier mid** (Sonnet 5.5, GPT-6.1 Sol class) | $52-54K | $169-180K | $556-600K | None |
| **D. Hosted open weights** (GLM-5.3 on Bedrock Global) | $67K | $247K | $869K | None. Weak case: cost above C, lower quality on most benchmarks, PRC-origin rules still apply |
| **E1. Private VPC, MI355X rental 24×7** (1, 2, 8 servers; 0.5, 1, 2 FTE) | $242K | $483K | $1.69M | None; supply and location unverified |
| **E2. Private VPC, Azure H200 at $84.80/hr, 24×7** | $803K | $1.61M | $6.18M | None, but on-demand costs more than buying: about $743K per server per year |
| **E3. Same, office hours only** (3,120 hours) | $325K | $649K | $2.36M | None; loses off-hour batch work |
| **F1. Hybrid, 20% restricted on-prem** | $71-90K (workstation, 2× RTX PRO, assumed $45K capex) | $231-327K (pilot tier) | $1.13-1.51M (2× 8-GPU) | No 8-GPU at 20 or 100; 2 servers at 400 |
| **F2. Hybrid, 50% restricted on-prem** | $117-129K (pilot tier) | $460-520K (1× 8-GPU, no N+1) | $1.60-1.84M (4× 8-GPU) | Pilot; 1 server; 4 servers |

**Break-evens [ESTIMATE].** Usage multiple at which frontier list pricing equals the on-prem stack, holding the token mix:

| Scale | vs Opus 5.5 | vs Sonnet 5.5 |
|---|---|---|
| 20 devs | 1.6× | 3.1× |
| 100 devs | 2.6× | 5.2× |
| 400 devs | 2.5× | 4.9× |

- Autonomous agents can reach these multiples. The prior note assumes one always-on agent equals 3-5 developers (assumption, unsourced), and [agent teams use about 7× tokens in plan mode](https://code.claude.com/docs/en/costs) [Vendor]. On-prem capacity is also consumed by the same growth, however, so the crossover needs load testing rather than this arithmetic.
- **Prior utilisation math (`onprem_models_compute.md`, based on [InferenceX throughput](https://inferencex.semianalysis.com/compare/glm-5-1-h200-vs-mi355x)):** on-prem matches hosted GLM uncached at about 15% utilisation and about 31% with 90% caching. The 95% hit assumption here raises that bar further.

**GPU decision rule (my judgment):**
1. Do not buy anything until classification yields the S3 and S2 share, and legal answers the cloud question.
2. Cloud approved and S3 below about 20%: **no 8-GPU server under about 100 developers**. A small on-prem tier serves S3 and continuity.
3. S3 around 50%, or sustained off-hour agent load near the break-even multiple: buy 8-GPU servers, starting with one or two.
4. Cloud refused: on-prem purchase stands, but only after the rented bake-off, as in the earlier report.
5. Do not treat on-demand hyperscaler VMs as a cheap alternative. Rent short-term for evaluation or burst only, unless reservation pricing changes this.

### Gaps
- No enterprise discount, PTU, Azure OpenAI list or Provisioned quote was obtained. Foundry prices for Fireworks-hosted models were not listed.
- The token mix is one vendor example. Open models may use more tokens per task, while cheaper models can use more too ([DeepSWE shows Sonnet 5 at $26.40 per task, above Opus 5 at $11.84](https://deepswe.datacurve.ai/)). Measure on real traces.
- Taiwan GPU cloud prices and Asian ND H200 availability were not verified. Taiwan-specific staff and power costs remain placeholders from the prior note.

## Quality differences on agentic coding, in terms of rework cost

### Takeaway
No published study measures rework cost for frontier versus open-weight models. Benchmarks imply opposite answers: a small gap on task completion and a gap of over 2× on mergeability, with the mergeability gap widening as tasks grow. My illustrative model shows the cost winner flips with the benchmark used, so this has to be settled by an internal bake-off that records human minutes.

### Cited Findings
Reused baseline: the capability section of `onprem_models_compute.md` covers DeepSWE, FrontierCode, Artificial Analysis v5, vendor score conflicts and harness effects. New or sharpened evidence follows.

- **[FrontierCode 2.0 primary post, 27 Oct 2026](https://cognition.com/blog/frontier-code-2.0)** [Vendor: Cognition sells a coding agent], best effort per model:

| Model | FC 1.1 | FC 2.0 |
|---|---|---|
| Claude Opus 5.5 | 54.6 | 45.5 |
| GPT-6 Astra | 53.3 | 42.8 |
| Grok 4.7 | 47.6 | 28.6 |
| Gemini 4 Argon | 47.5 | 26.3 |
| SWE-2 | 50.0 | 23.9 |
| DeepSeek V4.1 Flash | 43.9 | 19.3 |
| Kimi K3 | 44.2 | 17.7 |

- Version 2.0 touches [more files and lines in larger codebases, and thoroughness failures rose](https://cognition.com/blog/frontier-code-2.0). The post says western open models trail furthest, without much detail.
- **[BenchLM snapshot, 6 Nov](https://benchlm.ai/md/benchmarks/frontiercode-2.md)** [2nd]: Sonnet 5.5 41.0, Gemini 4 Carbon 40.0, GPT-6.1 Sol 39.0, Haiku 5.5 27.7, GPT-6 Luna 21.9, Mistral Large 4 19.7, Kimi K3.1 Pro 19.6, DeepSeek V4.1 Flash 1017 19.3. **GLM-5.3 is not listed.** Per-model harnesses differ here, whereas [Artificial Analysis describes one Stirrup harness and shows Opus 5.5 at 46.4](https://artificialanalysis.ai/evaluations/frontiercode-2-0); that conflict remains unresolved.
- **[DeepSWE v1.1, 22 Sep](https://deepswe.datacurve.ai/)** [3P], pass@1 and average cost per task: gpt-6-astra 74% $4.43; gemini-3.8-flash 74% $2.36; claude-opus-5 74% $11.84; gpt-5.6-sol 73% $6.46; glm-5.3 69% $3.99; kimi-k3 69% $4.65; gpt-5.6-luna 67% $0.61; glm-5.3-flash 63% $0.24; deepseek-v4-pro 63% $1.67; claude-sonnet-5 54% $26.40. Newer models (GPT-6.1, Opus 5.5, Sonnet 5.5, DeepSeek V4.1, Kimi K3.1) are absent.
- Passing tests overstates mergeability: [METR found scorer results about 24 points above maintainer merge decisions](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) [Prior].
- [SWE-2, reported as a post-trained Kimi K3](https://www.orcarouter.ai/de/blog/cognition-swe-2-release) [2nd], scores above Kimi K3 on both versions in Cognition's table, hinting that post-training on open weights can recover some gap, though this is a vendor result.

### Inferences
- **The gap appears to grow with task size.** Opus 5.5 to DeepSeek V4.1 Flash goes from 1.24× on FC 1.1 to 2.36× on the larger FC 2.0 tasks. This fits the [prior long-horizon evidence](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/README.md), but it comes from one vendor's benchmark.
- **"Open versus frontier" hides a third option.** On DeepSWE, cheap hosted models match or beat open flagships on both axes: gemini-3.8-flash (74%, $2.36) against glm-5.3 (69%, $3.99), and on FC 2.0 even Luna and Haiku 5.5 sit above listed open-weight models. If cloud is permitted, open weights' main remaining reason is sovereignty, not value. This price uses Gemini's temporary rate.
- **[ESTIMATE] Rework model, illustrative only.** Assume 20 minutes of review per task, plus 60 minutes of rework when the attempt fails (assumptions, no source), token cost from DeepSWE, and $40/hour loaded. Taiwan-specific rates were not sourced.

| Comparison | Added human minutes per task, open vs frontier | Total, frontier | Total, open | Cheaper |
|---|---|---|---|---|
| Opus 5 vs GLM-5.3, DeepSWE rates (74/69) | 3.0 | $35.57 | $29.72 | Open, by $5.85 |
| GPT-5.6 Sol vs GLM-5.3, DeepSWE (73/69) | 2.4 | $30.59 | $29.72 | Roughly even |
| Opus 5 vs GLM-5.3-Flash, DeepSWE (74/63) | 6.6 | $35.57 | $28.37 | Open, by $7.20 |
| Opus-class vs DeepSeek-class, FrontierCode 2.0 rates (45.5/19.3), with DeepSWE token costs | 15.7 | $46.97 | $49.60 | Frontier, by $2.63 |

  At $25/hour the last row flips to open by $1.30; at $60/hour frontier wins by $7.87. At an assumed 40 tasks per developer per month, the spread across rows is about −$290 to +$105/developer/month, the same order as the token bills above. Rework assumptions matter as much as prices. Failed attempts may also cost more than 60 minutes on large changes, which would favour frontier models further.
- **Routing implication:** send scoped fixes, tests, docs and pre-screen review to on-prem or cheap tiers, and send long-horizon or cross-cutting work to the strongest permitted model. Fresh-context verification stays in place on every tier, because [independent context appears to matter more than vendor choice](https://arxiv.org/html/2610.28667) [Prior, small preregistered sample].
- **Bake-off metrics:** first-pass merge rate, human minutes to merge, reopened and reverted PRs at 30 days, escaped defects, tokens per merged PR, with fixed harness versions. Use 50-100 internal tasks in S0 or S1 repos, on rented GPUs at target quantization, as the earlier report proposed.

### Gaps
- No independent study converts model choice into rework hours. My minutes are assumptions.
- No benchmark tests self-hosted, quantized open models under enterprise load, and [hosted scores may overstate self-hosted results](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md) [Prior].
- GLM-5.3 lacks a FrontierCode 2.0 entry, and no zh-TW agentic coding comparison exists.
