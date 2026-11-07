# Azure DevOps Services: Integration and Data Boundary for a Hybrid AI Factory (as of 2026-11-07)

Evidence legend: "Docs" means Microsoft Learn or vendor documentation read this session. Repo facts (stars, licenses, releases, source code) were read through the GitHub API on 2026-11-07. "Background" marks unverified prior knowledge. Learn pages show their `ms.date` where it matters, because several pages disagree with newer sources.

## Triggering agents with no inbound path to on-premises

### Takeaway
Self-hosted agents need outbound HTTPS only. Only Build Validation, schedules, pipeline resources and REST calls work with no extra infrastructure. A PR comment or work item change cannot natively trigger an Azure Repos pipeline, so it needs a service hook into a queue or incoming webhook, or an on-prem poller calling the Runs API. Recommended default: an on-prem dispatcher polling REST, plus Build Validation for PRs. Add Storage Queue only if polling latency or API load proves unacceptable.

### Cited Findings
**Self-hosted agent networking**
- "The agent always initiates this communication." It reports logs and job status, listens with an HTTP long poll, then receives a job-specific short-lived OAuth token and discards it after the job — [Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)
- Job payloads are encrypted with a per-agent key pair, whose public key is exchanged at registration — [Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)
- Agent firewall list for `dev.azure.com` orgs: `https://dev.azure.com`, `https://*.dev.azure.com`, `https://login.microsoftonline.com`, `https://management.core.windows.net`, `https://download.agent.dev.azure.com`, `https://*.vsassets.io`, `https://*.vsblob.visualstudio.com`, `https://*.vssps.visualstudio.com`, `https://*.vstmr.visualstudio.com`, `https://*.blob.core.windows.net`, `https://app.vssps.visualstudio.com`. The `{org}.visualstudio.com` family applies to orgs on the legacy domain — [Linux agent FAQ](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops)
- Outbound IPv4 per the allowlist page: `150.171.22.0/24`, `.23.0/24`, `.73.0/24`, `.74.0/24`, `.75.0/24`, `.76.0/24`. IPv6: `2620:1ec:50::/48`, `2620:1ec:51::/48`, `2603:1061:10::/48`. Service tags are not supported for outbound — [Allowlist, ms.date 2026-02-10](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)
- **Conflict:** the Linux agent FAQ also lists `13.107.6.0/24`, `13.107.9.0/24`, `13.107.42.0/24` and `13.107.43.0/24`, which the allowlist page omits — [Linux agent FAQ](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops)
- Port 22 is needed only for SSH Git, to `ssh.dev.azure.com` and `vs-ssh.visualstudio.com` — [Allowlist](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)
- Agent maintenance dates: Node.js 6, 10 and 16 runners are scheduled for removal on 2026-11-24. From agent v5.279.0, `mapDockerSocket` defaults to `false` for Linux container jobs — [Sprint 279](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update)

**Option comparison** (limits are cited below the table; latency and security columns are my assessment, since Microsoft publishes no trigger latency figures)

| Option | Event source | Added cloud dependency | Inbound to on-prem | Limits | Expected latency (inference) | Main security concern |
|---|---|---|---|---|---|---|
| Build Validation | PR create or source push | None | None | One agent job per queue | Seconds to queue, then pool wait | PR content is untrusted input to the agent |
| Scheduled pipeline polling WIQL | Cron | None | None | About 1,000 runs per pipeline per week; 10 per 15 minutes | About 10 minutes if sustained | Uses a parallel job per poll |
| Pipeline resource trigger | Another pipeline finishing | None | None | Not for templates; default-branch filter rules | Seconds after completion | Good for chaining stages, not for events |
| On-prem poller plus Runs API | REST query, then `POST .../runs` | None | None | 200 TSTU per sliding 5 minutes | Poll interval | Long-lived identity outside pipelines |
| Service hook to Storage Queue or Service Bus, on-prem consumer | Any event | Azure subscription and a queue | None (consumer pulls) | Storage TTL up to 7 days | Seconds (unmeasured) | Payload copy in your Azure region; use Minimal or None |
| Service hook to Incoming Webhook resource | Any event | None (**untested**) | None | Same service hook policy | Seconds (unmeasured) | No usable HMAC; template injection |
| Service hook to on-prem Web Hook | Any event | Public endpoint or relay | **Yes** | Public HTTPS only | Seconds | Violates the no-inbound goal |

