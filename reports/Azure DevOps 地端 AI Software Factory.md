# 以 Azure Pipelines 為骨幹自建 AI Software Factory

建議的做法是保留 Azure DevOps,以 **Azure Pipelines 加一個地端 webhook receiver 當確定性協調器**,PM 用 BookStack(加 CJK 搜尋 sidecar)探索,核准後的 URD 以穩定 ID 晉升到 Azure Repos,coding agent 一律以 headless CLI 在自託管 agent pool 的沙箱中執行,測試由 fresh-context agent 提案、確定性工具判定。先確認你用的是 Server 還是 Services,因為這決定整條整合路徑:**Microsoft 官方 MCP 只支援 Services 且表明沒有地端計畫,`az devops` 也不支援 Server,本研究找到的 Microsoft AI 功能沒有一項記載可用於 Server**,所以 Server 路線的整合層必須自己擁有。模型方面,私部門可用 GLM-5.3 或 DeepSeek-V4.1-Flash-1017;若受台灣公部門相關規範約束,中國來源權重應視為不可用,候選改為 Mistral Large 4、Nemotron 3 Ultra 與 Laguna S 2.1。**地端的正當理由是主權,不是成本**:本報告估算部門級約 US$568/人/月,高於前沿模型的 US$150 到 250,而且公開評測顯示 open-weight 在長程工作與合併品質上仍明顯落後。因此不要先買 GPU:先以租用硬體跑 50 到 100 個內部任務的 bake-off,並保留一條只給低敏感資料的雲端例外路徑。所有工具都未經本研究實機測試,本報告是評估起點,不是定論。

## 建議架構:Pipelines 當骨幹,agent 與模型都可替換

**閱讀方式。** 「**證據**」指可查來源的事實,「**判斷**」指本報告的推論,「【估算】」指以明示假設計算的數字;表格中名為「判斷」「預設」「定位」的欄位都是推論。地端狀態用五個詞:**已載明**(廠商文件描述無網路運作)、**可行**(架構允許、未見矛盾,但無廠商聲明)、**部分**(離線會失去指名功能)、**否**(需要雲端控制面)、**未確認**(只有次級或互相衝突的說法);「未稽核」表示本輪研究沒有檢查。授權、星數與版本由研究者於 2026-11-07 從 GitHub 或 Hugging Face API 讀取。沒有任何工具做過封包檢查,「已載明」不等於零外連。利益揭露:本報告由 Anthropic 的模型撰寫,比較中包含 Anthropic 模型,Claude Code 成本數字也來自 Anthropic 文件,請自行加權。

