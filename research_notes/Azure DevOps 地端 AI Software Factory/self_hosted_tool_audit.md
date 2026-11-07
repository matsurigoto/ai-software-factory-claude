# Self-Hosted, On-Prem and Air-Gapped Audit of the AI Software Factory Tool Layers (as of 2026-11-07)

Method and limits, for the report writer:
- Starting list: `reports/AI Software Factory 架構與工具選型.md`.
- Repo license, star, push and release data came from the GitHub API (`gh api`) on 2026-11-07. NuGet versions came from the NuGet flat-container API the same day.
- **No tool was installed or tested.** "Documented" means the vendor's documentation says so. It does not mean anyone verified zero egress.
- Web pages were read through a summarizing fetch tool. Some items rest on search summaries rather than a page I read in full. I flag them where the source is secondary.
- Local model quality and GPUs belong to another researcher. Azure DevOps integration depth also belongs to another researcher. This file covers Azure DevOps only where it decides air-gap viability.

Verdict legend used in all tables:
- **Documented**: vendor docs describe no-internet operation.
- **Feasible**: the architecture appears to allow it and nothing contradicts it, but no vendor statement was found.
- **Partial**: runs offline with named features lost.
- **No**: a cloud control plane or cloud API is required.
- **Unconfirmed**: only secondary claims, or conflicting ones.

## Which on-prem default and alternate should each layer use?

### Takeaway
Every layer has a workable on-prem default, but the shortlist changes: Outline loses its top spot for Windows/AD shops, Onyx needs custom connectors for Azure DevOps and SMB, Agent Framework's durable extension and the LangGraph server are not air-gap defaults, Nexus Community's caps look too small for agent traffic, and snyk/agent-scan must be dropped. No commercial coding agent was found that documents both air-gap and Azure DevOps Server support.

### Cited Findings
Evidence for each row is in the layer sections below.

| Layer | On-prem default (my judgment) | Alternate (my judgment) | Change from earlier report |
|---|---|---|---|
| Wiki | BookStack (MIT, native AD/LDAP) with a CJK search sidecar | Docmost Business (SSO and MCP paid; "Air-gapped ready"), or Outline if OIDC is already available | Outline was the default |
| Search add-on | OpenSearch or Meilisearch sidecar, chosen after a Traditional Chinese test | PGroonga where the app's SQL can be changed | New |
| Retrieval | Onyx Enterprise or Business self-hosted plus custom connectors | RAGFlow v1.0 for document-heavy, team-scoped use without ACL inheritance | Same tool, but Azure DevOps, SMB and SharePoint Server are gaps |
| Code knowledge | codebase-memory-mcp + Zoekt + Repomix | Sourcegraph Enterprise with an "air gapped" license tag | Zoekt added; Sourcegraph is the paid alternate |
| Orchestration | Temporal on Linux Kubernetes with Postgres, .NET workers, Agent Framework inside activities | Dapr Workflow (.NET) with Agent Framework | Dapr Agents is not an option for .NET |
| Skills | Private Git monorepo plus CI packaging, cisco skill-scanner in local mode | Vendored `vercel-labs/skills` with telemetry off | snyk/agent-scan removed |
| Test | Keep the deterministic tools, mirror the feeds | Opengrep or Semgrep CE with vendored rules | promptfoo needs extra flags and an egress block |
| Kubernetes | RKE2 (Linux control plane, optional Windows workers) | OpenShift if a subscription is acceptable; Talos for Linux-only clusters | New |
| Sandbox | agent-sandbox + Kata on bare metal or nested virtualization, gVisor where neither exists | Coder workspaces | Windows workloads are outside this isolation model |
| Observability | Langfuse OSS + OTel with `TELEMETRY_ENABLED=false` | MLflow (from earlier report; not re-audited) | Enterprise edition has an air-gap trap |
| Registry and mirrors | Harbor for images; Azure Artifacts or Artifactory for packages | Nexus Pro; Verdaccio and devpi for small installs | Nexus Community caps |
| Gateway | agentgateway | Agent Router, IBM ContextForge | Microsoft mcp-gateway is not on-prem ready |
| Secrets, identity, policy | OpenBao, SPIRE, OPA with `--disable-telemetry` | Same | Only OPA's phone-home was verified |
| Commercial agent | Factory Droid airgap build or Tabnine private install, both after a written Azure DevOps Server confirmation | OSS harness behind egress block | Cursor, Amp and Copilot excluded |

