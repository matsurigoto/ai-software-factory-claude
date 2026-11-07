# 以 Azure DevOps Services 組裝混合式 AI Factory

**建議架構是把 Azure DevOps Services 當作雲端的 ALM 與稽核骨幹,其餘全部放地端:由地端 dispatcher 主動輪詢事件並呼叫 Runs API,搭配 Build Validation 作為確定性協調器;coding agent 與 test agent 以 headless CLI 在自託管 agent pool 的沙箱中執行;PM 由 Onyx 進入,URD 草稿預設放 Azure DevOps Wiki,核准版以穩定 ID 進 Azure Repos。** 這個組合不需要任何從雲端進入地端的連線,因為自託管 agent 只走對外 HTTPS,而 Azure Repos 的 PR 留言與工作項目事件都不能原生觸發 pipeline。尚未決定的最大事項是模型推論能否上雲,它同時決定成本與 GPU 採購:【估算】以牌價和平均用量計,託管路線在 100 位與 400 位開發者時約為純地端模型成本的 20% 到 45%,所以地端的正當理由是主權與持續運作,不是省錢。建議的預設是**先不買 8-GPU 伺服器**,先分類資料並取得法務答覆;若獲准上雲且受限資料低於約 20%,採混合路由:受限等級只走地端且失敗就拒絕,其他等級走獲准的雲端端點。Onyx 可以當入口,但有三個前提必須寫進計畫:它沒有 Azure DevOps 連接器,匯入 API 沒有逐文件 ACL 欄位且省略參數時會公開給全體使用者,中文搜尋預設使用 `english` analyzer。另有兩個近期期限:全域 PAT 於 **2027-01-27** 失效,舊版 Node.js runner 排定於 2026-11-24 移除。本研究沒有實機測試任何工具,所有選擇都要經過文末的 PoC,包含繁體中文搜尋與模型測試。

## 建議架構:雲端只放 ALM,協調與 agent 都由地端對外連線

**閱讀方式。** 「**證據**」指可查來源的事實,「**判斷**」指本報告的推論,「【估算】」指以明示假設計算的數字,不是報價或實測。名為「判斷」「預設」「定位」的表格欄位都是推論。地端狀態用五個詞:**已載明**(廠商文件描述無網路運作)、**可行**(架構允許、未見矛盾,但無廠商聲明)、**部分**(離線或無雲端時會失去指名功能)、**否**(需要雲端控制面)、**未確認**。授權、星數與版本由研究者於 2026-11-07 從 GitHub API 讀取;多數文件是經摘要式工具讀取,不是逐字閱讀。沒有工具做過封包檢查,本報告也不是合規或法律結論。**利益揭露:本報告由 Anthropic 的模型撰寫,比較對象包含 Anthropic 產品(Claude Code、Claude 模型、Claude in Foundry),成本需求基準也取自 Anthropic 的文件,請自行加權。** 本報告取代先前兩份報告。

**判斷:** 下表是建議架構,設計依據是模式研究的一致方向:循序規劃任務上每種多 agent 變體都退步 39% 到 70% ([Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)),Stripe 的量產做法也是確定性節點穿插 agent 節點,CI 最多兩輪就退回人類 ([Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))。沒有任何來源以這個組合測得成效,也沒有找到按 repo 敏感度路由的公開企業案例。

| 層 | 職責 | 預設 | 位置 | 替代 |
|---|---|---|---|---|
| L0 ALM | Repos、Boards、Pipelines 控制面、Wiki、Test Plans | Azure DevOps Services(已定案) | Microsoft 雲端 | 無 |
| L1 PM 入口與知識 | 搜尋、問答、URD 訪談與起草 | Onyx(已定案)+ 自建 Azure DevOps 同步工作 | 地端 | RAGFlow 作對照組 |
| L2 需求權威 | 草稿、核准版規格、穩定 ID | Azure DevOps Wiki 放草稿(待決),Azure Repos 放核准版;Spec Kit 或 OpenSpec | 雲端 | BookStack 放草稿(地端) |
| L3 協調 | 階段、閘門、重試、核准 | 地端 dispatcher + Pipelines YAML + Build Validation + Environments | 控制面在雲端,dispatcher 在地端 | 第二階段才加 Temporal |
| L4 Agent 執行 | 設計、實作、重構、文件、遷移 | OpenCode 為主,Goose 為次,私有 skill monorepo;依推論路線調整 | 地端 | Claude Code 或 Codex CLI(若獲准雲端推論) |
| L5 驗證 | 審查與測試 agent | PR-Agent、fresh-context verifier、Playwright Test Agents、確定性工具 | 地端 | CodeRabbit、Qodo 單租戶 |
| L6 平台 | 沙箱、閘道、遙測、身分 | RKE2、agent-sandbox + Kata 或 gVisor、agentgateway、Langfuse OSS、憑證保管 | 地端 | Agent Router、Coder |
| L7 推論 | 模型服務與路由 | 分級路由(待決) | 依等級 | 純地端或純雲端 |

| 決策 | 狀態 | 建議預設 | 改變預設的條件 |
|---|---|---|---|
| ALM 平台、原始碼位置、PM 入口 | **已定案** | Azure DevOps Services;程式碼留 Microsoft 雲端;Onyx | 不重新討論;Onyx 的條件見 PM 節 |
| 模型推論能否上雲 | **待決** | 未核准前視為不可;目標狀態是混合路由,受限等級只走地端,其餘優先 Azure OpenAI Data Zone APAC | 法務拒絕:純地端。Claude 的品質差距在內部 bake-off 大到值得第二份雲端合約:改 Vertex Singapore 或 Bedrock JP geo。不重視地理位置而重視整合:Claude Hosted on Azure Global |
| URD 草稿位置 | **待決** | Azure DevOps Wiki | PM 不接受編輯器、草稿含不宜上雲內容,或頁面權限不足:BookStack |
| 是否受台灣相關限制 | **待決** | 條件式處理,不假設組織所在地 | 受公部門相關規範約束:中國來源模型與 embedding 視同不可用,見推論節 |
| GPU 採購 | 待上述結果 | 不買 8-GPU 伺服器;只租用做 bake-off | 見推論節的採購規則 |
| 事件觸發 | 建議 | 地端輪詢 + Build Validation | 輪詢延遲或 API 負載不可接受:加 Storage Queue |
| 協調器 | 建議 | Pipelines + dispatcher | 試點出現多日等待或跨 repo saga:加 Temporal |
| 規模 | 建議 | 150 人以上才建全套 | 少於 50 人:精簡為 Git skill、Pipelines、VM runner、Langfuse、閘道 |

**相較先前報告的修正。** 全域 PAT 日期是 2027-01-27,不是 12 月 1 日;遠端 MCP 已於 2026-08-05 GA,不是預覽。RAGFlow 現在有 Azure DevOps 連接器,Onyx 仍沒有。Web Hook 只能打公開 HTTPS 端點,所以先前的「地端 webhook receiver」改為輪詢。Wiki 預設由 BookStack 改為 Azure DevOps Wiki,理由是 Onyx 接手搜尋入口;這仍是待決事項。Azure DevOps Server 路線已不適用,相關比較不再列入。

## SDLC 流程:沒有入站路徑,由地端主動拉事件

**證據:** 自託管 agent「永遠由 agent 發起通訊」,以 HTTP 長輪詢接工作,取得單一 job 的短效 OAuth 權杖,job 結束後丟棄 ([Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops))。Azure Repos 不支援留言觸發,`/azp run` 只支援 GitHub ([Triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/triggers?view=azure-devops),搜尋摘錄),Web Hook consumer 則要求公開 HTTPS 端點,不能指向 loopback 或特殊範圍 IP ([Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops))。Microsoft 沒有公布觸發延遲,所以下表延遲欄是判斷。

