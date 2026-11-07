# Knowledge Base / Wiki and PM Requirements (URD/PRD) Tooling for a Multi-Department AI Software Factory (as of 2026-11-07)

Method note: GitHub stars, SPDX license, last push and latest release were pulled directly from the GitHub REST API on 2026-11-07. GitHub reports "NOASSERTION" for custom or mixed licenses, so I read the LICENSE text itself for those. Items resting only on third-party sources are labelled. Pricing was read from vendor pages where possible.

## Which options should be shortlisted?

### Takeaway
No single product covers this. The best-evidenced combination is a hybrid: a wiki for PM discovery (Outline first, BookStack if permissive licensing is mandatory, Confluence Cloud or Notion if SaaS is acceptable), Git-versioned specs with requirement IDs for approved requirements (Spec Kit or OpenSpec, plus StrictDoc or Sphinx-Needs where audit-grade traceability is needed), and an ACL-aware retrieval layer (Onyx) in front of agents. Traditional Chinese search is the largest unverified risk and needs a hands-on test before any wiki is selected.

### Cited Findings
Evidence behind each pick is cited in the sections below. The strongest disqualifiers or conditions found:
- GitNexus is PolyForm Noncommercial, which is a barrier for enterprise use — [LICENSE](https://github.com/abhigyanpatwari/GitNexus/blob/main/LICENSE)
- AFFiNE's backend may be used in production only with an AFFiNE subscription or other agreement — [backend LICENSE](https://github.com/toeverything/AFFiNE/blob/canary/packages/backend/server/LICENSE)
- Dify defines a tenant as one workspace and requires written authorization to run a multi-tenant environment from its source — [LICENSE](https://github.com/langgenius/dify/blob/main/LICENSE)
- Docmost's maintainer says CJK-capable search (Typesense driver) is in the enterprise edition — [issue #1477](https://github.com/docmost/docmost/issues/1477)
- Onyx permission sync needs a Business, Enterprise or Cloud plan — [Onyx connector docs](https://docs.onyx.app/admins/connectors/overview.md)
- Confluence Data Center becomes read-only on 2029-03-28, barring exceptions — [Atlassian](https://www.atlassian.com/licensing/data-center-end-of-life)

### Inferences
Recommended shortlist (my judgment, based on the evidence in this file):

| Layer | Self-hosted first choice | Self-hosted alternate | SaaS | Watch or avoid |
|---|---|---|---|---|
| PM wiki | Outline (BSL; built-in MCP) | BookStack (MIT; OIDC group-to-role sync; community MCP only) or Docmost Business (paid SSO, MCP, granular permissions) | Confluence Cloud + Rovo MCP, or Notion | Wiki.js 3 (watch MCP); AFFiNE (license); AppFlowy, SiYuan, Logseq (no department-permission evidence found) |
| Enterprise retrieval | Onyx (paid tier if ACL sync is required) | RAGFlow (document-heavy, team-scoped) | Onyx Cloud, Rovo Search | Open WebUI, AnythingLLM (permissions not verified); Khoj (release activity has slowed) |
| App building on knowledge | Dify (with licensing review) | None evaluated | Dify Cloud (plans not verified) | Running one Dify workspace per department without a commercial license |
| Knowledge graph or memory | Cognee (built-in tenancy) | Graphiti, LightRAG as engines behind your own authorization layer | Not evaluated | Exposing Graphiti's reference servers directly |
| Code knowledge | codebase-memory-mcp + Repomix | deepwiki-open (features unverified) | Devin/DeepWiki MCP; Sourcegraph MCP | GitNexus unless commercially licensed |
| Specs and traceability | Spec Kit or OpenSpec in repo | StrictDoc, Sphinx-Needs, Doorstop for audit-grade traceability | Kiro; ChatPRD for drafting | BMAD v7 until stable |
| Work tracking | Plane or OpenProject | Jira Data Center only with a migration plan | Jira + Rovo; Linear | None |

### Gaps
- No tool here was hands-on tested. Rankings rest on documents, licenses and repository data.
- No credible independent, enterprise-grade benchmark comparing these products on Traditional Chinese was found.

## Self-hostable and SaaS wiki/knowledge platforms: license, activity, permissions, MCP, API

### Takeaway
Outline is the only self-hostable wiki I verified with a documented, vendor-provided MCP server available to self-hosted installs and groups on every tier, but it is source-available (BSL), not open source. Docmost is the closest alternative, but its SSO, MCP and page-level permissions sit behind a paid tier. BookStack is the cleanest license with solid SSO but only community MCP servers. Every self-hosted candidate has CJK search doubts that must be tested.

### Cited Findings
Repository data (GitHub API, 2026-11-07):

| Product | Stars | Exact license | Last push | Latest release |
|---|---|---|---|---|
| [Outline](https://github.com/outline/outline) | 41,194 | [BSL 1.1](https://github.com/outline/outline/blob/main/LICENSE); converts to Apache-2.0 on 2030-10-23 | 2026-11-07 | v1.12.0 (2026-10-23) |
| [BookStack](https://github.com/BookStackApp/BookStack) | 19,131 | MIT | 2026-11-06 | v26.09.2 (2026-10-20) |
| [Wiki.js](https://github.com/requarks/wiki) | 29,200 | AGPL-3.0 | 2026-11-07 | 3.1.1 (2026-11-06) |
| [Docmost](https://github.com/docmost/docmost) | 22,249 | AGPL-3.0 for Open Source; paid editions need a key | 2026-11-06 | v0.97.0 (2026-10-25) |
| [AFFiNE](https://github.com/toeverything/AFFiNE) | 74,282 | [Mixed: MIT outside backend/native; EE license for backend](https://github.com/toeverything/AFFiNE/blob/canary/LICENSE) | 2026-11-06 | v0.27.4 (2026-08-18) |
| [AppFlowy](https://github.com/AppFlowy-IO/AppFlowy) | 78,340 | AGPL-3.0 | 2026-11-02 | 0.15.0 (2026-11-02) |
| [SiYuan](https://github.com/siyuan-note/siyuan) | 47,109 | AGPL-3.0 | 2026-11-06 | v3.8.9 (2026-10-29) |
| [Logseq](https://github.com/logseq/logseq) | 45,502 | AGPL-3.0 | 2026-11-07 | 2.0.2 (2026-10-07) |
| [PandaWiki](https://github.com/chaitin/PandaWiki) (found during research, not evaluated) | 10,475 | AGPL-3.0 | 2026-11-05 | Not checked |

Outline:
- The BSL Additional Use Grant allows use except for a "Document Service", defined as a commercial offering letting third parties other than your employees and contractors create teams and documents — [LICENSE](https://github.com/outline/outline/blob/main/LICENSE)
- Self-hosted pricing: Community free with limited features and no support; Business $4/user/month (10-user minimum); Enterprise $5/user/month (100-user minimum). Cloud is flat-rate: $10 (1-10 members), $79 (11-100), $249 (101-200), custom above. All billed as described on the page — [Outline pricing](https://www.getoutline.com/pricing)
- SSO, groups, user-level permissions and API plus webhooks are on all tiers; SAML 2.0 is self-hosted Business and Enterprise only — [Outline pricing](https://www.getoutline.com/pricing)
- Every workspace has built-in MCP since the 2026-02-18 changelog; assistants can search, read, create and edit documents — [changelog](https://www.getoutline.com/changelog/mcp)
- Self-hosted installs use `https://<your-domain>/mcp`; Streamable HTTP only; OAuth by default or API key; admins toggle it in Settings, Workspace, AI — [Outline MCP docs](https://docs.getoutline.com/s/guide/doc/mcp-6j9jtENNKL)
- The community server mcp-outline tells users to prefer the official one — [PyPI](https://pypi.org/project/mcp-outline), [README mirror](https://cdn.jsdelivr.net/gh/vortiago/mcp-outline@main/README.md)

Docmost:
- Three editions: Open Source (AGPL-3.0), Business and Enterprise; paid keys are validated offline — [editions](https://docmost.com/docs/editions)
- Business is $6/seat/month billed annually with a 10-seat minimum; Enterprise is custom. MCP, API keys, SSO (SAML/OIDC/LDAP), page-level permissions, attachment search and AI are shown for Business and Enterprise; SCIM, audit logs and SIEM are Enterprise — [pricing](https://docmost.com/pricing). The comparison table had unclear tier markers for some rows, so verify before quoting.
- A third-party aggregator lists $4/user/month — [xpay](https://www.xpay.sh/saas-pricing/docmost/); this conflicts with the vendor page, which I treat as authoritative.

BookStack:
- OIDC login with group-to-role sync through `OIDC_GROUPS_CLAIM`; the same docs link to LDAP and SAML pages — [OIDC docs](https://www.bookstackapp.com/docs/admin/oidc-auth/)
- Development has moved: the maintainer says BookStack is now managed on Codeberg and GitHub issues were closed — [issue #778 comment, 2026-04-28](https://github.com/BookStackApp/BookStack/issues/778). GitHub stars and issue data may therefore understate activity.
- MCP is community only: [pnocera/bookstack-mcp-server](https://github.com/pnocera/bookstack-mcp-server) (101 stars, MIT) and [ttpears/bookstack-mcp](https://github.com/ttpears/bookstack-mcp) (51 stars, MIT).

Wiki.js:
- Open issue #8140 shows "Implement MCP endpoint" and PAT scopes checked, with documentation unchecked — [issue #8140](https://github.com/requarks/wiki/issues/8140). I could not confirm this has shipped in 3.1.1.
- Community servers include [talosdeus/wiki-js-mcp](https://github.com/talosdeus/wiki-js-mcp) (50 stars, last push 2025-06-01, stale) and [jaalbin24/wikijs-mcp](https://github.com/jaalbin24/wikijs-mcp) (19 stars, no license declared).

AFFiNE:
- Backend terms say the software "may only be used in production" with the subscription terms or another agreement and a valid subscription for the correct seat count — [backend LICENSE](https://github.com/toeverything/AFFiNE/blob/canary/packages/backend/server/LICENSE)
- An open feature request asks for "MCP write access" — [issue #15652](https://github.com/toeverything/AFFiNE/issues/15652); another reports a credentials update breaking a community server — [issue #15251](https://github.com/toeverything/AFFiNE/issues/15251).

AppFlowy, SiYuan, Logseq, Obsidian:
- AppFlowy has open requests for an MCP server — [#8043](https://github.com/AppFlowy-IO/AppFlowy/issues/8043) and for MCP API tokens — [#8776](https://github.com/AppFlowy-IO/AppFlowy/issues/8776).
- SiYuan lists Docker deployment and a documented API — [README](https://github.com/siyuan-note/siyuan); MCP is available through community work such as [siyuan-plugins-mcp-sisyphus](https://github.com/yangtaihong59/siyuan-plugins-mcp-sisyphus) (123 stars, MIT).
- Logseq has an open PR for a local MCP server — [#13622](https://github.com/logseq/logseq/pull/13622); community option [ergut/mcp-logseq](https://github.com/ergut/mcp-logseq) (350 stars, MIT).
- Obsidian: the public [obsidian-releases](https://github.com/obsidianmd/obsidian-releases) repository has no declared license; community MCP [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) has 4,523 stars, MIT, last push 2026-08-31.

SaaS:
- Atlassian Rovo MCP: endpoint `https://mcp.atlassian.com/v2/mcp`; covers Jira, Confluence, JSM, Bitbucket Cloud and more; OAuth 2.1 or API token; "never grants access beyond what the user already has"; documented for Atlassian Cloud — [Atlassian developer docs](https://developer.atlassian.com/cloud/rovo-mcp/)
- GA on 2026-02-04 is reported only by a third party — [dev.to](https://dev.to/grove_chatforest/atlassian-mcp-server-jira-and-confluence-access-for-ai-agents-2158). A forum thread reports some MCP features depending on Rovo Dev licensing, and a moderator says this is not clearly documented — [Atlassian Community](https://community.atlassian.com/forums/Rovo-questions/MCP-server-only-working-with-Rovo-Dev-License/qaq-p/3188274).
- Community alternative [sooperset/mcp-atlassian](https://github.com/sooperset/mcp-atlassian): 6,093 stars, MIT, last push 2026-09-19.
- Confluence Data Center: new customer sales ended 2026-03-30; existing customers cannot buy expansions after 2028-03-30; read-only from 2029-03-28, with extensions by exception only — [Atlassian](https://www.atlassian.com/licensing/data-center-end-of-life)
- Notion MCP: `https://mcp.notion.com/mcp`, OAuth; bearer tokens for headless use only on Business and Enterprise; reads and updates what the authorizing user can access; guests refused; the local open-source server is "no longer actively maintained" — [Notion docs](https://developers.notion.com/guides/mcp/get-started-with-mcp). This is a staleness flag even though [makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server) still has 4,700 stars and recent pushes.
- Notion pricing is inconsistent across third-party blogs ($16-$20 for Business; SSO tier disputed) — e.g. [automationatlas](https://automationatlas.io/answers/notion-pricing-explained-2026/); I did not get an official figure.
- GitBook: published sites expose a read-only MCP server at `/~gitbook/mcp` — [GitBook docs](https://gitbook.com/docs/ai-and-search/mcp-servers-for-published-docs); its pricing page lists this across plans — [GitBook pricing](https://gitbook.com/pricing); a competitor says it is Ultimate-only — [Docsie](https://www.docsie.io/vs/document360-vs-gitbook-pricing). The vendor page is the better source.

CJK evidence:
- Outline: "Search should support CJK" (opened 2022) was auto-closed for inactivity on 2025-09-21, not visibly fixed — [issue #3207](https://github.com/outline/outline/issues/3207). Recent merged work covers [CJK fallback fonts](https://github.com/outline/outline/pull/13811), [CJK line height](https://github.com/outline/outline/pull/13822) and [Pinyin slugs](https://github.com/outline/outline/pull/14375), which shows attention to CJK display but not search.
- Docmost: an issue states the default English PostgreSQL tokenizer breaks mid-sentence Chinese and Japanese search; the maintainer replied that enterprise has a Typesense driver supporting CJK; another commenter questioned its Japanese quality — [issue #1477](https://github.com/docmost/docmost/issues/1477). A community PR to index Chinese closed unmerged — [PR #497](https://github.com/docmost/docmost/pull/497).
- BookStack: "Chinese search cannot find words in the middle of a sentence" dates from 2018 and was migrated to Codeberg — [issue #778](https://github.com/BookStackApp/BookStack/issues/778). Current behavior unverified.
- AppFlowy: a fix for CJK substring local search is still an open PR — [PR #9110](https://github.com/AppFlowy-IO/AppFlowy/pull/9110).
- Wiki.js: only old issues surfaced, such as [#520](https://github.com/requarks/wiki/issues/520). Current behavior unverified.

### Inferences
- Internal multi-department use appears to fall outside Outline's "Document Service" restriction, because users are employees and contractors. Use by external customers or subsidiaries could differ, so legal should confirm.
- Assume PostgreSQL-backed wikis fail mid-sentence Traditional Chinese search until proven otherwise. A sidecar index or the RAG layer may be required regardless of the wiki chosen. Test exact phrases, 2-character terms, and mixed Chinese-English terms.
- AFFiNE's mixed license makes it unsafe to describe as "MIT" despite its popularity. Treat free production self-hosting as a legal question.
- SiYuan, Logseq and Obsidian look better suited to individuals or Git-backed authoring than to governed department spaces.
- GitBook's MCP is for published docs, so it is a poor fit for in-progress internal URDs.
- A built-in MCP toggle at workspace level is not per-agent governance. A gateway or per-agent service accounts will probably be needed.

### Gaps
- Outline's docs do not state per-document MCP permission behavior, or whether Community Edition includes MCP. The API-key and OAuth models suggest user-scoped access, but this is unconfirmed.
- BookStack API quality and Docmost Community API availability were not verified.
- AppFlowy Cloud's server license and team permissions were not checked.
- Whether SiYuan or Logseq offer department-style permissions: no evidence found.
- Whether sooperset/mcp-atlassian supports Data Center was not verified in this session.
- Exact Confluence and Notion pricing and CJK search quality were not verified from official pages.
- Multi-department "spaces" were not compared feature by feature for Outline collections or BookStack shelves.

## AI knowledge and RAG engines: enterprise fit, multi-tenant permissions, connectors, ACL-aware retrieval

### Takeaway
Onyx has the clearest documented source-ACL inheritance (16 connectors) and an MCP server that advertises the caller's access controls, but ACL sync and group scoping are paid. RAGFlow and Dify offer team-scoped sharing rather than verified source-document ACL inheritance. Graph and memory engines are components, not enterprise knowledge platforms; only Cognee documents built-in tenancy.

### Cited Findings
Repository data (GitHub API, 2026-11-07):

| Engine | Stars | Exact license | Last push | Latest release |
|---|---|---|---|---|
| [Dify](https://github.com/langgenius/dify) | 160,581 | [Modified Apache-2.0 with extra conditions](https://github.com/langgenius/dify/blob/main/LICENSE) | 2026-11-07 | 1.19.0 (2026-11-04) |
| [Open WebUI](https://github.com/open-webui/open-webui) | 157,956 | [Custom Open WebUI License with branding clause](https://github.com/open-webui/open-webui/blob/main/LICENSE) | 2026-11-07 | v0.13.0 (2026-11-03) |
| [RAGFlow](https://github.com/infiniflow/ragflow) | 93,700 | Apache-2.0 | 2026-11-06 | v1.0.0 (2026-11-06) |
| [Mem0](https://github.com/mem0ai/mem0) | 68,532 | Apache-2.0 | 2026-11-06 | ts-v3.3.1 (2026-09-25) |
| [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) | 68,015 | MIT | 2026-11-07 | v1.19.0 (2026-10-26) |
| [LightRAG](https://github.com/HKUDS/LightRAG) | 40,587 | MIT | 2026-11-05 | v1.5.7 (2026-09-02) |
| [Khoj](https://github.com/khoj-ai/khoj) | 38,030 | AGPL-3.0 | 2026-08-02 (stale signal) | 2.0.0-beta.28 (2026-03-26) |
| [GraphRAG](https://github.com/microsoft/graphrag) | 36,583 | MIT | 2026-11-02 | v3.3.0 (2026-10-08) |
| [Onyx](https://github.com/onyx-dot-app/onyx) | 33,260 | [MIT for CE; Onyx Enterprise License for `ee` directories](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE) | 2026-11-07 | v5.1.0 (2026-11-06) |
| [Cognee](https://github.com/topoteretes/cognee) | 33,114 | Apache-2.0 | 2026-11-06 | v1.6.7 (2026-11-06) |
| [Graphiti](https://github.com/getzep/graphiti) | 32,606 | Apache-2.0 | 2026-11-06 | v0.31.2 (2026-10-20) |
| [GPT Researcher](https://github.com/assafelovic/gpt-researcher) | 30,524 | Apache-2.0 | 2026-10-10 | Not retrieved |
| [WeKnora](https://github.com/Tencent/WeKnora) (found during research, not evaluated) | 36,693 | Custom notice; not classified by GitHub, verify | 2026-11-03 | Not checked |

Onyx:
- 56 official connectors; 16 with permission sync: Google Drive, Gmail, Confluence, Jira, Slack (indexed only), GitHub, Box, Canvas, SharePoint (certificate auth), OneDrive, Teams, Outlook, Linear, HubSpot, YouTrack, Zoom — [Onyx docs](https://docs.onyx.app/admins/connectors/overview.md)
- Permission sync requires Business, Enterprise or Cloud; "Specific Groups" access requires Business or higher — [Onyx docs](https://docs.onyx.app/admins/connectors/overview.md)
- Permission matching is based on user email, and a public connector pointed at private data exposes it to all Onyx users — [permissioning docs](https://docs.onyx.app/enterprise_edition/permissioning)
- Business adds multiple OIDC/SAML providers (one works in every edition), user groups and permission sync; Enterprise adds custom admin roles and SCIM — [edition docs](https://docs.onyx.app/admins/getting_started/enterprise_edition.md)
- README: an MCP server lets Claude Code, Codex or other clients get company context "with the same access controls as the person running them" — [README](https://github.com/onyx-dot-app/onyx)

Dify:
- One tenant equals one workspace; operating a multi-tenant environment from source needs written authorization; frontend logo and copyright cannot be removed — [LICENSE](https://github.com/langgenius/dify/blob/main/LICENSE)
- Knowledge base access has three levels: Only Me, All team members, Partial team members, and authorized users can view, edit and delete — [Dify Enterprise docs 3.0.x](https://enterprise-docs.dify.ai/en/3.0.x/use/knowledge-base/knowledge-and-documents-maintenance/introduction)
- SSO and RBAC are positioned under Dify Enterprise — [README](https://github.com/langgenius/dify)
- A community MCP server warns that one Knowledge Base API key can access all visible datasets, so its allowlist is client-side — [dify-mcp on Glama](https://glama.ai/mcp/servers/salted-butter-joshua/dify-mcp)

RAGFlow:
- Permissions have layers for team membership, resource sharing scope and operation rights — [RAGFlow docs](https://ragflow.io/docs/permission_system_overview). I saw this in a search excerpt; a later direct fetch returned 404, so re-verify.
- An official MCP server exists; in one launch mode it reaches only a specified tenant's datasets; default port 9382 — [RAGFlow MCP docs](https://ragflow.io/docs/dev/launch_mcp_server)

Open WebUI:
- Branding must be preserved unless the deployment has 50 or fewer users in a 30-day window, the contributor has written permission, or an enterprise license allows it; effective from v0.6.6 — [Open WebUI license docs](https://docs.openwebui.com/license/)

Cognee, Graphiti, LightRAG, Mem0:
- Cognee: `ENABLE_BACKEND_ACCESS_CONTROL` enables tenants, roles and dataset permissions (read, write, delete, share); multi-tenant mode enforces Kùzu and LanceDB — [Cognee docs](https://docs.cognee.ai/core-concepts/multi-user-mode/permissions-system/overview). A third-party listing places permissions in a paid tier — [aitoolsatlas](https://aitoolsatlas.ai/compare/cognee-vs-mem0); this conflict is unresolved.
- Graphiti's reference servers "do not authenticate callers", and any caller can read, change or delete data in any `group_id` — [README](https://github.com/getzep/graphiti)
- LightRAG's README says to configure authentication before network exposure and states that hiding the admin UI is "not a security boundary" — [README](https://github.com/HKUDS/LightRAG)
- Mem0 describes multi-level memory for user, session and agent state — [README](https://github.com/mem0ai/mem0)
- A single Chinese-language comparison marks Graphiti and LightRAG as not multi-user and Mem0 as multi-user — [CSDN](https://aicoding.csdn.net/6a321d2910ee7a33f27e70d6.html) (secondary).

### Inferences
- If document-level ACL fidelity matters, budget for paid Onyx or build ACL filtering yourself. Onyx CE is suitable for pilots with non-sensitive data only.
- Onyx's permission-sync list includes Confluence and Jira but not Outline, BookStack or Docmost. A self-hosted wiki will likely need group-scoped or custom ACL handling, which favors Confluence in strict ACL settings.
- A department-per-workspace Dify design is the obvious way to trigger its license condition. Get a commercial license or use a single workspace, and have legal confirm.
- Graph engines add value for relationship-heavy retrieval (requirements, decisions, entities), but they should sit behind an authorization gateway.
- GPT Researcher fits report generation during discovery, not knowledge base hosting.

### Gaps
- AnythingLLM's permissions docs returned 403, so multi-user roles and document-level ACLs are unverified.
- Open WebUI's RBAC docs URL returned 404, so group and knowledge access control are unverified.
- Whether Onyx offers connectors for Outline, BookStack or Docmost was not checked. Onyx plan prices were not retrieved.
- RAGFlow and Onyx Traditional Chinese retrieval quality: no reliable benchmark found. RAGFlow's CJK parsing was not verified from a primary source.
- No evidence found of RAGFlow or Dify inheriting ACLs from source systems. This is absence of evidence, not proof of absence.
- Mem0 and GraphRAG access-control documentation was not found in primary sources.

## Codebase-knowledge tools: how they complement the PM wiki

### Takeaway
Code-knowledge tools answer "what exists and how it works", while the PM wiki answers "what should exist and why". They should be connected by requirement IDs rather than merged. Licensing and hosting vary greatly, and popular-looking GitNexus is noncommercial.

### Cited Findings
- DeepWiki MCP: `https://mcp.deepwiki.com/mcp`, free, no auth, public repositories only; Devin MCP at `https://mcp.devin.ai/mcp` needs an API key and covers private repos; enterprise keys need `X-Org-Id` — [DeepWiki MCP docs](https://docs.devin.ai/work-with-devin/deepwiki-mcp), [Devin MCP docs](https://docs.devin.ai/work-with-devin/devin-mcp.md)
- [deepwiki-open](https://github.com/AsyncFuncAI/deepwiki-open): 18,372 stars, MIT, last push 2026-11-06. It is an independent project, and its current feature set was not verified.
- Sourcegraph: the main repo is no longer public. [sourcegraph-public-snapshot](https://github.com/sourcegraph/sourcegraph-public-snapshot) is archived (last push 2024-09-02) and [cody-public-snapshot](https://github.com/sourcegraph/cody-public-snapshot) is archived (last push 2025-08-01). [Zoekt](https://github.com/sourcegraph/zoekt) (Apache-2.0, 2,036 stars) remains active.
- One competitor's blog says Sourcegraph is a single Enterprise plan from $16K/year, Cody Free and Pro ended, Amp is separate, Deep Search is metered at three per seat per month, and MCP is Enterprise-only — [Unblocked](https://getunblocked.com/blog/sourcegraph-alternatives/). An aggregator instead reads "$16000/user/yr" — [aitoolsatlas](https://aitoolsatlas.ai/tools/sourcegraph-cody/pricing). I did not verify any of this on Sourcegraph's own pricing page; see also the [changelog](https://sourcegraph.com/changelog).
- [Context7](https://github.com/upstash/context7): 64,299 stars, MIT, but the README says the API backend, parsing engine and crawling engine are private; hosted at `https://mcp.context7.com/mcp` with an API key.
- [Repomix](https://github.com/yamadashy/repomix): 29,173 stars, MIT, last push 2026-11-07.
- [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp): 48,245 stars, MIT, last push 2026-11-04.
- [GitNexus](https://github.com/abhigyanpatwari/GitNexus): 48,311 stars, [PolyForm Noncommercial 1.0.0](https://github.com/abhigyanpatwari/GitNexus/blob/main/LICENSE), last push 2026-11-06.

### Inferences
- Suggested split: the wiki and specs supply intent; DeepWiki, Devin or Sourcegraph supply architecture and search; a code-graph MCP supplies call and dependency relationships; Context7 supplies third-party library docs, not company knowledge; Repomix supplies one-off context packs.
- Agents can reconcile the layers by checking a requirement ID in the spec against the code and tests found through code search, flagging any drift.
- Context7's MIT badge does not make it self-hostable as a whole, since the backend is private. Check data-handling terms before sending it internal queries.
- Star counts for the two code-graph projects are not evidence of production maturity. I did not assess quality or star authenticity.

### Gaps
- Sourcegraph's official pricing, MCP docs and self-hosted licensing were not read directly.
- Whether DeepWiki-generated wikis can be self-hosted, or support Chinese output, in the commercial product was not found.
- deepwiki-open's MCP, private-repo and local-model support could not be confirmed from a README query that returned no matches.
- Context7 private-repo and enterprise terms were not verified.

## Requirements tools: AI PRD/URD generation, spec tooling, traceability from requirement IDs to code and tests

### Takeaway
None of the tools verified here enforces end-to-end requirement-to-code-to-test traceability on its own in mainstream AI workflows. Spec Kit has the best-verified ID scheme and coverage analysis (requirement to task), and StrictDoc, Sphinx-Needs and Doorstop are stronger for formal linkage but have far smaller communities. Plan on a CI convention to close the code and test links.

### Cited Findings
Repository data (GitHub API, 2026-11-07):

| Tool | Stars | Exact license | Last push | Latest release |
|---|---|---|---|---|
| [GitHub Spec Kit](https://github.com/github/spec-kit) | 148,212 | MIT | 2026-11-07 | v1.3.2 (2026-11-06) |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 74,176 | MIT | 2026-11-07 | v1.16.1 (2026-11-07) |
| [Plane](https://github.com/makeplane/plane) | 62,094 | AGPL-3.0 | 2026-11-06 | v1.4.3 (2026-10-28) |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 55,085 | [MIT, with added attribution text](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/LICENSE) | 2026-11-06 | v6.12.4 (2026-10-14) |
| [OpenProject](https://github.com/opf/openproject) | 17,061 | GPL-3.0 | 2026-11-07 | v17.9.6 (2026-10-30) |
| [Doorstop](https://github.com/doorstop-dev/doorstop) | 693 | [LGPL-3.0](https://github.com/doorstop-dev/doorstop/blob/develop/LICENSE.md) | 2026-11-04 | v3.2 (2026-07-10) |
| [StrictDoc](https://github.com/strictdoc-project/strictdoc) | 448 | [Apache-2.0](https://github.com/strictdoc-project/strictdoc/blob/main/LICENSE) | 2026-11-06 | 0.30.4 (2026-10-22) |
| [Sphinx-Needs](https://github.com/useblocks/sphinx-needs) | 321 | MIT | 2026-11-07 | Latest tag is a sub-package, verify |

Spec-driven tools:
- Spec Kit's spec template uses `FR-001` functional requirements, `SC-001` success criteria, prioritized user stories and `[NEEDS CLARIFICATION]` markers — [spec-template](https://github.com/github/spec-kit/blob/main/templates/spec-template.md)
- Its task template uses `T010 [P] [US1] Contract test for [endpoint] in tests/contract/...`, tying tasks to stories and test paths — [tasks-template](https://github.com/github/spec-kit/blob/main/templates/tasks-template.md)
- `/speckit.analyze` maps tasks to requirements by inference or explicit IDs and outputs a coverage table of requirement key, has-task and task IDs, treating some zero-coverage requirements as critical — [analyze command](https://github.com/github/spec-kit/blob/main/templates/commands/analyze.md)
- OpenSpec uses plain Markdown `### Requirement:` and `#### Scenario:` with changes archived after completion — [README](https://github.com/Fission-AI/OpenSpec); its delta model (ADDED, MODIFIED, REMOVED) is described by a secondary source — [issoh.co.jp](https://www.issoh.co.jp/md/tech/8732)
- BMAD: stable line v6.12, with V7 in preview alpha whose skills, install path and docs "change often", ETA mid-November 2026 — [README](https://github.com/bmad-code-org/BMAD-METHOD). It has a BRD template and PRD skill — [template-brd](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/skills/bmod-method/doc-templates/template-brd.md), [PRD checklist](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/skills/bmad-prd/assets/prd-validation-checklist.md), and an enterprise testing add-on, listed in the README.
- Kiro specs produce requirements.md, design.md and tasks.md; Analyze Requirements flags gaps; property-based "Correctness" testing is IDE only per the capability table — [Kiro docs](https://kiro.dev/docs/specs/). Requirements use EARS: "WHEN ... THE SYSTEM SHALL" — [feature specs](https://kiro.dev/docs/specs/feature-specs/). Requirement-to-task traceability claims come from a vendor-adjacent blog — [codemyspec](https://codemyspec.com/blog/kiro-vs-openspec).

Formal traceability:
- Doorstop stores each item (requirement, test case, etc.) as a YAML file, builds a document tree, validates traceability and publishes — [README](https://github.com/doorstop-dev/doorstop)
- StrictDoc supports File relations from requirements to code, source traceability sections, ReqIF, and experimental JUnit, Robot Framework and nextest report support; parsing SDoc nodes from comments is experimental, C/C++ and Rust only — [User Guide](https://strictdoc.readthedocs.io/en/stable/stable/docs/strictdoc_01_user_guide.html)
- Sphinx-Needs lets docs define and link requirements, specs and test cases, with import and export for JIRA, GitHub and spreadsheets; sphinx-test-reports imports XML results as linkable needs — [docs](https://sphinx-needs.readthedocs.io/en/latest/). Companions are small: [sphinx-test-reports](https://github.com/useblocks/sphinx-test-reports) (36 stars), [sphinx-codelinks](https://github.com/useblocks/sphinx-codelinks) (5 stars).

AI PRD and PM systems:
- ChatPRD: Free; Pro $15/month or $179/year with hosted MCP and REST API; Teams $29/seat/month or $349/seat/year; Enterprise custom, with SSO. Linear and Notion integrations are listed; Confluence not named — [pricing](https://www.chatprd.ai/pricing)
- Linear MCP: `https://mcp.linear.app/mcp`, read-only variant at `/mcp/readonly`; OAuth 2.1 or API key; issues, projects, comments; SSE deprecated — [Linear docs](https://linear.app/docs/mcp)
- Aha! announced an MCP server on 2026-05-20, available to all customers — [Aha! blog](https://www.aha.io/blog/introducing-the-aha-software-mcp-server)
- Productboard: official help describes Spark as an MCP client — [Productboard help](https://support.productboard.com/hc/en-us/articles/48218564085139). A vendor blog claims an official server — [Storyflow](https://storyflow.so/blog/best-mcp-servers-for-product-managers-2026); I could not confirm this. Community servers exist, such as [productboard-mcp](https://glama.ai/mcp/servers/kenjihikmatullah/productboard-mcp).
- OpenProject MCP: Enterprise add-on on Professional, Premium or Corporate plans; `/mcp`; work packages, relations, projects and more; runs with the user's permissions; wiki pages not listed — [OpenProject docs](https://www.openproject.org/pt/docs/system-admin-guide/integrations/mcp-server/). Community CE option: [openproject-ce-mcp](https://pypi.org/project/openproject-ce-mcp/).
- Plane: official [plane-mcp-server](https://github.com/makeplane/plane-mcp-server) (357 stars, MIT); self-hosted suitability is claimed by a third-party guide — [mcp.directory](https://mcp.directory/blog/plane-mcp-complete-guide-2026).
- Jira: a vendor blog says acceptance criteria usually sit in description text that nothing enforces — [TitanApps](https://titanapps.io/blog/?p=10624).

### Inferences
Recommended traceability convention (my proposal, not a standard from any cited source):
1. **Discovery:** PMs work in the wiki. Stable IDs are assigned on promotion, for example `URD-PAY-012`, with a department or domain prefix.
2. **Promotion:** Approved requirements move to Git as Spec Kit `FR-`, OpenSpec requirements, or StrictDoc items. Each records its source wiki URL and revision.
3. **Work:** Jira, Linear, Plane or OpenProject issues carry the ID; tasks reference it, as in `T010 [FR-012]`.
4. **Code:** PR templates or commit trailers carry `Implements: FR-012`; use StrictDoc markers or Sphinx-Needs links for formal projects.
5. **Tests:** Test names or tags carry the ID, and JUnit XML feeds StrictDoc or sphinx-test-reports.
6. **CI gate:** Fail builds on orphan requirements, unknown IDs, or changed requirements without test updates, and publish the matrix.

Additional views:
- Spec Kit's analysis stops at tasks by default. Code and test coverage needs custom CI.
- OpenSpec's minimal syntax lowers adoption costs but lacks a verified stable-ID scheme. Add an ID convention.
- For regulated departments, use StrictDoc or Sphinx-Needs for formal records. Weigh their small communities against the commitment.
- Pin BMAD to v6.12 until v7 stabilizes.
- A URD does not map exactly to any template found. Adapt BMAD's BRD template or Spec Kit's spec template, with user-language needs kept separate from system requirements.
- Keep OpenProject out of the plan as wiki access for agents unless wiki tools are confirmed.

### Gaps
- Jira Rovo's AI requirements features and native traceability: no official documentation found. Marketplace tools (e.g. Xray, Requirements Yogi) were not researched.
- OpenSpec's stable-ID behavior and Kiro's requirement numbering and task references were not confirmed from primary docs.
- StrictDoc's exact source-code marker syntax was not visible in the fetched page.
- Productboard's and Aha!'s traceability to code: not found. Kiro's pricing and data residency were not checked.
- Traditional Chinese support for ChatPRD is not stated. Spec Kit and OpenSpec are Markdown-based and should carry Chinese text, but agent quality in Traditional Chinese was not tested.
- Plane Pages or Wiki edition limits were not verified.

## Architecture patterns: docs-as-code, wiki as source of truth, hybrid, and the LLM wiki pattern

### Takeaway
The LLM wiki pattern is verifiably Karpathy's, but it is explicitly a personal, moderate-scale "idea file" and not an enterprise architecture. For this context a hybrid is the most defensible design: wiki for collaborative discovery, Git for approved and versioned specs, and a permission-aware index for retrieval. This recommendation is my synthesis, since I found no rigorous comparative study.

### Cited Findings
- Karpathy's gist "LLM Wiki" calls itself an "idea file" to paste into an agent. It has three layers: raw sources (immutable), an LLM-maintained Markdown wiki, and a schema file such as CLAUDE.md or AGENTS.md. Operations are ingest, query and lint — [gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- The gist says the index-file approach works at moderate scale, roughly 100 sources and a few hundred pages, and larger wikis may need search tooling — [gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Secondary sources date it to April 2026 — [VentureBeat](https://venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an), [zby ingest](https://zby.github.io/commonplace/sources/karpathy-llm-wiki.ingest/). The GitHub gist API returned 502 twice, so I could not confirm the timestamps directly.
- Implementations: [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) (21,433 stars, GPL per its [license file](https://github.com/nashsu/llm_wiki/blob/main/LICENSE), last push 2026-10-21) and [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) (16,230 stars, MIT, last push 2026-10-25).
- Birgitta Böckeler, on Martin Fowler's site (2025-10-15), defines spec-first, spec-anchored and spec-as-source. She found Kiro and Spec Kit mostly spec-first, Kiro heavy for a small bug, Spec Kit's Markdown repetitive to review, and that agents do not reliably follow all instructions — [martinfowler.com](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html). This is over a year old, and both tools have since shipped many releases.
- One practitioner argues that artifacts feeding AI tools should be versioned and tracked rather than kept in a siloed wiki — [John Munn](https://medium.com/@johnmunn/documents-as-code-c216d0cb4b6a). This is opinion, not evidence.
- A critique reported second-hand says spec-first assumes nothing is learned during implementation — [dev.to](https://dev.to/pacheco/spec-driven-development-isnt-waterfall-but-it-keeps-ending-up-there-4eei).

### Inferences
| Pattern | Strengths | Weaknesses | Fit |
|---|---|---|---|
| Docs-as-code only | Diffs, review, branch alignment, agent-friendly | Barrier for non-technical PMs and departments; Git permissions are repo-level | Engineering-heavy teams |
| Wiki as source of truth | Easy collaboration, granular permissions | Weak versus-code linkage; drift; agent writes hard to review | Knowledge with no direct build impact |
| Hybrid (recommended) | Each audience gets a suitable tool; approved specs become reviewable | Needs promotion workflow and ID discipline; two stores can diverge | Multi-department factory |
| LLM wiki layer | Compounding synthesis; low retrieval infrastructure at small scale | Summary drift, errors frozen in pages; no native ACL or multi-author governance | One domain or department, derived content only |

Proposed reference flow:
1. **Capture:** Interviews, meeting notes and documents are stored as immutable raw sources with department-level access.
2. **Draft:** PMs and agents develop in wiki spaces. An LLM wiki-style agent may ingest, cross-link and lint, with generated pages labelled and citing sources.
3. **Approve:** Sign-off promotes the URD through a PR to Git with stable IDs. Git is authoritative for approved requirements, and the wiki links to or renders it read-only to prevent two sources of truth.
4. **Index:** Onyx or a similar layer indexes both stores, applying ACLs at retrieval.
5. **Consume:** Agents use MCP for the wiki, the repo directly for specs, and code-knowledge MCP for implementation facts.
6. **Audit:** CI checks traceability, and lint jobs raise stale or contradictory content.

Key design rules:
- Agent writes to approved requirements go through PR review only. Agents may write freely only to draft space.
- Scale limits matter: at a multi-department level, the gist's own index-only approach will likely be outgrown. Keep search or RAG alongside it.
- Use per-user or per-agent identity for MCP. A shared service account would bypass the user-scoped permission model that Atlassian, Notion and OpenProject document.
- Keep the Markdown export path open for any wiki, since it preserves portability and supports Git-based agent consumption.

### Gaps
- No rigorous empirical study comparing docs-as-code, wiki-first and hybrid for AI-agent consumption was found. Most sources are practitioner opinions.
- No documented large-enterprise, multi-department LLM wiki deployment was found. Its risks in the table are my assessment, reinforced by commentary I did not verify in depth.
- Bidirectional wiki-Git sync reliability was not researched. Docmost has an open [lossless Markdown round-trip PR](https://github.com/docmost/docmost/pull/2540), which suggests that round-trip fidelity is not yet solved there.