**Limits and mechanics**
- Scheduled runs: "around 1000 runs per pipeline per week" and "10 runs per pipeline per 15 minutes". Cron is UTC. Without `always: true`, a schedule skips when code has not changed, and `always: true` runs even if a prior run is in progress — [Scheduled triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops)
- Build Validation with an Automatic trigger queues on PR creation or a push to an existing PR. Manual is user-queued. Policy options include blocking or optional, path filter, source-update-only and validity duration — [Branch policies](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops)
- Comment triggers such as `/azp run` "are supported only for GitHub repositories", and Azure Repos Git uses branch policies for PR triggers (search extract of Microsoft's page) — [Triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/triggers?view=azure-devops)
- Pipeline resource triggers run on the completion of another pipeline and support branch, tag and stage filters. Filters are read from the "Default branch for manual and scheduled builds", and the trigger block cannot live in a template — [Pipeline triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/pipeline-triggers?view=azure-devops)
- Runs API: `POST https://dev.azure.com/{org}/{project}/_apis/pipelines/{id}/runs?api-version=7.1`, accepting `templateParameters`, `variables`, `stagesToSkip` and `previewRun`, with scope `vso.build_execute` — [Run Pipeline](https://learn.microsoft.com/en-us/rest/api/azure/devops/pipelines/runs/run-pipeline?view=azure-devops-rest-7.1)
- Rate limit is a global 200 TSTU in any sliding five-minute window, returning HTTP 429 when blocked — [Rate limits](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops). Service principals and managed identities have the same limits as users — [SP and MI](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)

**Event IDs**
- PR comment: `ms.vss-code.git-pullrequest-comment-event`, filterable by repository and branch. PR created: `git.pullrequest.created`. PR updated: `git.pullrequest.updated`. Run state: `ms.vss-pipelines.run-state-changed-event` — [Events](https://learn.microsoft.com/en-us/azure/devops/service-hooks/events?view=azure-devops)
- Work items: `workitem.created`, `workitem.updated`, `workitem.deleted`, `workitem.restored`, filterable by `areaPath` and `workItemType` — [Events](https://learn.microsoft.com/en-us/azure/devops/service-hooks/events?view=azure-devops). `workitem.commented` is also an event — [Consumers](https://learn.microsoft.com/en-us/azure/devops/service-hooks/consumers?view=azure-devops)

**Service hook consumers and failure behaviour**
- Service Bus queue and topic support connection string or an Azure Resource Manager service connection. Storage Queue supports account key or service connection, with visibility timeout and TTL capped at 604,800 seconds. All accept `resourceDetailsToSend` — [Consumers](https://learn.microsoft.com/en-us/azure/devops/service-hooks/consumers?view=azure-devops)
- Web Hook consumers require a "public HTTPS endpoint" and "can't target localhost (loopback) or special range IPv4/IPv6 addresses". Minimal or None payloads force callers back through normal permission checks — [Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops). This answers the earlier open question for Services.
- Inbound source for Asia Pacific is `20.195.68.0/24` (Southeast Asia), or the `AzureDevOps` service tag — [Allowlist](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)
- Failures: 410 disables the subscription. 408, 502, 503 and 504 retry up to eight times over about 183 seconds. Other failures put it on probation, during which "it loses any new events", with at most seven retries — [Troubleshoot](https://learn.microsoft.com/en-us/azure/devops/service-hooks/troubleshoot?view=azure-devops)

**Incoming Webhook resource**
- `resources.webhooks` with an Incoming Webhook service connection accepts `POST https://dev.azure.com/{org}/_apis/public/distributedtask/webhooks/{name}?api-version=6.0-preview`. Anyone who knows the org and connection name can trigger it unless a secret is set, and then an HMAC-SHA1 checksum is verified (search extract of Microsoft's pages) — [Resources](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops); [Service connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops)
- Payload fields are consumed as compile-time `${{ parameters.<alias>.<path> }}` — [Resources](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops)
- The Web Hook consumer offers only static `httpHeaders` and basic auth, and header values are visible to anyone with subscription access — [Consumers](https://learn.microsoft.com/en-us/azure/devops/service-hooks/consumers?view=azure-devops)

### Inferences
- **PR comment or work item change to pipeline: yes, but only indirectly.** Paths: (a) on-prem poller to Runs API; (b) service hook to Storage Queue, consumer to Runs API; (c) service hook to Incoming Webhook, if a PoC confirms it.
- Option (c) is attractive because it adds no Azure resource and no on-prem listener, but three risks must be resolved. No source shows a service hook pointing at the same org's webhook endpoint. Service hooks cannot compute the HMAC, so the secret looks unusable and the endpoint would be unauthenticated. Expanding comment text through `${{ }}` is a YAML and script injection vector, so pass only IDs and refetch.
- A sustained schedule tops out near a 10-minute cadence (1,008 runs per week), and each poll takes a parallel job. A long agent run on the single free job would block polling. Run the poller as an on-prem daemon instead.
- Probation drops events, so any event-driven design still needs a reconciliation poll.
- Queue cost in option (b): a real but small Azure dependency, holding only IDs if Minimal or None is used. It does not change the boundary much, since source is already in Microsoft's cloud.
- Polling PR comments is the expensive case: WIQL does not cover threads, so the poller must list threads per active PR. Mind the TSTU limit. This is the best reason to adopt option (b) later.
- Allowlist the agent FAQ's IP superset, because the two Learn pages disagree.
- Background: Service Bus uses AMQP on 5671 or WebSockets over 443, and Storage Queue uses HTTPS 443. Verify at implementation.

### Gaps
- No published latency or SLA for service hook delivery or long-poll job pickup. Measure in the PoC.
- Service hook to same-org Incoming Webhook is untested and undocumented.
- The full work item filter list (for example, changed fields) was not extracted.
- The YAML `pr:` and comment-trigger statements come from search extracts of the Learn page, not a direct read.

## Identity: Entra, workload identity federation, secretless agents, global PAT retirement

### Takeaway
**The global PAT date is 2027-01-27.** Microsoft's announcement postponed it from 2026-12-01 on 10/23, and the December date survives only in stale text. Agents inside pipelines can run with no stored secret, using the job token or a federated Azure DevOps service connection. Long-running on-prem services outside pipelines cannot be fully secretless without an external OIDC issuer or Azure Arc, so plan on a certificate-based service principal.

### Cited Findings
**Global PAT date**
- Authoritative: the announcement shows "December 1, 2026 → January 27, 2027" and "Update 10/23: This retirement has been postponed to avoid disruption during the December holiday season." The planned March 15 creation block was also dropped ("Update 03/05"), and creation continues until decommissioning — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/retirement-of-global-personal-access-tokens-in-azure-devops/)
- The PAT page's top notice agrees: "All existing global PATs stop working on January 27, 2027." Its best-practices list still says December 1, 2026, which is a leftover in the same document (`updated_at` 2026-11-03) — [PAT docs](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops)
- Stale sources with December 1: [Sprint 270 notes, last updated 2026-03-06](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/general/sprint-270-update); [Microsoft Q&A answer, 2026-10-04, before the update](https://learn.microsoft.com/en-ca/answers/questions/6025099/exact-end-of-life-dates-for-azure-devops-oauth-and); [Backstage #35394](https://github.com/backstage/backstage/issues/35394); [Mend](https://docs.mend.io/integrations/latest/azure-devops-global-pat-and-oauth-1.0-deprecation)
- Identify global PATs by setting Access scope to "All accessible organizations", status Active — [Azure DevOps Blog](https://devblogs.microsoft.com/devops/retirement-of-global-personal-access-tokens-in-azure-devops/)
- Org-scoped PATs remain. Entra-backed orgs mark a PAT inactive if its user does not sign in within 90 days. Token use is not audited: "Token access events aren't currently logged" — [PAT docs](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops)
- Legacy Azure DevOps OAuth: retirement announced for 2026, exact end date unspecified, per a moderator — [Microsoft Q&A](https://learn.microsoft.com/en-ca/answers/questions/6025099/exact-end-of-life-dates-for-azure-devops-oauth-and)

**Service principals and managed identities**
- Requires an org connected to an Entra tenant, and each identity is added to each org explicitly. Azure Repos needs Basic or higher, Stakeholder does not include Repos, and each paid identity is billed per org with no Visual Studio benefit — [SP and MI](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)
- Application identities "can't create PATs or SSH keys", cannot use the web portal, and Conditional Access for workload identities covers service principals but not managed identities — [SP and MI](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)
- Managed identity is "Best for workloads hosted on Azure". A service principal uses federation, certificate or secret, in that order of preference, and requests `https://app.vssps.visualstudio.com/.default`. Tokens are opaque — [SP and MI](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)

**Secretless paths for on-prem agents**
- Registration: PAT with **Agent Pools (read, manage)** is used "only at the time of registering the agent" — [PAT registration](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/personal-access-token-agent-registration?view=azure-devops). SP registration from agent 3.227.1 uses client ID, tenant ID and a client secret, "used only during agent registration" — [SP registration, ms.date 2026-04-22](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/service-principal-agent-registration?view=azure-devops)
- Runtime: the agent uses a listener token and a short-lived job token for the scoped identity — [Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)
- Azure DevOps service connection: pipelines authenticate to Azure DevOps with "PAT-free authentication" through Entra workload identities, with per-pipeline permission and audit logging. `AzureCLI@3` with `azureDevOpsServiceConnection` can run `az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798`. The page, ms.date 2026-03-23, still carries a stale "rolling out this week" note — [Entra service connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/add-devops-entra-service-connection?view=azure-devops)
- Federation outside Azure: supported for any Kubernetes cluster including on-prem, SPIFFE/SPIRE and other compute with an external IdP. Entra uses the OIDC Discovery issuer URL "to fetch the keys", and tokens must be RS256 — [WIF overview](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation); [Create trust](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust)
- Azure Arc-enabled servers can use a managed identity through local IMDS; the doc's example is Key Vault — [Arc MI](https://learn.microsoft.com/en-us/azure/azure-arc/servers/managed-identity-authentication)

### Inferences
| Workload | Recommended identity | Stored secret on-prem |
|---|---|---|
| Agent inside a pipeline, repo and PR writes in own project | `System.AccessToken` with scoped job authorization | None |
| Agent needing a dedicated identity or cross-project access | Azure DevOps service connection (WIF) | None; needs Azure CLI and `login.microsoftonline.com` |
| Agent registration | Short-lived org-scoped PAT or SP secret, revoked afterward | Bootstrap only, plus the agent key pair |
| On-prem dispatcher, RAG indexer, Backstage, MCP gateway | Service principal with certificate in on-prem vault | Certificate key |
| Same, strict no-secret | WIF with an internal issuer whose discovery and JWKS documents are publicly hosted, or Arc MI | None, but adds an exposure or Azure Arc |

- Entra must fetch keys, so an internal-only issuer would not work. Publishing only the static discovery and JWKS documents satisfies this without opening a route to the workloads (inference).
- A pipeline can hand its job token to a local MCP process via `--authentication envvar`, giving the agent a credential that expires with the job. This is the cleanest pattern found, but is untested.
- Use Entra as the default and org-scoped PATs only for tools without Entra support. Because PAT use is not logged, prefer identities that show up in audit.
- Separate identities per agent role, each with Basic ($6 per month, see Native features). Read-only agents should use Readers-style groups.

### Gaps
- No documentation shows certificate or federated auth for `config.sh --auth SP`. Docs describe client secret only.
- No source confirms Arc MI working against Azure DevOps. The SP page frames MI as Azure-hosted.
- Microsoft's post labels updates "10/23" with no year; 2026 is inferred from context and the Learn page's current date.
- Whether the January 27 cutoff is immediate or phased is not documented.

## Data boundary: storage, region, logs, training, audit

### Takeaway
Pick **Asia Pacific**, the only nearby geography; the doc's region table lists Singapore, and there is no Taiwan geography. Customer data stays in the geography with stated exceptions, but token data and non-US/EU/UK profile data live in the US, and prereleases may be stored in the US or globally. Self-hosted agent console logs do go to the cloud by default, and the documented opt-out is `--disableloguploads`. Azure DevOps pages make no AI-training statement, which remains an open item for legal.

### Cited Findings
**Storage and region**
- Repos and work items: Azure Blob holds source file contents and work item attachments. Azure SQL holds project metadata, versioned source history and work item details — [Data protection](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops)
- Geographies: Australia, Brazil, Canada, Asia Pacific, EU, India, UK, US — [Data locations, ms.date 2026-03-17](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops)
- The inbound IP table's only Asia Pacific region is Southeast Asia (Singapore) — [Allowlist](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)
- Default is the closest location, selectable at creation. Later moves go through a migration with Microsoft support — [Data locations](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops); [Data protection](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops)
- Customer data "includes source code, work items, test results, geo-redundant mirrors, and offsite backups" — [Data locations](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops)
- **Exceptions:** identity and profile data for users outside the US, EU and UK is stored in the US. PATs and SSH keys are stored in a US datacenter. Named-user tenant allowlist references reside in US, EU and Singapore. Transfers may occur for support, troubleshooting and legal compliance. "Preview, beta, or other prerelease services typically store data in the United States but might store it globally" — [Data locations](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops)
- Remote MCP runs in Europe and the United States, routes by lowest latency, and acts "as a pass-through and doesn't store your data" — [Data locations](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops)

**Protection**
- SQL uses TDE; blobs use service-side encryption. Data is replicated to a second region in the same geography, with roughly six blob copies. SQL backups are retained 28 days, deleted orgs are recoverable within 28 days, and deletion can take up to 70 days. Customer-deleted assets are not restored — [Data protection](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops)
- Microsoft production access is just-in-time and logged. Court-ordered disclosure attempts owner notice when legally allowed. Certifications include ISO 27001:2022, SOC 1 and SOC 2 Type 2, and C5. SLA is 99.9% — [Data protection](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops)

**AI training**
- The Azure DevOps data protection and location pages contain no statement on model training; "training" there refers to staff — [Data protection](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops)
- Adjacent Microsoft statement: foundation models via Azure OpenAI and Copilots do not use Customer Data for training without permission (Q&A answer, not product terms) — [Microsoft Q&A](https://learn.microsoft.com/en-au/answers/questions/1703034/does-microsoft-ai-systems-such-as-azure-azure-mach); Security Copilot says this is in the Product Terms — [FAQ](https://learn.microsoft.com/en-us/copilot/security/faq-data-compliance)
- GitHub: Copilot Business and Enterprise data is not used for training — [GitHub Docs](https://docs.github.com/copilot/how-tos/manage-your-account/managing-copilot-policies-as-an-individual-subscriber). From April 24, 2026, Free, Pro and Pro+ interaction data is used unless opted out, while members of paid orgs are excluded — [GitHub Blog](https://github.blog/news-insights/company-news/updates-to-github-copilot-interaction-data-usage-policy/); [FAQ](https://github.com/orgs/community/discussions/188488)

**Pipeline logs**
- The agent "reports the logs and job status" to the service — [Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)
- **Opt-out confirmed:** `--disableloguploads`: "don't stream or send console log output to the server. Instead, you may retrieve them from the agent host's filesystem after the job completes." — [Linux agent](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops); [Windows agent](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/windows-agent?view=azure-devops)
- Other upload paths: `##vso[task.uploadfile]` and pipeline artifacts. `system.debug` turns on verbose logs, and `Agent.Diagnostic` follows it — [Review logs](https://learn.microsoft.com/en-us/azure/devops/pipelines/troubleshooting/review-logs?view=azure-devops)
- Masking is best effort: "We make an effort to mask secrets". Substrings are never masked, secrets should not be structured data, and some operating systems log command lines — [Secret variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/set-secret-variables?view=azure-devops)
- Retention: project-level only (per-pipeline rules are gone), with days settings for runs, PR runs, and artifacts, symbols and attachments. Deleting a run removes logs, artifacts, test results and metadata. Runs can be marked to retain indefinitely — [Retention](https://learn.microsoft.com/en-us/azure/devops/pipelines/policies/retention?view=azure-devops)
- A 30-day default is referenced only indirectly — [Build History API](https://learn.microsoft.com/en-us/rest/api/azure/devops/build/history/get?view=azure-devops-rest-7.1)

**Audit**
- "Auditing is still in public preview", Entra-backed orgs only, with 90-day retention. Streams target Splunk (on-prem or cloud), Azure Monitor Logs and Event Grid. Events reach Splunk within half an hour or less — [Streaming](https://learn.microsoft.com/en-us/azure/devops/organizations/audit/auditing-streaming?view=azure-devops); [Auditing](https://learn.microsoft.com/en-us/azure/devops/organizations/audit/azure-devops-auditing?view=azure-devops)
- On-prem Splunk needs inbound HEC from Azure DevOps IPs — [Allowlist](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)

### Inferences
- Logs are not named in the customer data list. Treat them as customer data, but ask Microsoft to confirm residency.
- Log minimization, in order: run agents in a pool with `--disableloguploads` and ship host logs to on-prem storage. Keep agent transcripts out of stdout regardless. Never use `system.debug` on agent pools. Keep artifacts and test attachments on-prem, pushing only pass or fail and links. Set short retention on run and PR-run days. Scrub at the source, since masking is not a control.
- Side effects need a PoC: reviewers lose log text in the UI. MCP `pipelines_build_log` and any failure-analysis tool will see nothing.
- Azure Pipelines still receives step names, status, timing, test results and tags. Keep prompts, plan text and diffs out of them.
- No inbound audit option: scheduled export to on-prem SIEM, or Event Grid into a queue that a local consumer pulls. An on-prem Splunk stream breaks the no-inbound goal.
- Avoid preview features for sensitive code, because of the US or global storage clause. This specifically covers Copilot code review while it is in preview.

### Gaps
- Microsoft Products and Services DPA and Product Terms were not retrieved. Obtain the exact training language, and confirm Azure DevOps coverage, before finalizing.
- Maximum run retention and PR-run default on Services were not found.
- Whether `--disableloguploads` suppresses all timeline detail or affects Marketplace tasks is undocumented.
- The Asia Pacific secondary replication region is not stated. Background: Azure's Southeast Asia pair is East Asia (Hong Kong), which matters if Hong Kong replication is a concern.
- Whether audit REST export is suitable for automation was not read this session.

## Official Azure DevOps MCP Server

### Takeaway
Remote has been **GA since 2026-08-05**, so the earlier "preview" note is stale. Local stays supported with no retirement date. For this architecture, use **local (v2.11.0 or later)** inside pipelines with a short-lived token, because remote has no documented non-interactive path and transits US or EU instances. Treat it as read-mostly and keep write actions in a separate stage.

### Cited Findings
| Aspect | Remote `https://mcp.dev.azure.com/{org}` | Local `@azure-devops/mcp` |
|---|---|---|
| Status | [GA 2026-08-05](https://devblogs.microsoft.com/devops/azure-devops-remote-mcp-server-ga/) | Supported; "[will eventually replace](https://github.com/microsoft/azure-devops-mcp)" wording with no date, and "[continue to support and maintain](https://devblogs.microsoft.com/devops/azure-devops-remote-mcp-server-ga/)" |
| Auth | [Entra only; Entra-backed org; no MSA orgs](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server) | [`interactive`, `azcli`, `envvar` (`ADO_MCP_AUTH_TOKEN`), `pat`, plus an `env` example](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/GETTINGSTARTED.md) |
| Tool scoping | [`X-MCP-Toolsets`, `X-MCP-Tools`](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server) | [`-d` domains](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/GETTINGSTARTED.md) |
| Read-only | [`X-MCP-Readonly: true`](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server) | No flag seen in [`src/index.ts`](https://github.com/microsoft/azure-devops-mcp/blob/main/src/index.ts) |
| Data path | [EU or US instance, pass-through](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops) | Runs locally, calls your org |

- Repo: MIT; v2.11.0 (2026-10-22), v2.10.0 (2026-09-09) — [Releases](https://github.com/microsoft/azure-devops-mcp/releases)
- Client support: the GA post (Aug 5) lists Claude Code and Cursor as unsupported — [Blog](https://devblogs.microsoft.com/devops/azure-devops-remote-mcp-server-ga/). **Newer docs supersede this:** Cursor and Claude Code now work with a custom Entra app registration (delegated permissions on app `2a72489c-aab2-4b65-b93a-a91edccf33b8`, public client, browser sign-in through `/mcp`). Claude Desktop and Codex remain unsupported on remote — [Remote MCP, ms.date 2026-09-21](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server)
- Tool groups: `repos`, `advsec`, `wit`, `pipelines`, `wiki`, `work`, `testplan`, `elm`. `elm` (Enterprise Live Migration, including `abandon`) is on by default in `all`. `wit_query_by_wiql` is Insiders only. OAuth scopes and user permissions, not tool filters, grant authority — [Remote MCP](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server)
- Rate-limit guidance was added to remote tool responses — [Sprint 279](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update)
- Learn's claim that "sensitive project information never leaves your network" sits on a page that also describes a Microsoft-hosted endpoint — [Overview](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview)

**Security advisories**
- The repo lists 0 published GitHub security advisories (API, 2026-11-07) — [Repo](https://github.com/microsoft/azure-devops-mcp/security)
- **PR-description prompt injection:** Manifold Security showed hidden HTML comments in a PR description hijacking agents through the PR tool, which lacked spotlighting. No CVE, and Microsoft called it a known class of AI risk — [CSA note, 2026-08-03](https://labs.cloudsecurityalliance.org/research/csa-research-note-azure-devops-mcp-pr-comment-hijack-2026080/). The proof of concept triggered a cross-project pipeline and exfiltrated a wiki page (search extract; the article returned 403 to direct fetch) — [The Hacker News](https://thehackernews.com/2026/07/microsoft-azure-devops-mcp-flaw-lets.html)
- **Later fixes, which make "unpatched" claims stale for local:** [#1553, 2026-08-27](https://github.com/microsoft/azure-devops-mcp/commit/c3e4cd05) spotlights PR content; [#1563, 2026-08-31](https://github.com/microsoft/azure-devops-mcp/commit/ba994781) adds repo, wiki and work item content; [#1570, 2026-09-02, tagged MSRC-135517](https://github.com/microsoft/azure-devops-mcp/commit/a2891613) centralizes wrapping. `repositories.ts` now imports [`content-safety`](https://github.com/microsoft/azure-devops-mcp/blob/main/src/shared/content-safety.ts)
- [#1680, 2026-10-09](https://github.com/microsoft/azure-devops-mcp/commit/ea8a7f6b) fixes an MSRC case in attachment downloads (filename validation)
- **Misattribution:** third parties say CVE-2026-32211 affects `@azure-devops/mcp` — [DEV](https://dev.to/michael_onyekwere/cve-2026-32211-what-the-azure-mcp-server-flaw-means-for-your-agent-security-14db). The CVE record is titled "Azure MCP Server" with product "Azure Web Apps" and does not name this package — [CVE record](https://cveawg.mitre.org/api/cve/CVE-2026-32211)
- CVE-2026-42826 (CVSS 10.0, 2026-05-07) is an Azure DevOps service information disclosure, not MCP-specific — [CVE record](https://cveawg.mitre.org/api/cve/CVE-2026-42826)

### Inferences
- Pin local at v2.11.0 or later. By release dates, v2.10.0 is the first with spotlighting and v2.11.0 the first with the attachment fix.
- Spotlighting is only a delimiter hint to the model, so it does not remove the risk. Controls that do: project-scoped identity, no auto-approved writes, separate read and write stages, and egress limits.
- Local has no read-only switch, so enforce it through identity permissions and `-d` domains, or an allowlisting MCP gateway.
- With remote, always set a toolset, because `all` includes destructive `elm` actions.
- Claude Code on remote still depends on browser sign-in, so it fits developer desktops rather than headless agents.

### Gaps
- No documentation on service principal or client-credential use of remote MCP.
- Whether the hosted remote server received the same spotlighting fixes is not stated.
- No MSRC advisory text for MSRC-135517 was found.
- Local retirement date: none published.

## Microsoft native features on Services

### Takeaway
**Copilot coding agent does not support Azure Repos, and Microsoft says there are no current plans.** Boards to Copilot requires a GitHub repo. Copilot code review works on Azure Repos, but runs on Microsoft-hosted agents only, so it conflicts with the on-prem preference. For self-hosted capacity, budget $15 for each extra parallel job.

### Cited Findings
| Feature | Status | Constraint | Source |
|---|---|---|---|
| Copilot coding agent on Azure Repos | **Not supported** | Dan Hellem, 2026-10-01: "We currently do not have plans to integrate the GitHub cloud agent with Azure Repos." Move repos to GitHub for it | [Blog comment](https://devblogs.microsoft.com/devops/updates-to-copilot-code-reviews-for-azure-repos/#comment-5490) |
| Copilot for Azure Boards | GA, Dec 2025 to Jan 2026 rollout | Sends work items to Copilot, which drafts PRs in a GitHub repo | [Blog](https://devblogs.microsoft.com/devops/github-copilot-for-azure-boards/); [Sprint 268](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/boards/sprint-268-update) |
| Copilot code review, Azure Repos | Public preview | See below | [Sprint 279](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update); [Docs](https://learn.microsoft.com/en-us/azure/devops/repos/git/copilot-code-reviews?view=azure-devops) |
| Enterprise Live Migrations | Public preview | Azure Repos to GitHub Enterprise Cloud with data residency, keeping Boards and Pipelines | [Sprint 279](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update) |

**Copilot code review constraints** — [Docs, ms.date 2026-09-29](https://learn.microsoft.com/en-us/azure/devops/repos/git/copilot-code-reviews?view=azure-devops)
- Billed through Azure Cost Management on a linked subscription. Repos of 10 GB or less and PRs of 100 changed files or fewer.
- "Copilot always leaves a Comment review. It never approves", so it does not satisfy required reviewers or block merges. It does not re-review on push or read replies.
- **Conflict:** "Self-hosted agent pools and Managed DevOps Pools are not currently supported" in the docs, while Sprint 279 (2026-09-04) lists "Managed DevOps Pools support". The docs are newer. Self-hosted is excluded in both.

**Pricing** — [Azure DevOps pricing](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/) (Microsoft labels these estimates)
- Self-hosted parallel jobs: 1 free with unlimited minutes, $15 per extra. Basic: first 5 users free, then $6 per user per month (search extract). Basic + Test Plans: $52 per user per month (search extract).
- Advanced Security: **Code Security $30** and **Secret Protection $19** per active committer per month. Legacy customers keep the bundled product unless migrated through support, and committers are deduplicated per Azure subscription — [Billing](https://learn.microsoft.com/en-us/azure/devops/repos/security/github-advanced-security-billing?view=azure-devops). Autofix uses AI credits — [Sprint 279](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update)

**Parallel jobs** — [Parallel jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops)
- Charged by concurrent jobs, not agents registered: "you can register any number of self-hosted agents". "There are no time limits on self-hosted jobs."
- Private projects get one job, plus one per active Visual Studio Enterprise subscriber in the org.

**Test Plans**
- MCP exposes plans, suites, cases, runs and results — [Remote MCP](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server). The Azure DevOps API also accepts results from automation, as described in earlier notes.

### Inferences
- The factory cannot rely on Microsoft's coding agent with code in Azure Repos. Self-host CLI agents on pipelines, which matches the on-prem preference anyway.
- Copilot code review sends code to Microsoft-hosted compute. Treat it as optional, and prefer on-prem PR-Agent or similar as the primary reviewer.
- Concurrency budget: N simultaneous agents plus verifier jobs costs (N minus free jobs) × $15 per month. Application identities need their own Basic seats and do not get Visual Studio benefits. Count Visual Studio Enterprise subscribers first.
- Advanced Security, at $49 per committer per month for both, is optional if Semgrep, Trivy and Gitleaks-style scanning is run on-prem. Its push protection is the feature hardest to replicate outside the server. Note it keeps findings in Microsoft's cloud.
- Buy Test Plans seats only for people authoring manual tests. Automated results can be published without them, but license needs should be checked.

### Gaps
- Test Plans license requirements for read or automated publishing were not verified this session.
- Copilot code review's data-handling page was not found. Whether Copilot data terms cover this Azure-billed route is unconfirmed.
- Exact per-review cost was not verified here; [InfoQ](https://www.infoq.com/news/2026/09/copilot-code-review-azure-repos/) reports per-review billing.

## ado-aw and OpenHands

### Takeaway
ado-aw is an active but low-adoption, pre-1.0 Copilot-only project: copy its three-stage pattern, and pilot at most. OpenHands' Azure DevOps integration is real and Services-only, but needs a commercial Enterprise license beyond 30 days per year and inbound reachability for service hooks.

### Cited Findings
**ado-aw**
- [githubnext/ado-aw](https://github.com/githubnext/ado-aw): MIT, 33 stars, v0.54.0 (2026-10-27), last push 2026-11-06, 68 open issues. Commits come mostly from bots and one human (`jamesadevine`, 279). 30 listed releases since at least 2026-06-09 — GitHub API.
- Compiles Markdown into Agent, Detection and SafeOutputs stages; the write token is "Never exposed to the agent" — [README](https://github.com/githubnext/ado-aw)
- "Currently only `copilot` (GitHub Copilot CLI) is supported." BYOK provider config takes `base-url`, `type`, `wire-api`, `token` or `api-key` — [Engine](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md)
- Named agent pools are supported, and the standalone default is `vmImage: ubuntu-22.04`. AWF uses Squid plus Docker — [Targets](https://github.com/githubnext/ado-aw/blob/main/docs/targets.md)
- Runtime pulls from GitHub and `ghcr.io/github/...`, and docs address pools that cannot reach them, with mirrored binaries and checksums — [Supply chain](https://github.com/githubnext/ado-aw/blob/main/docs/supply-chain.md)

**OpenHands**
- Core [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands): MIT, 92,852 stars, v1.37.0 (2026-11-06). It has no `enterprise/` directory now (API returned 404), so older fork statements describing that path are stale.
- **Exact enterprise license:** [OpenHands/enterprise](https://github.com/OpenHands/enterprise) is "PolyForm Free Trial License 1.0.0", "**not** an open source license. Usage is limited to 30 days per calendar year without a commercial license." [OpenHands-Cloud](https://github.com/OpenHands/OpenHands-Cloud) carries the same license.
- Integration needs Enterprise (Replicated or Helm) and an Entra app with client secret. It uses **user-delegated OAuth**, and service principals are not mentioned. Outbound to `login.microsoftonline.com` and `dev.azure.com`. For Service Hook triggers, Azure DevOps "must reach" the OpenHands URL. `@openhands` mentions work in PR and work item comments, with per-org install. "Azure DevOps Server is not covered". The page states no GA or beta label — [OpenHands docs](https://docs.openhands.dev/enterprise/integrations/azure-devops)

### Inferences
- ado-aw needs a GitHub Copilot CLI engine, even under BYOK to a local model, so it adds GitHub CLI and container supply chain to the on-prem pool. Whether BYOK avoids a Copilot license was not confirmed.
- OpenHands' `@mention` flow requires a public inbound URL, which conflicts with the no-inbound stance. It could sit behind a relay, or the MIT agent SDK or CLI could be used from pipelines without the Enterprise integration.
- User-delegated OAuth means agents act as the person, which gives good attribution but makes service-identity automation weaker.

### Gaps
- Commercial pricing for OpenHands Enterprise is not public (contact form).
- No independent production report of ado-aw was found.
- Whether a self-hosted MIT OpenHands can receive Azure DevOps triggers through a custom dispatcher was not tested.

## Connectors reaching Services from on-prem

### Takeaway
All connectors can work outbound-only except where they need vendor SaaS (Semgrep PR comments). The key correction is **RAGFlow now has an Azure DevOps connector and Onyx does not**. Both new options unlocked on Services are Entra auth for Backstage and Semgrep PR comments.

### Cited Findings
| Tool | License / version | Services path | Auth | New compared with Server | Watch out |
|---|---|---|---|---|---|
| [RAGFlow](https://github.com/infiniflow/ragflow) | Apache-2.0; v1.0.0 (2026-11-06) | Native connector for Azure Repos source files and PRs, default `https://dev.azure.com`, custom base URL since 2026-09-16 — [source](https://github.com/infiniflow/ragflow/blob/main/internal/syncer/connector/azure_devops.go) | PAT — [UI](https://github.com/infiniflow/ragflow/blob/main/web/src/pages/user-setting/data-source/constant/azure-devops-constant.tsx) | Base URL suggests Server may also work (untested) | Open bug: [#20972](https://github.com/infiniflow/ragflow/issues/20972) sync fails on `content_hash` length, fix [#20973](https://github.com/infiniflow/ragflow/pull/20973) open; added 2026-08-25, so young |
| [Onyx](https://github.com/onyx-dot-app/onyx) | MIT except `ee` dirs under the Onyx Enterprise License — [LICENSE](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE); v5.1.0 | **No official connector.** Tree has only `azure_datalake`. [PR #6496](https://github.com/onyx-dot-app/onyx/pull/6496) closed unmerged 2026-10-16. Docs list GitHub and GitLab — [Docs](https://docs.onyx.app/admins/connectors/overview.md) | N/A | None | Custom connector or Git mirror needed |
| [Backstage](https://github.com/backstage/community-plugins) | Apache-2.0 | `PersonalAccessToken`, `ClientSecret`, `ManagedIdentity`, `ManagedIdentityClientAssertion` — [#35394](https://github.com/backstage/backstage/issues/35394) | Entra or org-scoped PAT | **Entra auth works on Services** but not Server | Set `organizations` on PATs; maintainers discuss deprecating Services PATs. Issue's Dec 1 date is stale |
| SonarQube Server | Commercial; Community Build | Compatible with both products, with an instance PAT and project PAT — [Sonar](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration) | PAT | Same | PR decoration needs Developer, Enterprise or Data Center; "Community Build is not supported" — [Sonar docs assistant](https://docs.sonarsource.com/sonarqube-server/discovering/code-analysis/pull-request-analysis.md) |
| Semgrep | Commercial platform for PR comments | "Semgrep currently supports repositories hosted by Azure DevOps Cloud." — [Semgrep](https://semgrep.dev/docs/semgrep-appsec-platform/azure-pr-comments) | PAT, Full access at setup, then narrower | **PR comments work on Services**, not Server | Requires `SEMGREP_APP_TOKEN` and the SaaS platform, adding a cloud dependency |
| [Trivy task](https://github.com/aquasecurity/trivy-azure-pipelines-task) | MIT; v1.20.7 (2026-09-23); [Trivy](https://github.com/aquasecurity/trivy) Apache-2.0, v0.76.2 | Official Marketplace task with results UI — [README](https://github.com/aquasecurity/trivy-azure-pipelines-task) | Pipeline job | Marketplace install is direct | Docker support is fully supported only on Linux agents |
| ZAP | [zaproxy](https://github.com/zaproxy/zaproxy) Apache-2.0; v2.17.0 (2025-12-15) | Community extensions such as [CSE-DevOps](https://marketplace.visualstudio.com/items?itemName=CSE-DevOps.zap-scanner) and [PSG](https://marketplace.visualstudio.com/items?itemName=solucionespsg.owasp-zap-scan-psg) | N/A | Same | No official ZAP Azure task found; Docker CLI step preferred |

### Inferences
- Prior notes' "global PAT plus project PAT" for SonarQube means Sonar's instance-level setting, not a global-scoped token. Verify it is org-scoped before January 27, 2027.
- RAG recommendation: for repos and PRs, RAGFlow's connector can be used after the bug fix. For work items and wiki, the connector's described scope ("source files and pull requests") does not cover them, so write a small indexer. The wiki is also a Git repo, as earlier notes show.
- Onyx on Services needs a custom connector built on the REST API. Reading closed PR #6496 could save effort, but its code was not reviewed.
- Trivy needs vulnerability DB access or a mirror. ZAP scans should run on-prem against on-prem test targets. Results stay local if posted back only as summaries (background on the DB mirror).
- Avoid full-access PATs for connectors. Use a dedicated Basic-access identity, and read-only scopes where supported.
- No connector requires inbound access for basic pull or CI operation, so these fit outbound-only networking.

### Gaps
- Exact RAGFlow scope for work items or wiki was inferred from a code comment rather than a full code review. Also, no public test against a real org was found.
- SonarQube edition facts came from its docs assistant endpoint, not a static page.
- Semgrep and Sonar Entra support and org-scoped PAT behaviour were not verified.
- No current Backstage docs page was retrieved directly; credential kinds come from the issue.
- Marketplace extension maintenance for ZAP was not verified; no 2026 changelog was found.
