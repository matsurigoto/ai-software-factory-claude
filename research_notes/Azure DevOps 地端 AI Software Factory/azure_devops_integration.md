# AI Software Factory on Azure DevOps with an On-Premises Preference (as of 2026-11-07)

Evidence legend: "Docs" = vendor documentation read this session. "Source" = code in the repo read this session. "Secondary" = third-party or search-summary evidence only. Repo stars, licenses and release dates were read from the GitHub API on 2026-11-07.

## Azure DevOps Server: release, lifecycle, gaps against Services, GHAS

### Takeaway
Azure DevOps Server is actively maintained: the 25H2-generation release is in support and patched monthly, and a 26H2 RC shipped on 2026-10-27. The core ALM surface (Pipelines, Repos, Boards, Test Plans, Wiki, webhooks, PATs) is there, but almost every new AI and security feature from Microsoft is limited to Services, and `az devops` does not work on Server at all.

### Cited Findings
**Version and lifecycle**
- Azure DevOps Server follows the Modern Lifecycle Policy. Support start is 2025-12-09, and retirement shows "In Support" with no end date — [Microsoft Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/azure-devops-server)
- Latest listed patch is Patch 9 (2026-10-13, security fixes). Patches have been roughly monthly since Patch 1 (2026-02-10). RTW was 2025-12-09 and RC was 2025-10-07. Direct upgrade is supported from 2022, 2020, 2019 and TFS 2015 or newer — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)
- The product is unnumbered, but release notes also call this release "25H2" (Patch 1 and Patch 5 entries) — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)
- 26H2 RC was announced 2026-10-27 with an updated Boards UI, target branches in the PR branch dropdown, improved pipeline run debugging and a new Test Run Hub. No RTW date is given. The post does not mention AI, Copilot, MCP or Advanced Security. RC-to-RTW direct upgrade will be supported — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599)
- Blog commenters complain about missing MFA and SAML SSO, and one reports a TF400367 error on the RC. This is unverified user feedback — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599)
- TFS 2017 Extended Support ends 2027-01-12 (search summary of Microsoft's post) — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599)
- A third-party tracker lists 2022 Update 2 extended support ending 2028-01-11. Its snapshot looked stale (latest patch shown was Patch 3) — [endoflife.date](https://endoflife.date/azure-devops-server)

**Features confirmed on Server (docs carry the "Azure DevOps Server" applies-to label)**
- Pipelines environments, approvals and checks, plus commit and work item traceability per deployment. Environment roles differ slightly from Services — [Environments](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops-server)
- Patch 7 fixed Manual Validation tasks getting stuck after approval — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)
- Web Hooks service hooks — [Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops-server)
- Test Plans can use YAML pipelines for configuration and automated test execution. Publish Code Coverage v2 supports diff coverage PR comments. Repo and PR Markdown previews render Mermaid. Sparse checkout requires agent v3.248.0 or v4.248.0 — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)
- Work item comments are capped at 1,000 per item through the REST API. This limit matters for agents that post progress to work items — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)
- Extensions on Server install from a Local Gallery (`http://{server}/_gallery`) — [Install Search](https://learn.microsoft.com/en-us/azure/devops/project/search/install-configure-search?view=azure-devops-server)

**REST API**
- A community MCP's auth guide says on-prem releases cap the API version, with Server 2022 at `7.0` against `7.1` on Services. Tools then fail with "The requested REST API version of <x> is out of range for this server" — [Tiberriver256 auth guide](https://github.com/Tiberriver256/mcp-server-azure-devops/blob/main/docs/authentication.md)
- Microsoft states "several required API endpoints are not yet available for on-premises deployments" — [MS MCP FAQ](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/FAQ.md)

**CLI**
- "The Azure DevOps CLI extension works only with Azure DevOps Services (cloud). It doesn't support Azure DevOps Server (on-premises)." — [Azure DevOps CLI](https://learn.microsoft.com/en-us/azure/devops/cli/?view=azure-devops)

**Auth**
- Microsoft recommends Active Directory for on-prem user management and lists Windows authentication among supported methods. No source found describes Entra sign-in for Server — [Security overview](https://learn.microsoft.com/en-us/azure/devops/organizations/security/about-security-identity)
- PATs exist on Server, scoped to a collection and sent via HTTP Basic. IIS Basic Authentication must stay disabled or PATs fail. PAT lifecycle APIs, PAT policies and PAT audit content sit in Services-only doc sections — [PAT docs](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-server)
- Service principal and managed identity auth are not supported on Server (Backstage changelog, not a Microsoft statement) — [Backstage changelog](https://backstage.io/docs/releases/v1.15.0-next.1-changelog)
- **Live risk:** NTLM support was scheduled for removal from libcurl in September 2026. Git over HTTPS against Server fails for anyone relying on NTLM, and IIS "Negotiate" can silently fall back to NTLM. Microsoft points to Kerberos, or SSH where Kerberos is not feasible, and advises against downgrading Git — [Azure DevOps Blog, 2026-07-01](https://devblogs.microsoft.com/devops/upcoming-change-ntlm-removal-in-git-libcurl-impact-to-azure-devops-server-customers/)
- Services only: global PATs are being retired. The same page gives two dates, 2027-01-27 and 2026-12-01, so verify before relying on either — [PAT docs](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops)

**GitHub Advanced Security for Azure DevOps**
- Listed as Services only, not supported on Server (partner page) — [innFactory](https://innfactory.de/en/cloud/azure/products/github-advanced-security-devops/)
- A 2023 analyst note said support would not be added to Server — [Directions on Microsoft](https://www.directionsonmicrosoft.com/?p=123427)
- A community fork of the MS MCP returns "not supported" for Advanced Security tools on-prem — [Discussion #996](https://github.com/microsoft/azure-devops-mcp/discussions/996)

### Inferences
- Plan on REST API or MCP for all Server automation, since `az boards`, `az repos` and `az pipelines` cannot be assumed. Any agent recipe built on `az` must be rewritten.
- Pin a service account's Git auth to PAT over HTTPS, Kerberos or SSH before rollout. A PAT is sent as HTTP Basic, so it should not depend on NTLM, but no source confirmed this directly.
- Upgrading to 26H2 RTW is a feature decision for Boards and Test Plans, not an AI decision. The announcement offers nothing for AI.

### Gaps
- No Microsoft primary page was found that states GHAS is excluded from Server. Evidence is secondary, though consistent.
- Whether libcurl's NTLM removal actually shipped on schedule, and in which Git for Windows version, was not verified.
- The full 26H2 feature list was not retrieved. Visual Studio Magazine returned 403, and the Learn release notes page still shows Patch 9 and 25H2 content.
- REST API version cap for current Server (25H2 and 26H2) was not verified. Only the Server 2022 row was read.
- From background knowledge, not verified this session: Microsoft-hosted agents are unavailable to Server, so all agents are self-hosted.

## Official Azure DevOps MCP Server and community options for Server

### Takeaway
The official MCP server supports Services only, both local and remote, and Microsoft says it has no plans for on-prem. On Server you must use a community server, and Tiberriver256 is the most credible option found. Treat all of them as code to review, pin and test.

### Cited Findings
- `microsoft/azure-devops-mcp`: MIT, 2,083 stars, v2.11.0 (2026-10-22), pushed 2026-11-06 — [GitHub](https://github.com/microsoft/azure-devops-mcp)
- FAQ: "This MCP Server supports only Azure DevOps Services... We currently do not have plans to support Azure DevOps on-prem." — [FAQ](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/FAQ.md)
- Learn's MCP page is labelled "Azure DevOps Services" only. The remote endpoint is hosted by Azure DevOps, uses Entra and needs an Entra-backed org. The local server stays available for clients such as Claude Code, Cursor and Codex that cannot complete the remote auth flow — [Learn MCP overview](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview)
- Remote is `https://mcp.dev.azure.com/{organization}`, in preview and recommended, with new development focused there. Local runs `npx @azure-devops/mcp` on Node 20+ over stdio, and the repo says remote will eventually replace it — [README](https://github.com/microsoft/azure-devops-mcp)
- On 2026-03-12 maintainer Dan Hellem wrote "we do not have plans to enable this for on-prem" for now, and that the team would reconsider local support after the remote server is complete. Reasons cited were maintenance overhead, server lag on new APIs and auth changes. Users were still asking on 2026-11-04 — [Discussion #996](https://github.com/microsoft/azure-devops-mcp/discussions/996)

Community options:

| Project | License | Maintenance signal | Server auth | Notes |
|---|---|---|---|---|
| [Tiberriver256/mcp-server-azure-devops](https://github.com/Tiberriver256/mcp-server-azure-devops) | MIT | 399 stars, v0.1.54 (2026-11-02) | [PAT only](https://github.com/Tiberriver256/mcp-server-azure-devops/blob/main/docs/authentication.md) | README positions it for Server, and the docs cover API version caps |
| [mcp-devops-onpremise](https://pypi.org/project/mcp-devops-onpremise) | Apache-2.0 | PyPI 2.1.0, [source](https://github.com/zwitbaum/mcp-devops-on-prem) | NTLM and PAT | The only one found listing NTLM (stars not checked) |
| [AzureDevOpsServer.Mcp](https://www.nuget.org/packages/AzureDevOpsServer.Mcp) | Not checked | NuGet 0.2.1 latest; preview-level earlier | PAT | .NET 10; docs list Server 2019, 2020, 2022+ |
| [@ahouben fork](https://github.com/ahouben/azure-devops-mcp) | MIT | 2 stars, last push 2026-03-12 | PAT | Patched MS fork; 47 read tools passed on-prem, writes skipped, per [its author](https://github.com/microsoft/azure-devops-mcp/discussions/996). Looks stale |
| [conradprogrammer AzureDevOpsMCP](https://lobehub.com/pl/mcp/conradprogrammer-azuredevopsmcp) | Not checked | Listing only | Not checked | .NET 10, clone and build |

### Inferences
- Default for Server: Tiberriver256 with a dedicated low-scope PAT, and set the API version explicitly.
- The agent's Windows-integrated NTLM path (mcp-devops-onpremise) inherits the deprecation pressure above, so prefer PAT.
- An option with less exposure is a small in-house MCP that exposes only the factory's verbs against REST, since single-maintainer MCPs hold write credentials.

### Gaps
- None of the community servers was tested against Server 25H2. The fork's report is the author's own and against an unspecified server version.

## Microsoft agent features: what is cloud-only, what can run disconnected

### Takeaway
None of Microsoft's agent features was found to run against Azure DevOps Server, and most need a cloud link even from Services. The only disconnected path found is Copilot CLI in BYOK or air-gapped mode, and that is documented for GitHub Enterprise Server, not for Azure DevOps.

### Cited Findings
| Feature | Services | Server | Evidence |
|---|---|---|---|
| Copilot for Azure Boards (work item to draft PR) | GA Jan 2026 | Not documented | The user selects a "target GitHub repository", so code must be on GitHub, not Azure Repos — [Sprint 268](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/boards/sprint-268-update) |
| Custom agents and model choice in that flow | Yes | Not documented | [Feb 2026 blog](https://devblogs.microsoft.com/devops/azure-boards-integration-with-github-copilot-includes-custom-agent-support/); [Sprint 275](https://learn.microsoft.com/en-au/azure/devops/release-notes/2026/sprint-275-update) |
| Copilot code review for Azure Repos | Public preview (Aug 2026) | Not mentioned | Details below — [InfoQ](https://infoq.com/news/2026/09/copilot-code-review-azure-repos); [GitHub changelog](https://github.blog/changelog/2026-06-02-github-copilot-code-review-for-azure-repos-is-now-in-technical-preview/) |
| Azure SRE Agent connector | Yes | Not mentioned | Auth is Entra user OAuth or managed identity for live access, and managed identity or PAT for the wiki connector. It indexes wiki into a vector index with 24-hour sync, in an Azure-hosted service — [SRE connector](https://learn.microsoft.com/en-us/azure/sre-agent/ado-connector) |
| Remote MCP | Preview | No | [Learn MCP overview](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview) |
| Foundry | Client of remote MCP; BYOK target in ado-aw | Not applicable | [Learn MCP overview](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview); [ado-aw engine docs](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md) |

- Copilot code review details: it bills per review through Azure, leaves Comment reviews only and cannot approve or block. It is limited to repos of 10 GB or less and PRs of 100 or fewer files, and **self-hosted agents are not supported**. Budgets notify but do not stop reviews — [InfoQ](https://infoq.com/news/2026/09/copilot-code-review-azure-repos)
- GHES docs say most Copilot features need a GitHub Enterprise Cloud presence, while Copilot CLI can be configured for disconnected or air-gapped GHES without a Copilot license — [About Copilot on GHES](https://docs.github.com/en/enterprise-server@latest/copilot/copilot-on-ghes/about-copilot-on-ghes)

### Inferences
- With Server, assume no Microsoft-provided AI features, and build the factory from third-party agents and local models.
- With Services, the Copilot Boards flow still does not touch Azure Repos, so it does not help a team whose code stays in Azure Repos.

### Gaps
- Foundry Local and other disconnected Microsoft inference options were not researched.
- Whether Copilot CLI's disconnected mode works with an Azure DevOps Server remote was not found. It is plausible since the CLI operates on a local checkout, but it is untested.
- Current Boards and Copilot Learn page was not retrieved (404 on the guessed URL), so licensing prerequisites are unverified.

## Coding agents and autonomous pipelines with Azure Repos and Azure Pipelines

### Takeaway
Treat every coding agent as a CLI that runs in a self-hosted pipeline agent, using git and REST. OpenHands, the most ADO-aware open option, explicitly excludes Server from its integration, and `ado-aw` is effectively Services-oriented. A pipeline template calling a headless CLI is the portable pattern.

### Cited Findings
Repository facts (GitHub API, 2026-11-07):

| Agent | Repo | License | Stars | Latest release | ADO-specific signal found |
|---|---|---|---|---|---|
| OpenCode | [anomalyco/opencode](https://github.com/anomalyco/opencode) | MIT | 225,087 | v1.19.5 (2026-10-28) | None in code search |
| Codex CLI | [openai/codex](https://github.com/openai/codex) | Apache-2.0 | 133,499 | rust-v0.167.0 (2026-11-06) | None in code search; local MS MCP client per [Learn](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview) |
| OpenHands | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | MIT | 92,840 | v1.37.0 (2026-11-06) | Services integration in Enterprise |
| Cline | [cline/cline](https://github.com/cline/cline) | Apache-2.0 | 71,410 | v4.1.26 (2026-10-26) | None in code search |
| Goose | [aaif-goose/goose](https://github.com/aaif-goose/goose) | Apache-2.0 | 56,314 | v1.57.0 (2026-10-28) | None in code search |
| Aider | [Aider-AI/aider](https://github.com/Aider-AI/aider) | Apache-2.0 | 49,993 | v0.86.0 (2025-08-09), last push 2026-05-22 | Not searched; **stale** |
| Continue | [continuedev/continue](https://github.com/continuedev/continue) | Apache-2.0 | 36,427 | v2.0.0-vscode (2026-06-19) | Only an MCP-naming test matched |
| Tabby | [TabbyML/tabby](https://github.com/TabbyML/tabby) | GitHub reports NOASSERTION | 33,922 | v0.32.0 (2026-01-25), last push 2026-06-30 | None in code search; **slowing** |
| Qwen Code | [QwenLM/qwen-code](https://github.com/QwenLM/qwen-code) | Apache-2.0 | 29,057 | v0.26.9 (2026-11-06) | None in code search |
| Kilo Code | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) | MIT | 27,885 | v7.8.27 (2026-11-05) | None in code search |

- The code search negatives are weak evidence, because GitHub code search can return empty on rate limits or indexing gaps. Do not read them as proof of no support.
- OpenHands: integration needs an Enterprise install (Replicated or Helm), reaches `login.microsoftonline.com` and `dev.azure.com`, and "Azure DevOps Server is not covered". Triggers are Boards comments, Repos PR comments and Service Hooks — [OpenHands docs](https://docs.openhands.dev/enterprise/integrations/azure-devops)
- Claude Code: no official Anthropic guide for Azure Pipelines was found. The building block is `claude -p` with `--allowedTools` and JSON output — [claudecn automation guide (third party)](https://claudecn.com/en/docs/claude-code/automation/). A low turn cap is suggested by [SFEIR (third party)](https://institute.sfeir.com/en/claude-code/claude-code-headless-mode-and-ci-cd/tips/)
- ReviewPR is a community example of ADO webhook to Claude to PR thread comment. It flags prompt injection through PR descriptions and CLAUDE.md, and runs on Azure Functions — [Optimizely blog](https://world.optimizely.com/blogs/kennyg/dates/2026/5/reviewpr-an-azure-function-that-reviews-your-azure-devops-pull-requests-with-claude/)

**Triggering from a work item or PR**
- Azure Pipelines can't be triggered by PR comments. YAML `pr:` triggers don't work with Azure Repos Git, so use a Build Validation branch policy instead — [PR-Agent docs](https://docs.pr-agent.ai/installation/azure/); confirmed for Semgrep setups by [Semgrep docs](https://semgrep.dev/docs/kb/semgrep-ci/azure-self-hosted-ubuntu)
- Webhooks send event JSON. A subscription can be created through an API and can send minimal payloads so the receiver must call back through normal permissions — [Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops-server)
- **Watch out:** the same doc says webhooks "can't target localhost (loopback) or special range IPv4/IPv6 addresses", and it links IANA's special-purpose registry. That registry covers RFC1918 ranges. The note is not gated to Services, while Qodo's on-prem guide says Server webhooks stay "within your internal network" — [Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops-server); [Qodo on-prem guide](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md)
- External callers start a run through the Run Pipeline REST API with template parameters and queue-time variables (third-party description) — [Port docs](https://docs.port.io/workflows/build-workflows/action-nodes/integration-actions/azure-devops/)

### Inferences
- Recommended flow: work item tag or state change → service hook → small on-prem webhook receiver → Run Pipeline REST call with the work item ID → self-hosted agent pool in a sandbox → agent CLI → `git push` plus PR through REST → Build Validation runs the verifiers.
- PR comment commands need the same receiver, because pipelines cannot react to comments natively.
- Agent choice can stay independent from SCM choice, which matches the earlier report's "portable assets" position.
- Avoid building on Aider or Tabby without a maintenance check.

### Gaps
- No end-to-end example of OpenCode, Goose, Qwen Code, Kilo or Cline on Azure Pipelines against Server was found. Feasibility is inferred from their being CLIs or MCP clients.
- Whether Server webhooks really reject private addresses must be tested on the first day of a PoC, since the receiver design depends on it.
- CLI headless capabilities for each agent were not re-verified here.

## Code review tools for Azure DevOps Server

### Takeaway
PR-Agent is the only open-source option with code evidence for arbitrary Server URLs, though its docs show only `dev.azure.com`. Qodo has an on-prem guide that mentions Server but conflicting matrices. CodeRabbit and Greptile lack evidence of Server support.

### Cited Findings
| Tool | License / terms | Services | Server | Evidence |
|---|---|---|---|---|
| PR-Agent ([The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent)) | MIT; 15,320 stars; v0.49.0 (2026-10-25) | Documented | Likely; undocumented | See below |
| Qodo | Commercial | Single and multi-tenant | **Conflicting** | See below |
| CodeRabbit self-hosted | Enterprise, 500+ seats | Listed | Not stated | [Overview](https://docs.coderabbit.ai/self-hosted/overview); [Azure DevOps guide](https://docs.coderabbit.ai/self-hosted/azure-devops) |
| Greptile self-hosted | Commercial | Not listed | Not listed | [Deployment options](https://www.greptile.com/docs/deployment-options); "custom code host integrations on request" per [self-host page](https://www.greptile.com/docs/security/selfhost) |
| Copilot code review | Azure-billed preview | Yes | Not mentioned; self-hosted agents unsupported | [InfoQ](https://infoq.com/news/2026/09/copilot-code-review-azure-repos) |
| SonarQube Server | Commercial and community tiers | Yes | **Yes** | [Sonar integration page](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration) |

- PR-Agent docs: a pipeline runs the `pragent/pr-agent` container, or a webhook handles "PR created" and "PR commented" with Basic auth. Docs show `org = https://dev.azure.com/...` only, with an `[azure_devops_server]` section for `agent_identity` and webhook credentials — [PR-Agent Azure docs](https://docs.pr-agent.ai/installation/azure/)
- PR-Agent source: `Connection(base_url=org, creds=BasicAuthentication("", auth_token))`, and a code comment says `azure_devops.org` "may be a bare org name or a full collection URL" — [azuredevops_provider.py](https://github.com/The-PR-Agent/pr-agent/blob/main/pr_agent/git_providers/azuredevops_provider.py)
- PR-Agent removed the `azure.com` URL restriction in a PR merged 2023-10-17 — [PR #381](https://github.com/Codium-ai/pr-agent/pull/381)
- Qodo's deployment matrix lists Azure DevOps as cloud only, with on-prem unsupported — [Deployment models](https://docs.qodo.ai/install-and-configure/deployment-model-support)
- Qodo's on-prem guide addresses Server: "For Azure DevOps Server, that's access within your internal network." OAuth needs Entra with admin roles, and PAT is the alternative — [Qodo on-prem guide](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md)
- CodeRabbit's listed self-managed platforms are GitHub Enterprise Server, GitLab self-managed and Bitbucket Data Center. Azure DevOps Server is absent, though "Azure DevOps" is a supported platform — [Azure DevOps guide](https://docs.coderabbit.ai/self-hosted/azure-devops)
- A Qodo blog claims CodeRabbit is cloud-only on Azure DevOps. This is vendor marketing, and CodeRabbit's self-hosted docs partly contradict it — [Qodo blog (search summary)](https://www.qodo.ai/?p=20997)

### Inferences
- First candidate: PR-Agent with a collection URL, PAT and a local OpenAI-compatible model. Check that its `azure-devops` Python SDK calls are accepted by Server's API version.
- Ask Qodo and CodeRabbit in writing for Server version support before evaluating either.

### Gaps
- PR-Agent against Server 25H2 was not verified by any public report found. Issue search returned nothing Server-specific.
- SonarQube edition required for PR decoration and current Server version compatibility were not confirmed.

## Requirements traceability on Azure Boards, Test Plans, specs and the Wiki

### Takeaway
Boards plus environments and Test Plans provide a strong built-in trace from work item to deployment on both products. `AB#` itself is a GitHub feature and does not apply to Azure Repos. The built-in Wiki is viable for a PM wiki on size and Mermaid, but it has unverified Chinese search quality and should not be chosen without a CJK test.

### Cited Findings
- `AB#{ID}` links from GitHub text (commit message, PR description, issue description) and not from comments or PR titles. Server 2020+ integrates with GitHub Enterprise Server only, and "Other Git repositories: Integration isn't supported." On Server with GHES, linking runs on an hourly pull — [Link GitHub](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github)
- Deployment jobs show commits and work items deployed to each environment — [Environments](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops-server)
- Test Plans can execute automated tests through YAML pipelines, and 26H2 adds a Test Run Hub — [Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver); [Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599)
- Spec tooling: [github/spec-kit](https://github.com/github/spec-kit) MIT, 148,221 stars, v1.3.2 (2026-11-06); [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) MIT, 74,177 stars, v1.16.1 (2026-11-07).

**Wiki**
- Mermaid: sequence, Gantt, flowchart, class, state, user journey, pie, requirement, gitgraph, ER and timeline are listed. There is "limited syntax support": most HTML tags, Font Awesome, `flowchart` syntax (use `graph`) and long arrows are unsupported — [Markdown guidance](https://learn.microsoft.com/en-us/azure/devops/project/wiki/markdown-guidance)
- The doc lists `flowchart` as a supported diagram type and also says the `flowchart` keyword is unsupported, so test before writing templates — [Markdown guidance](https://learn.microsoft.com/en-us/azure/devops/project/wiki/markdown-guidance)
- Git-backed storage with `.attachments`; 18 MB page, 19 MB attachment and 235-character path limits; case-sensitive unique titles; hyphens replace spaces in file names (mirror and GitHub copy, not the live page) — [Wiki naming conventions](https://github.com/MicrosoftDocs/azure-devops-docs/blob/main/docs/project/wiki/includes/wiki-naming-conventions.md); [file structure mirror](https://dotnet.territoriali.olinfo.it/en-us/azure/devops/project/wiki/wiki-file-structure)
- Search on Server uses a **modified Elasticsearch 7.17.0** that works only in that form and on OpenJDK 11. It is admin-configured and not extensible, needs up to 150% of repo size in disk, and has no HTTPS (IPSec is suggested). It needs a full reindex after database rollback — [Install Search](https://learn.microsoft.com/en-us/azure/devops/project/search/install-configure-search?view=azure-devops-server)
- Stakeholders only see provisioned wiki results, not published wikis, and have no code search. Prefix wildcards are not allowed. Boolean and phrase search are documented — [Search](https://learn.microsoft.com/en-us/azure/devops/project/search/get-started-search?view=azure-devops-server)
- Readers question the Log4j version bundled in Search (unverified comment) — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599)

### Inferences
- With Azure Repos, use native work item links and PR policies plus a checked trace trailer such as `Work-Item: 1234`, enforced by Build Validation, instead of `AB#`.
- Spec Kit and OpenSpec files live in Git, so the adaptation is thin: a spec header carries the work item ID, and the pipeline syncs status and tests through REST.
- Wiki verdict: keep the published (code-as-wiki) mode for agent-readable knowledge since it is plain Git Markdown, and index it with your own CJK-aware retrieval. A separate wiki is only justified if PMs need richer permissions or editing, which was not assessed here.

### Gaps
- **No source was found on CJK tokenization for wiki, work item or code search.** Treat this as untested. Run Traditional Chinese queries on real pages. One search returned only unrelated Purview docs.
- Native Azure Repos linking syntax (`#ID` in commits) comes from background knowledge and was not read this session.
- No public Spec Kit or OpenSpec integration with Azure Boards was searched for or confirmed. The adaptation above is my design, not a documented feature.
- Page-level wiki permissions and Test Plans licensing were not researched.

## Orchestration: Azure Pipelines, Temporal, and a gh-aw equivalent

### Takeaway
A gh-aw equivalent does exist as `githubnext/ado-aw`, but its engine, network allowlist and credential proxy all point at Services. On Server, use Pipelines YAML with environments as the delivery gate, and optionally Temporal above it for long-running cross-system state.

### Cited Findings
- `githubnext/ado-aw`: MIT, 33 stars, v0.54.0 (2026-10-27). It compiles Markdown agents into pipelines with Agent, Detection and SafeOutputs stages. Safe outputs cover PRs, work items, wiki pages and build tags, and targets are `standalone`, `1es`, `job` and `stage` — [Project site](https://githubnext.github.io/ado-aw/); [README](https://github.com/githubnext/ado-aw)
- ado-aw currently supports only the `copilot` engine (GitHub Copilot CLI). BYOK via `engine.provider` accepts a base URL with `openai`, `azure` or `anthropic` formats — [Engine docs](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md)
- ado-aw's compiler and firewall binaries come from GitHub Releases, and its default allowlist is `dev.azure.com`, `*.visualstudio.com` and `github.com` hosts — [Network docs](https://github.com/githubnext/ado-aw/blob/main/docs/network.md)
- ado-aw's `ado-proxy` design states "Azure DevOps Services only: no Azure DevOps Server or sovereign clouds". This statement covers that sidecar, not necessarily the entire tool. Its audit docs parse on-prem URLs — [ado-proxy design](https://github.com/githubnext/ado-aw/blob/main/docs/ado-proxy-design.md); [audit docs](https://github.com/githubnext/ado-aw/blob/main/docs/audit.md)
- For comparison, `github/gh-aw` is MIT, 5,585 stars, v1.0.1 (2026-10-27) — [GitHub](https://github.com/github/gh-aw)
- Temporal: `temporalio/temporal` is MIT, 24,238 stars, v1.32.2 (2026-11-02) — [GitHub](https://github.com/temporalio/temporal)
- No Temporal-plus-Azure-DevOps guide or official connector was found. Building blocks are service hooks with branch and result filters, and the Run Pipeline REST API — [Service hooks](https://learn.microsoft.com/azure/devops/service-hooks/); [Port docs](https://docs.port.io/workflows/build-workflows/action-nodes/integration-actions/azure-devops/)

### Inferences
- Start with Pipelines only: templates, environments, approvals and Build Validation. Add Temporal only if the first pilot shows waits, retries or cross-repo sagas that pipelines handle badly.
- Temporal pattern: an activity calls Run Pipeline, then a `build.complete` service hook hits a receiver that signals the workflow by run ID, with polling as a fallback. Treat Pipelines as the executor and audit trail, and Temporal as the state machine.
- ado-aw's design is worth copying even where it can't run: a read-only agent stage, a separate detection stage, and an executor holding the write token.
- ado-aw has low adoption and a pre-1.0 version, so it is a watch item even on Services.

### Gaps
- Nobody has publicly shown ado-aw running on Server. Whether compile-time settings can override hosts was not determined.
- Pipeline concurrency, timeout and job-length limits on self-hosted agents were not checked.

## Backstage and security tools in Azure Pipelines on-prem

### Takeaway
The Backstage community plugin supports Server with a PAT. For security scanning, SonarQube is explicitly Server-compatible. Semgrep scans can run on self-hosted agents, but PR comments are documented as cloud only. Trivy and ZAP were not verified and can fall back to CLI steps.

### Cited Findings
- `@backstage-community/plugin-azure-devops-backend` README: "`organization` is your Azure DevOps Services (cloud) Organization name or for Azure DevOps Server (on-premise) this will be your Collection name" — [README](https://github.com/backstage/community-plugins/blob/main/workspaces/azure-devops/plugins/azure-devops-backend/README.md)
- Repo `backstage/community-plugins`: Apache-2.0, 427 stars, pushed 2026-11-07 — [GitHub](https://github.com/backstage/community-plugins)
- Service principal and managed identity auth don't work with Server — [Backstage changelog](https://backstage.io/docs/releases/v1.15.0-next.1-changelog). A Taiwanese practitioner confirms a PAT-based setup — [iThome](https://ithelp.ithome.com.tw/articles/10350793)
- Roadie's doc saying Server is unsupported dates to 2022 and looks stale for the OSS plugin, though it may still be accurate for Roadie's SaaS — [Roadie](https://roadie.io/docs/integrations/azure-devops/)
- SonarQube integration works with both Server and Services and uses a global PAT plus a project-level PAT — [Sonar docs](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration). Extension 8.2.3 (2026-06-19) adds severity labels in Azure PR decoration — [Sonar 9.9 extension page](https://docs.sonarsource.com/sonarqube-server/9.9/analyzing-source-code/scanners/sonarqube-extension-for-azure-devops)
- One 2025.1 LTS user reports quality gate status worked but no PR comments appeared, unresolved — [Sonar community](https://community.sonarsource.com/t/trouble-getting-pr-decoration-working-with-sonarqube-server-2025-1-lts-and-azure-devops/151196)
- Semgrep: PR comments "currently support repositories hosted by Azure DevOps Cloud" — [Semgrep PR comments](https://semgrep.dev/docs/semgrep-appsec-platform/azure-pr-comments). Self-hosted agent install via pipx, with Build Validation needed for PR scans and `SEMGREP_PR_ID` and `SEMGREP_BASELINE_REF` for diff-aware runs — [Semgrep self-hosted](https://semgrep.dev/docs/kb/semgrep-ci/azure-self-hosted-ubuntu)

### Inferences
- For a uniform on-prem path, run scanners as CLI or container steps emitting SARIF or JUnit, gate on exit code, and post findings with a small REST step. This avoids dependence on Marketplace extensions and cloud back-ends.
- Local Gallery installation of Marketplace extensions needs a compatibility check on each Server upgrade.

### Gaps
- Trivy and OWASP ZAP integrations on Azure Pipelines were not confirmed by any source this session. Both being CLI or container tools is background knowledge.
- Backstage Scaffolder and catalog discovery behaviour on Server 25H2 was not tested.

## Migration: when to leave Azure DevOps

### Takeaway
Stay on Azure DevOps if Boards, Test Plans and existing pipelines are deeply used, and accept that you will own the integration glue. Moving is justified mainly if you want a vendor-supported self-hosted agent platform, where GitLab has the strongest evidence. GHES does not solve this by itself, because its Copilot features still lean on cloud.

### Cited Findings
- GitLab Duo Agent Platform Self-Hosted: GitLab 19.0 added open models including Devstral 2, GLM-5.1, Kimi-K2.6 and MiniMax-M2.7. For isolated networks, open models on local inference are the only path, with vLLM as the primary pattern. Offline-licensed customers need the Self-Hosted add-on — [GitLab blog, 2026-05-21](https://about.gitlab.com/blog/more-ai-models-for-duo-agent-platform-self-hosted/)
- GHES: most Copilot features need Enterprise Cloud, while Copilot CLI can work disconnected — [About Copilot on GHES](https://docs.github.com/en/enterprise-server@latest/copilot/copilot-on-ghes/about-copilot-on-ghes). A community write-up runs similar agents on Actions runners, not official — [Ben Leane](https://benleane83.github.io/blog/2025-12-04-ghes-coding-agent)
- Azure DevOps Server's GHES integration exists for Boards — [Link GitHub](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github)

### Inferences
| Situation | Likely best |
|---|---|
| Heavy Boards, Test Plans and audit dependence; Windows/AD estate; small agent scope | Keep Azure DevOps Server, add PR-Agent, community MCP, CLI agents |
| Already on Services and cloud is merely disliked | Consider staying on Services with self-hosted agents and local models; the MCP, OpenHands and Qodo paths open up. Code still sits in Microsoft's cloud |
| Hard no-cloud plus wanting a supported agent platform | Evaluate GitLab self-managed with Duo Self-Hosted, with licence cost as the main check |
| Want the GitHub ecosystem and can accept some cloud | GHES plus GitHub Enterprise Cloud link; hybrid path keeps Boards through the supported GHES integration |
| Small team, minimal licences, willing to own everything | Gitea or Forgejo, with Boards and Test Plans replaced elsewhere. Least-evidenced option here |

- Migration triggers worth tracking: community MCP breakage on Server upgrades, a need for AI features Microsoft ships only to Services, and PAT or auth changes that break agent service accounts.
- A pilot on Azure DevOps Server costs little and its assets (specs, agent instructions, CLI wrappers) mostly port to another SCM. The REST-specific receiver and trace steps would need rewriting.
- Which row applies depends on whether the user is on Server or Services today, which the user has not stated. That is the first question to settle.

### Gaps
- Gitea and Forgejo AI and agent status, GitLab licence pricing, and ADO-to-GitLab migration tooling were not researched.
- No total cost comparison was attempted.
- GHES conclusions rely on docs and a search summary. No 2026 GHES-specific agent announcement was found.