**判斷:** 下表是建議架構。設計依據是模式研究的一致方向:Google Research 顯示獨立多 agent 把錯誤放大至 17.2 倍,而循序規劃任務上每種多 agent 變體都退步 39% 到 70% ([Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/));Stripe 的量產做法也是確定性節點穿插 agent 節點,CI 最多兩輪就退回人類 ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2));Faros 的廠商觀察資料則顯示高採用組審查時間約 5 倍、每週部署反降 11.7% ([Faros](https://faros.ai/research/ai-acceleration-whiplash))。沒有任何來源以這個組合測得成效。

| 層 | 職責 | 地端預設 | 替代 | Azure DevOps 接點 |
|---|---|---|---|---|
| L1 知識與需求 | PM 探索、URD、核准規格 | BookStack + OpenSearch 或 Meilisearch sidecar;Spec Kit 或 OpenSpec | Azure DevOps Wiki 單獨使用;Docmost Business | 核准版在 Azure Repos(可發布為 code wiki),Boards 持工作項目 |
| L2 檢索 | 依權限檢索文件與程式碼 | Onyx 付費版 + 自建連接器;codebase-memory-mcp + Zoekt + Repomix | RAGFlow v1.0;Sourcegraph Enterprise | 以內部 Git mirror 工作建索引,不依賴各工具的 ADO 連接器 |
| L3 協調 | 階段、閘門、重試、核准 | Pipelines YAML + Environments + Build Validation + webhook receiver | 需要時加 Temporal;Dapr Workflow | Service Hook、Run Pipeline REST |
| L4 Agent 執行 | 設計、實作、重構、文件、遷移 | OpenCode 為主、Goose 為次,私有 skill monorepo | Factory Droid 或 Tabnine(書面確認後) | 自建或社群 MCP + git + REST |
| L5 驗證 | 審查與測試 | PR-Agent、fresh-context verifier、Playwright Test Agents、確定性工具 | Qodo(確認後) | Build Validation、Test Plans、Publish Code Coverage |
| L6 平台 | 沙箱、閘道、遙測、身分、映像 | RKE2、agent-sandbox + Kata 或 gVisor、agentgateway、Langfuse OSS、OpenBao + SPIRE + OPA、Harbor | OpenShift、Agent Router、Coder | Azure Artifacts 可兼套件 feed |
| L7 模型 | 推論與配額 | vLLM 或 SGLang,依組織資格選模型 | llm-d(超過一兩台節點) | 無;閘道依 repo 分級路由 |

**判斷:** 流程上,所有寫入都走 Azure DevOps 既有的稽核表面。Pipelines 無法被 PR 留言觸發,YAML `pr:` 觸發器也不適用於 Azure Repos Git,必須用 Build Validation 分支政策 ([PR-Agent 文件](https://docs.pr-agent.ai/installation/azure/),[Semgrep](https://semgrep.dev/docs/kb/semgrep-ci/azure-self-hosted-ubuntu)),所以留言指令與工作項目事件都要經過 receiver。下列 agent stage 的切法借自 [ado-aw](https://github.com/githubnext/ado-aw) 的 Agent、Detection、SafeOutputs 設計:這個工具本身不適合 Server,但它的權限分離值得照抄。

```
Boards 工作項目(標籤或狀態變更)
  → Service Hook → 地端 webhook receiver
  → Run Pipeline REST(帶 work item ID)
  → 自託管 agent pool 的沙箱:唯讀 Agent stage → Detection stage → 持寫入權杖的 Executor stage
  → git push + REST 建 PR
  → Build Validation(確定性閘門 + fresh-context verifier + PR-Agent 建議)
  → 人類審查合併 → Environments 核准與部署 → 部署紀錄回連工作項目
```

| SDLC 階段 | Azure DevOps 觸發 | Agent 與 skill(判斷) | 權威狀態 | 人類閘門 | 確定性檢查 |
|---|---|---|---|---|---|
| 探索、URD 草稿 | 人工 | PM 助理 agent,只能寫草稿區;BMAD BRD 模板改作 URD | BookStack | PM 審閱 | 生成內容標註並附來源 |
| 核准與晉升 | 工作項目狀態 | 轉換 agent 開規格 PR | Azure Repos + Boards | 需求負責人核准 PR | 每條驗收條件有 ID 且可測 |
| 設計 | 工作項目標籤 | 主要 harness + Spec Kit 或 OpenSpec、domain modeling skill;規劃類任務可依分級升級模型 | Azure Repos | 依風險核准 ADR | 需求對任務覆蓋 |
| 實作 | Run Pipeline | 單一寫手 + TDD skill,在沙箱中 | 分支、Pipeline run | 無(受閘門約束) | Lint、型別、單元、secret scan |
| 審查 | Build Validation | PR-Agent 接本地模型,與作者分離,先建議不阻擋 | PR threads | 人類合併 | SAST、SCA、SonarQube、測試竄改、追溯 trailer |
| 驗證 | Build Validation 或排程 | Fresh-context verifier 只取規格與公開介面;Playwright Test Agents | 受保護測試路徑、Test Plans | 測試 oracle 變更 | Mutation、Pact、Schemathesis、E2E |
| 發布 | Environments | Agent 只提案,部署由 pipeline 執行 | 環境部署紀錄 | Approvals and checks | Trivy、ZAP、k6、SLO |
| 維護 | 排程(離峰) | 批次 agent 做依賴升級、flaky test、遷移;Healer 只開 PR | PR | 依變更類別 | 新增 skip 一律失敗 |

**證據與判斷:追溯。** `AB#` 是 GitHub 整合語法,文件明列其他 Git repo 不支援 ([Link GitHub](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github)),不要拿來用在 Azure Repos。Environments 能顯示每次部署的 commit 與工作項目 ([Environments](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops-server)),Test Plans 能以 YAML pipeline 執行自動測試 ([Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver))。本報告提案以 Azure Repos 原生工作項目連結搭配 `Work-Item: 1234`、`Implements: FR-012` 兩個 trailer,由 Build Validation 強制檢查孤立需求與未知 ID。Agent 回報進度時要注意單一工作項目 REST 留言上限 1,000 則 ([Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)),應彙總寫入而不是逐步留言。

**判斷:** 下表是需要你拍板的決策,預設偏向低鎖定與可退場。

| 決策 | 預設 | 推翻預設的條件 |
|---|---|---|
| Server 或 Services | 先確認現況;目標狀態為 Server | 已在 Services、敏感程式碼已獲准上雲:留在 Services,只把推論與 agent 放地端 |
| 留在或離開 Azure DevOps | 留下並自持整合層 | Boards 與 Test Plans 用量淺,且要廠商支援的地端 agent 平台:評估 GitLab Self-Managed |
| 隔離程度 | Default-deny 出站 + 鏡像 feed,不做實體 air-gap | 法規或合約明文要求實體隔離:每年加 1 到 2 FTE 與單向傳輸流程 |
| 協調器 | Pipelines + receiver,暫不用 Temporal | 試點出現多日等待、跨 repo saga 或重試狀態,Pipelines 明顯處理不佳 |
| PM wiki | BookStack 寫草稿,Azure Repos 存核准版 | Azure DevOps Wiki 通過繁中實測且 PM 接受其編輯體驗:省掉 BookStack |
| 檢索 | Onyx 付費版 + 自建連接器 | 預算不允許:每部門一個社群版實例,接受只有粗粒度隔離 |
| Harness | OpenCode 為主、Goose 為次 | Bake-off 差距明顯,或 Tabnine、Factory 書面確認支援 Azure DevOps Server 且離線授權可接受 |
| 程式碼審查 | PR-Agent | 對 Server 版本實測失敗,或 Qodo 書面確認 Server 支援且召回明顯較佳 |
| 模型路線 | 依組織資格分流(見模型節) | 律師確認本地中國來源權重可用或不可用 |
| 硬體 | 先租後買;Pilot 級開始,2 台 8-GPU 為部門目標 | Bake-off 品質不足、實測利用率過低,或 S3 資料很少:改用私有 VPC 推論 |
| 雲端例外 | S0 與 S1 可走例外路徑,S3 永不 | 政策零例外:關閉路由,接受品質與重工成本 |
| 規模下限 | 150 人以上才建全套 | 少於 50 人:精簡為 BookStack、Git skill、Pipelines、VM runner、Langfuse、閘道(依據見人力節) |

**相較先前建議的修正。** GitHub 路徑不再適用,gh-aw 與 Copilot 路線隨之退出。Outline 不再是 wiki 預設,因為它沒有 LDAP,離線 AD 環境要先多架 OIDC 服務。Temporal 從預設降為條件式。snyk/agent-scan 因必須把 skill 內容送往 Snyk 而移除 ([agent-scan](https://github.com/snyk/agent-scan))。GitLab 的離線部署已有文件(需 18.9+ 與 offline cloud license),先前說法過時 ([GitLab](https://docs.gitlab.com/ee/administration/gitlab_duo_self_hosted/offline_deployment/))。Factory 的 air-gap 僅指 Droid airgap build,Factory Private 是 Factory 預設營運控制面的 Private Preview ([airgap 文件](https://docs.factory.com/enterprise/airgapped-deployment.md),[Factory Private](https://factory.com/news/factory-private))。SonarQube Community 不足以做 PR 閘門,據論壇來源 PR 分析與 decoration 從 Developer Edition 起 ([Sonar 社群](https://community.sonarsource.com/t/pullrequest-decoration-quality-gate/46232))。Continue 已轉唯讀,不應再列候選 ([Continue](https://github.com/continuedev/continue));Serena 應用程式是 GPL-3.0-or-later,不是 MIT ([Serena](https://github.com/oraios/serena))。

## Server 與 Services 是兩條不同的整合路徑

### Azure DevOps Server:核心 ALM 健康,AI 全靠自己

**證據:** Server 採 Modern Lifecycle,支援起始日 2025-12-09,目前無終止日 ([Microsoft Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/azure-devops-server))。25H2 世代的最新 patch 是 2026-10-13 的 Patch 9,大致每月一次 ([Server release notes](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver))。26H2 RC 於 2026-10-27 發布,內容是 Boards UI、Test Run Hub 與 pipeline 除錯,公告完全沒提 AI、Copilot、MCP 或 Advanced Security,也沒有 RTW 日期 ([Azure DevOps Blog](https://devblogs.microsoft.com/devops/?p=73599))。**判斷:** 升級 26H2 是 Boards 與 Test Plans 的決定,與 AI 無關。

**證據:** 四個 Server 限制會直接影響設計。第一,**NTLM 風險**:libcurl 排定 2026 年 9 月移除 NTLM,依賴 NTLM 的 Git over HTTPS 會失敗,IIS Negotiate 也可能悄悄退回 NTLM,Microsoft 建議改用 Kerberos 或 SSH ([Azure DevOps Blog](https://devblogs.microsoft.com/devops/upcoming-change-ntlm-removal-in-git-libcurl-impact-to-azure-devops-server-customers/));實際是否如期出貨未查證。第二,**REST API 版本上限**:社群指南記載 Server 2022 卡在 7.0,超出就回 out-of-range 錯誤 ([Tiberriver256](https://github.com/Tiberriver256/mcp-server-azure-devops/blob/main/docs/authentication.md)),25H2 與 26H2 的上限未查證。第三,**服務帳號**:PAT 限單一 collection、走 HTTP Basic,IIS Basic Authentication 必須關閉 ([PAT 文件](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-server));Backstage changelog 指出 service principal 與 managed identity 不適用於 Server ([Backstage](https://backstage.io/docs/releases/v1.15.0-next.1-changelog))。第四,**Webhook 位址**:文件稱不能指向 loopback 或特殊範圍 IP,且連到涵蓋 RFC1918 的 IANA 清單,未限定只適用 Services ([Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops-server)),而 Qodo 的地端指南又說 Server webhook 在內網 ([Qodo](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md))。**判斷:** 這個矛盾直接影響 receiver 設計,是 PoC 第一天要測的項目;失敗就改輪詢或給 receiver 一個可路由的位址。

### 差異總表

| 能力 | Services | Server | 影響 |
|---|---|---|---|
| 官方 MCP | [本機 + 遠端預覽](https://github.com/microsoft/azure-devops-mcp);[Learn 僅標示 Services](https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview) | **否**;[FAQ](https://github.com/microsoft/azure-devops-mcp/blob/main/docs/FAQ.md) 與[維護者](https://github.com/microsoft/azure-devops-mcp/discussions/996)均稱無計畫 | Server 用社群或自建 MCP |
| `az devops` | 支援 | [**否**](https://learn.microsoft.com/en-us/azure/devops/cli/?view=azure-devops) | 所有 agent recipe 改寫為 REST |
| Copilot for Boards | [GA,但目標是 GitHub repo](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/boards/sprint-268-update) | 未記載 | 程式碼留在 Azure Repos 時兩邊都用不上 |
| Copilot code review | [公開預覽;只留言不阻擋;不支援自託管 agent;10 GB 與 100 檔上限;預算只通知](https://infoq.com/news/2026/09/copilot-code-review-azure-repos) | 未提及 | 與地端推論不相容 |
| Azure SRE Agent | [Azure 託管,wiki 建向量索引](https://learn.microsoft.com/en-us/azure/sre-agent/ado-connector) | 未提及 | 內容會進入 Azure |
| GHAS for Azure DevOps | 支援 | [合作夥伴頁](https://innfactory.de/en/cloud/azure/products/github-advanced-security-devops/)稱不支援,無 Microsoft 一手聲明 | 以 Semgrep 或 Opengrep、Trivy 取代 |
| 身分 | Entra、service principal | [AD、Windows 驗證](https://learn.microsoft.com/en-us/azure/devops/organizations/security/about-security-identity);未見 Entra 登入說明 | 服務帳號 + 低權限 PAT |
| OpenHands 整合 | [Enterprise 版,需連 `login.microsoftonline.com`](https://docs.openhands.dev/enterprise/integrations/azure-devops) | [明文不涵蓋](https://docs.openhands.dev/enterprise/integrations/azure-devops) | Server 不把 OpenHands 當 ADO 整合 |
| ado-aw | [僅 Copilot engine,可 BYOK](https://github.com/githubnext/ado-aw/blob/main/docs/engine.md) | [`ado-proxy` 明文僅 Services](https://github.com/githubnext/ado-aw/blob/main/docs/ado-proxy-design.md);無公開案例 | 33 星且未達 1.0,兩邊都只列觀察 |
| Semgrep PR 留言 | [支援](https://semgrep.dev/docs/semgrep-appsec-platform/azure-pr-comments) | 文件僅稱 Cloud;[掃描可跑自託管 agent](https://semgrep.dev/docs/kb/semgrep-ci/azure-self-hosted-ubuntu) | Server 上自己以 REST 回寫 |
| Search | 託管 | [修改版 Elasticsearch 7.17.0 + OpenJDK 11,無 HTTPS,磁碟最多達 repo 150%](https://learn.microsoft.com/en-us/azure/devops/project/search/install-configure-search?view=azure-devops-server) | 維運與 CJK 都是風險 |
| 兩邊都有 | | [Environments、approvals](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops-server)、[Webhooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops-server)、[Test Plans YAML、Mermaid、diff coverage](https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver)、[Backstage 外掛](https://github.com/backstage/community-plugins/blob/main/workspaces/azure-devops/plugins/azure-devops-backend/README.md)、[SonarQube](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration) | 骨幹可共用 |

**證據:** Server 上的社群 MCP 中,[Tiberriver256](https://github.com/Tiberriver256/mcp-server-azure-devops)(MIT,399 星,v0.1.54,2026-11-02)最可信,README 針對 Server 定位,但僅支援 PAT;[mcp-devops-onpremise](https://pypi.org/project/mcp-devops-onpremise)(Apache-2.0)是唯一列出 NTLM 的選項;[@ahouben fork](https://github.com/ahouben/azure-devops-mcp) 看起來已停滯。沒有一個經過 Server 25H2 測試。**判斷:** 先用 Tiberriver256 搭專用低權限 PAT 並明確設定 API 版本。這些單一維護者的 MCP 握有寫入憑證,因此正式環境較好的做法是**自建一個小型 MCP,只暴露 factory 需要的動詞**;這也是 Server 路線上真正屬於你的整合資產。

### 如果你現在在 Services

**判斷:** 在 Services 上,程式碼已在 Microsoft 雲端,「盡量不用雲端」的真正問題變成「還能把多少留在地端」。留在 Services 的好處是具體的:官方 MCP、Entra 服務身分、OpenHands、Qodo 單租戶與 CodeRabbit 的路徑都打開了。但仍有三件事沒有改變:Copilot for Boards 不碰 Azure Repos,Copilot code review 不支援自託管 agent,推論與 agent 沙箱還是要自己放地端。

**判斷:** 同一份架構在 Services 上要改三點。第一,官方 MCP 取代社群 MCP,但要注意遠端版需要 Entra,而本機版 README 稱未來會被遠端取代 ([README](https://github.com/microsoft/azure-devops-mcp)),對隔離網路是中期風險。第二,service hook 由雲端發出,地端 receiver 必須能被抵達,這會引入入站路徑,較好的替代是由地端輪詢,或由自託管 agent 主動拉工作。第三,全域 PAT 即將退役,同一頁文件給了 2027-01-27 與 2026-12-01 兩個日期,須確認 ([PAT 文件](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops))。

**未驗證但最關鍵的一點:** 本研究沒有查 Services 移回 Server 的路徑。就撰寫者的背景知識,官方遷移工具的方向是 Server 到 Services,反向可能需要自行搬移 repo 與工作項目,且歷史與連結會有損失,必須先向 Microsoft 確認。若屬實,「從 Services 回到 Server」的成本可能不低於遷往 GitLab,因此不能預設它是便宜選項。

### 留下或遷移的分岔

**判斷:** 除非有下表右側的明確觸發條件,否則留在 Azure DevOps。留下的代價是整合層自持,但試點資產大多可攜:規格、agent 指令、skill 與 CLI wrapper 都能帶走,只有 receiver 與 REST 追溯步驟要重寫。

| 你的狀況 | 最可能的選擇 | 主要證據與代價 | 轉向的觸發條件 |
|---|---|---|---|
| Boards、Test Plans、稽核依賴深;Windows/AD 環境 | **留在 Azure DevOps Server**,加 PR-Agent、MCP、CLI agent | 自持整合;Microsoft 不提供 AI | 社群 MCP 在升級時持續損壞;認證變更持續打斷服務帳號 |
| 已在 Services,只是不喜歡雲端 | **留在 Services**,自託管 agent + 本地模型 | 整合最順,程式碼仍在雲端 | 法遵把程式碼納入硬性地端範圍 |
| 硬性零雲端,且要廠商支援的 agent 平台 | **評估 GitLab Self-Managed + Duo Self-Hosted** | [離線需 18.9+ 與 offline license](https://docs.gitlab.com/ee/administration/gitlab_duo_self_hosted/offline_deployment/),[離線授權需 add-on,Agent Platform 自 18.8 GA](https://docs.gitlab.com/administration/gitlab_duo_self_hosted/);價格未研究;[先前查到自託管模型仍計 credit](https://docs.gitlab.com/subscriptions/gitlab_credits),離線情境待確認 | 報價與 ADO 遷移成本可接受 |
| 小團隊、願意全部自持 | Gitea 或 Forgejo,另找 Boards 與 Test Plans 替代 | 本研究未研究其 AI 狀態,證據最少 | 只有在授權成本是主要限制時 |

**證據與判斷:** GitLab 是唯一找到**同時有離線文件與自託管 agent 平台**的 SCM 廠商,它解決的是廠商支援,不是模型問題:19.0 新增的 open 模型是 Devstral 2、GLM-5.1、Kimi-K2.6、MiniMax-M2.7,隔離網路只能走本地 open 模型 ([GitLab](https://about.gitlab.com/blog/more-ai-models-for-duo-agent-platform-self-hosted/))。其中三個是中國來源,而 MiniMax-M2.7 的授權在未獲書面許可下禁止商用 ([授權](https://huggingface.co/MiniMaxAI/MiniMax-M2.7/raw/main/LICENSE))。所以遷移不會消除模型節的任何限制。GitHub 路徑已排除;回頭看 GHES 也解不了地端問題,其 Copilot 多數功能仍需 Enterprise Cloud ([GHES 文件](https://docs.github.com/en/enterprise-server@latest/copilot/copilot-on-ghes/about-copilot-on-ghes))。

## 逐層選型:每層都有地端預設,但付費牆常落在多部門最需要的功能

**判斷:** 整體可離線,但代價不是某一張授權,而是**持續的匯入流程**:Trivy DB、規則包、Nuclei 模板、Playwright 瀏覽器、套件、映像、模型都要有受控傳輸路徑。開放核心授權也傾向收走多部門最需要的功能,包括 Onyx 權限同步、Langfuse 專案 RBAC 與稽核、Coder 治理、Docmost SSO、SonarQube PR decoration。編列預算時不要因為「開源」就把這些算成零。

### L1 知識與需求:Wiki 的最大未知是繁體中文搜尋

**證據:** 沒有任何候選有官方的繁中搜尋召回數據。Outline 的 CJK issue 是因閒置被關閉,不是修復 ([Issue #3207](https://github.com/outline/outline/issues/3207));第三方指 BookStack 以空白分詞,不利 CJK ([bookstack-meilisearch](https://backiee.wasmer.app/http_github_com/johnroyer/bookstack-meilisearch));Azure DevOps 也找不到 wiki、工作項目或程式碼搜尋的 CJK 分詞來源。

| PM wiki | 授權 | 狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| [BookStack](https://github.com/BookStackApp/BookStack) | MIT | v26.09.2(2026-10-20) | 可行;[LDAP 功能只連 LDAP 伺服器](https://www.bookstackapp.com/docs/admin/ldap-auth/) | [原生 AD 與群組對應](https://www.bookstackapp.com/docs/admin/ldap-auth/);僅[社群 MCP](https://github.com/pnocera/bookstack-mcp-server);[Onyx 連接器無權限同步](https://docs.onyx.app/admins/connectors/overview.md) | **預設** |
| Azure DevOps Wiki | 隨產品 | Git 儲存 | 已載明(地端產品本身) | [18 MB 頁、19 MB 附件、235 字元路徑](https://github.com/MicrosoftDocs/azure-devops-docs/blob/main/docs/project/wiki/includes/wiki-naming-conventions.md);[Mermaid 語法有限,文件對 `flowchart` 自相矛盾](https://learn.microsoft.com/en-us/azure/devops/project/wiki/markdown-guidance);[Stakeholder 只看 provisioned wiki 結果,無前綴萬用字元](https://learn.microsoft.com/en-us/azure/devops/project/search/get-started-search?view=azure-devops-server) | 核准版與 agent 可讀層;通過繁中測試可取代 BookStack |
| [Docmost](https://github.com/docmost/docmost) | AGPL-3.0;付費功能需金鑰 | v0.97.0 | [Business 標示 Air-gapped ready](https://docmost.com/pricing);離線授權機制未說明 | [SSO、頁面權限、MCP 需 Business,$6/席、10 席起](https://docmost.com/pricing) | 替代 |
| [Outline](https://github.com/outline/outline) | [BSL 1.1](https://github.com/outline/outline/blob/main/LICENSE) | v1.12.0 | 可行;[`ENABLE_UPDATES=false`](https://raw.githubusercontent.com/outline/outline/main/.env.sample) | 無 LDAP([#8168](https://github.com/outline/outline/issues/8168));[SAML 需付費](https://www.getoutline.com/pricing);內建 MCP | 已有 OIDC 時才考慮 |
| [Wiki.js](https://github.com/requarks/wiki) | AGPL-3.0 | 3.1.1 | 部分;[週期性聯繫開發者服務](https://docs.requarks.io/install/requirements/internet)(摘要) | v3 遙測開關未確認;[基本搜尋據報僅標題](https://GitHub.com/requarks/wiki/discussions/7166) | 不優先 |

| CJK 搜尋 | 授權 | 狀態 | 地端 | 條件 | 定位 |
|---|---|---|---|---|---|
| [OpenSearch](https://github.com/opensearch-project/OpenSearch) | Apache-2.0 | 3.9.0 | 可行;[IK 可由本地 zip 安裝,為社群 fork](https://toscode.mulanos.cn/mirrors/opensearch-analysis-ik) | [IK 字典以簡體為主,建議先繁轉簡](https://support.huaweicloud.com/intl/en-us/usermanual-css/css_01_0324.html);應同時測 2-gram | 實測後二選一 |
| [Meilisearch](https://github.com/meilisearch/meilisearch) | [MIT + EE 部分 BUSL-1.1](https://github.com/meilisearch/meilisearch/blob/main/LICENSE) | v1.56.1 | 可行;[`MEILI_NO_ANALYTICS=true`](https://www.meilisearch.com/docs/resources/help/telemetry.md) | 繁中行為未確認;[BookStack 整合僅 PoC](https://backiee.wasmer.app/http_github_com/johnroyer/bookstack-meilisearch) | 實測後二選一 |
| [PGroonga](https://github.com/pgroonga/pgroonga) | PostgreSQL License | 4.1.0 | 可行 | [Zulip 視為中日文實驗選項](https://zulip.com/help/configure-multi-language-search);需改應用 SQL | 能改 SQL 時 |
| [Typesense](https://github.com/typesense/typesense) | GPL-3.0 | v30.2 | 可行 | [分詞](https://server.hk/blog/how-to-deploy-typesense-on-hong-kong-vps-algolia-alternative-2026/)[說法衝突](https://railway.com/deploy/typesense-cjk--deploy-typesense-cjk-search-on-railway) | 觀察 |

**判斷:** 用 sidecar 爬 wiki 比改 wiki 的搜尋程式碼務實,且同一個 sidecar 可索引 BookStack 與 Azure Repos 的 Markdown。Agent 對核准需求的寫入一律走 PR。需求工具只是 Git 中的檔案,整合很薄:規格標頭帶工作項目 ID,pipeline 以 REST 同步狀態與測試;這是本報告的設計,沒有找到 Spec Kit 或 OpenSpec 與 Azure Boards 的公開整合。

| 需求與規格 | 授權 | 狀態 | 地端 | 條件 | 定位 |
|---|---|---|---|---|---|
| [Spec Kit](https://github.com/github/spec-kit) | MIT | v1.3.2;148k 星 | 檔案本身可行,CLI 未稽核 | [`/speckit.analyze`](https://github.com/github/spec-kit/blob/main/templates/commands/analyze.md) 止於任務層,程式碼與測試追溯要自建 | 大型或跨團隊 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | MIT | v1.16.1;74k 星 | 同上 | 穩定 ID 需自訂慣例 | 日常變更 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | [MIT + 額外聲明](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/LICENSE) | v6.12.4 | 未稽核 | [BRD 模板](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/skills/bmod-method/doc-templates/template-brd.md)可改作 URD | 方法參考 |
| [StrictDoc](https://github.com/strictdoc-project/strictdoc)、[Sphinx-Needs](https://github.com/useblocks/sphinx-needs) | Apache-2.0、MIT | 活躍 | 未稽核 | 社群較小 | 受監管部門 |

**證據:** 有一項 88,052 個 PR 的研究發現有規格與無規格的缺陷率無顯著差異,但其工件多為 issue 與 ticket 引用,SDD 工具本身未被測試 ([ESEM 2026](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ESEM.2026.84))。**判斷:** 這裡的理由是追溯與閘門,不是已證實的缺陷下降。需求階段自動化幾乎沒有成效證據,PM 這段應從輔助起步。

### L2 檢索:Onyx 能離線,但接不上 Azure DevOps

| 檢索 | 授權 | 狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| [Onyx](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE) | MIT;`ee` 另計 | v5.1.0 | **已載明**:[`ONYX_AIRGAPPED=true`、每個服務 `DISABLE_TELEMETRY=true`、手動授權檔](https://docs.onyx.app/security/self_hosted/faq) | [**沒有 Azure DevOps 連接器**](https://github.com/onyx-dot-app/onyx/tree/main/backend/onyx/connectors)、[無 SMB](https://docs.onyx.app/admins/connectors/overview.md)、[SharePoint 僅 Online](https://docs.onyx.app/admins/connectors/official/sharepoint);ACL 同步付費,[總覽](https://docs.onyx.app/admins/connectors/overview.md)與 [Confluence 頁](https://docs.onyx.app/admins/connectors/official/confluence)對方案說法衝突;[Business $20/人/月](https://onyx.app/pricing) | **預設** |
| [RAGFlow](https://github.com/infiniflow/ragflow) | Apache-2.0 | v1.0.0(2026-11-06);[RC 為 Go 重寫](https://newreleases.io/project/github/infiniflow/ragflow/release/v1.0.0-rc1) | 未確認;[僅 Zilliz FAQ](https://zilliz.com/ai-faq/does-ragflow-support-airgapped-deployment) | 未見 ACL 繼承證據;生產歷史很短 | 文件密集、團隊範圍 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | 修改版 Apache-2.0 | 1.19.0 | 部分;[`CHECK_UPDATE_URL` 留空](https://docs.dify.ai/en/self-host/configuration/environments) | **一個 workspace 算一個 tenant,多租戶需書面授權** | 應用建置,勿每部門一個 workspace |
| 自建 OpenSearch | Apache-2.0 | 3.9.0 | 可行 | 全部自建 | 僅有人力時 |

**判斷:** Azure DevOps Server 加 SMB 的環境,實際做法是為 Azure DevOps wiki、repo、工作項目與檔案分享寫自訂匯入工作,以部門群組做粗粒度存取;逐文件 ACL 是客製工作,不是勾選項目。Onyx 以 email 比對使用者,masked email 不會匹配 ([Onyx](https://docs.onyx.app/admins/connectors/official/confluence)),AD mail 屬性要先整理。Onyx 對 Confluence Data Center 的權限同步雖然最成熟,但 Data Center 從 2029-03-28 起唯讀 ([Atlassian](https://www.atlassian.com/licensing/data-center-end-of-life)),不宜為此新導入。

| 程式碼知識 | 授權 | 狀態 | 地端 | 條件 | 定位 |
|---|---|---|---|---|---|
| [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | MIT | v0.12.1;48.2k 星 | 已載明(自述無遙測、無背景連線) | 星數增長快,從原始碼建置並審查;Windows 有 Defender 誤報 | **預設** |
| [Zoekt](https://github.com/sourcegraph/zoekt) | Apache-2.0 | 活躍,無 release | 可行 | 釘 commit;以 Git mirror 索引 | **預設** |
| [Repomix](https://github.com/yamadashy/repomix) | MIT | v1.18.1 | 可行;[用 `./` 強制本機](https://repomix.com/guide/remote-repository-processing) | 遙測未確認 | **預設** |
| [Serena](https://github.com/oraios/serena) | GPL-3.0-or-later;CLA | v2.1.0 | [已載明建置時預載語言伺服器](https://oraios.github.io/serena/01-about/020_programming-languages.html) | [C# 需 .NET 10+ 與 `pwsh`](https://oraios.github.io/serena/01-about/020_programming-languages.html) | 語意編輯選配 |
| [Sourcegraph](https://sourcegraph.com/docs/admin/deploy) | 專有 | 企業版 | 部分;[無 air-gap 標記授權可能停機](https://handbook.sourcegraph.com/departments/data-analytics/how-to-airgap-disable-telemetry)(摘要,可能過時) | [有 Server 專用欄位](https://sourcegraph.com/docs/admin/code_hosts/azuredevops);[Deep Search 不支援 BYOK](https://sourcegraph.com/docs/deep-search) | 付費替代,先書面確認 |
| [DeepWiki-Open](https://github.com/AsyncFuncAI/deepwiki-open) | MIT | 無 release | 未確認 | 未見 Azure DevOps | 不採用 |

### L3 協調:先用 Pipelines,Temporal 等試點證明需要才加

**判斷:** 本輪兩份筆記在此有張力:工具稽核把 Temporal 列為協調預設,Azure DevOps 研究則主張先用 Pipelines。本報告採後者並以 Temporal 作第二階段,因為工作項目、核准與部署紀錄已有現成人類介面,Temporal 又帶來一整個 stateful 平台。若加入,模式是 activity 呼叫 Run Pipeline,`build.complete` service hook 以 run ID 通知 workflow,輪詢作備援。沒有找到 Temporal 與 Azure DevOps 的官方連接器。

| 引擎 | 授權 | 狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| Azure Pipelines | 隨產品 | Server 與 Services 皆有 | 已載明(Server) | 無 PR 留言觸發;併行與逾時限制未查 | **預設** |
| [Temporal](https://github.com/temporalio/temporal) | MIT | v1.32.2 | 可行;無廠商 air-gap 文件 | [shard 數不可變、升級可能每兩週且循序、無內建 RBAC 與稽核](https://docs.temporal.io/self-hosted-guide/production-checklist);[**資料庫清單未列 SQL Server**](https://docs.temporal.io/references/configuration);[.NET SDK 1.21.1 穩定](https://api.nuget.org/v3-flatcontainer/temporalio/index.json) | 第二階段 |
| [Dapr](https://docs.dapr.io/operations/hosting/self-hosted/self-hosted-airgap/) Workflow | Apache-2.0 | [Dapr.Workflow 1.19.0](https://api.nuget.org/v3-flatcontainer/dapr.workflow/index.json) | **已載明**,含 Windows bundle 範例 | Dapr Agents 僅 Python;耐久性需實測 | Temporal 太重時 |
| [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) | MIT | [1.28.0](https://api.nuget.org/v3-flatcontainer/microsoft.agents.ai/index.json) | 可行(程序內) | [Durable 套件僅 preview,連 Durable Task Scheduler](https://learn.microsoft.com/en-us/agent-framework/integrations/durable-extension),不作離線骨幹 | .NET 階段內邏輯 |
| [LangGraph](https://github.com/langchain-ai/langgraph) 程式庫 | MIT | 活躍 | 可行 | [Server 為商業;帳務遙測僅離線授權可關](https://docs.langchain.com/langsmith/self-host-egress) | Python 階段內 |
| [Argo Workflows](https://github.com/argoproj/argo-workflows) | Apache-2.0 | v4.1.6 | 可行 | 不適合長時間人類閘門 | 批次 fan-out |
| [ado-aw](https://github.com/githubnext/ado-aw) | MIT | v0.54.0;33 星 | 不適用;[二進位來自 GitHub Releases](https://github.com/githubnext/ado-aw/blob/main/docs/network.md) | 見差異表 | 只抄設計 |

### L4 Coding agent:沒有商用產品同時記載 air-gap 與 Azure DevOps Server

**證據:** 本研究找不到任何廠商文件同時聲明 air-gap 與 Azure DevOps Server 支援;搜尋未果不等於不支援。OSS agent 的 Azure DevOps 程式碼搜尋多半為空,這是弱證據,因為 code search 可能受限流影響。也沒有找到 OpenCode、Goose、Qwen Code、Kilo 或 Cline 在 Azure Pipelines 對 Server 的端到端範例,可行性是由它們是 CLI 或 MCP client 推得。

| OSS agent | 授權 | 狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| [OpenCode](https://github.com/anomalyco/opencode) | MIT | v1.19.5;225k 星 | 部分;[第三方稽核 v1.18.34:無總開關、17 條外連路徑、4 條無設定、Zen 託管模型自動載入](https://particula.tech/blog/opencode-offline-mode-air-gapped) | 必設 `enabled_providers`;[Skill 權限 allow、ask、deny](https://opencode.ai/docs/skills) | **主要** |
| [Goose](https://github.com/aaif-goose/goose) | Apache-2.0 | v1.57.0 | 可行;[遙測](https://mintlify.com/block/goose/guides/environment-variables)[變數名稱衝突](https://last9.io/docs/integrations/goose/),查原始碼 | Stripe 曾 fork 為基礎 | **次要** |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | Apache-2.0 | v0.26.9 | 未稽核 | 若受來源限制,工具本身也要過法務 | 私部門候選 |
| [Codex CLI](https://github.com/openai/codex) | Apache-2.0 | rust-v0.167.0 | 未稽核 | 本地模型能力未重驗 | bake-off 候選 |
| [Cline](https://github.com/cline/cline)、[Kilo Code](https://github.com/Kilo-Org/kilocode) | Apache-2.0、MIT | v4.1.26、v7.8.27 | 未稽核 | [Cline 2026-02 npm 遭投毒](https://thehackernews.com/2026/02/cline-cli-230-supply-chain-attack.html?m=1) | IDE 導向 |
| [OpenHands](https://github.com/OpenHands/OpenHands) | MIT | v1.37.0 | 未稽核 | [ADO 整合排除 Server](https://docs.openhands.dev/enterprise/integrations/azure-devops) | Services 路線 |
| [Aider](https://github.com/Aider-AI/aider)、[Tabby](https://github.com/TabbyML/tabby)、[Continue](https://github.com/continuedev/continue) | 見 repo | 陳舊、趨緩、唯讀 | 未作新標準 | 維運風險 | **不作新標準** |

| 商用 agent | 授權 | 狀態 | 地端 / air-gap | Azure DevOps Server | 定位 |
|---|---|---|---|---|---|
| Tabnine Enterprise | 商用 | 價格未公開 | [**已載明**:Kubernetes,完全 air-gapped](https://docs.tabnine.com/main/welcome/readme/architecture/deployment-options.md) | [未記載](https://docs.tabnine.com/main/sitemap.md) | 書面確認後評估 |
| Factory Droid | 商用,Custom Deployment | GA 狀態未說明 | [**已載明**:無 Factory 請求;失去帳號、Router、分享、網頁搜尋、自動更新](https://docs.factory.com/enterprise/airgapped-deployment.md) | 未記載,通用 Git remote | 書面確認後評估 |
| Coder Agents | 商用 | [2026-09-09 GA](https://coder.com/blog/coder-agents-ga) | 廠商主張 | 未確認 | 搭 Coder 時查 |
| Sourcegraph Amp、Devin Desktop | 商用 | 不明 | 未確認 | 無 | 不納入規劃 |
| Cursor | 商用 | [規劃留在 Cursor 雲端](https://cursor.com/blog/self-hosted-cloud-agents) | **否** | 未列名 | 排除 |

**判斷:** 你提到各階段套用不同 agent。沒有研究直接比較「單一主要 agent」與「每階段 best-of-breed」;但每多一個 harness 就多一個需要做默認拒絕、遙測檢查與升級回歸的面,而在地端這筆成本高於雲端。因此建議**階段差異主要由 skill、模型層級與權限表達,harness 只分「寫手」與「審查者」兩個角色**。Fresh-context 驗證的證據也支持「獨立 context 比換模型重要」:預註冊小樣本中 verifier 偵測率 95.4%,建構者 85.1%,跨供應商比較則有混淆 ([arXiv 2610.28667](https://arxiv.org/html/2610.28667))。

| 階段 | Harness 與模型層級 | Skill | 權限 |
|---|---|---|---|
| 需求、設計 | OpenCode + 主模型;S0、S1 可升級 | OpenSpec 或 Spec Kit、domain modeling | 只寫規格分支 |
| 實作 | OpenCode + 主模型 | TDD、diagnosing bugs、部門 skill | 沙箱內寫單一分支 |
| 小修、測試、文件 | OpenCode 或 Goose + 快速層 | 文件模板、測試 skill | 同上 |
| 審查 | PR-Agent + 主模型 | 組織規則 | 只寫 PR thread |
| 遷移、批次 | Goose headless,離峰 | 遷移 playbook | 分批,agent 不直接寫遠端 |
| DevOps | 唯讀或 plan-only | IaC plan 審查 | Push 與部署由 pipeline 持有 |

| 審查與掃描 | 授權 | 狀態 | 地端 / Server | 條件 | 定位 |
|---|---|---|---|---|---|
| [PR-Agent](https://github.com/The-PR-Agent/pr-agent) | MIT | v0.49.0;15.3k 星 | 可能支援:[程式碼接受 collection URL](https://github.com/The-PR-Agent/pr-agent/blob/main/pr_agent/git_providers/azuredevops_provider.py),[文件只示範 `dev.azure.com`](https://docs.pr-agent.ai/installation/azure/) | 無公開 Server 25H2 回報;SDK API 版本要測 | **預設** |
| Qodo | 商用 | 洽詢 | **衝突**:[矩陣稱 cloud only](https://docs.qodo.ai/install-and-configure/deployment-model-support),[地端指南提及 Server](https://docs.qodo.ai/on-prem/git-integration/install/azure-devops.md) | 要書面版本支援 | 確認後評估 |
| CodeRabbit | 企業,500 席起 | 洽詢 | [自管平台未列 Server](https://docs.coderabbit.ai/self-hosted/azure-devops) | 要書面確認 | 確認後評估 |
| Greptile | 商用 | 洽詢 | [未列 Azure DevOps](https://www.greptile.com/docs/deployment-options) | | 不優先 |
| [SonarQube](https://github.com/SonarSource/sonarqube) | Community LGPL-3.0;付費版另計 | 26.11 | [文件支援 Server 與 Services](https://docs.sonarsource.com/sonarqube-server/devops-platform-integration/azure-devops-integration);離線授權未確認 | [PR decoration 需 Developer 以上(論壇)](https://community.sonarsource.com/t/pullrequest-decoration-quality-gate/46232);[有 2025.1 LTS 不留言的回報](https://community.sonarsource.com/t/trouble-getting-pr-decoration-working-with-sonarqube-server-2025-1-lts-and-azure-devops/151196) | 付費版作閘門 |

**證據與判斷:Skill。** 純 Git 就夠了。[OpenCode 讀 `.claude/skills` 與 `.agents/skills`](https://opencode.ai/docs/skills),[Factory airgap 保留已安裝 skill](https://docs.factory.com/enterprise/airgapped-deployment.md),[Claude Code marketplace 可指向任何 Git host](https://code.claude.com/docs/en/plugin-marketplaces)。做法是 Azure Repos 上的 skill monorepo,加分支政策與打包 CI,可用 [Azure Artifacts Universal Packages](https://learn.microsoft.com/azure/devops/artifacts) 當發行通道。掃描用 [cisco skill-scanner](https://github.com/cisco-ai-defense/skill-scanner) 的本地模式,LLM judge 可接本地端點,但其自測規則掃描在 HIGH 等級只抓 7.7%,配 LLM 達 MEDIUM 以上為 66.7%,本地模型的結果會不同,人工審查不可省。[SkillsBench](https://arxiv.org/html/2602.12670v1) 顯示軟體工程 skill 平均僅提升 4.5 個百分點,自動生成為 -1.3,所以每個 skill 都要做 with/without 配對評測,無效就退役;依賴網路工具的公開 skill 在離線環境會靜默失效。

### L5 Test agent:確定性工具離線很穩,LLM 工具會縮水

**判斷:** 維持「agent 提案、工具判定、人類擁有 oracle」。本地模型會削弱 AI pentest 與紅隊生成,所以閘門必須放在確定性工具上。**證據:** 通過測試本身是弱證據:METR 請維護者審閱 296 個通過評分器的 patch,評分器分數比合併決定高約 24.2 個百分點 ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/));Playwright healer 認為功能損壞時會把測試標成 skip ([Playwright](https://playwright.dev/docs/test-agents)),在 CI 上看起來像綠燈,因此任何新增 skip 都要讓 CI 失敗。

| 測試層 | 工具 | 授權與狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| E2E agent | [Playwright Test Agents](https://playwright.dev/docs/test-agents) | Apache-2.0;v1.65.0 | 可行;[`PLAYWRIGHT_DOWNLOAD_HOST` 接內部鏡像](https://playwright.dev/docs/browsers) | OpenCode loop 接本地模型的品質無證據 | **預設**,先測 |
| 單元、變異 | Harness + TDD;[PIT](https://github.com/hcoles/pitest)、[Stryker.NET](https://github.com/stryker-mutator/stryker-net) | Apache-2.0;1.30.2、5.0.0 | 可行;鏡像 Maven、NuGet | 遙測未查 | **預設** |
| API、契約 | [Schemathesis](https://github.com/schemathesis/schemathesis)、[Pact](https://docs.pact.io/telemetry) | MIT;Apache-2.0 或 MIT | 可行;[`PACT_DO_NOT_TRACK=true`](https://docs.pact.io/telemetry) | Schemathesis 遙測未驗證 | **預設** |
| 效能 | [k6](https://github.com/grafana/k6) | AGPL-3.0;v2.3.0 | 可行;[`K6_NO_USAGE_REPORT`](https://grafana.com/docs/k6/latest/set-up/usage-collection/) | 預設回報用量 | **預設** |
| SAST | [Semgrep CE](https://github.com/semgrep/semgrep)、[Opengrep](https://github.com/opengrep/opengrep) | LGPL-2.1;v1.184.0、v1.33.1 | 可行;[規則需 vendor](https://mhchlagou.github.io/lintel/0.2.4/scanners/opengrep/) | [社群規則限制有爭議,來源為 fork 背後的 Aikido](https://aikido.dev/blog/launching-opengrep-why-we-forked-semgrep) | **預設** |
| SCA、映像 | [Trivy](https://github.com/aquasecurity/trivy) | Apache-2.0;v0.76.2 | **已載明**:[鏡像 DB 與 checks](https://trivy.dev/docs/latest/guide/advanced/self-hosting/);`--skip-version-check --disable-telemetry` | [DB 每 6 小時建置](https://github.com/aquasecurity/trivy-db);[2026 年 3 月曾遭入侵](https://github.com/advisories/GHSA-69fq-xp46-6x23),釘 digest | **預設** |
| DAST | [ZAP](https://github.com/zaproxy/zaproxy)、[Nuclei](https://github.com/projectdiscovery/nuclei) | Apache-2.0、MIT | 部分;[`-silent`,曾有 bug](https://www.zaproxy.org/blog/2025-10-21-zap-was-not-so-silent/);[`-duc`](https://docs.projectdiscovery.io/opensource/nuclei/running) | Nuclei 失去 Interactsh 與 `-ai` | **預設** |
| AI pentest | [Strix](https://github.com/usestrix/strix) | Apache-2.0;v1.7.0 | 部分;`LLM_API_BASE` 可接本地,但推薦模型為託管型 | 本地能力未測;僅限書面授權環境 | 試驗 |
| AI 評測 | [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai)、[DeepEval](https://github.com/confident-ai/deepeval) | MIT、Apache-2.0 | 可行;[Inspect 支援 vLLM、SGLang](https://inspect.aisi.org.uk/providers.html) | DeepEval 遙測未確認 | 模型與 skill 回歸 |
| 紅隊 | [garak](https://github.com/NVIDIA/garak)、[PyRIT](https://github.com/microsoft/PyRIT)、[promptfoo](https://github.com/promptfoo/promptfoo) | Apache-2.0、MIT、MIT | 部分;[promptfoo 文件自承旗標不保證隔離,且失去多類外掛](https://www.promptfoo.dev/docs/faq) | [第三方稱停用後仍有事件送 `r.promptfoo.app`](https://particula.tech/blog/promptfoo-self-hosted-red-teaming-alternatives) | 擋出站才用 |

**判斷:** 建議的 feed 節奏是 Trivy 至少每日,規則與模板至少每週;這是本報告提案。掃描結果以 SARIF 或 JUnit 輸出、依 exit code 判定、用小 REST 步驟回寫 PR,可避開對 Marketplace 擴充與雲端後端的依賴,也省去每次 Server 升級都要檢查 [Local Gallery](https://learn.microsoft.com/en-us/azure/devops/project/search/install-configure-search?view=azure-devops-server) 相容性。

### L6 平台:Agent 沙箱必須放 Linux

| 元件 | 工具 | 授權與狀態 | 地端 / air-gap | 條件 | 定位 |
|---|---|---|---|---|---|
| Kubernetes | [RKE2](https://github.com/rancher/rke2) | Apache-2.0;v1.37.2+rke2r1 | [**已載明**](https://docs.rke2.io/install/airgap);[Windows Server 2019、2022 worker](https://docs.rke2.io/install/windows_airgap) | 控制面在 Linux | **預設** |
| | [OpenShift](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/disconnected_environments/installing-mirroring-installation-images)、[Talos](https://docs.siderolabs.com/talos/v1.12/platform-specific-installations/air-gapped) | 商用;MPL-2.0 | 已載明 | OpenShift 價格未查;Talos 假設僅 Linux | 替代 |
| 沙箱 | [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) + Kata 或 gVisor | Apache-2.0;v1.0.10 | 自述無雲端相依 | [VM 節點跑 Kata 需巢狀虛擬化](https://northflank.com/blog/what-are-kata-containers),與 VMware 團隊早談 | **預設** |
| 工作區 | [Coder](https://github.com/coder/coder) | AGPL-3.0;v2.38.6 | [**已載明**:全功能離線,授權離線驗證](https://coder.com/docs/install/airgap) | [治理在付費方案;SKU 名稱已變](https://coder.com/pricing) | 需 Windows VM 工作區時 |
| 閘道 | [agentgateway](https://github.com/agentgateway/agentgateway) | Apache-2.0;v1.7.0 | 可行 | [只有近似 token 限流,全域需外部服務](https://agentgateway.dev/docs/kubernetes/latest/llm/rate-limit/);[有高危 MCP advisory](https://github.com/agentgateway/agentgateway/security/advisories) | **預設** |
| | [Agent Router](https://github.com/theagentrouter/agent-router) | Apache-2.0;v1.2.1 | 可行;需 Redis | [CEL 成本](https://theagentrouter.ai/docs/capabilities/traffic/usage-based-ratelimiting/);[11-06 六則 advisory](https://github.com/theagentrouter/agent-router/security/advisories) | 替代 |
| | [LiteLLM](https://github.com/BerriAI/litellm) | MIT(`enterprise/` 除外);v1.108.2 | 可行 | 預算功能最完整,安全紀錄最差(見模型節) | 僅釘簽章容器 |
| | [Microsoft mcp-gateway](https://github.com/microsoft/mcp-gateway)、[Kong](https://developer.konghq.com/plugins/ai-rate-limiting-advanced/) | MIT;Apache-2.0 | 地端生產未記載;token 限流屬企業版 | | 不採用 |
| 觀測 | [Langfuse](https://github.com/langfuse/langfuse/blob/main/LICENSE) + OTel | MIT(`ee` 除外);v4.76.0 | [OSS 可 `TELEMETRY_ENABLED=false`](https://langfuse.com/self-hosting/security/telemetry) | [**企業版遙測不可關,走 PostHog Cloud**](https://langfuse.com/self-hosting/security/telemetry);[RBAC、稽核屬企業版](https://langfuse.com/self-hosting/license-key) | **預設**;付費前先問 |
| 密鑰、身分 | [OpenBao](https://github.com/openbao/openbao)、[SPIRE](https://github.com/spiffe/spire) | MPL-2.0、Apache-2.0 | 外連行為未查 | | **預設** |
| 政策 | [OPA](https://github.com/open-policy-agent/opa) | Apache-2.0;v1.22.2 | 可行;[`--disable-telemetry`](https://openpolicyagent.org/docs/latest/privacy/) | | **預設** |
| 映像 | [Harbor](https://github.com/goharbor/harbor) | Apache-2.0;v2.14.6 | 離線安裝未在已讀頁確認 | | **預設** |
| 套件 | [Azure Artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/upstream-sources?view=azure-devops-server) | 隨 Server | 上游套件留存於 feed,但需連線才能取新版 | 由 DMZ 匯入 | **預設** |
| | [Nexus Community](https://help.sonatype.com/en/nexus-repository-tiers-and-feature-availability.html)、[Artifactory](https://docs.jfrog.com/security/docs/xray-and-jfrog-external-db-sync) | CE 條款;商用 | CE 上限 40,000 components、每日 100,000 requests;Artifactory 有離線資料包 | [第三方價格衝突](https://www.vendr.com/marketplace/jfrog) | CE 太小;Artifactory 付費替代 |
| 目錄 | [Backstage 外掛](https://github.com/backstage/community-plugins/blob/main/workspaces/azure-devops/plugins/azure-devops-backend/README.md) | Apache-2.0 | 支援 Server(collection + PAT) | Server 25H2 未測 | 100 人以上再加 |

**判斷:** Windows 節點可做 .NET Framework 建置,但 agent 沙箱放 Linux。若 agent 必須在 Windows 上工作,就走 VM 工作區,密度較低、控制也不同。部門隔離放基礎設施層,每部門獨立 namespace、閘道金鑰、憑證、Langfuse project 與配額,因為應用層 tenancy 在這類工具上反覆出問題,例如 [IBM ContextForge 的跨團隊接管](https://github.com/IBM/mcp-context-forge/security/advisories)。

**證據與判斷:遙測。** 許多工具預設回報,旗標常不完整,甚至會隨版本改變:OpenCode 的目錄主機在一個 patch 版內就變了 ([Particula](https://particula.tech/blog/opencode-offline-mode-air-gapped))。旗標只是衛生,**default-deny 出站才是控制**;尤其模型驅動的 shell 能連到網路允許的任何地方。擋掉端點也可能造成故障,授權檢查就是例子。驗收方式是每個工具在 default-deny 加 DNS 記錄下跑過啟動、正常使用與升級,每次升級重做,設定釘在 Helm values,被移除就讓 CI 失敗。BookStack、Docmost、RAGFlow、Temporal、Harbor、OpenBao、SPIRE、Argo、Serena、Repomix、Inspect、garak、Strix 的遙測未經驗證,這不等於乾淨。

## 地端模型與硬體:主權站得住腳,成本站不住腳

### 能力落差:常見評測下約 5 分,合併品質下超過 2 倍

**證據:** 落差大小取決於量什麼,下表應全部呈現給決策者。分數不可跨列比較。

| 評測 | 測什麼 | 前沿 | 可部署 open-weight | 注意 |
|---|---|---|---|---|
| [DeepSWE v1.1](https://deepswe.datacurve.ai/) | 同一 harness 的 113 題 | GPT-6 Astra、Opus 5 等 74% | GLM-5.3、Kimi K3 69%;GLM-5.3-Flash、V4-Pro 63%;Qwen3.8-Max 57% | 這裡唯一固定 harness 的比較;[LLM Stats 53 筆 0 筆驗證](https://llm-stats.com/benchmarks/deepswe-1.1) |
| [FrontierCode 2.0](https://artificialanalysis.ai/evaluations/frontiercode-2-0) | 132 題私有任務,以可合併性評分 | Opus 5.5 46.4%;GPT-6 Astra 42.4% | [Mistral Large 4 19.7、K3.1 Pro 19.6、V4.1-Flash-1017 19.3](https://benchlm.ai/md/benchmarks/frontiercode-2.md) | Cognition 賣 coding agent;harness 說法衝突 |
| [AA Index v5](https://artificialanalysis.ai/leaderboards/models?open_weights=true) | 綜合,[25% coding,主要英文](https://www.implicator.ai/artificial-analysis-index-v5-open-model-gap/) | Opus 5.5 59;Gemini 4 Carbon 60 | K3.1 Pro 46;DeepSeek 33;GLM-5.3 32 | 對繁中沒有資訊 |
| [ProgramBench](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/README.md) | 長程 | Opus 5.5 65.0 | K3.1 31.5;DeepSeek 29.1;GLM-5.3 19.0 | 廠商表格 |

**證據:** 三個現象提醒不要只看單一數字。廠商自報不能互換:DeepSeek 自報 V4.1-Flash-1017 為 77.0,高於它列的 Opus 5 ([模型卡](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017/raw/main/README.md));而 Poolside 的表格把 V4-Pro 在 DeepSWE 列為 9.0%,與 Datacurve 的 63% 相差約七倍,本研究無法解釋 ([Laguna 模型卡](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md))。託管分數可能高估自託管結果:Qwen 官方服務比釋出權重多了預設 1M context 與 non-thinking 模式 ([模型卡](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md))。引擎本身也會壞:vLLM 除錯 Kimi K2 時發現 chat template 參數被靜默丟棄,修正後 schema 準確率約 76% ([vLLM](https://vllm.ai/blog/Kimi-K2-Accuracy))。

**判斷:** 誠實的對外說法是:本地模型能承擔相當大部分的工作,包括範圍明確的修復、測試、文件、CI triage 與審查預篩;現有證據**不支持**宣稱它們在無人值守的長程工程或首次合併品質上與前沿對等。「全部工作都能地端跑」是一項**持續運作能力**,品質較低,不是等值替代,要預留更多審查與重試。

### 模型:兩條路線,由組織資格決定

| 模型 | 來源 | 授權 | 規模 | 代理證據 | 地端狀態與條件 | 定位 |
|---|---|---|---|---|---|---|
| [GLM-5.3](https://huggingface.co/zai-org/GLM-5.3) | CN | [類 MIT;僅超大型 MaaS 需審查](https://huggingface.co/zai-org/GLM-5.3/raw/main/LICENSE) | 753B | DeepSWE 69;[Toolathlon 73.0](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md) | 8×H200 FP8 偏緊【估算】 | 私部門主力 |
| [DeepSeek-V4.1-Flash-1017](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017) | CN | MIT | 511GB;[參數量來源衝突](https://geotoolbox.ai/blog/best-open-source-llms) | 廠商 77.0,待驗 | 同級 | 私部門主力 |
| [GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash) | CN | MIT | 321B | DeepSWE 63;[$0.24/task](https://deepswe.datacurve.ai/) | | 快速層 |
| [Kimi K3.1 Pro](https://huggingface.co/moonshotai/Kimi-K3.1-Pro) | CN | [MaaS 門檻;內部用豁免](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/LICENSE) | 2.78T;1,561GB | AA 46 | [需 MI355X 級](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/) | 僅記憶體經濟成立時 |
| [Mistral Large 4](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B) | FR | [Ecosystem License;MaaS 需商業授權](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B/raw/main/LICENSE) | 1,061GB | FrontierCode 19.7 | 8×B200 或 MI355X【估算】 | 受限路線主力 |
| [Nemotron 3 Ultra](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16) | US | [OpenMDW-1.1](https://openmdw.ai/license/1-1/) | 561B | [TB 2.1 56.4;Toolathlon 34.3](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) | 工具呼叫偏弱 | 受限備選 |
| [Laguna S 2.1](https://huggingface.co/poolside/Laguna-S-2.1) | Poolside(總部未驗證) | OpenMDW-1.1 | 118B;235GB | [廠商:DeepSWE 40.4,TB2.1 70.2](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) | 多卡可跑 | 受限 Pilot;來源須先查 |
| [Gemma 4 31B](https://huggingface.co/google/gemma-4-31B-it)、[gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b) | US | Apache-2.0 | 31B;117B | 無可信分數;gpt-oss 陳舊 | 工作站可跑 | 受限輕量層 |
| [Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) | CN | Apache-2.0 | 27.8B;56GB | [DeepSWE 42.2(未驗證)](https://llm-stats.com/benchmarks/deepswe-1.1) | 單張 96GB | 私部門 Pilot |

**授權陷阱。證據:** [Mistral Medium 3.5](https://huggingface.co/mistralai/Mistral-Medium-3.5-128B/raw/main/LICENSE) 在公司月營收超過 US$20M 時不授予任何權利;[MiniMax-M3](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/LICENSE) 的商業定義含支援商業營運的 API;[Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next/raw/main/LICENSE) 對任何 MaaS 或 AI Work Assistant 業務都要授權,但內部使用豁免;[Jina v5](https://huggingface.co/jinaai/jina-embeddings-v5-text-small) 是 CC-BY-NC-4.0。**判斷:** 多部門情境下最容易出事的是集團結構:若部門分屬不同法人,factory 可能被視為對第三方提供服務,要請律師判斷關係企業是否算第三方。

**台灣相關限制(條件式)。** 本段只在組織受台灣相關規範約束時適用,本報告不假設組織所在地。**證據:** 數位發展部副部長 2025-11-20 表示政府機關「已被禁止使用中國製生成式 AI 模型」,但報導未指明法律文件或開源處理 ([Taiwan News](https://www.taiwannews.com.tw/en/news/6248783))。2025 年初的 DeepSeek 措施涵蓋部會、公立學校、國營事業與準政府機構,報導並延伸至關鍵基礎設施人員與政府捐助基金會 ([The Star](https://www.thestar.com.my/aseanplus/aseanplus-news/2025/02/01/taiwan-bans-government-agencies-from-using-deepseek))。2019 年行政院原則禁止公部門採購或使用中國廠牌資通訊產品,含軟體;公立大學與研究機構經核准可用於研究 ([CNA](https://cna.com.tw/news/aipl/202502030237.aspx))。國安局 2025 年 11 月測試了五個中國 App,對象是 App,不是自架權重 ([Taiwan News](https://www.taiwannews.com.tw/en/news/6245677))。**未見私部門禁令的報導**,數發部僅稱持續提醒 ([Taipei Times](https://www.taipeitimes.com/News/taiwan/archives/2025/11/20/2003847542))。

**判斷:** 「自架就沒有資料外流問題」只回答了傳輸風險,沒有回答官方提到的資訊操弄理由,也沒有回答中國廠牌軟體原則。因此機關、國營事業、以及可能的承包商,在律師確認前應**視同不可用**;本研究找不到明確規範本地權重的法律文字。這個限制不只影響 LLM:bge-m3 與 Qwen3-Embedding 也屬中國來源,Qwen Code 這類 harness 也應一併盤點。微調或蒸餾模型要有血統紀錄,[TAIDE](https://huggingface.co/taide) 與 [Breeze](https://huggingface.co/MediaTek-Research) 模型太小且缺乏寫碼證據,最多用於繁中起草或防護,例如 [Breeze-Guard-26](https://huggingface.co/MediaTek-Research/Breeze-Guard-26)。受限路線的代價也不平均:Mistral Large 4 在 FrontierCode 上並不輸中國模型,代價主要在 1TB 級記憶體、授權審查與缺少獨立工具呼叫數據;Nemotron 與 Laguna 的能力代價則明顯。不論哪條路線,2026 年都沒有獨立的繁中評測,要自建繁中需求轉程式碼、簡體字洩漏率、台灣用語與敏感內容行為的測試集。

| Embedding、rerank | 來源 | 授權 | 狀態 | 條件 | 定位 |
|---|---|---|---|---|---|
| [Qwen3-Embedding-4B](https://huggingface.co/Qwen/Qwen3-Embedding-4B)、[Reranker](https://huggingface.co/Qwen/Qwen3-Reranker-4B) | CN | Apache-2.0 | [廠商 MTEB(Code) 約 78 對 bge-m3 約 50](https://arxiv.org/pdf/2506.05176) | 中文表格以簡體為主(判斷) | 私部門首選 |
| [bge-m3](https://huggingface.co/BAAI/bge-m3)、[reranker](https://huggingface.co/BAAI/bge-reranker-v2-m3) | CN | MIT、Apache-2.0 | 2024 年;[設定錯誤可使 Recall@1 從 80% 掉到 15%](https://docs.apiyi.com/en/live/2026-08/bge-m3-relaunch)(廠商) | 混合檢索基線 | 基線 |
| [EmbeddingGemma](https://huggingface.co/google/embeddinggemma-300m)、[GTAIDE](https://huggingface.co/taide/embeddinggemma-GTAIDE-300m-2605) | US、TW | 未驗證、other | 中文表現未驗證 | 需內部評測 | 受限路線 |

### Serving 與閘道:引擎要包在驗證閘道後面

**證據:** 所有引擎 2026 年都有嚴重漏洞:vLLM 有 [`code_revision` RCE](https://github.com/vllm-project/vllm/security/advisories/GHSA-h3rc-6mm3-gc2m) 與[繞過 `trust_remote_code=False` 的 CVE](https://github.com/vllm-project/vllm/security/advisories/GHSA-3c86-2m5g-59q7);SGLang 有 [CERT/CC 登記的未驗證 ZMQ pickle RCE](https://www.kb.cert.org/vuls/id/665416),[0.5.10 修復](https://advisories.gitlab.com/pypi/sglang/CVE-2026-3059/);Ollama 有 [Bleeding Llama](https://www.cyera.com/blog/bleeding-llama-a-critical-memory-leak-in-the-worlds-most-popular-local-ai-platform);llama.cpp 有 [RPC RCE](https://github.com/ggml-org/llama.cpp/security/advisories/GHSA-j8rj-fmpv-wcxw)。

| Serving | 授權 | 狀態 | 地端條件 | 定位 |
|---|---|---|---|---|
| [vLLM](https://github.com/vllm-project/vllm) | Apache-2.0 | v0.33.0rc3;96k 星 | [parser 最廣](https://docs.vllm.ai/en/latest/features/tool_calling/);`--tool-strict-level parameter`;advisory 密集 | **預設**,與 SGLang 對測 |
| [SGLang](https://github.com/sgl-project/sglang) | Apache-2.0 | v0.5.23 | [硬體矩陣明確](https://docs.sglang.io/cookbook/autoregressive/DeepSeek/DeepSeek-V4) | **預設**,與 vLLM 對測 |
| [llm-d](https://github.com/llm-d/llm-d) | Apache-2.0 | [v1.0.0(10-20)](https://github.com/llm-d/llm-d/releases/tag/v1.0.0) | 優先級與流量控制;才剛 1.0 | 超過一兩台再加 |
| [KServe](https://github.com/kserve/kserve) | Apache-2.0 | v0.21.0 | | 已在用才選 |
| NVIDIA NIM | 商用 | [AI Enterprise 約 $4,500/GPU/年(次級)](https://pi3g.com/nvidia-ai-enterprise-subscription-cost-in-2026/) | 離線授權未確認 | 買支援時才選 |
| [Ollama](https://github.com/ollama/ollama)、[llama.cpp](https://github.com/ggml-org/llama.cpp)、[LocalAI](https://github.com/mudler/LocalAI) | MIT | 活躍 | [負載下明顯落後(次級)](https://pub.towardsai.net/vllm-vs-ollama-vs-llama-cpp-vs-sglang-ollama-collapses-to-41-tokens-under-load-8f7a850d1e07) | 開發者桌面 |
| [TGI](https://github.com/huggingface/text-generation-inference) | Apache-2.0 | 封存、維護模式 | README 推薦 vLLM 與 SGLang | **不採用** |

**判斷:** 只開文字端點,因為 vLLM 近期多數 advisory 在多媒體路徑,寫碼 agent 很少需要。引擎埠只讓閘道連。依模型卡選引擎與 parser,每月一次安全更新窗是現實下限。

**證據:** LiteLLM 的預算功能最完整,含 team、key、user 的預算與時窗,但無 PostgreSQL 時全域預算會 fail open ([文件](https://docs.litellm.ai/docs/proxy/users));它也是最危險的選項:PyPI 1.82.7 與 1.82.8 於 2026-03-24 遭植入憑證竊取程式,官方容器據稱未受影響 ([LiteLLM](https://docs.litellm.ai/blog/security-update-march-2026)),其後又有 [API key 驗證 SQL 注入](https://github.com/BerriAI/litellm/security/advisories/GHSA-r75f-5x8p-qvmc)、[Host header 繞過驗證](https://github.com/BerriAI/litellm/security/advisories/GHSA-4xpc-pv4p-pm3w)等嚴重 advisory。**判斷:** 預設 agentgateway,以 OIDC JWT 帶部門 claim,除非每團隊預算 UX 比攻擊面更重要。地端的「預算」應該是 **GPU 容量份額,不是金額**。起始權重可設為 input ×1、cached ×0.1、output ×4,再以 GPU 秒校準;這些權重是假設。每部門給保證份額加 burst,互動優先於批次。agentgateway 與 Agent Router 都在回應完成後才計費,失控串流會超支一個回應,因此每個身分要有 max-token 與併行上限。

### 硬體分級:8-GPU 伺服器才是起點,工作站只是個人工具

**【估算】容量推導。** 從 Claude Code 文件的範例 session 與 $13/開發者/活躍日 ([Anthropic,廠商](https://code.claude.com/docs/en/costs)) 反推,每位開發者 8 小時約 4.3 output tok/s;乘上尖峰 ×3 與 open 模型冗長、重試、背景 agent ×3,得約 **40 output tok/s**。供給面,8×H200 跑 GLM 5/5.1 在 34 tok/s/user 下約 5,920 tok/s ([InferenceX](https://inferencex.semianalysis.com/compare/glm-5-1-h200-vs-mi355x)),再為長 context 與多租戶打五折,得 **約 74 位開發者/台,範圍 40 到 100**。一個常駐 agent 假設等於 3 到 5 位開發者。這條鏈的每一個係數都是假設,且沒有公開負載測試重現 100k+ context 的多租戶情境,所以必須以實際 trace 重播驗證。

| 【估算】 | Pilot | 部門級 | 多部門 |
|---|---|---|---|
| 開發者(常駐 agent) | 10 到 25(少量) | 50 到 100(至多約 10) | 300 到 500(至多約 50) |
| 硬體 | 4×RTX PRO 6000 或 1×DGX Station | 2× 8-GPU(H200 或 B200;MI355X 為實測替代) | 約 8× 8-GPU 含 N+1,400G 網路,共享儲存 |
| 主模型:私部門 | Qwen3.8-27B;V4-Flash 用 2 卡 | GLM-5.3 或 V4.1-Flash-1017 | 同左,加 GLM-5.3-Flash 快速層 |
| 主模型:受限 | Laguna S 2.1 或 Gemma 4 | Mistral Large 4(需 B200 或 MI355X) | 同左,加輕量層 |
| 預期品質 | 長程弱(DeepSWE 約 40%) | 部分評測接近前沿,合併品質遠低 | 同左 |
| Capex | 約 $90K | 約 $1.05M | 約 $4.2M |
| IT / 設施電力(PUE 1.4) | 3.5 / 4.9 kW | 28 / 39 kW | 112 / 157 kW |
| 冷卻 | 機房氣冷 | 高密度氣冷或液冷 | 液冷或背門,3 到 4 櫃 |
| GPU 維運人力 | 0.5 FTE | 1.5 FTE | 4 FTE |

**判斷:** 這張表有三個讀法要注意。第一,Pilot 硬體**跑不動**部門級模型,所以用它驗證品質會得到錯誤答案,這正是 PoC 要租硬體的原因。第二,上限組合沒有餘裕:部門級 100 人加 10 個 agent 約 130 到 150 人當量,兩台全開約 148,沒有 N+1;多部門 500 人加 50 個 agent 達 650 到 750 人當量,超過 8 台的 592。常駐 agent 數才是真正的容量變數。第三,受限路線的 Mistral Large 4 約 1.06TB,放上約 1.13TB 的 H200 幾乎不剩 KV cache 空間【估算】,會把硬體選擇往 B200 或 MI355X 推。

| 硬體 | 記憶體 | 價格(皆為次級來源) | 供貨 | 功耗與條件 |
|---|---|---|---|---|
| [HGX H200 8-GPU](https://intuitionlabs.ai/articles/nvidia-ai-gpu-pricing-guide) | 約 1.13TB(規格憑記憶) | $315K 到 $420K | [經銷商稱有貨](https://viperatech.com/news) | 約 11 kW【估算】 |
| [HGX B200 8-GPU](https://www.besthub.dev/articles/nvidia-server-gpu-2026-buying-guide-specs-prices-compliance-channels-9c6195bac2c4) | 約 1.5TB | $400K 到 $500K | [8 到 20 週,客製 26 週以上](https://viperatech.com/news) | [10 到 15 kW](https://intuitionlabs.ai/articles/nvidia-hgx-data-center-requirements) |
| MI355X 8-GPU | [288GB × 8,約 2.3TB](https://www.tomshardware.com/pc-components/gpus/amd-announces-mi350x-and-mi355x-ai-gpus-claims-up-to-4x-generational-gain-up-to-35x-faster-inference-performance) | 無可信購價;[租 $2.59/GPU-hr](https://www.gpuperhour.com/rent/mi355x) | 未驗證 | [1,400W/GPU,主要液冷](https://www.tomshardware.com/pc-components/gpus/amds-instinct-mi355x-accelerator-will-consume-1-400-watts) |
| [RTX PRO 6000](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) | 96GB | 2026-10 零售近 $16K | [2026 年三次漲價,供貨緊](https://viperatech.com/product/nvidia-rtx-pro-6000-blackwell-Series) | 無 NVLink |
| [DGX Station GB300](https://www.tomshardware.com/desktops/nvidias-gb300-powered-dgx-station-desktop-tower-listed-for-nearly-usd100-000-online-enterprise-ai-powerhouse-now-available-to-buy-for-mere-mortals-with-lots-of-cash) | [748GB,含 496GB LPDDR5X](https://wccftech.com/nvidia-dgx-station-upgraded-gb300-blackwell-ultra-desktop-superchip/) | $85K 到 $125K | [4 到 12 週](https://pi3g.com/?p=31780) | 高速部分約 252GB【估算】 |
| [DGX Spark](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/)、[Mac Studio M5 Ultra](https://techradar.com/pro/apple-will-now-sell-you-a-mac-studio-with-512gb-of-ram-but-itll-cost-you-as-much-as-a-good-sized-family-car) | 128GB;至多 512GB | 約 $6,950;$22K 到 $30K | 有貨 | 記憶體頻寬限速;只作個人工具 |

**證據與判斷:** InferenceX 顯示 MI355X 在這些工作負載的 tok/s/chip 約為 H200 的 1.7 到 3.9 倍 ([InferenceX](https://inferencex.semianalysis.com/compare/gptoss-120b-h200-vs-mi355x)),但它是內插數字、未標示引擎,且軟體成熟度與 AMD 的投入有關 ([FourWeekMBA](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/))。AMD 值得一測,但隔離環境拿不到快速修補,不應未經 PoC 就採購。

### TCO:部門級約 US$568/人/月,成本理由成立的門檻很窄

**【估算】** 假設:三年直線折舊;支援 10% capex;PUE 1.4;US$0.134/kWh(以 [NT$4.29 工業平均](https://www.bnext.com.tw/article/74485/taipower-electric-valance-increase)除以匯率 32 作為佔位值,資料已舊,也不代表組織所在地);每 FTE US$120K(佔位);不含機房改裝、NVIDIA AI Enterprise 與殘值;人力只計 GPU 維運。

| 【估算】 | 折舊 | 支援 | 電力 | 人力 | **年總計** | 每人每月 |
|---|---|---|---|---|---|---|
| Pilot,20 人 | $30K | $9K | $6K | $60K | **$105K** | **約 $437** |
| 部門級,100 人 | $350K | $105K | $46K | $180K | **$681K** | **約 $568** |
| 多部門,400 人 | $1.4M | $420K | $184K | $480K | **$2.48M** | **約 $518** |

**【估算】** 對照組是前沿模型的企業平均 $150 到 $250/人/月 ([Anthropic,廠商](https://code.claude.com/docs/en/costs)),100 人一年 $180K 到 $300K。以 $200/月計,要約 284 人才打平部門級成本,已超出這套硬體的估算容量;以重度用量 $30/活躍日、每月 22 天計,約 86 人打平。換成利用率:16 顆 H200 滿載一年約 373B output tokens,每百萬 output 成本 100% 利用率時 $1.82,20% 時 $9.12。以 8:1 input/output 比計,與託管 GLM-5.3 ([OpenRouter $1.085/$3.41](https://openrouter.ai/compare/deepseek/deepseek-v4.1-flash/z-ai/glm-5.3)) 打平約需 **15% 利用率**;假設 90% 快取命中且快取價為 input 的 20%,打平則需 **31%**。與 Opus 5.5 ([第三方報價 $4/$20](https://www.gradually.ai/en/llm-comparison/claude-opus-5.5-vs-deepseek-v4.1-flash/)) 打平約 7%,但兩者能力不對等。朝九晚六的使用型態大約只有 10% 到 25%(假設),所以離峰要用批次 agent 填滿,否則成本論述不成立。這裡的 $568 也還沒算平台人力:加上 5 到 7 FTE 後,部門級約 $1,070 到 $1,270/人/月。不過不論模型放哪,平台人力大半都要付,不應計入模型比較。

**證據:** 公開的打平主張彼此矛盾,且多有利害關係:[SitePoint 稱 4 到 6 個月](https://www.sitepoint.com/local-llms-vs-cloud-api-cost-analysis-2026/);[賣雲的 Spheron 說雲端勝](https://www.spheron.network/blog/llm-inference-on-premise-vs-cloud/),[賣地端的 VRLA 說地端勝](https://vrlatech.com/llm-inference-on-premise-vs-cloud-cost-2026/)。折舊也沒有共識:[Microsoft 用 6 年、Meta 5.5 年](https://fourweekmba.com/ai-nvidia-gpu-depreciation-durability-debate-huang-2026/),而[賣評估的 American Compute 估 H100 在 2028 年只剩牌價 5% 到 7%](https://www.amcompute.com/appraisals/h100)。模型更迭也快,DeepSWE 上 [GLM-5.2 44% 到 GLM-5.3 69%](https://deepswe.datacurve.ai/);本報告採三年折舊,是因為旗艦權重 2026 年內從數百 GB 成長到 1.5TB 以上,記憶體要留餘裕。

**誠實結論(判斷)。** 如果決策理由是省錢,在目前的平均開發者用量下,地端**不成立**,而且相對託管 open-weight 的 API 更不成立。它只在三種情況下合理:資料主權或法規是硬性要求,需要在外部 API 中斷或被限制時持續運作,或者常駐批次 agent 能把利用率撐到 30% 以上。前兩者是風險與合規預算,應該如此對內陳述,不要包裝成 ROI。你用了「盡量」這個詞,所以真正該問的是:**有多少程式碼屬於絕對不能離開的等級?** 這個比例才決定該買多少 GPU。

### 混合逃生口:路由預先設計好,預設不開

**判斷:** 建議從第一天就把路由做進閘道,因為把任務類別移到另一層應該只是設定,不是重寫。下列分級是本報告設計,在閘道依 repo 分級與身分強制,任何判斷失敗都預設拒絕;定期測試 fallback,否則真正需要時用不了。

| 分級 | 預設 | 允許的例外 | 備註 |
|---|---|---|---|
| S3 受限 | 僅地端 | 無 | 失敗就失敗 |
| S2 機密 | 地端 | 核准的私有 VPC open-weight burst | 同權重,控制面自有 |
| S1 內部 | 地端 | 兩次失敗或規劃類任務,升級至私有雲端前沿端點 | 例外路徑,不是預設 |
| S0 公開 | 任意 | | |

| 部署方案 | 地端內容 | 雲端內容 | 適合 | 主要代價 |
|---|---|---|---|---|
| A 純地端 | 全部 | 無 | S3 占多數,或政策零例外 | 品質落差全部由人吸收 |
| **B 地端 + 例外路徑(預設)** | 骨幹、資料、主推論 | S0、S1 的前沿升級 | 「盡量」不用雲端 | 例外審批與路由測試 |
| C 地端骨幹 + 私有 VPC 推論 | 骨幹、資料 | [租用 GPU 跑相同 open 權重](https://www.gpuperhour.com/rent/mi355x) | S3 很少、GPU 預算不足 | 程式碼離開機房;利用率風險轉給供應商 |

**證據:** 雲端端點的條件要逐一確認。Claude 可經 Bedrock、Google Agent Platform 或 Foundry 計費到你的雲端帳號,文件稱不回傳指標給 Anthropic ([Anthropic](https://code.claude.com/docs/en/costs))。**若需要境內處理**,目前證據並不樂觀:Bedrock 相容表已讀部分顯示 Claude 5.5 系列在 `ap-east-2` 僅全域跨區推論 ([AWS](https://docs.aws.amazon.com/bedrock/latest/userguide/models-region-compatibility.html));Azure Taiwan North 2026 年 9 月仍分階段向特定客戶開放 ([Focus Taiwan](https://focustaiwan.tw/business/202609010030));資料存放在境內也不代表推論在境內。**判斷:** 公部門關聯資料走這條路要另外取得核准。

### 模型供應鏈:Safetensors 不是終點

**證據:** Pickle 掃描器一再被繞過 ([picklescan advisories](https://github.com/mmaitre314/picklescan/security/advisories)),`trust_remote_code=False` 也不是完整保證 ([CVE-2026-90553](https://github.com/vllm-project/vllm/security/advisories/GHSA-3c86-2m5g-59q7)),chat template 同時是功能風險與攻擊面 ([vLLM](https://github.com/vllm-project/vllm/security/advisories/GHSA-542m-9234-8gvf))。連安全工具都可能是入口:LiteLLM 的入侵被追溯到 CI 中的 Trivy ([LiteLLM](https://docs.litellm.ai/blog/security-update-march-2026))。**判斷:** 下表是本報告提案的流程。掃描偵測不到被下毒的權重,只有行為測試能部分補上,且沒有找到獨立的權重後門偵測評估。

| 步驟 | 動作 |
|---|---|
| 匯入(連線 DMZ) | 依確切 commit 下載,記錄 SHA 與每檔 SHA-256,保存授權快照;拒絕 `.bin`、`.pt`、`.pkl` 與需要遠端程式碼的 repo |
| 檢查(無憑證主機) | [modelscan](https://github.com/protectai/modelscan) 加第二掃描器 fail-closed;比對 config、tokenizer、chat template 的差異 |
| 行為測試 | 工具呼叫一致性([Kimi-Vendor-Verifier](https://github.com/MoonshotAI/Kimi-Vendor-Verifier) 風格)、內部回歸、繁中套件、外洩與不安全程式碼誘餌 |
| 簽章傳輸 | 內部金鑰簽章([model-transparency](https://github.com/sigstore/model-transparency) 或 cosign),單向傳輸後重驗 |
| 服務 | 唯讀掛載、釘容器 digest、無出站、HF offline 模式、非 root |
| 更新 | 每季模型、每月引擎;保留上一版模型與 parser 以便回滾;每個 agent commit 記錄模型與引擎版本 |

## 人力:2 到 3 人起步,全地端多部門可到 10 到 13 人

**證據:** 沒有公開案例給出這套自建堆疊的團隊規模。最好的通用基準是 DX 對 39 家公司的調查:中央開發者生產力團隊占 2% 到 6%、平均 4.7%,但**不含 SRE、雲端維運與安全** ([DX,摘要](https://newsletter.getdx.com/p/devprod-headcount-benchmarks-q1-2026));其他比例說法互相衝突且無研究基礎 ([1:8 到 1:12](https://platformengineeringcost.com/team-structure),[20:1](https://tianpan.co/forum/t/20-1-developer-to-platform-engineer-ratio-what-are-elite-platform-teams-actually-doing-differently/3881))。Temporal 文件只說人力與技能是可觀的持續成本,沒有數字 ([Temporal](https://docs.temporal.io/self-hosted-guide/production-checklist))。

| 【估算】工作線 | 試點 FTE | 生產 FTE |
|---|---|---|
| Kubernetes、儲存、ingress、升級 | 0.5 到 1 | 1 到 1.5 |
| Temporal 或 Dapr、Postgres | 0 到 0.25(先用 Pipelines) | 0.5 到 1 |
| Wiki、搜尋 sidecar、檢索、連接器 | 0.5 | 1 到 1.5 |
| 閘道、沙箱、身分、密鑰、政策 | 0.5 | 1 |
| 觀測與成本歸屬 | 0.25 | 0.5 |
| 測試工具鏈、掃描器、skill registry | 0.25 | 0.5 到 1 |
| **小計(平台,含 Azure DevOps 整合)** | **2 到 3** | **5 到 7** |
| GPU 與模型維運 | +0.5 | +1.5(部門級)或 +4(多部門) |
| Air-gap:feed 傳輸、鏡像、授權與升級演練 | +0.5(若實施) | +1 到 2(若實施) |
| **總計** | **2.5 到 4** | **6.5 到 10.5(部門級);9 到 13(多部門)** |

**判斷:** 這些數字假設 100 到 300 位開發者、一到兩個叢集。生產數字在 150 人以上最合理;少於 50 人時,全套不太可能回本,應精簡。少於 4 人時,stateful 工作流程引擎的 on-call 很脆弱,這是延後 Temporal 的另一個理由。職能上最難補的不是 Kubernetes,而是能同時懂 Azure DevOps REST、模型 parser 與評測的人,因為 Server 路線的整合層與模型回歸都沒有廠商可以依靠。授權預算也要另列,例如 Onyx 150 人一年 $36,000 ([Onyx](https://onyx.app/pricing)),SonarQube、Coder Premium 與套件庫則需報價。

## 先驗證再採購:四階段路線與 PoC 清單

**判斷:** 以下時程與門檻都是提案。起點選有強測試、重複性高的工作,這與 Stripe 的 flaky test 工單及 Spotify 的維護遷移相符 ([Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents),[Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint))。

| 時程 | 範圍 | 新增元件 | 進入下一階段的條件 |
|---|---|---|---|
| 第 0 到 2 週 | 確認事實 | Server 版本、用量盤點、資料分級、法律諮詢 | 知道 Server 或 Services、S3 比例、模型資格 |
| 第 2 到 10 週 | **採購前 PoC** | 租用 GPU、閘道、沙箱、一條端到端流程 | 下表阻擋項全部通過 |
| 第 3 到 6 個月 | 1 到 2 個團隊的受限試點:flaky test、依賴升級、小修 | Pilot 級或部門級硬體、PR-Agent、skill monorepo、BookStack;全部人審 | 變更失敗率與漏網缺陷率持平,審查容量撐得住 |
| 第 7 到 12 個月 | Spec-to-PR、第二個部門 | 規格晉升、fresh-context verifier、Onyx、配額分帳;視需要加 Temporal | Lead time 改善且品質持平;利用率達規劃值 |
| 第 12 個月後 | 分級自主 | 特定變更類別自動合併;降級規則 | 該類別失敗率經證實偏低 |

**【估算】** PoC 硬體很便宜,所以沒有理由跳過:以單一供應商的 MI355X 租價 $2.59/GPU-hr,8 GPU 連續兩週約 US$7,000,不到部門級 capex 的 1%。限制是必須與計畫購買的硬體相符,且該供應商實際供貨未驗證;租用雲端 GPU 也不適合 S3 程式碼,應以 S0、S1 或脫敏 repo 測試。

| PoC 項目 | 做法 | 通過標準(提案) | 失敗時的改動 |
|---|---|---|---|
| Webhook 私有 IP(阻擋) | Service Hook 打內網 receiver | 事件到達且可驗證 | 改輪詢或可路由位址 |
| Git 認證(阻擋) | 關閉 NTLM,PAT、Kerberos、SSH 全測 | 服務帳號穩定 clone 與 push | 調整 IIS 與憑證 |
| REST 與 MCP(阻擋) | 對實際 Server 版本跑 Tiberriver256 的讀寫;確認 API 上限 | 建 PR、更新工作項目、發留言成功 | 自建 MCP |
| 端到端追溯 | 工作項目到部署紀錄整條跑通 | 每一跳可回查;trailer 被強制 | 調整 pipeline 模板 |
| PR-Agent | Collection URL + PAT + 本地模型 | 能回寫 PR thread | 自寫審查步驟或問 Qodo |
| 繁中搜尋(阻擋) | 真實 URD 上測完整片語、句中二字詞、中英混合;比較 Azure DevOps Wiki、BookStack、sidecar 的 2-gram 與 IK | 達內部召回基準 | 換 sidecar 或 wiki |
| 模型 bake-off(阻擋) | 50 到 100 個內部任務,固定 harness,目標量化,自託管 | 比較首次合併率、人工分鐘、重工,對照前沿基線 | 換模型或走方案 B、C |
| 容量 | 以真實 agent trace 重播 | 尖峰延遲可接受時的人當量 | 調整 tier、限制常駐 agent |
| 工具呼叫 | Vendor Verifier 風格,vLLM 與 SGLang 對測 | Schema 正確率與穩定性達標 | 換引擎、parser 或模型 |
| 繁中模型品質 | 需求轉程式碼、簡體洩漏率、台灣用語、敏感內容 | 達內部基準 | 加後處理或換模型 |
| 出站(阻擋) | 每個工具 default-deny + DNS 記錄,跑啟動、使用、升級 | 無未申報外連,失敗模式可接受 | 替換工具 |
| 部門隔離(阻擋) | 以 A 部門身分嘗試存取 B 部門文件、MCP、trace、金鑰 | 零跨部門成功 | 架構改為獨立實例 |
| 沙箱與 Prompt injection | VMware 巢狀虛擬化;在 PR 描述與 repo 指令檔植入惡意指令 ([ReviewPR 提醒](https://world.optimizely.com/blogs/kennyg/dates/2026/5/reviewpr-an-azure-function-that-reviews-your-azure-devops-pull-requests-with-claude/)) | 無權杖外洩,出站被擋 | 收緊 stage 分離 |
| 廠商書面答覆 | Tabnine、Factory、Qodo、CodeRabbit、Onyx、Sourcegraph:Server 版本支援、離線授權、更新機制、完整外連清單 | 有書面答覆 | 不納入 |
| 法律 | 模型來源、MaaS 條款、集團法人結構、BSL 與 AGPL、TAIDE 授權 | 取得書面意見 | 換路線 |
| 離線演練(若實施) | 一次完整 feed bundle 與引擎修補走單向傳輸 | 一個月內可重複 | 調整人力或放寬隔離 |
| AMD(若考慮) | 目標模型與引擎上實測 | 穩定度與吞吐達標 | 回到 NVIDIA |

**判斷:** KPI 要配對,不能只看 PR 數。交付面看 lead time、部署頻率、變更失敗率;品質面看漏網缺陷率;瓶頸面看審查時間;agent 面看無人工修改的合併率、每 PR 的 CI 輪數與 30 天 churn;地端專屬面則看 GPU 利用率、每合併 PR 的 GPU 秒、例外路徑使用率與模型升級回歸結果。

## 已知的未知數

**平台。** 沒有工具經本研究實機測試或封包檢查。Server 25H2 與 26H2 的 REST API 上限、社群 MCP 與 PR-Agent 對它們的相容性都未驗證;26H2 完整功能清單未取得;libcurl 移除 NTLM 是否如期、出現在哪個 Git for Windows 版本也未查。Services 回遷 Server 的路徑未研究,Microsoft-hosted agent 不適用於 Server 這一點也只是背景知識。GHAS 不支援 Server 沒有 Microsoft 一手聲明。Foundry Local 等 Microsoft 離線推論、Azure Local 與 AKS Arc 都沒有研究。Copilot CLI 的離線模式雖然[記載於 GHES](https://docs.github.com/en/enterprise-server@latest/copilot/copilot-on-ghes/about-copilot-on-ghes),能否搭配 Azure DevOps Server 未測,不應現在就當選項。

**工具與知識。** 所有候選的繁中搜尋都缺官方數據,Azure DevOps 也不例外。Onyx 自建連接器能否帶逐文件 ACL、權限同步需要哪個方案、離線授權如何續約,都要書面答覆。Azure DevOps Wiki 的頁面層權限與 Test Plans 授權未研究。本地模型驅動 Playwright Test Agents、Strix 與 skill-scanner 的效果都沒有證據。

**模型與成本。** 沒有任何 benchmark 測過真實企業 harness 下自託管量化模型的表現,也沒有中立的自託管工具呼叫比較、繁中評測或 100k+ context 多租戶負載測試。本地中國來源權重是否落入 2019 年原則,找不到法律文字;Mistral Large 4 與 Devstral 2 的完整條款、TAIDE 授權都要法律審閱。MI355X 購價、在地報價與交期、FTE 成本、機房費用都缺資料,目前電價也是舊值,所有 TCO 都應以實際報價重算。GitLab 價格與 ADO 到 GitLab 的遷移工具未研究。

**成效。** 沒有找到端到端需求到測試的受控 factory 研究,需求階段自動化尤其缺乏成效證據;這份架構是由零散證據推論而來,不是經驗證的藍圖。

## 結論

這輪研究把問題從「挑哪些工具」變成「按什麼順序花錢」。直覺的順序是先買 GPU、再裝工具、再找使用情境;證據支持的順序正好相反:先花兩週弄清楚 Server 或 Services 與資料分級,花兩個月以不到 1% 的硬體預算測出模型品質、繁中搜尋與 webhook 這類阻擋項,最後才決定要不要花七位數美元。這個順序能把最大的不確定性留在最便宜的階段處理,而且部分結果可能直接讓硬體採購消失。

另一個值得帶走的觀點是,Microsoft 把 AI 留給 Services,對 Server 使用者其實釐清了責任邊界:沒有官方路線可等,整合層就是你的資產。一個只暴露少數動詞的自建 MCP、一套 pipeline 模板與 receiver、Git 中的規格與 skill、加上你的內部評測集,這些比任何單一 agent 都耐久,也是日後遷往 GitLab 或更換模型時能帶走或快速重寫的部分。建議把這些當作這個計畫真正要累積的東西,GPU 與模型則當作每一到三年就會換掉的耗材。