| 觸發方式 | 事件來源 | 新增雲端依賴 | 入站 | 限制 | 預期延遲(判斷) | 主要風險 | 定位 |
|---|---|---|---|---|---|---|---|
| Build Validation | PR 建立或推送 | 無 | 無 | [分支政策選項](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops) | 秒級入列,再等 pool | PR 內容是不可信輸入 | **預設** |
| 地端 dispatcher + Runs API | REST 查詢後 [`POST .../runs`](https://learn.microsoft.com/en-us/rest/api/azure/devops/pipelines/runs/run-pipeline?view=azure-devops-rest-7.1) | 無 | 無 | [滑動 5 分鐘 200 TSTU](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops) | 輪詢週期 | pipeline 外的長效身分 | **預設** |
| 排程 pipeline 查 WIQL | Cron | 無 | 無 | [每週約 1,000 次、15 分鐘 10 次](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops) | 持續約 10 分鐘 | 每次輪詢占 parallel job | 不建議 |
| Pipeline resource | 上一條 pipeline 完成 | 無 | 無 | [依預設分支讀篩選條件](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/pipeline-triggers?view=azure-devops) | 秒級 | 僅串接階段 | 串接用 |
| Service hook → Storage Queue 或 Service Bus | [任何事件](https://learn.microsoft.com/en-us/azure/devops/service-hooks/events?view=azure-devops) | Azure 訂閱與佇列 | 無 | [TTL 上限 7 天](https://learn.microsoft.com/en-us/azure/devops/service-hooks/consumers?view=azure-devops) | 秒級(未測) | 佇列留副本,payload 應設 Minimal 或 None | 第二選擇 |
| Service hook → Incoming Webhook 資源 | 任何事件 | 無 | 無 | **同組織組合無文件,未測** | 秒級(未測) | [無法算 HMAC](https://learn.microsoft.com/en-us/azure/devops/service-hooks/consumers?view=azure-devops);[`${{ }}` 展開](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops)有注入風險 | PoC 才考慮 |
| Service hook → 地端 Web Hook | 任何事件 | 公開端點或 relay | **有** | 公開 HTTPS | 秒級 | 違反無入站目標 | 排除 |

**判斷:** 預設是輪詢。Service hook 有 probation 機制,期間「會遺失新事件」([Troubleshoot](https://learn.microsoft.com/en-us/azure/devops/service-hooks/troubleshoot?view=azure-devops)),所以即使改事件驅動也要保留對帳輪詢。輪詢的貴處是 PR 留言:WIQL 不涵蓋 threads,必須逐一列出進行中 PR 的 threads,這是日後加 Storage Queue 的最佳理由。佇列只放 ID 時對資料邊界影響很小,因為程式碼本來就在 Microsoft 雲端。Agent 主機的防火牆應採 FAQ 版的 IP 超集,因為[允許清單頁](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)與 [Linux agent FAQ](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops) 列的範圍不一致。

```
Boards 工作項目 / PR threads(Azure DevOps Services)
   ↑ 地端 dispatcher 以憑證式 service principal 輪詢(對外 443)
   ↓ POST /_apis/pipelines/{id}/runs,只帶 ID,agent 再回查內容
依資料分級隔離的自託管 agent pool(長輪詢接工作;--disableloguploads)
   唯讀 Agent stage → Detection stage → 持寫入權杖的 SafeOutputs stage
   ↓ git push + REST 建 PR
Build Validation:確定性閘門 + fresh-context verifier + PR-Agent 建議
   ↓
人類審查合併 → Environments 核准 → 部署紀錄回連工作項目
```

**判斷:** 三階段切法借自 [ado-aw](https://github.com/githubnext/ado-aw) 的設計,寫入權杖不暴露給 agent。這個工具本身只宜試用:33 星、未達 1.0,且[只支援 Copilot CLI 引擎](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md),值得抄的是權限分離。沒有廠商發布過 Claude Code、Codex、OpenCode、Goose、Copilot CLI 或 Factory 的完整 Azure Pipelines 範例,可行性是由 CLI 行為推得,agent 自身的 REST 與 MCP 步驟要自己寫。

| SDLC 階段 | 觸發 | Agent 與 skill(判斷) | 權威狀態 | 人類閘門 | 確定性檢查 |
|---|---|---|---|---|---|
| 探索、URD 草稿 | PM 在 Onyx 發起 | Onyx URD assistant;BMAD BRD 模板改作 URD | Wiki 草稿 | PM 審閱 | 生成內容標註並附來源 |
| 核准與晉升 | 工作項目狀態,dispatcher | 轉換 agent 開規格 PR | Azure Repos + Boards | 需求負責人核准 PR | 每條驗收條件有 ID 且可測 |
| 設計 | 工作項目標籤,dispatcher | 主要 harness + Spec Kit 或 OpenSpec、domain modeling;依等級使用最強獲准模型 | Azure Repos | 依風險核准 ADR | 需求對任務覆蓋 |
| 實作 | Runs API | 單一寫手 + TDD skill,沙箱內 | 分支、run | 無(受閘門約束) | Lint、型別、單元、secret scan |
| 審查 | Build Validation | PR-Agent,與作者分離,先建議不阻擋 | PR threads | 人類合併 | SAST、SCA、測試竄改、trailer |
| 驗證 | Build Validation 或排程 | Fresh-context verifier 只取規格與公開介面;Playwright Test Agents | 受保護測試路徑、Test Plans | 測試 oracle 變更 | Mutation、Pact、Schemathesis、E2E |
| 發布 | Environments | Agent 只提案,部署由 pipeline 執行 | 環境部署紀錄 | Approvals and checks | Trivy、ZAP、k6、SLO |
| 維護 | 排程(離峰) | 批次 agent 做依賴升級、flaky test;Healer 只開 PR | PR | 依變更類別 | 新增 skip 一律失敗 |

**判斷:Agent 的 Azure DevOps 操作用本機 MCP。** 遠端 MCP [已於 2026-08-05 GA](https://devblogs.microsoft.com/devops/azure-devops-remote-mcp-server-ga/),但它[只在歐洲與美國執行](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops),也沒有文件記載非互動路徑,Claude Code 與 Cursor 目前要靠[自訂 Entra app 與瀏覽器登入](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server),適合開發者桌面。Pipeline 內應釘 [`@azure-devops/mcp`](https://github.com/microsoft/azure-devops-mcp) v2.11.0 或更新版,以 [`envvar` 認證](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/GETTINGSTARTED.md)帶入隨 job 失效的權杖;這個做法未測。**證據:** Manifold Security 曾示範以 PR 描述隱藏指令劫持 agent ([CSA](https://labs.cloudsecurityalliance.org/research/csa-research-note-azure-devops-mcp-pr-comment-hijack-2026080/)),其後 [#1553](https://github.com/microsoft/azure-devops-mcp/commit/c3e4cd05) 與 [#1570](https://github.com/microsoft/azure-devops-mcp/commit/a2891613) 加入 spotlighting,[#1680](https://github.com/microsoft/azure-devops-mcp/commit/ea8a7f6b) 修補附件下載。第三方把 CVE-2026-32211 算到這個套件上,但 [CVE 紀錄](https://cveawg.mitre.org/api/cve/CVE-2026-32211)並未點名它。**判斷:** Spotlighting 只是給模型的分隔提示,真正的控制是專案範圍身分、不自動核准寫入、讀寫分階段與出站限制。本機版沒有唯讀旗標,要靠身分權限與 `-d` 限定領域。

**判斷:追溯。** `AB#` 是 GitHub 整合語法,文件明列其他 Git repo 不支援 ([Link GitHub](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github)),不要用在 Azure Repos。本報告提案使用原生工作項目連結,加上 `Work-Item: 1234`、`Implements: FR-012` 兩個 trailer,由 Build Validation 強制檢查孤立需求與未知 ID。需求工具與 Azure Boards 沒有公開整合,這是本報告的設計。

## PM 流程:Onyx 當入口,但不當需求的權威來源

**判斷:** PM 在部門範圍的 Onyx URD assistant 訪談與起草,草稿寫入 Wiki;核准後的 URD 帶 `URD-PAY-012` 這類穩定 ID,以 PR 進 Azure Repos 並同步工作項目;同步工作再把 Wiki、工作項目與 repo 內容寫回部門範圍的 Onyx connector pair,寫回時標明草稿或核准狀態。Coding agent 透過 Onyx 的唯讀 MCP 查詢企業知識,但實作依據一律是 repo 內的核准規格。Onyx 寫 Wiki 這一步要用 Custom Agent 的 OpenAPI 或 MCP action 自建,或由 PM 手動貼上,兩者都未測。需要核准 PR 的 PM 要有 Basic 授權,因為 [Stakeholder 不含 Repos](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)。

| Onyx 條件或風險 | 證據 | 處置(判斷) |
|---|---|---|
| 沒有 Azure DevOps 連接器 | [官方清單](https://docs.onyx.app/admins/connectors/overview.md)未列;[PR #6496](https://github.com/onyx-dot-app/onyx/pull/6496) 2026-10-16 未合併關閉 | 自建同步工作;可先讀該 PR,但其程式碼未經審閱 |
| 匯入沒有逐文件 ACL | [Ingestion API](https://docs.onyx.app/developers/guides/index_files_ingestion_api.md) 只有 `cc_pair_id` 這個存取手段,省略時進預設 pair 並「對所有使用者公開」 | 每部門一個 pair;**缺 `cc_pair_id` 時同步工作必須失敗**;隔離是部門級,不是文件級 |
| 付費方案說法衝突 | [方案頁](https://docs.onyx.app/deployment/enterprise_edition.md)稱權限同步與群組在 Business;[連接器頁](https://docs.onyx.app/overview/core_features/connectors.md)稱需 Enterprise;[SharePoint 頁](https://docs.onyx.app/admins/connectors/official/sharepoint)又是另一種說法 | 取得書面答覆後才編預算 |
| SSO 說法衝突 | [文件](https://docs.onyx.app/deployment/enterprise_edition.md)稱單一 SSO 所有版本可用;[定價頁](https://onyx.app/pricing)把 OIDC/SAML 列在 Enterprise,Business 為 US$20/人/月 | 同上;先確認 Entra SSO 落在哪一版 |
| 沒有需求工作流程 | [Custom Agents](https://docs.onyx.app/overview/core_features/agents.md) 未記載模板、審核、版本或核准 | 不當權威來源;流程留在 Repos |
| 中文搜尋預設英文 | [`OPENSEARCH_TEXT_ANALYZER` 預設 `english`](https://github.com/onyx-dot-app/onyx/blob/main/backend/onyx/configs/app_configs.py),既有索引需重建 ([schema](https://github.com/onyx-dot-app/onyx/blob/main/backend/onyx/document_index/opensearch/schema.py)) | 灌資料前先定案,見下方測試 |
| 來源權限不會自動帶入 | [BookStack 連接器無權限同步](https://docs.onyx.app/admins/connectors/overview.md);以 [email 比對使用者](https://docs.onyx.app/admins/connectors/official/confluence) | 部門級存取;整理 Entra 的 mail 屬性 |
| MCP 能力 | [唯讀工具含 `search_web`、`open_urls`](https://docs.onyx.app/overview/onyx_anywhere/mcp_server.md),文件權限適用 | 受限等級關閉對外工具,以出站政策兜底 |
| Craft | [可在沙箱產文件,經核准動作開 PR](https://docs.onyx.app/overview/core_features/craft.md);未提 Wiki,Azure DevOps app 未查 | 第二階段再評估 |
| 離線與授權 | [`ONYX_AIRGAPPED=true`、每個服務 `DISABLE_TELEMETRY=true`](https://docs.onyx.app/security/self_hosted/faq);[授權本地驗證,可手動上傳](https://docs.onyx.app/admins/billing/overview.md) | 以 default-deny 出站驗收;續約流程要確認 |
| 依賴 | [`pyproject` 釘 `litellm==1.100.4`](https://github.com/onyx-dot-app/onyx);LiteLLM 2026 年有[供應鏈事件](https://docs.litellm.ai/blog/security-update-march-2026),對 Onyx 的影響未查 | 升級前查 advisory;Onyx 不對外開放 |
| 硬體 | [次級來源:5,000 人以下建議 8 vCPU、16 GB、500 GB;10 GB 文字例需 9 核 35 GB](https://wz-it.com/en/blog/onyx-self-hosted-installation/) | 以實際資料量重估 |
| Onyx 自己的 LLM | 問答與起草同樣受推論路線約束;[預設偏英文,embedding 與 rerank 可換](https://docs.onyx.app/multilingual_setup) | 純地端時,PM 端也需要 GPU 與 embedding 容量,不在開發者成本表內 |

### URD 草稿位置:預設 Azure DevOps Wiki,但這是對「盡量不上雲」的明確例外

**判斷:** 預設選 Wiki,因為 Onyx 成為搜尋入口後,Wiki 搜尋較弱的影響變小,而 Azure DevOps 同步工作反正要寫。代價要講清楚:你只核准了原始碼留在 Microsoft 雲端,URD 草稿會把訪談內容、客戶需求與未公開計畫也帶上去,需要明確決定這是否可接受。

| 比較 | Azure DevOps Wiki(預設) | BookStack(替代) |
|---|---|---|
| 授權與狀態 | 隨產品;Git 儲存 | [MIT,v26.09.2](https://github.com/BookStackApp/BookStack) |
| 地端狀態 | **否**,內容在 Azure DevOps 的地理位置 | 可行,自行託管 |
| Onyx 整合 | 無連接器,用自建同步 | [有連接器,無權限同步](https://docs.onyx.app/admins/connectors/overview.md) |
| Agent 整合 | [官方 MCP `wiki` 工具組](https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server),發布型 wiki 是純 Git Markdown | [僅社群 MCP](https://github.com/pnocera/bookstack-mcp-server) |
| 編輯與限制 | [18 MB 頁、235 字元路徑](https://github.com/MicrosoftDocs/azure-devops-docs/blob/main/docs/project/wiki/includes/wiki-naming-conventions.md);[Mermaid 語法有限,文件對 `flowchart` 自相矛盾](https://learn.microsoft.com/en-us/azure/devops/project/wiki/markdown-guidance) | PM 接受度未測 |
| 原生中文搜尋 | 未找到分詞來源,未測 | [第三方稱以空白分詞](https://backiee.wasmer.app/http_github_com/johnroyer/bookstack-meilisearch);兩者都靠 Onyx 補 |
| 新增維運 | 無 | 一個 stateful 服務 |
| 改選條件 | 預設 | PM 拒絕 Wiki 編輯器;草稿不得上雲;頁面權限不足(Wiki 頁面權限未研究) |

兩個選項都把核准版放 Azure Repos,所以這項決定可逆,搬移的是草稿,不是需求權威。

### 繁體中文搜尋:OpenSearch 最可調,但還沒有人證明它最好

**證據:** 沒有找到任何選項的獨立近期繁中評測。OpenSearch 有 [`cjk_bigram`](https://docs.opensearch.org/3.6/analyzers/token-filters/cjk-bigram/);[SmartCN 只處理簡體](https://blog.mimacom.com/elasticsearch-chinese-language),[IK 也以簡體為主](https://phabricator.wikimedia.org/T158202)(2017 年開發者說明),[STConvert](https://gitee.com/infinilabs/analysis-stconvert) 可轉換字形。[Meilisearch Charabia](https://github.com/meilisearch/charabia) 的繁中行為未確認,[Zulip 把 PGroonga 列為中日文實驗選項](https://zulip.com/help/configure-multi-language-search)。Onyx 使用 `cjk` analyzer 能否端到端運作也未驗證。**判斷:** 字形轉換解決不了軟體/软件、資料庫/数据库這類用語差異,要維護內部同義詞表。

| 測試組 | 配置 | 定位(判斷) | 條件 |
|---|---|---|---|
| 1 | Onyx 預設 | 基線 | 無 |
| 2 | Onyx + `cjk` analyzer + 多語 embedding,例如 [bge-m3](https://huggingface.co/BAAI/bge-m3) | **第一建議** | bge-m3 為中國來源,受台灣限制時改測 [EmbeddingGemma](https://huggingface.co/google/embeddinggemma-300m) 或 [GTAIDE](https://huggingface.co/taide/embeddinggemma-GTAIDE-300m-2605),兩者中文表現未驗證 |
| 3 | Onyx + IK + STConvert | 僅在第 2 組不足時 | 可能要改碼或 fork |
| 4 | [RAGFlow](https://github.com/infiniflow/ragflow) | 對照組 | [舊 mirror 顯示先繁轉簡](https://huggingface.co/spaces/Starowo/ragflow/blame/bcd8078e49b8d619491752eb22a9ecc80dcd8524/rag/nlp/rag_tokenizer.py),現行程式碼未驗證 |

## 逐層比較:原生整合多半帶雲端或入站條件,CLI harness 最貼合

### 知識、檢索與需求

| 工具 | 授權 | 狀態 | 地端狀態 | 條件 | 定位 |
|---|---|---|---|---|---|
| [Onyx](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE) | MIT;`ee` 另計 | v5.1.0;33,266 星 | 已載明 | 見上表 | **入口(已定案)** |
| [RAGFlow](https://github.com/infiniflow/ragflow) | Apache-2.0 | v1.0.0(2026-11-06) | 未確認;[僅 Zilliz FAQ](https://zilliz.com/ai-faq/does-ragflow-support-airgapped-deployment) | [原生連接 Azure Repos 檔案與 PR](https://github.com/infiniflow/ragflow/blob/main/internal/syncer/connector/azure_devops.go),[PAT 認證](https://github.com/infiniflow/ragflow/blob/main/web/src/pages/user-setting/data-source/constant/azure-devops-constant.tsx);[#20972 同步失敗](https://github.com/infiniflow/ragflow/issues/20972)未修;未見 ACL 繼承 | 對照組;程式碼檢索備選 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | 修改版 Apache-2.0 | 1.19.0 | 部分 | 一個 workspace 算一個 tenant,多租戶需書面授權 | 不用於部門隔離 |
| [Docmost](https://docmost.com/pricing) | [AGPL-3.0](https://github.com/docmost/docmost);付費功能需金鑰 | v0.97.0 | Business 標示 Air-gapped ready | SSO、頁面權限、MCP 需 Business,$6/席、10 席起 | Wiki 替代 |
| [Outline](https://github.com/outline/outline) | [BSL 1.1](https://github.com/outline/outline/blob/main/LICENSE) | v1.12.0 | 可行;[`ENABLE_UPDATES=false`](https://raw.githubusercontent.com/outline/outline/main/.env.sample) | [SAML 需付費](https://www.getoutline.com/pricing);[CJK issue 因閒置關閉](https://github.com/outline/outline/issues/3207) | 僅在已偏好時 |
| [Spec Kit](https://github.com/github/spec-kit) | MIT | v1.3.2 | 檔案可行,CLI 未稽核 | [`/speckit.analyze`](https://github.com/github/spec-kit/blob/main/templates/commands/analyze.md) 止於任務層 | 大型功能 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | MIT | v1.16.1 | 同上 | 穩定 ID 需自訂慣例 | 日常變更 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | [MIT + 額外聲明](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/LICENSE) | v6.12.4 | 未稽核 | [BRD 模板](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/skills/bmod-method/doc-templates/template-brd.md)可改作 URD | 模板來源 |
| [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | MIT | v0.12.1 | 已載明(自述) | 從原始碼建置並審查 | 程式碼知識 |
| [Zoekt](https://github.com/sourcegraph/zoekt)、[Repomix](https://github.com/yamadashy/repomix) | Apache-2.0、MIT | 活躍;v1.18.1 | 可行 | 釘 commit;[Repomix 用 `./` 強制本機](https://repomix.com/guide/remote-repository-processing) | 程式碼知識 |

**證據與判斷:** 一項 88,052 個 PR 的研究發現有無規格的缺陷率沒有顯著差異,但其工件多為 issue 與 ticket 引用,SDD 工具本身未被測試 ([ESEM 2026](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ESEM.2026.84))。採用規格工具的理由是追溯與閘門,不是已證實的缺陷下降;PM 這段應從輔助起步。RAGFlow 的連接器描述不含工作項目與 Wiki,這部分無論如何要自建索引。

### 協調

| 引擎 | 授權 | 狀態 | 地端狀態 | 條件 | 定位 |
|---|---|---|---|---|---|
| Azure Pipelines + dispatcher | 隨產品;dispatcher 自建 | GA | 部分:控制面在雲端,執行在地端 | [1 個免費自託管 job,每增加 1 個 $15](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/);[按並行計費,無時間限制](https://learn.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops) | **預設** |
| [Temporal](https://github.com/temporalio/temporal) | MIT | v1.32.2 | 可行;無廠商 air-gap 文件 | [升級與 shard 負擔](https://docs.temporal.io/self-hosted-guide/production-checklist);無 Azure DevOps 連接器 | 第二階段 |
| [Dapr](https://docs.dapr.io/operations/hosting/self-hosted/self-hosted-airgap/) Workflow | Apache-2.0 | [Dapr.Workflow 1.19.0](https://api.nuget.org/v3-flatcontainer/dapr.workflow/index.json) | 已載明 | 耐久性需實測 | Temporal 太重時 |
| [ado-aw](https://github.com/githubnext/ado-aw) | MIT | v0.54.0;33 星 | 部分;[需拉 GitHub 與 ghcr.io,文件有鏡像作法](https://github.com/githubnext/ado-aw/blob/main/docs/supply-chain.md) | 僅 Copilot CLI 引擎;BYOK 是否免 Copilot 授權未確認 | 抄設計,至多試用 |
| [OpenHands Enterprise](https://docs.openhands.dev/enterprise/integrations/azure-devops) | [PolyForm Free Trial,每年 30 天](https://github.com/OpenHands/enterprise) | 價格未公開 | 部分 | Service hook 要能連到它的 URL,**有入站**;使用者委派 OAuth | 不符無入站目標 |

### Coding agent:能跑在自己 pipeline 又能接自託管端點的,都是 CLI harness

**證據:** Microsoft 明言「目前沒有計畫」把 GitHub cloud agent 整合到 Azure Repos ([部落格留言,2026-10-01](https://devblogs.microsoft.com/devops/updates-to-copilot-code-reviews-for-azure-repos/#comment-5490)),Copilot for Boards 也要[目標 GitHub repo](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/boards/sprint-268-update)。原生支援 Azure Repos 的 Devin、Cursor、OpenHands Enterprise 都各有雲端或入站條件。

| Agent | 授權與狀態 | Azure Repos | 地端 harness 狀態 | 自託管端點 | 條件 | 定位(判斷) |
|---|---|---|---|---|---|---|
| [OpenCode](https://github.com/anomalyco/opencode) | MIT;v1.19.5 | 無原生;git、REST、MCP | 部分;[第三方稽核 17 條外連路徑,無總開關](https://particula.tech/blog/opencode-offline-mode-air-gapped) | [OpenAI 與 Anthropic 相容,內建 Azure、Bedrock、Vertex](https://opencode.ai/docs/providers) | 必設 `enabled_providers` | **主要** |
| [Goose](https://github.com/aaif-goose/goose) | Apache-2.0;v1.57.0 | 同上 | 可行 | [Azure OpenAI 支援 Entra;vLLM 可接](https://goose-docs.ai/docs/getting-started/providers) | 企業政策層薄 | **次要** |
| [Claude Code](https://code.claude.com/docs/en/microsoft-foundry) | 專有 | 未找到專屬文件;git、CLI | 可行;[雲端平台路線預設關閉部分遙測](https://code.claude.com/docs/en/data-usage) | [經 gateway,但不支援非 Claude 模型](https://code.claude.com/docs/en/llm-gateway) | [`allowedProviders` 可釘閘道](https://code.claude.com/docs/en/llm-gateway);transcript 明文存 30 天 | 獲准 Claude 路線時;需第二個 harness 跑本地模型 |
| [Codex CLI](https://github.com/openai/codex) | Apache-2.0;rust-v0.167.0 | 無原生 | 未稽核 | [`model_providers`;需 Responses 格式](https://learn.chatgpt.com/docs/config-file/config-advanced);[vLLM 有文件](https://docs.vllm.ai/en/stable/serving/integrations/codex/) | [#31875 對 Azure GPT-5.6 失敗,未解](https://github.com/openai/codex/issues/31875) | 獲准 OpenAI 路線時,先測 |
| [Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models) | v1.0.107;授權標 NOASSERTION | 無文件 | 部分;`COPILOT_OFFLINE=true` | BYOK:Azure、Anthropic、OpenAI、Ollama | 憑證存明文;[失去 `/delegate`、GitHub MCP](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli) | 觀察 |
| [Factory Droid](https://docs.factory.com/cli/byok/overview) | 商用 | [整合頁 404](https://docs.factory.ai/integrations/azure-devops),視為 git + CLI | [有 airgap build](https://docs.factory.com/enterprise/airgapped-deployment.md) | `customModels`;30B 以下表現差 | 價格與 ADO 需書面確認 | 商用候選 |
| [Tabnine](https://docs.tabnine.com/main/welcome/readme/architecture/deployment-options.md) | 商用 | [帳號連接與索引](https://docs.tabnine.com/main/administering-tabnine/release-notes) | 已載明 | [私有端點,框架為 Chat](https://docs.tabnine.com/main/administering-tabnine/managing-your-team/settings/models-settings);agent 涵蓋未確認 | [品牌已改為 Tricentis](https://tabnine.com/platform) | 書面確認後 |
| [Cursor](https://cursor.com/docs/integrations/azure-devops) | 商用 | 原生,Cloud Agents 公開測試;Bugbot 有限開放 | 否;[規劃留在 Cursor 雲端](https://cursor.com/blog/self-hosted-cloud-agents) | 未驗證 | 新增資料處理者 | 僅 S0、S1 |
| [Devin](https://docs.devin.ai/integrations/azure-devops) | 商用 | 原生,僅 Git | [否;大腦永遠在 Cognition 雲端](https://docs.devin.ai/enterprise/deployment/overview) | 不支援 | 持 repo 寫入憑證 | 僅 S0、S1 |
| [OpenHands](https://github.com/OpenHands/OpenHands) 核心 | MIT;v1.37.0 | 僅 Enterprise 有整合 | 未稽核 | [LiteLLM 支援的模型](https://docs.openhands.dev/openhands/usage/llms/llms) | 可由自建 dispatcher 呼叫,未測 | SDK 備選 |

**判斷:** 你規劃各階段套用不同 agent 與 skill。沒有研究直接比較單一主要 agent 與每階段 best-of-breed,但每多一個 harness 就多一個默認拒絕、遙測檢查與升級回歸面,所以建議**階段差異主要由 skill、模型層級與權限表達,harness 只分寫手與審查者**。Harness 的選擇受推論路線約束:混合路由下,OpenCode、Goose、Factory 能以同一 harness 連兩層;選 Claude Code 就要接受第二個 harness 處理本地模型,這是架構限制,不只是偏好。獨立 context 也比換模型重要:預註冊小樣本中 verifier 偵測率 95.4%,建構者 85.1% ([arXiv 2610.28667](https://arxiv.org/html/2610.28667))。

| 階段 | Harness | 模型層級(依等級允許範圍內) | Skill | 權限 |
|---|---|---|---|---|
| 需求、設計 | 主要 harness | 最強 | OpenSpec 或 Spec Kit、domain modeling | 只寫規格分支 |
| 實作、跨模組 | 主要 harness | 最強 | TDD、diagnosing bugs、部門 skill | 沙箱內寫單一分支 |
| 小修、測試、文件 | 主要或次要 | 快速或地端 | 文件模板、測試 skill | 同上 |
| 審查 | PR-Agent | 中階 | 組織規則 | 只寫 PR thread |
| 遷移、批次 | Goose headless,離峰 | 快速或地端 | 遷移 playbook | 分批,agent 不直接寫遠端 |
| DevOps | 唯讀或 plan-only | 中階 | IaC plan 審查 | 部署由 pipeline 持有 |

**證據與判斷:Skill。** Skill 放在 Azure Repos 的 monorepo,加分支政策與 CODEOWNERS。[OpenCode 可讀 `.claude/skills` 與 `.agents/skills`](https://opencode.ai/docs/skills),所以可攜性成本低。不要假設 skill 有效:[SkillsBench](https://arxiv.org/html/2602.12670v1) 顯示軟體工程 skill 平均只提升 4.5 個百分點,自動生成為 -1.3,每個 skill 都要做 with/without 配對評測。[Cisco skill-scanner](https://github.com/cisco-ai-defense/skill-scanner) 可本地執行,但其自測規則掃描在 HIGH 等級只抓 7.7%,人工審查不能省。

### 審查

| 工具 | 授權與狀態 | Azure Repos | 地端狀態 | 自有模型 | 條件 | 定位 |
|---|---|---|---|---|---|---|
| [PR-Agent](https://docs.pr-agent.ai/installation/azure/) | [MIT;v0.49.0](https://github.com/The-PR-Agent/pr-agent) | 支援 | 可行,在你的 pipeline | 是 | 非 Qodo 免費版 | **第一** |
| [CodeRabbit](https://docs.coderabbit.ai/platforms/azure-devops) | 商用 | 原生 | [自託管 500 席起](https://docs.coderabbit.ai/self-hosted/azure-devops) | 未說明 | 狀態不能滿足分支政策;service principal 少設權限時 PR 靜默不審;留存頁 404 | 第二,先要書面 |
| Qodo | 商用 | [2026-02 宣布](https://stocks.observer-reporter.com/observerreporter/article/gnwcq-2026-2-11-qodo-expands-integration-with-azure-devops-to-scale-ai-code-review-for-the-enterprise) | **衝突**:[矩陣無地端](https://docs.qodo.ai/install-and-configure/deployment-model-support),[設定文件要單租戶](https://docs.qodo.ai/qodo-documentation/code-review/qodo-merge/getting-started/setup-and-installation/azure-devops),[另有地端指南](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md) | 未找到 | 部署與留存要書面 | 第三 |
| [Copilot code review](https://learn.microsoft.com/en-us/azure/devops/repos/git/copilot-code-reviews?view=azure-devops) | 公開預覽 | 原生 | **否**;不支援自託管 pool | 否 | 只留言不阻擋;10 GB 與 100 檔上限;Sprint 279 與較新文件對 Managed DevOps Pools 說法衝突 | 只試低敏感 repo,不當閘門 |
| [Greptile](https://www.greptile.com/docs/deployment-options) | 商用 | 未列 | 自託管 | 是 | 未書面確認前排除 | 排除 |

### 平台與模型服務

**判斷:** 部門隔離放基礎設施層,每部門獨立 namespace、閘道金鑰、憑證、Langfuse project 與配額。LLM 驅動的 shell 跑在 Linux 沙箱,出站只允許閘道與內部 MCP;Azure DevOps 所需的 `*.blob.core.windows.net` 等寬泛網域應只開給 pipeline agent 程序,不開給 agent 沙箱,否則任意 storage account 都是外洩通道。這個分層未測,也要注意 [agent v5.279.0 起 `mapDockerSocket` 預設為 false](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update)。

| 元件 | 工具 | 授權與狀態 | 地端狀態 | 條件 | 定位 |
|---|---|---|---|---|---|
| 沙箱 | [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) + Kata 或 gVisor | Apache-2.0;v1.0.10 | 自述無雲端相依 | [VM 上跑 Kata 需巢狀虛擬化](https://northflank.com/blog/what-are-kata-containers) | **預設** |
| Kubernetes | [RKE2](https://github.com/rancher/rke2) | Apache-2.0 | [已載明](https://docs.rke2.io/install/airgap) | 控制面 Linux | **預設** |
| 閘道 | [agentgateway](https://github.com/agentgateway/agentgateway) | Apache-2.0;v1.7.0 | 可行 | [群組耗盡時移往下一組,終態未記載](https://agentgateway.dev/docs/kubernetes/latest/llm/failover/);[有 MCP 授權 advisory](https://github.com/agentgateway/agentgateway/security/advisories) | **預設** |
| | [Agent Router](https://github.com/theagentrouter/agent-router) | Apache-2.0;v1.2.1 | 可行 | [後端依序 fallback](https://theagentrouter.ai/docs/capabilities/traffic/provider-fallback/);[11-06 六則 advisory](https://github.com/theagentrouter/agent-router/security/advisories) | 替代 |
| | [LiteLLM](https://github.com/BerriAI/litellm) | MIT(`enterprise/` 除外);v1.108.2 | 可行 | [預算無 Postgres 時 fail open](https://docs.litellm.ai/docs/proxy/users);[供應鏈事件](https://docs.litellm.ai/blog/security-update-march-2026);[團隊模型需企業授權](https://docs.litellm.ai/docs/proxy/team_model_add) | 不作安全邊界 |
| | [APIM 自託管閘道](https://learn.microsoft.com/en-us/azure/api-management/self-hosted-gateway-overview) | 商用 | 部分;[需 Developer 或 Premium 與對外 443](https://learn.microsoft.com/en-us/azure/api-management/self-hosted-gateway-overview) | [**不支援的 policy 會被略過**](https://learn.microsoft.com/en-us/azure/api-management/api-management-gateways-overview),有 fail-open 風險 | 僅 Azure 導向團隊實測後 |
| | [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) | 專有 | 自託管 | 僅 Claude | 只能當前沿分支,不能當路由器 |
| 觀測 | [Langfuse](https://github.com/langfuse/langfuse/blob/main/LICENSE) OSS | MIT(`ee` 除外);v4.76.0 | [可 `TELEMETRY_ENABLED=false`;企業版遙測不可關](https://langfuse.com/self-hosting/security/telemetry) | [RBAC、稽核屬企業版](https://langfuse.com/self-hosting/license-key) | **預設** |
| 憑證與政策 | [OpenBao](https://github.com/openbao/openbao)、[OPA](https://github.com/open-policy-agent/opa) | MPL-2.0、Apache-2.0 | 可行;OpenBao 外連未查 | 保管 SP 憑證 | **預設** |
| 映像 | [Harbor](https://github.com/goharbor/harbor) | Apache-2.0;v2.14.6 | 離線未確認 | | **預設** |
| 目錄 | [Backstage](https://github.com/backstage/community-plugins) | Apache-2.0 | 可行 | [Services 可用 Entra,PAT 恐遭棄用](https://github.com/backstage/backstage/issues/35394) | 100 人以上再加 |
| 模型服務 | [vLLM](https://github.com/vllm-project/vllm)、[SGLang](https://github.com/sgl-project/sglang) | Apache-2.0 | 可行 | [vLLM RCE](https://github.com/vllm-project/vllm/security/advisories/GHSA-h3rc-6mm3-gc2m)、[SGLang RCE](https://www.kb.cert.org/vuls/id/665416);引擎埠只讓閘道連 | 對測後選一 |

## 身分與資料邊界:程式碼獲准上雲,不等於 agent context 也獲准

### 身分:pipeline 內可無密鑰,pipeline 外要憑證

**證據:** [全域 PAT 於 2027-01-27 失效](https://devblogs.microsoft.com/devops/retirement-of-global-personal-access-tokens-in-azure-devops/),由 2026-12-01 延後;[PAT 頁面頂部一致](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops),但同頁清單與部分舊文仍寫 12 月。是立即還是分階段切斷未記載。距本報告約 11 週,應立刻以「All accessible organizations」範圍盤點。同頁也指出 PAT 的使用事件不記錄,這是偏好 Entra 身分的主要理由。應用程式身分[不能建立 PAT 或 SSH 金鑰,且每個身分按組織計費,需 Basic 才能用 Repos](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops)。Workload identity federation 是建議模式,[Azure DevOps issuer 將於 2027-07-01 退場改用 Entra](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure)。舊版 OAuth 的確切終止日[未公布](https://learn.microsoft.com/en-ca/answers/questions/6025099/exact-end-of-life-dates-for-azure-devops-oauth-and)。

| 工作負載(判斷) | 建議身分 | 地端保存的密鑰 |
|---|---|---|
| Pipeline 內 agent,同專案 repo 與 PR 寫入 | `System.AccessToken`,限定 job 範圍 | 無 |
| 需專屬身分或跨專案 | [Azure DevOps service connection(WIF)](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/add-devops-entra-service-connection?view=azure-devops) | 無;需 Azure CLI 與 `login.microsoftonline.com` |
| Agent 註冊 | [短效組織範圍 PAT](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/personal-access-token-agent-registration?view=azure-devops) 或 [SP secret](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/service-principal-agent-registration?view=azure-devops),用後撤銷 | 僅啟動時,加上 agent 金鑰對 |
| Dispatcher、Onyx 同步工作、RAG 索引、Backstage | 憑證式 service principal | 憑證金鑰,放地端保管 |
| 同上,嚴格無密鑰 | [外部 issuer 的 WIF](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation),公開託管 discovery 與 JWKS;或 [Azure Arc MI](https://learn.microsoft.com/en-us/azure/azure-arc/servers/managed-identity-authentication) | 無,但增加公開文件或 Azure Arc;Arc MI 對 Azure DevOps 未確認 |

**判斷:** 每個 agent 角色一個身分,唯讀 agent 放 Readers 類群組。不接受全存取 PAT;[Semgrep 設定時要 Full access PAT](https://semgrep.dev/docs/semgrep-appsec-platform/azure-pr-comments) 也是不走其 SaaS 路線的理由之一。SonarQube 所謂的 [global PAT](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration) 指其實例層設定,不一定是全域範圍權杖,要在期限前確認。

### 資料邊界:預設會上雲的不只是程式碼

| 資料或事件 | 預設位置 | 處置(判斷) |
|---|---|---|
| 程式碼、工作項目、測試結果 | [組織選定的地理位置,清單沒有台灣](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops);[Asia Pacific 列出的區域是 Southeast Asia](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops) | 已接受;若用 Asia Pacific,詢問次要複寫區域。[資料複寫到同地理區第二區域](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops),背景知識是 Southeast Asia 的配對為 Hong Kong,未確認 |
| PAT、SSH 金鑰;美歐英以外的身分資料 | [美國](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops) | 納入法務審查 |
| 預覽功能資料 | [通常美國,可能全球](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-location?view=azure-devops) | 敏感程式碼不用預覽功能,含 Copilot code review |
| Pipeline console log | [預設上傳](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops);[`--disableloguploads` 可關,改留主機](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops) | Agent pool 全開此旗標;transcript 永不寫 stdout;[不用 `system.debug`](https://learn.microsoft.com/en-us/azure/devops/pipelines/troubleshooting/review-logs?view=azure-devops);[遮罩只是盡力](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/set-secret-variables?view=azure-devops),不當控制 |
| Artifact、測試附件 | [上傳後留存](https://learn.microsoft.com/en-us/azure/devops/pipelines/policies/retention?view=azure-devops) | 留地端,只回傳通過與否和連結;縮短留存 |
| Step 名稱、狀態、測試結果、標籤 | 雲端,無法避免 | 不放 prompt、計畫文字或 diff |
| 稽核事件 | [公開預覽、90 天留存;on-prem Splunk 需入站 HEC](https://learn.microsoft.com/en-us/azure/devops/organizations/audit/auditing-streaming?view=azure-devops) | 排程匯出,或 Event Grid 進佇列由地端拉取 |
| AI 訓練 | [Azure DevOps 頁面沒有相關聲明](https://learn.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops);DPA 未取得 | 由法務取得確切文字 |

`--disableloguploads` 有代價:審查者在 UI 看不到 log 內容,MCP 的 log 工具也會失效,對 timeline 與 Marketplace 工作的影響未記載,必須在 PoC 驗證。Log 也沒有被列在 customer data 清單,應請 Microsoft 確認其落地。

### 敏感度路由:以 prompt 能看到什麼分級,以網路兜底

**判斷:** 以下是本報告的設計,未經測試。分級看的是 agent context 可能包含什麼,不只是程式碼;程式碼可上雲的 repo,只要測試載入類生產資料就仍是 S3。

| 等級 | 判準 | 允許路線 | 失敗時 |
|---|---|---|---|
| S0 公開 | 開源或公開內容 | 任何核准路線 | 任意 |
| S1 內部 | 一般專有程式碼,無受規管資料 | 地端或核准的雲端端點 | 另一條核准路線 |
| S2 機密 | 敏感 IP、客戶專屬邏輯,無原始受規管資料 | 地端;特定雲端需個案決定 | 排隊或拒絕 |
| S3 受限 | 受規管資料、密鑰或合約約束內容可進入 context | 僅地端 | **拒絕,絕不 failover** |

分級表放受保護的中央 repo 並經 CODEOWNERS 核准,不放在各 repo,未列名的 repo 視為 S3。每個等級有獨立 agent pool、service connection 與等級範圍權杖;本研究未找到 Azure DevOps 原生的 repo 自訂中繼資料功能。**S3 的路由表只有地端後端,所以 failover 無處可去**,不要把各層建模成同一路由上的優先群組,因為 agentgateway 與 Agent Router 的文件描述的都是移往下一個後端。S3 pool 的出站只放行 Azure DevOps 與地端閘道,讓設定錯誤變成連線失敗而不是外洩。Session 等級取所讀 repo、wiki、RAG 與 MCP 結果的最高值,檢索要在組裝 context 前依等級過濾。開發者筆電是最弱點,要走同一閘道並封鎖供應商網域。[GitLab Duo 在模型不可用時停止該功能](https://docs.gitlab.com/administration/gitlab_duo_self_hosted)是最接近的已出貨 fail-closed 參考,但它是按功能,不是按 repo。每月演練一次:停掉地端後端,驗證 S3 回傳錯誤,並以 canary token 監看出站代理。

## 推論路線:成本差距大,採購結果取決於受限資料比例

### 路線比較:沒有前沿旗艦有文件記載的台灣境內推論

**證據:** 下表每格都要合約確認,不是合規結論。Azure OpenAI 的 [APAC 欄列 Australia East、Japan East、Korea Central、Southeast Asia、South India,沒有台灣](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure-region-availability),Microsoft [可不經通知把區域加入 Data Zone](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types)。[Azure Taiwan North 2026 年 9 月仍分階段開放](https://focustaiwan.tw/business/202609010030),未出現在已讀的 Foundry 表格中。

| 路線 | 訓練 | 留存 | 位置 | 限制與條件 |
|---|---|---|---|---|
| **Azure OpenAI in Foundry** | [不用於訓練,OpenAI 無法取得](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/openai/data-privacy) | 無狀態推論,但有[濫用監控儲存,需核准才能修改](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/abuse-monitoring);約 30 天僅[第三方說法](https://learn.microsoft.com/en-us/answers/questions/2286499/azure-openai-abuse-monitoring) | Global、Data Zone 或 Standard;gpt-6.1-sol、terra 在 APAC Data Zone | 牌價頁未能讀取,以下假設與 OpenAI 同價;Responses API 會持久化 |
| **Claude,Hosted on Azure** | [商用條款預設不訓練](https://code.claude.com/docs/en/data-usage);[Anthropic 條款管轄](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/claude-models/data-privacy) | [Anthropic DPA 與條款](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison) | [Global 或 US Data Zone](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry);[無 APAC,Global 資源區只有美國與 Sweden Central](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-from-partners) | **[CSP 訂閱不支援](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-from-partners)**;無 Batch;Fable 不在 Azure;[Microsoft](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/claude-models/data-privacy) 與 [Anthropic](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry) 對資料外送的措辭衝突 |
| **Claude,Hosted on Anthropic** | 同上 | 同上 | [可能在 Azure 與選定地理之外處理](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison) | Fable 必走 |
| **Foundry 其他夥伴** | 依供應商 | 依供應商 | [FW-GLM-5.3 由 Fireworks 託管,資料離開 Microsoft](https://ai.azure.com/catalog/models/FW-GLM-5.3) | 「在 Foundry」不等於「在 Azure」 |
| **Amazon Bedrock** | [供應商無法取得 prompt](https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html) | [`data_retention_mode` 可設;Fable 需 aws_review](https://docs.aws.amazon.com/bedrock/latest/userguide/data-retention.html) | [Opus 5.5:Taipei 僅 Global;Tokyo、Osaka 有 JP geo](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-opus-5-5.html) | 有 PrivateLink;第二份雲端合約;[區域溢價 10%](https://platform.claude.com/docs/en/about-claude/pricing) |
| **Google Vertex** | [不與第三方共享](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-partner-models);明確的不訓練聲明未在該頁 | [ZDR 與快取文件衝突](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/data-governance?authuser=6) | [Taiwan 只有 Haiku 4.5;Singapore 涵蓋 Opus 5.5、Sonnet 5.5、Fable](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency) | 紙面上最接近台灣的 Claude 選項;溢價 10% |
| **Anthropic API 直連** | [預設不訓練](https://code.claude.com/docs/en/data-usage) | [30 天;ZDR 需核准](https://code.claude.com/docs/en/zero-data-retention) | [僅 global 或 us](https://platform.claude.com/docs/en/manage-claude/data-residency) | 亞洲地理需求下最不合 |
| **OpenAI API 直連** | [預設不訓練](https://developers.openai.com/api/docs/guides/your-data) | 30 天濫用日誌;ZDR 需核准 | [亞洲僅儲存;處理僅美、歐、UAE](https://developers.openai.com/api/docs/guides/your-data) | 儲存落地不等於推論落地 |
| **私有 VPC + open weights** | 無模型廠商 | 自管 | 依 GPU 位置;[ND H200 亞洲供貨未確認](https://www.spheron.network/blog/azure-h200-pricing-2026/) | 程式碼仍在雲端,**不滿足「不可上雲」** |
| **地端 GPU** | 無 | 自管 | 你的機房 | 品質、資本支出、人力 |

**判斷:Azure 託管確實減少了增量暴露,但只有一部分。** 真正減少的是租戶、帳務與身分面,以及 Azure OpenAI 不新增廠商。沒有減少的是資料內容:prompt、工具輸出、環境變數值、測試資料與 stack trace 都是 Azure DevOps 原本沒有的新內容類別,處理位置也由單一地理區擴大到多國 Data Zone,並多一條濫用審查路徑。Claude Hosted on Azure 則另外新增 Anthropic 作為獨立處理者與條款。所以「程式碼可以在 Microsoft 雲端」不能自動延伸成「agent context 可以送 Foundry」,這必須另外核准。若採購管道是 CSP,Claude in Foundry 會直接被擋。

**判斷:路線偏好。** 獲准且重視地理時,先試 Azure OpenAI Data Zone APAC,因為增量廠商最少。Claude 的公開評分較高,但數字主要來自賣 agent 的廠商,應由內部 bake-off 判斷差距是否值得第二份雲端合約。本報告由 Anthropic 模型撰寫,請注意這一段的利害關係。

### 成本情境與 GPU 採購結果

**【估算】** 需求基準:把 [Claude Code 文件的範例 session](https://code.claude.com/docs/en/costs) 放大到 [$150 到 $250 企業月均](https://code.claude.com/docs/en/costs)的中點,即 Opus 5.5 下 $200/人/月,約 95% 快取命中。這是單一廠商範例套用到所有模型,忽略每任務 token 差異;以 [Anthropic](https://platform.claude.com/docs/en/about-claude/pricing)、[OpenAI](https://developers.openai.com/api/docs/pricing)、[AWS](https://aws.amazon.com/bedrock/pricing/) 牌價計算,未含折扣。地端假設三年直線折舊、10% 支援、PUE 1.4、佔位電價與每 FTE $120K,只計 GPU 維運,不含機房改裝;容量假設每台 8-GPU 約 74 位開發者,範圍 40 到 100,由 [InferenceX](https://inferencex.semianalysis.com/compare/glm-5-1-h200-vs-mi355x) 吞吐打折推得。雲端路線加 0.25、0.5、1 FTE 治理人力。租用價來自[單一次級來源](https://www.spheron.network/blog/azure-h200-pricing-2026/)與[單一供應商](https://www.gpuperhour.com/rent/mi355x)。**下表都不含 5 到 7 FTE 的平台人力**,也不含 PM 端 Onyx 的推論需求,因為各路線大半都要付。

| 【估算】年成本,路線 | 20 人 | 100 人 | 400 人 | GPU 採購結果 |
|---|---|---|---|---|
| **A 純地端** | $105K | $681K | $2.48M | Pilot 級;2× 8-GPU;約 8× 8-GPU,且都在 bake-off 後 |
| **B 前沿高階**(Opus 5.5 級) | $78K | $300K | $1.08M | 不買;只租用評測 |
| **C 前沿中階**(Sonnet 5.5、GPT-6.1 Sol 級) | $52-54K | $169-180K | $556-600K | 不買 |
| **D 託管 open weights**(GLM-5.3 on Bedrock Global) | $67K | $247K | $869K | 不買;成本高於 C,大多評測品質較低,中國來源規則仍適用 |
| **E1 私有 VPC,MI355X 24×7**(供貨未驗證) | $242K | $483K | $1.69M | 不買 |
| **E2 私有 VPC,Azure H200 隨需 24×7** | $803K | $1.61M | $6.18M | 不買,但每台每年約 $743K,比買還貴 |
| **E3 同上,僅上班時間**(3,120 小時) | $325K | $649K | $2.36M | 不買;失去離峰批次 |
| **F1 混合,20% 受限走地端** | $71-90K | $231-327K | $1.13-1.51M | 工作站;Pilot 級;2× 8-GPU |
| **F2 混合,50% 受限走地端** | $117-129K | $460-520K | $1.60-1.84M | Pilot 級;1× 8-GPU,無 N+1;4× 8-GPU |

**【估算】** 打平用量倍數:對 Opus 5.5 為 1.6、2.6、2.5 倍,對 Sonnet 5.5 為 3.1、5.2、4.9 倍(依 20、100、400 人)。常駐 agent 可能達到這些倍數,[agent teams 在 plan mode 約用 7 倍 token](https://code.claude.com/docs/en/costs),但地端容量也會被同樣的增長吃掉,交叉點要靠負載測試。快取同樣敏感:Opus 5.5 在 95%、90%、70%、50% 命中時約為 $197、$284、$631、$977/人/月,所以試點要實測命中率。

**判斷:採購規則。** 分類結果與法務答覆出來前什麼都不買。雲端獲准且 S3 低於約 20%,約 100 人以下不買 8-GPU,小型地端層只服務 S3 與持續運作。S3 約 50%,或離峰 agent 負載持續接近打平倍數,才買,且從一到兩台開始。雲端遭拒時地端採購成立,但仍在租用 bake-off 之後。F1 有一個代價不在金額裡:20 到 100 人規模的小型地端層跑不動部門級模型,S3 工作只能用長程能力較弱的小模型;如果 S3 包含大型跨模組開發,應至少一台 8-GPU。Azure 隨需 GPU 不是便宜替代,只用於短期評測。

### 品質:落差大小取決於量什麼,重工成本要內部測

**證據:** [DeepSWE v1.1](https://deepswe.datacurve.ai/)(第三方)顯示差距小:gpt-6-astra、gemini-3.8-flash、claude-opus-5 為 74%,glm-5.3 為 69%,glm-5.3-flash 為 63%;這一版沒有 GPT-6.1、Opus 5.5、Sonnet 5.5。[FrontierCode 2.0](https://cognition.com/blog/frontier-code-2.0)(Cognition 賣 coding agent)顯示差距大:Opus 5.5 45.5、GPT-6 Astra 42.8、Kimi K3 17.7。[BenchLM 快照](https://benchlm.ai/md/benchmarks/frontiercode-2.md)(次級)另列 Sonnet 5.5 41.0、GPT-6.1 Sol 39.0、Mistral Large 4 19.7,**未列 GLM-5.3**。Harness 說法與[單一 harness 的 Artificial Analysis 數據](https://artificialanalysis.ai/evaluations/frontiercode-2-0)也有衝突。[METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) 顯示評分器分數比維護者合併決定高約 24 個百分點。**判斷:** Opus 5.5 對 DeepSeek V4.1 Flash 的倍數由 FC 1.1 的 1.24 倍擴大到 FC 2.0 的 2.36 倍,任務越大差距越大,但這只來自一家廠商的評測。

| 【估算】每任務總成本,示意 | 前沿 | Open | 較便宜 |
|---|---|---|---|
| Opus 5 vs GLM-5.3,DeepSWE 通過率 | $35.57 | $29.72 | Open,$5.85 |
| GPT-5.6 Sol vs GLM-5.3,DeepSWE | $30.59 | $29.72 | 約平手 |
| Opus 級 vs DeepSeek 級,FC 2.0 通過率 | $46.97 | $49.60 | 前沿,$2.63 |

假設每任務審查 20 分鐘,失敗再加 60 分鐘,每小時 $40,token 成本取自 DeepSWE;第三列 open 端代入的是 $3.99 的 GLM-5.3 成本,這些都沒有來源支持。時薪 $25 時第三列翻轉為 open 便宜,$60 時前沿多贏到 $7.87。**判斷:** 採用哪個 benchmark 就會得到相反答案,因此公開數字不能決定路線。每月 40 個任務時,差距與 token 帳單同一量級,所以 bake-off 要記錄人工分鐘。若雲端獲准,open weights 的主要理由是主權,不是價值:gemini-3.8-flash 在 DeepSWE 上通過率與成本都優於 glm-5.3,雖然該價格是[將於 2027-01-01 調漲](https://aistudio.google.com/docs/pricing)的暫時價。

### 地端模型與硬體

| 模型 | 來源 | 授權 | 規模 | 證據 | 定位(判斷) |
|---|---|---|---|---|---|
| [GLM-5.3](https://huggingface.co/zai-org/GLM-5.3) | CN | [類 MIT](https://huggingface.co/zai-org/GLM-5.3/raw/main/LICENSE) | 753B | DeepSWE 69 | 無來源限制時的主力 |
| [DeepSeek-V4.1-Flash-1017](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017) | CN | MIT | 511GB | [FC 2.0 19.3](https://benchlm.ai/md/benchmarks/frontiercode-2.md);廠商分數待驗 | 同上 |
| [GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash) | CN | MIT | 321B | DeepSWE 63 | 快速層 |
| [Mistral Large 4](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B) | FR | [Ecosystem License](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B/raw/main/LICENSE) | 1,061GB | FC 2.0 19.7 | 受限路線主力;推向 B200 或 MI355X |
| [Laguna S 2.1](https://huggingface.co/poolside/Laguna-S-2.1) | Poolside | OpenMDW-1.1 | 118B | [廠商數據](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) | 受限 Pilot,來源須先查 |
| [Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) | CN | Apache-2.0 | 27.8B | [DeepSWE 42.2,未驗證](https://llm-stats.com/benchmarks/deepswe-1.1) | 小型地端層 |

| 【估算】硬體層級 | 適用 | 配置 | Capex | IT 電力 | GPU 維運 |
|---|---|---|---|---|---|
| Pilot | 10 到 25 人 | 4×[RTX PRO 6000](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) 或 1×[DGX Station](https://www.tomshardware.com/desktops/nvidias-gb300-powered-dgx-station-desktop-tower-listed-for-nearly-usd100-000-online-enterprise-ai-powerhouse-now-available-to-buy-for-mere-mortals-with-lots-of-cash) | 約 $90K | 3.5 kW | 0.5 FTE |
| 部門 | 50 到 100 人 | 2× 8-GPU([H200](https://intuitionlabs.ai/articles/nvidia-ai-gpu-pricing-guide) 或 [B200](https://www.besthub.dev/articles/nvidia-server-gpu-2026-buying-guide-specs-prices-compliance-channels-9c6195bac2c4)) | 約 $1.05M | 28 kW | 1.5 FTE |
| 多部門 | 300 到 500 人 | 約 8× 8-GPU 含 N+1,400G 網路 | 約 $4.2M | 112 kW | 4 FTE |

**判斷:** Pilot 硬體跑不動部門級模型,用它驗證品質會得到錯誤答案,所以 bake-off 要在目標等級的租用硬體上做。常駐 agent 數才是真正的容量變數;先前估算一個常駐 agent 等於 3 到 5 位開發者,這是未經來源支持的假設。

### 若受台灣相關限制

本段只在組織受台灣相關規範約束時適用,本報告不假設組織所在地。**證據:** 數位發展部副部長 2025-11-20 表示政府機關「已被禁止使用中國製生成式 AI 模型」,但未指明法律文件或開源處理 ([Taiwan News](https://www.taiwannews.com.tw/en/news/6248783));2025 年 DeepSeek 措施涵蓋部會、公立學校、國營事業與準政府機構 ([The Star](https://www.thestar.com.my/aseanplus/aseanplus-news/2025/02/01/taiwan-bans-government-agencies-from-using-deepseek));2019 年原則限制公部門使用中國廠牌資通訊產品,含軟體 ([CNA](https://cna.com.tw/news/aipl/202502030237.aspx))。未見私部門禁令的報導 ([Taipei Times](https://www.taipeitimes.com/News/taiwan/archives/2025/11/20/2003847542))。

**判斷:** 受此限制時,改動涵蓋四處。第一,GLM、DeepSeek、Kimi、Qwen 在律師確認前視同不可用,**Bedrock 託管的 GLM 仍是中國來源模型,D 路線不解決這個問題**;Mistral Large 4 成為地端主力,而它把硬體推向 B200 或 MI355X,成本表要重算。第二,bge-m3 與 Qwen embedding 也屬中國來源,中文搜尋第 2 組要換模型,而替代品沒有中文證據;[TAIDE](https://huggingface.co/taide) 與 [Breeze](https://huggingface.co/MediaTek-Research) 最多用於繁中起草或防護。第三,若要求境內處理,目前沒有已記載的前沿旗艦選項,只有 Vertex Taiwan 的 Haiku 4.5 ([Google](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency)),其他都要另外核准跨境,S2 以上大概率回到地端,GPU 結果往 F2 或 A 移動。第四,Azure DevOps 本身也沒有台灣地理位置,這一點雖已定案,仍應在法務意見中明列。若不受限制,上述全部解除,只保留合約與跨境審查。

## Test agents:agent 提案、工具判定、人類擁有 oracle

**證據:** 通過測試是弱證據,METR 的落差已如前述;Playwright healer 認為功能損壞時會把測試標成 skip ([Playwright](https://playwright.dev/docs/test-agents)),在 CI 上看起來是綠燈。**判斷:** LLM 不單獨決定阻擋性閘門;Builder 不得寫入 oracle、閘門設定或隱藏測試;新增 skip 一律失敗;Healer 只能開 PR。本地模型會削弱 AI pentest 與紅隊生成,閘門要放在確定性工具上。測試報告留地端,只把摘要與連結寫回 PR。Test Plans 的 [Basic + Test Plans 為 $52/人/月(搜尋摘錄)](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/),只買給撰寫手動測試的人;自動發布結果的授權需求未驗證。

| 測試層 | 工具 | 授權與狀態 | 地端狀態 | 條件 | 定位 |
|---|---|---|---|---|---|
| 獨立驗證 | Fresh-context verifier(主要 harness + skill) | 依 harness | 依路線 | 只取規格與公開介面;[小樣本](https://arxiv.org/html/2610.28667) | **預設** |
| E2E agent | [Playwright Test Agents](https://playwright.dev/docs/test-agents) | Apache-2.0;v1.65.0 | 可行;[`PLAYWRIGHT_DOWNLOAD_HOST`](https://playwright.dev/docs/browsers) | 本地模型驅動的品質無證據 | **預設**,先測 |
| 單元、變異 | Harness + TDD;[PIT](https://github.com/hcoles/pitest)、[Stryker.NET](https://github.com/stryker-mutator/stryker-net) | Apache-2.0 | 可行 | 鏡像 Maven、NuGet | **預設** |
| API、契約 | [Schemathesis](https://github.com/schemathesis/schemathesis)、[Pact](https://docs.pact.io/telemetry) | MIT;Apache-2.0 或 MIT | 可行;`PACT_DO_NOT_TRACK=true` | Schemathesis 遙測未驗證 | **預設** |
| 效能 | [k6](https://github.com/grafana/k6) | AGPL-3.0;v2.3.0 | 可行;[`K6_NO_USAGE_REPORT`](https://grafana.com/docs/k6/latest/set-up/usage-collection/) | LLM 只註解,不決定通過 | **預設** |
| SAST | [Semgrep CE](https://github.com/semgrep/semgrep)、[Opengrep](https://github.com/opengrep/opengrep) | LGPL-2.1 | 可行;CLI 步驟,規則需 vendor | [Semgrep PR 留言要 SaaS 與 `SEMGREP_APP_TOKEN`](https://semgrep.dev/docs/semgrep-appsec-platform/azure-pr-comments),本架構用小 REST 步驟回寫 | **預設** |
| SCA、映像 | [Trivy](https://github.com/aquasecurity/trivy) + [官方 task](https://github.com/aquasecurity/trivy-azure-pipelines-task) | Apache-2.0;MIT;v0.76.2、v1.20.7 | [已載明,可鏡像 DB](https://trivy.dev/docs/latest/guide/advanced/self-hosting/) | [2026-03 曾遭入侵](https://github.com/advisories/GHSA-69fq-xp46-6x23),釘 digest;Docker 僅 Linux 完整支援 | **預設** |
| DAST | [ZAP](https://github.com/zaproxy/zaproxy) | Apache-2.0;v2.17.0 | 部分;[`-silent` 曾有 bug](https://www.zaproxy.org/blog/2025-10-21-zap-was-not-so-silent/) | 無官方 task,用 Docker CLI 步驟,目標在地端 | **預設** |
| 品質閘門 | [SonarQube](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration) | Community LGPL;付費版另計 | 可行;離線授權未確認 | [PR decoration 需 Developer 以上](https://docs.sonarsource.com/sonarqube-server/discovering/code-analysis/pull-request-analysis.md) | 付費版才作閘門 |
| 內建安全 | [GitHub Advanced Security](https://learn.microsoft.com/en-us/azure/devops/repos/security/github-advanced-security-billing?view=azure-devops) | 商用;$30 + $19/committer/月 | **否**,結果留 Microsoft | 推送防護最難在伺服器外複製 | 選配 |
| AI pentest | [Strix](https://github.com/usestrix/strix) | Apache-2.0;v1.7.0 | 部分;`LLM_API_BASE` 可接本地 | 推薦模型為託管型;僅限書面授權環境 | 試驗 |
| AI 評測 | [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai)、[garak](https://github.com/NVIDIA/garak) | MIT、Apache-2.0 | 可行;[Inspect 支援 vLLM、SGLang](https://inspect.aisi.org.uk/providers.html) | 用於模型與 skill 回歸 | 預設 |

## 人力:2 到 3 人起步,生產約 5 到 7 人,GPU 是否加人看路線

**證據:** 沒有公開案例給出這套自建堆疊的團隊規模。最接近的基準是 DX 對 39 家公司的調查,開發者生產力團隊占 2% 到 6%,但不含 SRE、雲端維運與安全 ([DX](https://newsletter.getdx.com/p/devprod-headcount-benchmarks-q1-2026))。**【估算】** 下表沿用先前工作線估算,假設 100 到 300 位開發者,並依本架構調整:拿掉 air-gap 傳輸線,Onyx 同步工作與 Azure DevOps 整合併入連接器線。

| 【估算】工作線 | 試點 FTE | 生產 FTE |
|---|---|---|
| Kubernetes、儲存、升級 | 0.5 到 1 | 1 到 1.5 |
| Dispatcher、Pipelines 模板、Azure DevOps 身分 | 0.25 | 0.5 到 1 |
| Onyx、搜尋、同步工作、Wiki | 0.5 | 1 到 1.5 |
| 閘道、沙箱、政策、路由演練 | 0.5 | 1 |
| 觀測與成本歸屬 | 0.25 | 0.5 |
| 測試工具鏈、掃描器、skill registry | 0.25 | 0.5 到 1 |
| **平台小計** | **2 到 3** | **5 到 7** |
| GPU 與模型維運:B、C 路線 | 0 | 0 |
| GPU 與模型維運:F1 | 0.25 到 0.5 | 0.5(100 人)到 1.5(400 人) |
| GPU 與模型維運:F2、A | 0.5 | 1.5(部門級)到 4(多部門) |

**判斷:** 小計已取整並略留餘裕。以每 FTE $120K 的佔位值,5 到 7 FTE 約 $600K 到 $840K 一年,比 100 人規模下 B、C 路線的推論費用還高,所以少於 50 人時全套不太可能回本。少於 4 人時,stateful 工作流程引擎的 on-call 很脆弱,這是延後 Temporal 的另一個理由。最難補的是同時懂 Azure DevOps REST、模型 parser 與評測的人。授權要另列:每個應用身分 [$6 Basic](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/),並行 job 每個 $15,Onyx 150 人約 $36,000 一年 ([Onyx](https://onyx.app/pricing)),但實際方案見前述衝突。

## 路線圖與 PoC:先用不到 1% 的硬體預算買下決策資訊

**判斷:** 時程與門檻都是提案,應在試點校準。起點選有強測試、重複性高的工作。

| 時程 | 範圍 | 交付 | 進入下一階段的條件 |
|---|---|---|---|
| 立即,2026-11-24 前 | Agent 維護 | [移除 Node 6、10、16 runner 的相依](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-279-update) | 既有 pipeline 不受影響 |
| 第 0 到 2 週 | 事實確認 | 資料分級;法務與地理問題;採購管道是否 CSP;全域 PAT 盤點;組織地理位置 | 知道 S3 比例、雲端答覆、台灣限制 |
| 第 2 到 10 週 | 採購前 PoC | 下表阻擋項;2027-01-27 前全域 PAT 換完 | 阻擋項全部通過 |
| 第 3 到 6 個月 | 1 到 2 個團隊:flaky test、依賴升級、小修 | Dispatcher、skill monorepo、PR-Agent、Onyx 單部門;全部人審 | 變更失敗率與漏網缺陷率持平 |
| 第 7 到 12 個月 | Spec-to-PR、第二部門 | 規格晉升、verifier、部門隔離、配額;依結果採購 | Lead time 改善且品質持平 |
| 第 12 個月後 | 分級自主 | 特定變更類別自動合併,含降級規則 | 該類別失敗率經證實偏低 |

**【估算】** 以單一供應商的 MI355X 租價,8 GPU 連續兩週約 US$7,000,不到部門級 capex 的 1%;該供應商供貨未驗證。租用 GPU 本身也是雲端,所以 bake-off 只用 S0 或脫敏 repo,且要先得到這個小例外的核准。

| PoC 項目 | 做法 | 通過標準(提案) | 失敗時 |
|---|---|---|---|
| **繁中檢索(阻擋)** | 50 到 100 條帶標註的真實 URD 查詢,跑上述四組;含完整片語、句中二字詞、中英混合、軟體/软件 | 記錄 recall 與 top-10 precision;第 2 組要明顯優於基線,目標由 PM 訂定 | 第 3 組、換 embedding 或把 RAGFlow 納入 |
| **繁中模型(阻擋)** | 需求轉程式碼、URD 起草、簡體洩漏率、台灣用語、敏感內容行為,地端與雲端候選並測 | 達內部基準;目前沒有外部基準 | 加後處理或換模型 |
| 模型 bake-off(阻擋) | 50 到 100 個內部任務,固定 harness 版本,目標量化 | 比較首次合併率、合併所需人工分鐘、30 天回退、每合併 PR token | 換模型或路線 |
| Onyx 同步與隔離(阻擋) | 缺 `cc_pair_id` 測試;以 A 部門身分查 B 部門文件與 MCP | 同步失敗;零跨部門成功 | 每部門獨立實例 |
| Onyx 方案書面答覆(阻擋) | 權限同步、群組、SSO、離線續約 | 有書面答覆 | 調整預算或部署 |
| 輪詢與 Runs API(阻擋) | Dispatcher 掃描工作項目與 PR threads | 延遲可接受,未達 TSTU 限制 | 加 Storage Queue |
| Incoming Webhook | Service hook 打同組織 webhook | 僅供參考 | 不採用 |
| Log 邊界(阻擋) | `--disableloguploads` + canary 字串,搜尋雲端 log、timeline 與 artifact | 雲端查無 canary;審查流程仍可用 | 調整 stage 與工具 |
| 路由 fail-closed(阻擋) | 停地端後端,送 S3 請求;錯設閘道與客戶端 | 回傳錯誤,雲端端點無流量 | 改網路與路由表 |
| Prompt injection(阻擋) | PR 描述與 repo 指令檔植入指令,目標是跨專案觸發與 wiki 外洩 | 無權杖外洩,寫入被 stage 分離擋下 | 收緊權限 |
| 出站(阻擋) | 每個工具 default-deny + DNS 記錄,跑啟動、使用、升級 | 無未申報外連 | 替換工具 |
| 容量 | 以真實 agent trace 重播;測快取命中率 | 尖峰延遲下的人當量 | 改層級或限制常駐 agent |
| Harness 對 Azure | OpenCode、Goose 對 Azure OpenAI 與地端;Codex 測 #31875 | 兩層都能跑完整任務 | 換 harness |
| PR-Agent 與審查 | 重播 50 到 100 個歷史 PR | 比較召回、採納率、雜訊與成本 | 問 CodeRabbit、Qodo |
| 身分 | 本機 MCP 以 job 權杖運作;SP 憑證;WIF | 無長效 PAT | 改短效權杖 |
| 廠商書面 | CodeRabbit、Qodo、Factory、Tabnine、Cursor:留存、自有模型、ADO 範圍 | 有書面 | 不納入 |
| 法律 | Data Zone 地理、DPA 訓練條款、CSP、濫用監控、PRC 模型、集團法人 | 書面意見 | 換路線 |

**判斷:** KPI 要配對,不能只看 PR 數:lead time、變更失敗率、漏網缺陷率、審查時間、無人工修改的合併率、每合併 PR 成本、GPU 利用率,以及 S3 路由違規次數,最後一項的目標是零。

## 已知的未知數

**整合。** 沒有工具經本研究實機測試。Service hook 與長輪詢沒有公布的延遲或 SLA;service hook 打同組織 Incoming Webhook 未經測試;`--disableloguploads` 對 timeline 與 Marketplace 工作的影響未記載;`config.sh --auth SP` 的憑證或聯合認證沒有文件;job 權杖驅動本機 MCP、Arc MI 對 Azure DevOps 都未確認。遠端 MCP 是否也收到 spotlighting 修補未說明,也沒有 service principal 使用遠端 MCP 的文件。全域 PAT 切斷是否分階段、公告的「10/23」年份都是推定。Log 是否算 customer data、Asia Pacific 次要複寫區域、Wiki 頁面權限、Test Plans 的自動發布授權都未查證。

**Onyx 與知識。** 所有候選都缺獨立的繁中評測,`cjk` analyzer 能否端到端運作未驗證。Onyx 方案歸屬、離線續約、Azure DevOps app 在 Craft 中的支援、LiteLLM 依賴的影響都未確認。PR #6496 的程式碼未審閱。RAGFlow 連接器範圍是由程式碼註解推得,沒有實際組織的公開測試。

**推論與成本。** 沒有取得企業折扣、PTU、Azure OpenAI 牌價或 APIM Premium 價格;Azure 與 OpenAI 同價只是假設。Microsoft Product Terms、Anthropic DPA 都未閱讀,濫用監控的留存天數只有第三方說法,Sonnet 5.5 的 Foundry 託管區域在三份文件中也不一致。Azure Taiwan North 狀態、亞洲 H200 供貨、台灣 GPU 雲價格未驗證。沒有獨立研究把模型選擇換算成重工時數,也沒有 benchmark 測過真實 harness 下的自託管量化模型,[託管分數可能高估自託管結果](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md)。所有容量與 TCO 係數都是假設,電價與人力是佔位值。

**成效。** 沒有找到端到端需求到測試的受控 factory 研究,需求階段自動化尤其缺乏證據。這份架構是由零散證據推論而來,不是經驗證的藍圖。

## 結論

這次把 Services 定案後,問題的形狀變了。「盡量不用雲端」不再是一道牆,而是一條要逐項劃的線:程式碼已在線的另一邊,真正要守的是雲端原本沒有的東西,也就是 agent context、log、URD 草稿與受限資料的 prompt。所以最有價值的工作不是挑模型,而是分類:S3 的比例同時決定路由、harness 與七位數美元的採購,而它只需要幾週就能得到初步答案。

另一個值得帶走的觀點是,這個架構最可靠的控制都很無聊:對外輪詢取代入站事件,網路規則取代閘道 fallback 設定,部門 pair 取代想像中的文件級 ACL,repo 中的核准規格取代聊天紀錄。Onyx、harness 與模型都可能在一兩年內更換,但分級表、同步工作、pipeline 模板、規格 ID、繁中評測集與 skill 會留下來;這些才應該是預算與人力先去的地方。