Corrections to the earlier report:
- It said GitLab's air-gap handling was undocumented. GitLab now documents an [offline deployment](https://docs.gitlab.com/ee/administration/gitlab_duo_self_hosted/offline_deployment/) requiring 18.9+ and an offline cloud license.
- It treated Factory as air-gapped generally. The [airgap builds doc](https://docs.factory.com/enterprise/airgapped-deployment.md) supports this for Droid, but [Factory Private](https://factory.com/news/factory-private) is a Private Preview VPC option with Factory operating the control plane by default.
- It named Coder "AI Bridge and Boundaries". The [pricing page](https://coder.com/pricing) now shows AI Premium, AI Gateway and Agent Firewall naming, so confirm the SKU names.
- [Continue](https://github.com/continuedev/continue) is now read-only, and it was in scope only for this audit.
- [Serena](https://github.com/oraios/serena) is GPL-3.0-or-later for the application, with MIT only for SolidLSP.

### Inferences
- The stack is viable offline, but "viable" costs a recurring import process more than any single license. Trivy, Nuclei, ZAP, rule packs, Playwright browsers, packages, images and model catalogs all need a controlled transfer path.
- Open-core licensing tends to take away the features a multi-department setup needs most: Onyx permission sync, Langfuse project RBAC and audit logs, Coder governance, Docmost SSO and SonarQube PR decoration. Budget for those even if the stack is "open source".

### Gaps
- Nothing here was tested with a packet capture. Run each tool in a default-deny network before accepting a verdict.

## Wiki and knowledge: which wiki and CJK search work offline with Windows identity?

### Takeaway
BookStack is the best Windows fit because it has native AD/LDAP with group mapping under MIT, but its CJK search is the weakest point. Outline has no LDAP, so an offline AD shop needs an OIDC provider first. Every candidate needs a Traditional Chinese test, and no candidate has official evidence of good CJK search without an extra engine or paid driver.

### Cited Findings

| Tool | License and version | Identity | Air-gap | What is lost or paid |
|---|---|---|---|---|
| [Outline](https://github.com/outline/outline) | [BSL 1.1](https://github.com/outline/outline/blob/main/LICENSE) with a Document Service restriction; v1.12.0 (2026-10-23) | [Slack, Google, Entra, Discord, generic OIDC in `.env.sample`](https://raw.githubusercontent.com/outline/outline/main/.env.sample); [SAML only in Business and Enterprise](https://www.getoutline.com/pricing). LDAP absent; [LDAP group mapping request still open](https://github.com/outline/outline/issues/8168), and so is [local auth](https://github.com/outline/outline/issues/1881) | **Feasible**. Set `ENABLE_UPDATES=false`; integrations are blank by default ([`.env.sample`](https://raw.githubusercontent.com/outline/outline/main/.env.sample)) | Community is $0 with limited features; [Business $4 and Enterprise $5 per user per month](https://www.getoutline.com/pricing). License key behavior offline not stated |
| [BookStack](https://github.com/BookStackApp/BookStack) | MIT; v26.09.2 (2026-10-20) | [LDAP with AD examples (`sAMAccountName`, `objectGUID`), group-to-role sync, nested groups via `memberOf`](https://www.bookstackapp.com/docs/admin/ldap-auth/); [OIDC group mapping](https://www.bookstackapp.com/docs/admin/oidc-auth/) | **Feasible**. LDAP page says it talks only to the LDAP server for those functions | No vendor MCP ([community only](https://github.com/pnocera/bookstack-mcp-server)). Onyx connector has [no permission sync](https://docs.onyx.app/admins/connectors/overview.md) |
| [Docmost](https://github.com/docmost/docmost) | AGPL-3.0; paid features need a key; v0.97.0 | [SAML, OIDC, LDAP only in Business and Enterprise](https://docmost.com/pricing) | **Documented by tier**: Business listed as "Air-gapped ready", Enterprise as air-gapped deployment ([pricing](https://docmost.com/pricing)); offline license mechanics not stated | Community has no SSO, page permissions or MCP. Business is $6 per seat, 10-seat minimum. Typesense tier unclear on the page |
| [Wiki.js](https://github.com/requarks/wiki) | AGPL-3.0; 3.1.1 (2026-11-06) | [LDAP, SAML, Azure AD, OIDC per project site](https://hn.jakechampion.name/reader/https://js.wiki) | **Partial**. [Docs](https://docs.requarks.io/install/requirements/internet) describe periodic contact with a developer service, a sideload path, and modules needing internet (search summary; page body not readable) | [Basic search is reported as titles only](https://GitHub.com/requarks/wiki/discussions/7166); [Elasticsearch module](https://docs.requarks.io/search/elasticsearch) available. Those docs may describe v2, not v3 |

CJK search options:

| Option | License and version | Finding |
|---|---|---|
| [PGroonga](https://github.com/pgroonga/pgroonga) | PostgreSQL License; 4.1.0 (2026-10-09) | Claims full-text search for all languages. [Zulip offers it as an experimental self-host option for Chinese and Japanese](https://zulip.com/help/configure-multi-language-search) |
| [pg_cjk_parser](https://doc.pigsty.io/ext/e/pg_cjk_parser) | Not checked | Emits overlapping 2-grams for CJK; a Postgres-level option |
| [Meilisearch](https://github.com/meilisearch/meilisearch) | [MIT and BUSL-1.1 for EE parts](https://github.com/meilisearch/meilisearch/blob/main/LICENSE); v1.56.1 | Telemetry on until [`MEILI_NO_ANALYTICS=true`](https://www.meilisearch.com/docs/resources/help/telemetry.md). A [BookStack-Meilisearch proof of concept](https://backiee.wasmer.app/http_github_com/johnroyer/bookstack-meilisearch) exists and says not to use it in production |
| [Typesense](https://github.com/typesense/typesense) | GPL-3.0; v30.2 (2026-04-19) | [Secondary source says the default tokenizer lacks CJK segmentation](https://server.hk/blog/how-to-deploy-typesense-on-hong-kong-vps-algolia-alternative-2026/), and another [says ICU handles it with a locale](https://railway.com/deploy/typesense-cjk--deploy-typesense-cjk-search-on-railway). These are hosting vendor pages and they conflict |
| [OpenSearch](https://github.com/opensearch-project/OpenSearch) | Apache-2.0; 3.9.0 | [IK plugin for OpenSearch is a community fork, installable from a local zip](https://toscode.mulanos.cn/mirrors/opensearch-analysis-ik); [forum reports it was unreliable in the 2.0 era](https://forum.opensearch.org/t/opensearch-analysis-ik-plugin/9664). [Huawei advises Traditional-to-Simplified conversion before IK because IK dictionaries are mainly Simplified](https://support.huaweicloud.com/intl/en-us/usermanual-css/css_01_0324.html) |

- The Outline CJK issue [#3207](https://github.com/outline/outline/issues/3207) is closed, and the earlier report notes it closed through inactivity rather than a fix.
- BookStack search: [a third-party project says tokens are space-separated](https://backiee.wasmer.app/http_github_com/johnroyer/bookstack-meilisearch), which suits CJK poorly. I found no official statement.

### Inferences
- In an air-gapped site, Outline's Entra and Google providers are unusable, so only generic OIDC or paid SAML remains. An on-prem AD shop would need AD FS or Keycloak in front. That is one more service to run.
- A sidecar index that crawls the wiki is likely more practical than changing a wiki's search code. For Outline, a fork is a BSL and upgrade burden.
- For Traditional Chinese, 2-gram indexing avoids segmentation dictionary bias and should be in the test set alongside IK with conversion.

### Gaps
- No official Traditional Chinese recall data for any option. Smartcn offline install and Meilisearch Traditional Chinese behavior were not confirmed.
- Docmost's Typesense tier, BookStack telemetry, Docmost telemetry and the exact Wiki.js telemetry switch in v3 were not confirmed. [Wiki.js internet page](https://docs.requarks.io/install/requirements/internet) rendered no body for the fetch tool.
- Azure DevOps Wiki comparison is left to the Azure DevOps researcher.

## Retrieval: which layer can run offline with local LLMs, on-prem connectors and ACL sync?

### Takeaway
Onyx is the only candidate with documented air-gap settings and on-prem Confluence permission sync, but it has no Azure DevOps, SMB or evidenced SharePoint Server connector. RAGFlow and Dify can run locally, but I found no ACL inheritance evidence, and an air-gap statement for RAGFlow only from a third party.

### Cited Findings

| Requirement | Onyx v5.1.0 | RAGFlow v1.0.0 | Dify 1.19.0 | OpenSearch-based build |
|---|---|---|---|---|
| License | [MIT outside `ee`; `ee` under Onyx Enterprise License](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE) | [Apache-2.0](https://github.com/infiniflow/ragflow) | [Modified Apache-2.0: no multi-tenant use without written permission, one workspace = one tenant, logo retained](https://github.com/langgenius/dify/blob/main/LICENSE) | [Apache-2.0](https://github.com/opensearch-project/OpenSearch) |
| Air-gap | **Documented**: [`ONYX_AIRGAPPED=true`, `DISABLE_TELEMETRY=true` on every service, manual license file, `onyx-cli deploy upgrade --offline`](https://docs.onyx.app/security/self_hosted/faq) | **Unconfirmed**: [only a Zilliz FAQ](https://zilliz.com/ai-faq/does-ragflow-support-airgapped-deployment) | **Partial**: [`CHECK_UPDATE_URL` empty for air-gap](https://docs.dify.ai/en/self-host/configuration/environments); [third-party guide adds `MARKETPLACE_ENABLED=false`](https://besthub.dev/articles/why-offline-deployment-of-dify-is-so-challenging-10-common-pitfalls-and-solutions-97bf74880283) and repackaged plugins; [custom signature verification is a documented option](https://docs.dify.ai/en/develop-plugin/publishing/standards/third-party-signature-verification) | **Feasible** (self-built) |
| Local LLM | [Required for offline use; docs advise a large model, not a lightweight one](https://docs.onyx.app/security/self_hosted/faq). [Non-default embedding or reranking models must be pre-downloaded](https://docs.onyx.app/security/self_hosted/data_processing.md) | [Earlier full images bundled embeddings; a single edition since v0.22](https://github.com/Ox0400/ragflow) (mirror README) | Not re-verified | Self-built |
| Azure DevOps | **Not available**. [Connector tree on `main`](https://github.com/onyx-dot-app/onyx/tree/main/backend/onyx/connectors) has none, and [PR #6496](https://github.com/onyx-dot-app/onyx/pull/6496) is closed | Not found | Not checked | Custom |
| SMB file share | **Not listed** ([overview](https://docs.onyx.app/admins/connectors/overview.md)); [third-party guide also says no SMB and suggests a custom connector](https://wz-it.com/en/blog/onyx-self-hosted-installation/) | Not found | Not checked | Custom |
| SharePoint on-prem | **Online only as documented**: [needs Microsoft 365, Graph and Entra app](https://docs.onyx.app/admins/connectors/official/sharepoint) | [v1.0 connector needs an Entra app](https://ragflow.io/docs/v1.0.0/add_data_source/data_source_configuration); [earlier version was a stub until PR #15190](https://github.com/infiniflow/ragflow/pull/15190) | Not checked | Custom |
| Confluence Data Center | **Yes, with permission sync**: [PAT, space and page restrictions, email matching, 30-minute sync](https://docs.onyx.app/admins/connectors/official/confluence) | Search summary mentions a Cloud or Server/Data Center toggle, with [no ACL guarantee](https://ragflow.io/docs/add_confluence); page now returns 404 | Not checked | Custom |
| ACL sync | Paid. [Overview says Business, Enterprise and Cloud](https://docs.onyx.app/admins/connectors/overview.md); [Confluence page says self-hosted needs Enterprise Edition](https://docs.onyx.app/admins/connectors/official/confluence). **Sources conflict** | No evidence | SSO and RBAC in Enterprise per the earlier report; not re-verified | Self-built |
| Price | [Business $20 per user per month annual; Enterprise custom](https://onyx.app/pricing) | Free | Multi-tenant needs permission | Staff time |

- RAGFlow v1.0.0 shipped on 2026-11-06. [v1.0.0-rc1 was a comprehensive Go rewrite](https://newreleases.io/project/github/infiniflow/ragflow/release/v1.0.0-rc1), so production history on this code base is very short.
- The Onyx wiki connectors for [Outline and BookStack exist, but have no permission sync](https://docs.onyx.app/admins/connectors/overview.md).
- Onyx users are matched by email, and [masked Confluence emails do not match](https://docs.onyx.app/admins/connectors/official/confluence). AD accounts need a consistent mail attribute.

### Inferences
- In an Azure DevOps Server and SMB shop, the realistic Onyx design is Confluence DC (if present) with native sync, plus a custom ingestion job for Azure DevOps wiki, repos, work items and SMB, with coarse access by department group. Per-document ACL fidelity on those sources is custom work, not a checkbox.
- If Onyx's paid tier is unavailable, run one instance per department. This gives coarse isolation, not per-document ACLs.
- Dify remains an app builder; a workspace per department triggers its license term.

### Gaps
- Onyx offline license validation, renewal and expiry. Whether custom connectors can carry per-document ACLs in EE was not verified.
- Whether Business or Enterprise is required for permission sync self-hosted needs a written answer.
- RAGFlow final v1.0.0 release notes and telemetry; Dify connectors and SSO terms.

## Code knowledge: which tools work offline and with local LLMs?

### Takeaway
The local stack is strong because the best code tools do not need an LLM at all: codebase-memory-mcp, Zoekt and Repomix run offline and let the local-model harness do the thinking. Sourcegraph supports Azure DevOps Server fields, but air-gap depends on a license tag and Deep Search is unavailable with BYOK. DeepWiki-Open is the weak option for Azure DevOps.

### Cited Findings

| Tool | License and status | Offline | Local LLM | Azure DevOps and loss |
|---|---|---|---|---|
| [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | MIT; v0.12.1 (2026-10-16); 48.2k stars | **Documented locally**: no API key, no telemetry, no background network calls; only optional Lens chat model (Hugging Face) and install or updates reach out. Manual archive and Windows installer available | Not needed for backend | Works on local checkout. Windows: some hooks withheld, Defender false positive and SmartScreen warnings noted |
| [Zoekt](https://github.com/sourcegraph/zoekt) | Apache-2.0; active; no GitHub releases | **Feasible** | Not needed | No release tags, so pin a commit. ADO indexing not verified; generic Git mirror assumed |
| [Repomix](https://github.com/yamadashy/repomix) | MIT; v1.18.1 | **Feasible** for local mode; [remote mode needs internet; bare `owner/repo` may try GitHub; use `./` to force local](https://repomix.com/guide/remote-repository-processing) | Not needed | Secret detection behavior and telemetry not verified |
| [Serena](https://github.com/oraios/serena) | GPL-3.0-or-later with MIT SolidLSP; CLA; v2.1.0 | **Documented path**: [`serena download-ls-dependencies` at build time; repeat after upgrades](https://oraios.github.io/serena/01-about/020_programming-languages.html) | Via any MCP client | [C# default Roslyn needs .NET 10+ and `pwsh` on Windows; OmniSharp alternative](https://oraios.github.io/serena/01-about/020_programming-languages.html). JetBrains plugin excludes Rider |
| [DeepWiki-Open](https://github.com/AsyncFuncAI/deepwiki-open) | MIT; no releases | **Unconfirmed** | Ollama Dockerfile and guide exist | Named hosts are GitHub, GitLab and Bitbucket; no Azure DevOps found |
| [Sourcegraph](https://sourcegraph.com/docs/admin/deploy) | Proprietary, Enterprise plan; private repo; Helm or Docker Compose; no ARM in production | **Partial, license-dependent**: [handbook says recent versions need an "Allow air gapped" tag, and untagged licenses ping Sourcegraph.com or the instance shuts down](https://handbook.sourcegraph.com/departments/data-analytics/how-to-airgap-disable-telemetry). Handbook now redirects to Notion, so this came from a search summary and may be stale | [Deep Search is not available to BYOK customers](https://sourcegraph.com/docs/deep-search) | [ADO config has "Server OnPrem Only" fields; permission sync uses OAuth and user tokens](https://sourcegraph.com/docs/admin/code_hosts/azuredevops). Server applicability of permission sync not stated |

- Search for a current Sourcegraph air-gap doc found [marketing claims and a comparison saying deployment docs state no air-gap guidance](https://repowise.dev/blog/comparisons/best-air-gapped-self-hosted-code-analysis-tools). Treat this as needing a written answer from the vendor.
- [Cody Enterprise is described as air-gap capable only with a self-hosted model service and Gateway disabled](https://intuitionlabs.ai/articles/enterprise-ai-code-assistants-air-gapped-environments) (third-party, updated 2026-07-29).

### Inferences
- Deep Search is probably unavailable in a strict air-gap, because it needs the Sourcegraph Model Provider. This is my inference; Sourcegraph does not state it.
- Index from an internal Git mirror job rather than from each tool's code-host connector. This makes Zoekt, codebase-memory-mcp and Repomix independent of Azure DevOps Server API support.
- codebase-memory-mcp's rapid star growth deserves caution. Build it from source or review it like other third-party code.

### Gaps
- No benchmark for these tools with local models or large .NET solutions.
- Serena and Repomix telemetry not confirmed.

## Orchestration: which engine fits a likely Windows/.NET shop offline?

### Takeaway
Temporal is still the strongest default for long-running approvals if the team can run Linux Kubernetes and Postgres, and its .NET SDK is stable on NuGet. Agent Framework is fine for agent logic with local models, but its durable extension is a preview that connects to a Durable Task Scheduler backend, so do not use it as the offline backbone. LangGraph the library is free, while its server product is licensed.

### Cited Findings

| Engine | License and version | Air-gap | Operating burden | .NET fit |
|---|---|---|---|---|
| [Temporal](https://github.com/temporalio/temporal) | MIT; v1.32.2 (2026-11-02) | **Feasible**; [two Go binaries](https://docs.temporal.io/self-hosted-guide/deployment). No vendor air-gap doc found | [Immutable shard count, upgrades possibly biweekly and sequential across minors, no built-in RBAC or audit logging, trained admins needed](https://docs.temporal.io/self-hosted-guide/production-checklist). [Stores: Cassandra, MySQL, Postgres; Elasticsearch for visibility](https://docs.temporal.io/references/configuration) | [Temporalio 1.21.1 stable on NuGet](https://api.nuget.org/v3-flatcontainer/temporalio/index.json). Windows server host not confirmed. **No SQL Server store listed** |
| [Dapr](https://docs.dapr.io/operations/hosting/self-hosted/self-hosted-airgap/) + [Dapr Agents](https://github.com/dapr/dapr-agents) | Apache-2.0; Agents v1.1.0 | **Documented**: installer bundle, `dapr init --from-dir .`, Windows bundle example | Sidecar and scheduler to run; state store needed | [Dapr.Workflow and Dapr.AI 1.19.0 stable on NuGet](https://api.nuget.org/v3-flatcontainer/dapr.workflow/index.json). Dapr Agents lists **Python only**. [Agent Framework has a Dapr Conversation provider, for example Ollama-backed, on .NET 10](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/model-providers/dapr) |
| [Argo Workflows](https://github.com/argoproj/argo-workflows) | Apache-2.0; v4.1.6 | **Feasible**; Kubernetes-native | Requires Kubernetes; earlier report judged it weak for long human gates | No .NET-specific evidence |
| [LangGraph](https://github.com/langchain-ai/langgraph) library | MIT | **Feasible** | Checkpointer and node idempotency are your work | Python and JS focus; not .NET |
| LangSmith and Agent Server | Commercial | **Documented only by offline license**: [self-hosting is an Enterprise add-on](https://docs.langchain.com/langsmith/self-hosted); [`beacon.langchain.com` required and billing telemetry cannot be disabled unless offline mode](https://docs.langchain.com/langsmith/self-host-egress) | KEDA, control plane and data plane for full deployment | [Moderator: server binary needs an enterprise key in production](https://forum.langchain.com/t/best-practices-for-self-hosting-langgraph-server-oss-without-langgraph-keys/3779); a user disputes the scope, unconfirmed |
| [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) | MIT; [Microsoft.Agents.AI and .Workflows 1.28.0 stable](https://api.nuget.org/v3-flatcontainer/microsoft.agents.ai/index.json) | **Feasible** in process | [Ollama provider for C# and Python; no hosted tools, local MCP works, tool calling depends on the model](https://learn.microsoft.com/en-us/agent-framework/agents/providers/ollama) | Best native fit. [Durable Task package is 1.16.0-preview only](https://api.nuget.org/v3-flatcontainer/microsoft.agents.ai.durabletask/index.json) and [self-hosted workers connect to a Durable Task Scheduler backend; the emulator is described for local development](https://learn.microsoft.com/en-us/agent-framework/integrations/durable-extension) |

Temporal staffing evidence is thin:
- [A blog rule of thumb says teams under about five engineers without database operations skill usually prefer Cloud](https://markaicode.com/stack/temporal-postgresql-stack/). That is not a benchmark, and Cloud is excluded here.
- [Openmetal reports Postgres is now the common default and Cassandra visibility was removed in v1.24](https://openmetal.io/resources/blog/self-hosting-temporal-is-a-postgres-and-private-networking-decision-before-anything-else/).
- [Quo describes outgrowing Postgres and moving to Cassandra](https://www.quo.com/blog/postgres-to-cassandra/).
- [Temporal's own cost comparison favors Cloud, is vendor-run and nearly three years old](https://temporal.io/blog/benchmarking-latency-temporal-cloud-vs-self-hosted-temporal).

### Inferences
- For a Windows and .NET shop, the seam is: Linux-hosted engine, .NET workers, Agent Framework inside activities, and Azure DevOps as the human approval surface. .NET developers keep their language while the platform team takes on Linux.
- Do not assume the existing SQL Server estate can back Temporal. Plan Postgres skills.
- Dapr Workflow plus Agent Framework is the lighter alternate if Temporal's burden is too high; its production durability on your chosen state store needs a test.
- A staged path works: start with pipeline-driven flows (Azure DevOps researcher's scope) and add Temporal only when multi-day gates and replay justify it.

### Gaps
- Whether Temporal server runs supported on Windows. Whether Durable Task Scheduler has any supported non-Azure production backend for this extension. [Local alternatives were not found](https://learn.microsoft.com/en-us/agent-framework/integrations/durable-extension).
- Temporal server version-check or telemetry behavior: [configuration page does not mention it](https://docs.temporal.io/references/configuration); needs a source check.
- Dapr SQL Server state store support and Dapr scheduler on Windows nodes were not verified.

## Skills distribution: how do skills work privately, offline and on non-Claude harnesses?

### Takeaway
Plain Git is enough. `SKILL.md` directories are read by non-Claude harnesses such as OpenCode, Factory Droid and Tabnine CLI, and a private marketplace can be any Git URL. Scanning works offline with cisco skill-scanner in local mode, but snyk/agent-scan sends skill content to Snyk and should be excluded.

### Cited Findings
- [OpenCode reads `.claude/skills` and `.agents/skills`, plus HTTP catalogs, with allow, ask and deny per skill](https://opencode.ai/docs/skills).
- [Factory airgap mode keeps installed plugins and skills](https://docs.factory.com/enterprise/airgapped-deployment.md); [Tabnine CLI lists Agent Skills](https://docs.tabnine.com/main/sitemap.md).
- [Claude Code marketplaces are Git repositories, with `url` sources on any host, plus `archive`, `npm` and local path sources](https://code.claude.com/docs/en/plugin-marketplaces). An administrator can require a marketplace on every machine.
- [Azure Artifacts supports Universal Packages among its feed types](https://learn.microsoft.com/azure/devops/artifacts), a possible signed-release channel.
- [`vercel-labs/skills` supports about 79 agents, collects anonymous usage data, and `DISABLE_TELEMETRY=1` or `DO_NOT_TRACK=1` also turns off security-audit requests](https://github.com/vercel-labs/skills). Azure DevOps and private source support are in another doc I did not read.
- [cisco skill-scanner: static, bytecode, pipeline, correlation and behavioral analyzers have no key requirement; LLM judge can use Ollama or an OpenAI-compatible local server; VirusTotal, AI Defense and `--use-osv` need network](https://github.com/cisco-ai-defense/skill-scanner). The README does not explicitly promise zero network for the offline analyzers.
- [snyk/agent-scan needs a Snyk token and sends tool descriptions and skill content to Snyk's API; no local-only mode described](https://github.com/snyk/agent-scan). `--analysis-url` exists, but no on-prem service was documented.
- Earlier report caveat still applies: [Cisco's own test found rule scanning alone caught 7.7% at HIGH, 66.7% at MEDIUM or above with an LLM judge](https://github.com/cisco-ai-defense/skill-scanner).

### Inferences
- A monorepo on Azure DevOps Server, branch policies, and a CI job that packages each harness's format and runs the scanner is enough. An offline public-catalog sync would be a one-way import through the same review path.
- Local-LLM judge detection rates will differ from Cisco's figures and need their own test.
- Skills that assume web tools, public registries or specific model behavior will fail silently offline, so paired evaluation on the local model matters more.

### Gaps
- Signing and provenance for Agent Skills. No offline marketplace product beyond Git was found.
- Compatibility test of skills across harnesses with local models.

## Test: which tools run offline, and what feeds do they need?

### Takeaway
Deterministic tools work well offline, but their value decays unless feeds are imported regularly. LLM-based tools can use local endpoints, but capability varies and red-team coverage shrinks. SonarQube's free edition lacks PR decoration, which undercuts PR gating.

### Cited Findings

| Tool | License and version | Local LLM | External feed or download | Disable and loss |
|---|---|---|---|---|
| [Playwright Test Agents](https://playwright.dev/docs/test-agents) | Apache-2.0; v1.65.0 | [Loops: vscode, claude, codex, opencode; no provider named](https://playwright.dev/docs/test-agents) | [Browser binaries from Microsoft CDN](https://playwright.dev/docs/browsers) | [`PLAYWRIGHT_DOWNLOAD_HOST`, `PLAYWRIGHT_BROWSERS_PATH`](https://playwright.dev/docs/browsers) for internal mirror |
| [DeepEval](https://github.com/confident-ai/deepeval) | Apache-2.0; v4.2.4 | [Custom models; some metrics local NLP; Confident AI optional](https://github.com/confident-ai/deepeval) | Metric model downloads not checked | Telemetry variable not confirmed from docs |
| [promptfoo](https://github.com/promptfoo/promptfoo) | MIT; 0.125.1; OpenAI acquisition announced | [Ollama provider](https://www.promptfoo.dev/docs/red-team/troubleshooting/inference-limit/) | [Remote generation](https://www.promptfoo.dev/docs/red-team/troubleshooting/data-handling) | [`PROMPTFOO_DISABLE_TELEMETRY`, `PROMPTFOO_DISABLE_REMOTE_GENERATION`, `PROMPTFOO_DISABLE_UPDATE`, `PROMPTFOO_DISABLE_SHARING`, `PROMPTFOO_SELF_HOSTED`](https://www.promptfoo.dev/docs/faq). Docs: not a network isolation guarantee. Harmful, Bias, Medical, Financial and Remote-Only plugins lost. [Third party reports an acknowledgment event to `r.promptfoo.app` even when disabled](https://particula.tech/blog/promptfoo-self-hosted-red-teaming-alternatives) |
| [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) | MIT; 0.3.288 | [Hugging Face, vLLM, Ollama, llama-cpp, SGLang, OpenAI-compatible](https://inspect.aisi.org.uk/providers.html) | Datasets are task-dependent (not checked) | None found |
| [garak](https://github.com/NVIDIA/garak) | Apache-2.0; v0.17.1 | [Ollama generator or OpenAI-compatible; blogs differ on flags](https://hexisanoob.gitbook.io/hexisanoob/large-language-models/master-guide-to-ai-red-teaming-using-nvidia-garak) | Offline model data not confirmed | Check `--help` |
| [PyRIT](https://github.com/microsoft/PyRIT) | MIT; v1.1.0 | [`OpenAIChatTarget` replaces a dedicated Ollama target](https://azure.github.io/PyRIT/blog/2025-03-03) | Not checked | [Needs `initialize_pyrit_async` first](https://app.stationx.net/articles/pyrit-microsoft) |
| [Schemathesis](https://github.com/schemathesis/schemathesis) | MIT; v4.33.1 | Not needed | None by design (inference) | Telemetry unverified; docs URL 404 |
| [Pact](https://docs.pact.io/telemetry) | Apache-2.0 or MIT per language | Not needed | Broker self-hosted (not checked) | [`PACT_DO_NOT_TRACK=true`](https://docs.pact.io/telemetry) |
| [k6](https://github.com/grafana/k6) | AGPL-3.0; v2.3.0 | Not needed | None for core | [Usage report on by default; `K6_NO_USAGE_REPORT`](https://grafana.com/docs/k6/latest/set-up/usage-collection/) |
| [PIT](https://github.com/hcoles/pitest), [Stryker.NET](https://github.com/stryker-mutator/stryker-net) | Apache-2.0; 1.30.2; 5.0.0 | Not needed | Maven and NuGet mirrors | Telemetry not checked |
| [SonarQube](https://github.com/SonarSource/sonarqube) | Community LGPL-3.0; 26.11; [Server per instance by LOC](https://www.sonarsource.com/plans-and-pricing/) | Not needed | Offline plugin and license flow not confirmed | [Telemetry on by default; `sonar.telemetry.enable=false`](https://docs.sonarsource.com/sonarqube-server/instance-administration/system-functions/telemetry). [PR analysis and decoration start at Developer Edition](https://community.sonarsource.com/t/pullrequest-decoration-quality-gate/46232) (forum). A search summary of forum posts says Community covers the main branch only; the current edition matrix was not read. [ADO Server integration documented](https://docs.sonarsource.com/sonarqube/9.8/devops-platform-integration/azure-devops-integration) in an older doc version |
| [Semgrep CE](https://github.com/semgrep/semgrep), [Opengrep](https://github.com/opengrep/opengrep) | LGPL-2.1; v1.184.0, v1.33.1 | Not needed | [Rule packs; Opengrep can fetch at runtime; vendor to a local path](https://mhchlagou.github.io/lintel/0.2.4/scanners/opengrep/) | `--metrics=off` per that doc. [Aikido says Semgrep's change restricted community rules](https://aikido.dev/blog/launching-opengrep-why-we-forked-semgrep) (Aikido backs the fork) |
| [Trivy](https://github.com/aquasecurity/trivy) | Apache-2.0; v0.76.2 | Not needed | [DB, Java DB, checks, VEX Hub, Maven](https://trivy.dev/docs/latest/guide/advanced/air-gap/). [DB built every 6 hours; metadata interval 24 hours](https://github.com/aquasecurity/trivy-db) | [Mirror with `--db-repository`, `--java-db-repository`, `--checks-bundle-repository`](https://trivy.dev/docs/latest/guide/advanced/self-hosting/); `--skip-version-check --disable-telemetry`; `--offline-scan`; [`--skip-db-update`](https://trivy.dev/docs/v0.56/advanced/air-gap/). Embedded checks freeze at release |
| [ZAP](https://github.com/zaproxy/zaproxy) | Apache-2.0; v2.17.0 (2025-12-15) | Not needed | [Add-on marketplace; offline add-on load](https://www.zaproxy.org/docs/desktop/ui/dialogs/manageaddons) | [Check to `cfu.zaproxy.org`; `-silent`, but third-party add-ons may ignore it](https://www.zaproxy.org/faq/what-calls-home-does-zap-make); [project blog admits a silent-mode bug](https://www.zaproxy.org/blog/2025-10-21-zap-was-not-so-silent/) |
| [Nuclei](https://github.com/projectdiscovery/nuclei) | MIT; v3.11.1 (2026-08-08) | `-ai` is not local | Templates | [`-duc`, `DISABLE_NUCLEI_TEMPLATES_*`, `NUCLEI_TEMPLATES_DIR`, `-ni`](https://docs.projectdiscovery.io/opensource/nuclei/running). Interactsh and `-ai` lost |
| [Strix](https://github.com/usestrix/strix) | Apache-2.0; v1.7.0 | [`LLM_API_BASE` for Ollama or LM Studio; recommended models are hosted](https://github.com/usestrix/strix) | Sandbox image pulled on first run | Mirror image; telemetry not confirmed; local-model capability untested |

### Inferences
- A one-way feed bundle should be scheduled at least daily for Trivy, and weekly or better for rules and templates. These cadences are my proposal.
- Test Agents can reach a local model through the OpenCode loop. I found no evidence of how well it works, and browser planning needs strong tool calling.
- SonarQube Community is insufficient for PR gates. Developer Edition or above is the real cost line.
- Local models will weaken AI pentest and red-team generation, so keep deterministic checks as the gate.

### Gaps
- SonarQube offline license and plugin path; Developer vs Enterprise rule for multiple ALM setups ([forum sources conflict](https://community.sonarsource.com/t/pr-decoration-possible-for-sonarqube-and-azure-devops-services/33825)).
- Telemetry for DeepEval, Schemathesis, Strix, garak, Inspect, PIT and Stryker not confirmed from primary docs. `--skip-db-update` behavior after DB expiry unverified.

## Platform: Kubernetes, sandbox, workspaces, observability, identity, registries and gateways

### Takeaway
RKE2 is the most Windows-friendly documented option with air-gap and Windows workers, but agent sandboxes must be on Linux. Coder claims all features work offline with offline license validation. Langfuse OSS can run dark while Enterprise's telemetry cannot be disabled, so ask for an offline arrangement. Nexus Community's new caps make it a poor default.

### Cited Findings

Kubernetes:

| Distribution | License and version | Air-gap | Windows | Loss or cost |
|---|---|---|---|---|
| [RKE2](https://github.com/rancher/rke2) | Apache-2.0; v1.37.2+rke2r1 | [**Documented**: private registry, manual images, Hauler, or embedded mirror; SELinux RPMs staged by hand](https://docs.rke2.io/install/airgap) | [**Documented**: Windows Server 2019 and 2022 workers, Calico or Flannel](https://docs.rke2.io/install/windows_airgap) | Support subscription optional (not checked) |
| OpenShift | Commercial | [**Documented**: mirror with oc-mirror v2 from a connected system; `oc adm release mirror` deprecated in 4.22; small mirror registry included](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/disconnected_environments/installing-mirroring-installation-images) | [WMCO manages Windows nodes](https://huggingface.co/datasets/boricua/qna-ocp-4.15/viewer) (4.15-era secondary); disconnected steps not verified | Price not found |
| [Talos](https://github.com/siderolabs/talos) | MPL-2.0; v1.14.5 | [**Documented**, with NTP, registry mirrors and on-prem Image Factory; self-hosted Discovery Service needs a Sidero license](https://docs.siderolabs.com/talos/v1.12/platform-specific-installations/air-gapped) | Unconfirmed; assumed Linux only | [gVisor and Kata as extensions baked into images](https://oneuptime.com/blog/post/2026-03-03-list-available-system-extensions-for-talos-linux/markdown) (secondary) |

Sandbox and workspaces:
- [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) (Apache-2.0, v1.0.10) installs from manifests or Helm, uses `agents.x-k8s.io/v1beta1`, delegates to gVisor or Kata through RuntimeClass, needs Gateway API CRDs for some client connections, and claims no cloud dependency.
- [Kata needs a virtualization backend; a VM node needs nested virtualization](https://northflank.com/blog/what-are-kata-containers). [gVisor Systrap does not need hardware virtualization](https://vercel.com/i/kata-containers). [RKE2 has a Kata add-on guide](https://documentation.suse.com/cloudnative/rke2/latest/de/add-ons/kata.html).
- [Firecracker](https://github.com/firecracker-microvm/firecracker) (Apache-2.0, v1.17.0) was not re-audited.
- [Coder](https://github.com/coder/coder) (AGPL-3.0, v2.38.6): [**Documented**: all features work air-gapped; license keys validated offline; Terraform mirror required; `CODER_TELEMETRY_ENABLE=false`, `CODER_UPDATE_CHECK=false`, `CODER_DISABLE_TEMPLATE_BUILDER`, STUN off; external Postgres; VS Code Remote-SSH clients need Microsoft update hosts](https://coder.com/docs/install/airgap).
- [Coder pricing lists Community and Premium at up to 5 active agents, AI Premium unlimited; OIDC group sync, audit, HA under Premium; price by quote](https://coder.com/pricing). The table extraction lost check marks, so tier mapping is approximate.
- [Coder Agents GA on 2026-09-09 claims air-gapped support as an enterprise offering](https://coder.com/blog/coder-agents-ga) (vendor; models and Git providers unnamed).

Observability:
- [Langfuse](https://github.com/langfuse/langfuse/blob/main/LICENSE) (MIT outside `ee`, © ClickHouse, v4.76.0) [needs Postgres, ClickHouse, Redis or Valkey, S3-compatible storage; "Internet access is optional"](https://langfuse.com/self-hosting).
- [Enterprise-only: project RBAC, protected prompt labels, retention, audit logs, masking, SCIM and others](https://langfuse.com/self-hosting/license-key).
- [OSS can use `TELEMETRY_ENABLED=false`; Enterprise telemetry for license compliance cannot be disabled and goes through PostHog Cloud](https://langfuse.com/self-hosting/security/telemetry). [Government keys send none](https://langfuse.com/self-hosting/license-key). **Air-gapped Enterprise needs vendor confirmation.**

Identity and policy:
- [OpenBao](https://github.com/openbao/openbao) MPL-2.0 v2.7.2; [SPIRE](https://github.com/spiffe/spire) Apache-2.0 v1.16.0.
- [OPA](https://github.com/open-policy-agent/opa) Apache-2.0 v1.22.2: [`opa run` reports by default; `--disable-telemetry`; `OPA_TELEMETRY_SERVICE_URL` overrides the target](https://openpolicyagent.org/docs/latest/privacy/).

Registries and mirrors:

| Tool | License | Finding | Limit |
|---|---|---|---|
| [Harbor](https://github.com/goharbor/harbor) | Apache-2.0; v2.14.6 | [Installer and scanner adapters per docs](https://goharbor.io/docs/2.14.0/install-config/) | Offline installer and OCI-only scope not confirmed on the fetched page |
| [Nexus Community](https://help.sonatype.com/en/ce-onboarding.html) | Sonatype CE terms | [Caps 40,000 components, 100,000 requests per day since 3.87.0](https://help.sonatype.com/en/nexus-repository-tiers-and-feature-availability.html); [new components cannot be added until usage is back under both caps](https://www.cloudrepo.io/articles/sonatype-nexus-repository-pricing-guide) (competitor source). A search summary said proxy repositories count; I could not pin that to a page, so verify | Grace period and enforcement date conflict across sources |
| [Artifactory](https://docs.jfrog.com/security/docs/xray-and-jfrog-external-db-sync) | Commercial | [Xray offline DB packages; JFConnect offline entitlement](https://docs.jfrog.com/security/docs/xray-and-jfrog-external-db-sync) | [Third-party prices conflict](https://www.vendr.com/marketplace/jfrog); Curation air-gap not confirmed |
| [Azure Artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/upstream-sources?view=azure-devops-server) | Part of Azure DevOps Server | Upstream packages are saved to the feed and stay even if upstream goes away; a failing upstream stops metadata refresh for that protocol | Needs a connection for upstream; collaborator role saves new packages |
| [Verdaccio](https://github.com/verdaccio/verdaccio), [devpi](https://github.com/devpi/devpi), [BaGetter](https://github.com/bagetter/BaGetter) | MIT; no license detected by GitHub; MIT | Lightweight per-ecosystem options | BaGetter's last release was 2025-12-07 and last push 2026-04-08, so its momentum is low |

MCP and LLM gateways:
- [agentgateway](https://github.com/agentgateway/agentgateway) (Apache-2.0, v1.7.0): [standalone or Kubernetes](https://agentgateway.dev/docs/); [JWT, API keys, OAuth, CEL RBAC, OTel, Inference Routing for self-hosted models](https://raw.githubusercontent.com/agentgateway/agentgateway/main/README.md). Ollama or vLLM not named in the README.
- [Agent Router](https://github.com/theagentrouter/agent-router) (Apache-2.0, v1.2.1) and [IBM ContextForge](https://github.com/IBM/mcp-context-forge) (Apache-2.0, v1.1.0) not re-audited.
- [Microsoft mcp-gateway](https://github.com/microsoft/mcp-gateway) (MIT, no releases): cloud path needs Entra, Cosmos and AKS; local path is anonymous dev mode with Redis; on-prem production is undocumented; MCP 2026-07-28 only.

### Inferences
- Windows nodes are useful for .NET Framework build jobs, but agent sandboxes should run on Linux nodes. gVisor is Linux-oriented by general knowledge; I did not find a source specifically covering Windows containers. Agents that need Windows need a VM-based workspace model, for example Coder with a Windows template, with weaker density and different controls.
- VMware-hosted nodes need nested virtualization enabled for Kata, so that decision belongs with the infrastructure team early.
- Package mirror pattern for air-gap: a connected DMZ with quarantine and scanning, then one-way transfer to internal feeds; agents can reach only those feeds. This is my design proposal.

### Gaps
- OpenBao and SPIRE outbound behavior; Harbor offline installer and proxy scope; ClickHouse sizing; OpenShift prices, sandboxed containers and disconnected WMCO.
- Azure Local and AKS Arc were not researched. [Coder Azure Repos auth page returned 404](https://coder.com/docs/admin/external-auth/azure-repos).

## Commercial coding agents: which truly support air-gap with Azure DevOps Server?

### Takeaway
I found no vendor documentation stating both air-gap support and Azure DevOps Server support. Tabnine, Factory Droid and GitLab Duo document air-gap; GitLab implies moving SCM. Cursor and Amp do not meet the need on the evidence found, and Continue is frozen. All candidates need written vendor confirmation.

### Cited Findings

| Offering | Air-gap | Local models | Azure DevOps Server | Verdict |
|---|---|---|---|---|
| Tabnine Enterprise | **Documented**: [private install on Kubernetes; "completely air-gapped environment"](https://docs.tabnine.com/main/welcome/readme/architecture/deployment-options.md). [Vendor blog says self-hosted GA including governance](https://context.tabnine.com/2026/05/06/the-coding-agent-that-runs-in-your-environment-fully/) (search summary; page later returned 522) | [Self-Managed Models page](https://docs.tabnine.com/main/sitemap.md) | **Not documented**: [CLI Git integrations are GitHub Actions, Bitbucket, GitLab CI](https://docs.tabnine.com/main/sitemap.md); [Context Engine lists "Git repositories" and Perforce with no named hosts](https://docs.tabnine.com/main/administering-tabnine/managing-your-team/context-engine/data-sources.md) | Candidate; confirm generic Git and Azure DevOps Server |
| Factory Droid | **Documented** for [airgap builds under Custom Deployment via sales; no Factory requests; lost: accounts, Factory Router, sharing, Slack, readiness reports, built-in web search and fetch, auto-update, hosted analytics](https://docs.factory.com/enterprise/airgapped-deployment.md). GA status not stated. [Federal page claims air-gapped IL4 and IL5 inheritance](https://factory.ai/industries/federal) | [BYOK custom endpoints required](https://docs.factory.com/enterprise/airgapped-deployment.md) | **Not documented**; doc says Git remotes, no provider named | Candidate; CLI-level fit with generic Git remotes |
| Factory Private | **Not an air-gap claim** in the [primary announcement: Private Preview, Factory operates control plane by default](https://factory.com/news/factory-private). [Secondary coverage claims air-gapped networks](https://hermes-ai.net/news/factory-private-brings-autonomous-coding-droids-inside-enterprise-networks/). **Conflict** | Not stated | Not mentioned | Do not plan on it yet |
| GitLab Self-Managed Duo | **Documented**: [offline license needs Duo Agent Platform Self-Hosted add-on; fully self-hosted gateway and LLMs; Agent Platform GA from 18.8](https://docs.gitlab.com/administration/gitlab_duo_self_hosted/); [offline deployment needs 18.9+, offline cloud license, vLLM](https://docs.gitlab.com/ee/administration/gitlab_duo_self_hosted/offline_deployment/); [19.0 adds open models](https://about.gitlab.com/blog/more-ai-models-for-duo-agent-platform-self-hosted/) | Yes | **Not mentioned**; agents work on GitLab projects | Real air-gap, but a second SCM or a migration |
| GitLab Dedicated | Not researched in depth; [the page's offline text is about self-hosted, not Dedicated](https://docs.gitlab.com/administration/gitlab_duo_self_hosted/) | Unknown | No | Out of scope for "no cloud" unless confirmed |
| Sourcegraph Amp | **Unconfirmed**: [no deployment doc found; Amp spun out in late 2025](https://innfactory.ai/en/ai-harness/amp/) (third-party) | No BYOK evidence | None | Exclude until shown otherwise |
| Windsurf / Devin Desktop | **Unconfirmed**: [renamed June 2026; third parties cite Cloud, Hybrid, Self-Hosted](https://tech-insider.org/windsurf-vs-cursor-2026/); [FedRAMP notes conflict](https://github.com/COG-GTM/federal_security_compliance/pull/4). [The air-gap article does not mention it](https://intuitionlabs.ai/articles/enterprise-ai-code-assistants-air-gapped-environments) | Not found | None | Ask vendor; do not plan on it |
| Continue | **Possible, but frozen**: [README says read-only; final 2.0.0 removed telemetry and auth](https://github.com/continuedev/continue). Last commit 2026-07-21. [Third party cites an offline guide (VSIX, local model)](https://intuitionlabs.ai/articles/enterprise-ai-code-assistants-air-gapped-environments); that guide URL now 404s. [Cursor purchase reported by secondary sources](https://www.digitaltoday.co.kr/en/view/73581/cursor-acquires-open-source-coding-assistant-continue) | Yes via config | Repo-agnostic IDE tool; no dedicated integration found | Apache-2.0 snapshot only; no standardization |
| Cursor | **No**: [self-hosted agents move execution only; planning stays in Cursor's cloud](https://cursor.com/blog/self-hosted-cloud-agents); [press: not an offline deployment](https://thenewstack.io/cursor-self-hosted-coding-agents/) | No air-gap path | [Named private connections are GHES and GitLab; ADO not named](https://cursor.com/docs/cloud-agent/self-hosted) | Exclude. [One comparison claims air-gap by contract](https://tech-insider.org/windsurf-vs-cursor-2026/); no primary support |
| GitHub Copilot | **No**: [third party calls it cloud-only](https://intuitionlabs.ai/articles/enterprise-ai-code-assistants-air-gapped-environments) | No | Out of scope | Exclude |
| Coder Agents | [Vendor claim of air-gap](https://coder.com/blog/coder-agents-ga) | Models unnamed | Unconfirmed | Check; pairs with Coder workspaces |

OSS fallback, for comparison:
- [OpenCode](https://github.com/anomalyco/opencode) (MIT): [a third-party source audit of v1.18.34 lists no offline flag and 17 outbound paths, 4 with no setting; `OPENCODE_DISABLE_MODELS_FETCH`, `_AUTOUPDATE`, `_SHARE`, `_LSP_DOWNLOAD` exist; catalog host changed in a patch release; offline PR closed unmerged 2026-10-10](https://particula.tech/blog/opencode-offline-mode-air-gapped). [Zen provider autoloads free hosted models](https://particula.tech/blog/opencode-offline-mode-air-gapped), so explicitly set `enabled_providers`.
- [Goose](https://github.com/aaif-goose/goose) (Apache-2.0): [supports Ollama with env vars](https://www.dash0.com/guides/observing-goose-on-ollama-with-opentelemetry); telemetry variable is [`GOOSE_TELEMETRY_OFF` in one source](https://mintlify.com/block/goose/guides/environment-variables) and [`GOOSE_TELEMETRY_ENABLED` in another](https://last9.io/docs/integrations/goose/). **Conflict**.
- [Tabby](https://tabby.tabbyml.com/docs/administration/usage-collection/): usage collection on by default, `TABBY_DISABLE_USAGE_COLLECTION=1`; docs from 2024; Azure DevOps and 2026 status unverified.

### Inferences
- The likely path for any commercial agent is that it treats Azure DevOps Server as a plain Git remote. Clone and push would work, while PR, work item and pipeline operations become your own MCP or CLI layer. That is why this layer depends on the Azure DevOps researcher's integration findings.
- OSS harnesses are realistic only with default-deny egress, because a model-driven shell can reach anything the network allows.
- Ask each vendor for: written Azure DevOps Server support, offline license terms, update mechanism, full egress list, and a test build for a default-deny lab.

### Gaps
- No sales confirmation was obtained. Commercial pricing for Tabnine, Factory custom deployment and GitLab's add-on is not public or not found.
- [Search of Sourcegraph, Tabnine and others found no Azure DevOps Server plus air-gap statement](https://intuitionlabs.ai/articles/enterprise-ai-code-assistants-air-gapped-environments), but absence in my search is not proof of no support.

## Telemetry traps: what phones home, how to disable it, and which hosts to expect?

### Takeaway
Many of these tools report by default, and flags are often incomplete. Treat flags as hygiene and default-deny egress as the actual control. A blocked endpoint can also cause failures, as with license checks on Sourcegraph and possibly Langfuse Enterprise.

### Cited Findings

| Tool | Default behavior and destination | Switch | Trap |
|---|---|---|---|
| [Onyx](https://docs.onyx.app/security/self_hosted/faq) | Anonymous telemetry; GitHub for catalogs and release notes | `DISABLE_TELEMETRY=true` on every service; `ONYX_AIRGAPPED=true` | Two settings; every service |
| [Outline](https://raw.githubusercontent.com/outline/outline/main/.env.sample) | Update check with anonymized stats | `ENABLE_UPDATES=false` | Endpoint not named |
| [Wiki.js](https://docs.requarks.io/install/requirements/internet) | Periodic service contact, plus telemetry | Setting not confirmed for v3 | Some modules need internet |
| [Dify](https://docs.dify.ai/en/self-host/configuration/environments) | `updates.dify.ai` | `CHECK_UPDATE_URL` empty | Marketplace is separate |
| [Meilisearch](https://www.meilisearch.com/docs/resources/help/telemetry.md) | Anonymous analytics | `MEILI_NO_ANALYTICS=true` | |
| [LangSmith](https://docs.langchain.com/langsmith/self-host-egress) | `beacon.langchain.com` | Offline license only | Billing telemetry cannot be disabled |
| [Langfuse](https://langfuse.com/self-hosting/security/telemetry) | PostHog Cloud | `TELEMETRY_ENABLED=false` (OSS) | Enterprise cannot disable |
| [Sourcegraph](https://handbook.sourcegraph.com/departments/data-analytics/how-to-airgap-disable-telemetry) | Pings and update checks; license validation | License tag; `update.channel: none`; [`disableNonCriticalTelemetry`](https://docs.sourcegraph.com/@4.4/admin/pings) | May shut down without tag |
| [Coder](https://coder.com/docs/install/airgap) | Telemetry, updates, `registry.coder.com`, `registry.terraform.io`, `code-server.dev`, `open-vsx.org`, `v2-licensor.coder.com` | `CODER_TELEMETRY_ENABLE=false`, `CODER_UPDATE_CHECK=false` | VS Code client needs its own hosts |
| [SonarQube](https://docs.sonarsource.com/sonarqube-server/instance-administration/system-functions/telemetry) | Daily payload to two endpoints | `sonar.telemetry.enable=false` | Scanner data relayed through server |
| [Trivy](https://trivy.dev/docs/latest/guide/advanced/air-gap/) | `check.trivy.dev`; `ghcr.io`, `mirror.gcr.io`, Maven, GitHub | `--skip-version-check`, `--disable-telemetry`, `--offline-scan` | Must set both skip flags |
| [promptfoo](https://www.promptfoo.dev/docs/faq) | PostHog and `r.promptfoo.app` | See test table | Docs disclaim isolation |
| [k6](https://grafana.com/docs/k6/latest/set-up/usage-collection/) | Usage report | `K6_NO_USAGE_REPORT=true` | Extension subcommands need the env var |
| [Pact](https://docs.pact.io/telemetry) | Telemetry | `PACT_DO_NOT_TRACK=true` | Set in JVM surefire too |
| [OPA](https://openpolicyagent.org/docs/latest/privacy/) | Version report on `opa run` | `--disable-telemetry` | Flags vary by version |
| [ZAP](https://www.zaproxy.org/faq/what-calls-home-does-zap-make) | `cfu.zaproxy.org` | `-silent` | Past bug; third-party add-ons |
| [Nuclei](https://docs.projectdiscovery.io/opensource/nuclei/running) | Template and version checks | `-duc`, `DISABLE_NUCLEI_TEMPLATES_*` | `ENABLE_CLOUD_UPLOAD` must stay off |
| [Playwright](https://playwright.dev/docs/browsers) | Microsoft CDN at install | `PLAYWRIGHT_DOWNLOAD_HOST` | Install-time only as documented |
| [vercel skills](https://github.com/vercel-labs/skills) | Usage and security audit | `DISABLE_TELEMETRY=1` or `DO_NOT_TRACK=1` | Also disables audit |
| [OpenCode](https://particula.tech/blog/opencode-offline-mode-air-gapped) | `models.opencode.ai` hourly; package channels; hosted Zen | See above | Hosted provider autoloads; no global offline flag |
| [Goose](https://mintlify.com/block/goose/guides/environment-variables) | Anonymous usage | Name conflict | Verify in source |
| [Tabby](https://tabby.tabbyml.com/docs/administration/usage-collection/) | Usage stats | `TABBY_DISABLE_USAGE_COLLECTION=1` | Old doc |
| [Ollama](https://d-central.tech/local-ai-telemetry-audit/) | Desktop tray update check; headless silent | Run server binary | Vendor blog |
| [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | None claimed | None needed | Self-reported |
| [Continue 2.0.0](https://github.com/continuedev/continue) | Telemetry removed | None needed | Frozen project |

### Inferences
- Run an acceptance test: each tool in default-deny with DNS logging, through start, normal use and upgrade. Repeat after each upgrade, because OpenCode's catalog host [changed in a patch](https://particula.tech/blog/opencode-offline-mode-air-gapped).
- Pin these settings in Helm values and fail CI if removed.

### Gaps
- Unverified: BookStack, Docmost, RAGFlow, Temporal, Harbor, OpenBao, SPIRE, Argo, Serena, Repomix, Inspect, garak, Strix. Absence here means not verified, not clean.

## Staffing: what is the published evidence and a realistic platform-team size?

### Takeaway
No published case study of this self-hosted stack gives a team size. The best general benchmark is 2 to 6% of headcount for central developer productivity, but it excludes SRE, cloud operations and security, and on-prem Kubernetes estimates are soft. My estimate is 2 to 3 FTE for a pilot and 5 to 7 FTE for a production multi-department stack, plus 1 to 2 FTE for a true air-gap, excluding GPU operations.

### Cited Findings
- [DX Q1 2026 benchmark of 39 companies: 2 to 6% of headcount to central developer productivity, average 4.7%, falling above 1,000 engineers. It **excludes SRE, cloud operations and security teams**](https://newsletter.getdx.com/p/devprod-headcount-benchmarks-q1-2026) (search summary of the DX newsletter). DX is a vendor in this market.
- [A separate DX report page gives 19% for companies under 1,000 engineers](https://getdx.com/report/headcount-benchmarks-devprod-teams/), with scope unclear. **Conflict**, probably a definition difference.
- [One site promotes a 1:8 to 1:12 ratio](https://platformengineeringcost.com/team-structure); [a forum thread cites 20:1 at elite teams](https://tianpan.co/forum/t/20-1-developer-to-platform-engineer-ratio-what-are-elite-platform-teams-actually-doing-differently/3881). They conflict, and I found no research basis for either.
- Kubernetes operations: [usually 1.5 to 2 FTE, about 0.5 per cluster across ten clusters for control plane staffing](https://hidora.io/en/blog/kubernetes-self-hosted-vs-managed), [six to eight weeks to build then 0.5 to 1 FTE](https://enum.co/guides/managed-kubernetes-vs-in-house-cost-comparison), and [a Gcore model assuming 3 engineers self-managed vs 1 managed](https://gcore.com/learning/kubernetes-tco-comparison). All vendor or practitioner estimates.
- [Platform9's TCO report](https://platform9.com/media/Platform9_Kubernetes-TCO-Report.pdf) is marketing.
- Temporal: [docs call staffing and skills a significant ongoing cost](https://docs.temporal.io/self-hosted-guide/production-checklist) with no number.
- Air-gap: [internal mirrors, licenses that do not call home, and a process to move docs, models and updates across the gap are ongoing work; no headcount given](https://context7.com/guides/self-hosted-ai-coding-assistant) (vendor guide).

### Inferences
**All numbers below are my estimates, not published data.** They assume about 100 to 300 developers, one or two clusters, and no GPU operations.

| Workstream | Pilot FTE | Production FTE |
|---|---|---|
| Kubernetes, storage, ingress, upgrades | 0.5 to 1 | 1 to 1.5 |
| Temporal or Dapr, Postgres | 0 to 0.25 (pipelines first) | 0.5 to 1 |
| Wiki, search sidecar, retrieval, connectors | 0.5 | 1 to 1.5 |
| Gateway, sandbox, identity, secrets, policy | 0.5 | 1 |
| Observability and cost attribution | 0.25 | 0.5 |
| Test toolchain, scanners, skills registry | 0.25 | 0.5 to 1 |
| **Total** | **2 to 3** | **5 to 7** |
| Air-gap: feed transfer, mirrors, license and upgrade rehearsals | +0.5 | +1 to 2 |

- Sanity check: 4.7% of 150 engineers is about 7, but DX excludes infrastructure and security work that this stack includes, so the benchmark is not like for like. The production number fits best at 150 or more developers, and may be low if the team also owns security operations. At under 50 developers, a full stack is unlikely to pay back; prefer a reduced set (BookStack, Git skills, pipelines, RKE2 or a VM runner, Langfuse, gateway).
- Coverage matters: at under 4 people, on-call for a stateful workflow engine is fragile.
- Budget lines beyond labor: Onyx at $20 per user per month is $36,000 per year for 150 users, and SonarQube, Coder Premium and a repository manager are quote-based.

### Gaps
- No time-tracking or named on-prem case study. Conference talks from regulated firms would be the next source.
- GPU and model-serving staffing belongs to the models researcher and should be added to the totals.
