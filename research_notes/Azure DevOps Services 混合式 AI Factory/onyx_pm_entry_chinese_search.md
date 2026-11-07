# Onyx as PM entry point, and Chinese search (checked 2026-11-07)

User decisions so far: Azure DevOps Services; source code may stay in Microsoft's cloud; the user wants Onyx as the PM entry point. Not yet decided: where URD drafts live; whether model inference may use the cloud.

No tool here was installed or tested. Doc claims were read through a summarising fetch tool. Items marked [GH] were read from GitHub API or source files on 2026-11-07.

## Onyx facts
- [GH] 33,266 stars, v5.1.0 (2026-11-06), not archived. License: MIT outside `ee`; `ee` under the Onyx Enterprise License — https://github.com/onyx-dot-app/onyx/blob/main/LICENSE
- 56 official connectors per earlier notes. Permission sync: Google Drive, Gmail, Confluence, Jira, Slack (indexed), GitHub, Box, Canvas, SharePoint (certificate auth), OneDrive, Teams, Outlook, Linear, HubSpot, YouTrack, Zoom — https://docs.onyx.app/admins/connectors/overview.md
- No Azure DevOps connector in the docs. [GH] none in `backend/onyx/connectors`; PR #6496 closed unmerged 2026-10-16; issue #4148 closed not_planned 2025-08-18.
- Ingestion API: document fields include id, title, sections, source, metadata, owners; no per-document ACL field is documented. The only documented access lever is `cc_pair_id`; omitted means the default pair, "public to every user". Group Managers can ingest to pairs of their groups. — https://docs.onyx.app/developers/guides/index_files_ingestion_api.md . [GH] `backend/onyx/server/onyx_api/ingestion.py` gates on cc_pair access and defaults to DEFAULT_CC_PAIR_ID.
- MCP server: read tools `search_indexed_documents`, `search_web`, `open_urls`; no write tools described; document permissions apply; "available on all editions" — https://docs.onyx.app/overview/onyx_anywhere/mcp_server.md
- Custom Agents: instructions, knowledge scoping, actions, MCP/OpenAPI plugins, sharing to users or groups. No documented template, review, versioning or approval workflow — https://docs.onyx.app/overview/core_features/agents.md
- Craft: produces documents and other files in a sandbox using Onyx knowledge; apps can do approved actions such as updating issues or opening PRs; all editions, admin must enable. Wiki editing not mentioned; Azure DevOps app support not checked — https://docs.onyx.app/overview/core_features/craft.md
- Licensing conflicts:
  - https://docs.onyx.app/deployment/enterprise_edition.md lists permission sync, groups and multiple SSO under Business; custom roles, SCIM and log export under Enterprise; a single SSO provider works in every edition.
  - https://docs.onyx.app/admins/connectors/official/sharepoint says permission sync is Business and Enterprise on Cloud, plus Enterprise Edition when self-hosted.
  - https://docs.onyx.app/overview/core_features/connectors.md says permission sync requires Enterprise Edition.
  - https://onyx.app/pricing lists Business at US$20/user/month billed annually, OIDC/SAML SSO under Enterprise only, Enterprise by quote. https://docs.onyx.app/admins/billing/overview.md lists self-hosting as optional for Business.
- Audit logs on self-hosted; export described for Enterprise self-hosted admins — https://docs.onyx.app/admins/auditing/audit_logs.md
- License validated locally; air-gapped via manual license upload — https://docs.onyx.app/admins/billing/overview.md
- SharePoint connector: SharePoint Online through Graph; permission sync needs a certificate and Sites.FullControl.All (or fullcontrol with Sites.Selected). Reads standard web part text only — https://docs.onyx.app/admins/connectors/official/sharepoint
- [GH] Depends on `litellm==1.100.4` (pyproject); whether LiteLLM advisories affect Onyx's use was not checked.
- Hardware (secondary, Sep-Oct 2026): preferred 8 vCPU, 16 GB RAM, 500 GB under 5,000 users; a 10 GB text example implies at least 9 cores and 35 GB RAM; local LLM separate — https://wz-it.com/en/blog/onyx-self-hosted-installation/

## Judgment on Onyx (inference)
Adopt as the PM entry point for search, Q&A, URD interview and drafting, and as read-only MCP knowledge for coding agents. Do not treat it as the URD system of record. Flow: Onyx URD assistant agent → draft in wiki → approved URD with stable IDs as a PR in Azure Repos → sync job writes wiki, work items and repos into department-scoped Onyx pairs. The sync job must fail if `cc_pair_id` is missing.

Draft location is the user's open decision. Default proposal: Azure DevOps Wiki, because Onyx becomes the search entry so weak Wiki search matters less, and the Azure DevOps sync job has to be built anyway. Alternate: BookStack if PMs reject the Wiki editor or want drafts on-prem (Onyx has a BookStack connector with no permission sync).

## Chinese search facts
No independent, recent Traditional Chinese benchmark was found for any option.
- [GH] Onyx: `OPENSEARCH_TEXT_ANALYZER` env var, default `english` — https://github.com/onyx-dot-app/onyx/blob/main/backend/onyx/configs/app_configs.py . Schema references the analyzer by name and notes existing indices need reindexing; no custom analysis settings were seen in a grep of schema.py — https://github.com/onyx-dot-app/onyx/blob/main/backend/onyx/document_index/opensearch/schema.py . Whether `cjk` works end to end is unverified.
- Onyx multilingual docs: defaults are English-first; embedding and reranking can be swapped — https://docs.onyx.app/multilingual_setup
- OpenSearch has a `cjk_bigram` filter — https://docs.opensearch.org/3.6/analyzers/token-filters/cjk-bigram/
- SmartCN handles Simplified only — https://blog.mimacom.com/elasticsearch-chinese-language . IK confirmed Simplified only by its developer, in 2017 notes — https://phabricator.wikimedia.org/T158202 . STConvert converts between scripts — https://gitee.com/infinilabs/analysis-stconvert
- A search summary said SmartCN + STConvert tested best in Wikimedia's 2017 work — https://phabricator.wikimedia.org/T158203 (original not read; old).
- [GH] Meilisearch Charabia: jieba segmentation, compatibility decomposition + kvariant conversion — https://github.com/meilisearch/charabia . Traditional behaviour unconfirmed.
- RAGFlow: an old mirror converts Traditional to Simplified before segmentation — https://huggingface.co/spaces/Starowo/ragflow/blame/bcd8078e49b8d619491752eb22a9ecc80dcd8524/rag/nlp/rag_tokenizer.py . Current code unverified.
- PGroonga: claims all languages; Zulip lists it as experimental for Chinese and Japanese — https://zulip.com/help/configure-multi-language-search

## Judgment on Chinese search (inference)
OpenSearch or Elasticsearch is the most tunable, not proven best. Script conversion does not solve vocabulary differences such as 軟體/软件 and 資料庫/数据库, so an internal synonym list is needed. Test on the same labelled URD queries (50 to 100): (1) Onyx default baseline, (2) Onyx + `cjk` analyzer + multilingual embedding such as bge-m3, the first recommendation, (3) Onyx + IK + STConvert only if (2) falls short since it likely needs code changes or a fork, (4) RAGFlow as a control. Measure recall and top-10 precision.
