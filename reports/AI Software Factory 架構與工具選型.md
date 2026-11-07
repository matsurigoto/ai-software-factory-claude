# 以確定性骨幹組裝多部門 AI Software Factory

截至 2026-11-07,**沒有任何單一商用或開源產品能涵蓋從需求到維運的完整 AI Software Factory**,因此建議圍繞現有 SCM(GitHub 或 GitLab)自行組裝,而不是押注一個平台。架構核心有四點:協調器採確定性工作流程(預設 Temporal 加一層薄的 SDLC 服務),LLM 只在階段內做規劃與探索;知識層採混合式,PM 在 wiki(預設 Outline)做探索,核准後的 URD 以穩定需求 ID 晉升到 Git(Spec Kit 或 OpenSpec),再由具權限感知的檢索層(Onyx)供 agent 使用;開發階段每部門一個主要 harness,從全公司 2 到 3 個白名單中選,技能則以 `SKILL.md` 與 `AGENTS.md` 保持可攜;測試則讓 agent 提案、工具判定、人類擁有規格與 oracle。這個設計的依據是現有證據一致顯示瓶頸已移往審查與驗證:Faros 的遙測資料中,高採用組織的 task 增加,審查時間卻約為 5 倍、每週部署反而下降。最大的未驗證風險有三項:繁體中文搜尋品質、部門間隔離、資料境內處理,必須在選型前實測。所有工具都未經本研究實機測試,需求階段自動化也幾乎沒有成效證據,因此本報告的選擇是供評估的起點,不是定論。

## 先定案:六層架構、一寫多驗、資產自持

**閱讀方式。** 本報告彙整五份研究筆記,GitHub 星數、授權與版本由研究者於 2026-11-07 直接讀取 GitHub API。內文以「**證據**」標示可查證來源的事實,以「**判斷**」標示本報告的推論;表格中名為「判斷」或「建議」的欄位都是推論,名為「條件」的欄位則是來源記載的事實。多數結果由廠商自述,會在句中註明。利益揭露:本報告由 Anthropic 的模型 Claude 撰寫,研究期間 Claude Code 文件最容易讀取,因此其篇幅反映的是可得性,不代表排名。

**判斷:** 下表是建議的參考架構。整體方向與 Stripe、Spotify 公開架構的共同元素相符(隔離環境、集中工具閘道、確定性步驟、受限重試),但沒有任何來源以這個組合測得成效。

| 層 | 職責 | 預設選擇 | 替代 | 關鍵條件 |
|---|---|---|---|---|
| L1 知識與需求 | PM 探索、URD、核准後規格、工作追蹤 | Outline + Spec Kit 或 OpenSpec + 現有 tracker | BookStack、Docmost Business、Confluence Cloud、Notion;StrictDoc 或 Sphinx-Needs(稽核級) | 繁中搜尋須實測;Outline 為 BSL,非 OSI 開源 |
| L2 Context 檢索 | 依權限檢索企業知識與程式碼知識 | Onyx + codebase-memory-mcp、Repomix | RAGFlow;Devin/DeepWiki MCP | Onyx 權限同步須付費版 |
| L3 協調控制面 | 階段、閘門、重試、核准、預算 | Temporal + 自建薄 SDLC 服務;試點期可先用 SCM 工作流程 | Dapr Agents、gh-aw、GitLab Flows、Harness | 需要平台團隊;階段內圖形才用 LangGraph 或 Agent Framework |
| L4 Agent 執行 | 設計、實作、重構、文件、遷移 | 每部門一個主要 harness,白名單 2 到 3 個;私有 skill registry | 雲端 agent(Devin、Factory)處理 backlog | 一個分支或模組只有一個寫手;第三方 skill 視同供應鏈 |
| L5 驗證 | 測試生成、審查、確定性閘門 | Fresh-context verifier + 各類確定性工具 + Playwright Test Agents | QA Wolf、Diffblue、Applitools、XBOW | LLM 不得單獨決定阻擋性閘門 |
| L6 平台與治理 | 沙箱、閘道、遙測、身分、政策、目錄 | agent-sandbox + gVisor/Kata、agentgateway 或 Agent Router、Langfuse + OTel、OpenBao + SPIRE、OPA 或 Cedar、Backstage | Coder(付費治理)、Port、LiteLLM(釘版容器) | 部門隔離放基礎設施層,不信任應用層 tenancy |

**判斷:** 下表把架構落到 SDLC 流程。核心原則是狀態永遠留在自己掌控的系統(Git、tracker、工作流程引擎),agent 與 UI 都可替換。

| 階段 | Agent 做什麼 | 狀態存放 | 人類閘門 | 確定性檢查 |
|---|---|---|---|---|
| 探索、URD 草稿 | 訪談摘要、交叉連結、缺口 lint;agent 只能寫草稿區 | 不可變原始資料 + wiki | PM 審閱 | 生成頁面須標註並附來源 |
| 核准與晉升 | 將 wiki 內容轉為含穩定 ID 的規格 PR | Git(核准版權威來源) | 需求負責人簽核 PR | 每條驗收條件有 ID 且可測 |
| 設計 | 主要 agent 產出 ADR 與 plan | Git | 依風險分級核准 | 需求對任務覆蓋分析 |
| 實作 | 單一寫手 + TDD skill,在隔離沙箱中執行 | 分支、工作流程引擎 | 無(受閘門約束) | Lint、型別、單元測試、secret scan |
| 審查 | 與作者分離的 review agent,先建議不阻擋 | PR | 人類合併 | SAST、SCA、測試竄改檢查、追溯檢查 |
| 驗證 | Fresh-context verifier 只憑規格與公開介面寫測試 | 受保護測試路徑 | 處理測試 oracle 變更 | 變更碼 mutation、Pact、fuzz、E2E |
| 發布與維運 | Healer 開 PR;AI pentest 僅限授權環境 | 流水線、遙測 | 高風險例外與發布決策 | SLO、exploited 等級政策、自動回滾 |

**判斷:** 使用者需要先做的決策如下。預設值偏向低鎖定與可退場,每項都列出會推翻預設的條件。

| 決策 | 預設 | 何時改變 |
|---|---|---|
| 買、擴充或組裝 | 圍繞現有 SCM 組裝,購買 agent,自持規格、skill、MCP、評測與日誌 | 無平台團隊且可接受單一廠商時,改買 Factory 或 Devin;規模極大且技術棧特殊時才考慮 Stripe 式全自建 |
| 協調器形式 | 確定性工作流程,LLM 只在階段內 | 探索型工作(分解、triage、除錯假設)才交給 LLM 協調 |
| PM 知識來源 | 混合:wiki 探索,Git 為核准版權威 | 全工程團隊可用純 docs-as-code;文件不影響建置時可純 wiki |
| Coding agent 策略 | 每部門一個主要,全公司白名單 2 到 3 個,審查獨立 | 內部評測的增益大於多一套治理面的成本,或資料落地限制強制時才加入 best-of-breed |
| 模型路由 | 依敏感度分流:低敏感用前沿模型,敏感程式碼用自託管 open-weight | 若法遵不要求境內處理,可放寬;反之須先確認供應商條款 |
| Self-hosted 或 SaaS | 依資料分級逐層決定 | Atlassian 與 Notion 已是標準環境時,Confluence Cloud 或 Notion 的權限同步優勢可能勝出 |

**判斷:** 三條可比較的落地路徑如下,它們共用 L1、L5 與 L6 的中立元件,差別在錨點。沒有明確條件時,先以現有 SCM 選 A 或 B 的錨點,逐步引入 C 的中立控制面。

| 路徑 | 組成 | 適合 | 主要代價 |
|---|---|---|---|
| A:GitHub 原生 | Copilot + 第三方 Claude/Codex agents、gh-aw、Actions 與 CodeQL | 已在 GitHub Enterprise,要最少整合 | 第三方 agent 仍是 public preview;本機 VS Code agent 不受組織政策管轄;無台灣資料落地區域 |
| B:GitLab 自管 | GitLab Self-Managed + Duo Agent Platform + 自託管模型 | 需要本地推論且已用 GitLab | 自託管模型仍計 credit;air-gap 文件未說明,需與業務確認 |
| C:中立組裝 | Temporal、OpenCode/Goose/Codex、agent-sandbox、agentgateway、Langfuse、Backstage | 多部門、多模型、有資料分級,且有平台團隊 | 整合責任全在自己;Temporal 自管負擔未經本研究評估 |

## 證據指向同一件事:管線當骨幹,寫手要少,驗證要多

**證據:** 模式研究的方向相當一致。Google Research 的部落格版本顯示,獨立多 agent 把錯誤放大至 17.2 倍,集中式則控制在 4.4 倍;集中式在可平行的金融任務上提升 80.9%,但在循序規劃任務上,**每種多 agent 變體都退步 39% 到 70%** ([Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/))。同一論文的 v3 版改為 260 組設定、六項 benchmark,提升與退步數字為 80.8% 與 70.0%,且 benchmark 並非端到端 SDLC ([arXiv 2512.08296](https://arxiv.org/abs/2512.08296))。Anthropic 的多 agent 研究系統比單 agent 高出 90.2%,但 token 約為一般對話 15 倍,並明言多數程式撰寫這類需共享 context 的工作並不適合 ([Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system))。Cognition 的實務結論是讓寫入維持單執行緒,其他 agent 只貢獻判斷力 ([Cognition](https://cognition.com/blog/multi-agents-working))。

**證據:** 生產案例也站在同一邊,但都是自述。Stripe 的 blueprint 是以程式碼定義的狀態機,穿插確定性節點與 agent 節點,CI 最多跑兩輪就退回人類,每週有 1,300 件以上 agent 產出的 PR 合併,且人類仍審查 ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))。它還建立在多年開發者工具投資之上,並 fork goose 作為執行引擎 ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))。Spotify 將 Claude Agent SDK、Backstage 與 Soundcheck 結合,PR 頻率增加 76%,但作者指出待審 PR 同步變多,碎片化程式碼庫中 agent 也明顯較差 ([Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint))。反例是 Anthropic 用 16 個 agent 並行、不設 orchestrator agent,花費近 2 萬美元寫出約 10 萬行 Rust 的 C compiler,但成果並非可直接替換的編譯器,新功能也常破壞舊功能 ([Anthropic](https://www.anthropic.com/engineering/building-c-compiler))。

| 模式 | 支持證據 | 反面證據 | 建議位置(判斷) |
|---|---|---|---|
| 確定性管線 + agent 節點 | Stripe 量產;循序任務多 agent 退步 | 對新型工作適應力較低 | Factory 骨幹 |
| Orchestrator-worker | Google 4.4 倍圍堵;Anthropic 90.2% | 約 15 倍 token;manager 容易過度指定 | 研究、審查、測試生成、跨 repo map-reduce |
| Generator-verifier | Cognition 自述;fresh-context 研究 | 無過濾時易循環 | 每個寫碼階段 |
| Planner-worker 大規模 | [Cursor](https://cursor.com/blog/scaling-agents) 數百 worker | 高支出、漂移、整合負擔 | 有強 oracle 的遷移 |
| Swarm 或對等 | C compiler | 17.2 倍(僅為近似對照);Cognition 否定 | 僅限實驗 |

**證據:** 組織層級的回報遠不如個別任務樂觀。METR 最新追蹤中,回鍋開發者節省 18% 時間,但信賴區間含零(-38% 到 +9%),METR 自己也稱資料因選擇效應而不可靠,真實加速可能更高 ([METR](https://metr.org/blog/2026-02-24-uplift-update/))。CMU 的 806 個 Cursor 採用 repo 分析顯示,產出增加在兩個月後消退,靜態警告增加 30%、複雜度增加 41% 則持續存在 ([He et al.](https://arxiv.org/html/2511.04427v2))。Faros 對約 22,000 名開發者的遙測資料顯示,高採用組的 task 增加 33.7%,審查時間約 5 倍,每 PR 事故約 3 倍,**每週部署卻下降 11.7%**;這是銷售此類分析的廠商所做的觀察研究,未證明因果 ([Faros](https://faros.ai/research/ai-acceleration-whiplash))。McKinsey 經 CIO Dive 轉述,約四分之一企業有明顯加速,但 30% 的生產力反而下降 ([CIO Dive](https://www.ciodive.com/news/enterprises-bet-coding-agents-despite-ROI/830943/))。

**判斷:** 設計規則因此是「一個分支或模組一個寫手、多個讀者與驗證者、確定性的合併閘門」。當步驟順序已知、需可重播稽核、需等待數日的人類核准、副作用須冪等、預算須在模型外強制,或失敗後不應重跑已完成工作時,用確定性引擎。審查與驗證容量應在擴大生成量之前先增加。C compiler、Cursor、Stripe 的共同成功因素都是強外部 oracle,測試薄弱的部門不應預期這些成果能移植。

## 整包平台尚不存在,開源 factory 專案多半只能當參考

**證據:** Gartner 於 2026 年 9 月發布了 AI Software Factories 的 Innovation Insight,但全文付費,摘要以外的內容都經由被點名且自我定位為 horizontal platform 的 Port 轉述;Port 稱目前沒有隨插即用的完整方案,並提到 2028 年至少 45% 大型軟體組織將運作 factory 的預測 ([Gartner 摘要](https://www.gartner.com/en/documents/8397881),[Port](https://www.port.io/blog/gartner-names-port-ai-software-factory-platform))。比較文章也多由自我排名第一的廠商撰寫,例如 [Warp](https://www.warp.dev/articles/best-software-factory-platforms-2026),只適合用來發現名單。Thoughtworks Radar Vol. 35 沒有「software factory」條目,但把 agent sandboxing 放在 Adopt、風險分級人類監督放在 Trial,人在迴路劇場與以編碼吞吐量衡量生產力放在 Caution ([Thoughtworks](https://www.thoughtworks.com/radar))。

| 商用產品 | 涵蓋範圍 | 公開價格 | 部署與治理條件 | 判斷 |
|---|---|---|---|---|
| [GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises) | Issue 到 PR;Claude、Codex 為[第三方 agent](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents) | Business 1,900、Enterprise 3,900 AI credits/人/月;另計 Actions 分鐘 | 第三方 agent 為 public preview;預算須開啟停用選項才強制;[資料落地](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency)無台灣 | 已在 GitHub 者的最低整合錨點 |
| [GitLab Duo Agent Platform](https://docs.gitlab.com/subscriptions/gitlab_credits) | DevSecOps 流程 | 牌價 $1/credit;訂閱含部分額度 | Self-Managed、Dedicated、自託管模型;自託管仍計 credit,失敗流程也可能計費 | SCM 原生中本地推論路線文件最清楚 |
| [Harness](https://www.harness.io/press-and-news/harness-launches-autonomous-worker-agents-for-software-delivery) | CI/CD、測試、安全、FinOps | 未公開 | 2026 年新 agent 產品;[次級報導](https://letsdatascience.com/news/harness-adds-autonomous-ai-agents-for-devops-automation-e27b029c)指部分 GA 狀態有矛盾 | 已用 Harness 流水線時再評估 |
| [Factory](https://factory.com/pricing) | 寫碼代理為主 | Pro $20 到 Max $200;Teams $60/團隊 + $40/席;企業洽詢 | [Air-gapped 模式](https://docs.factory.ai/enterprise.md)、子組織、稽核;認證未經 Trust Center 確認 | 主權需求下的商用候選 |
| [Devin](https://devin.ai/pricing) | 規劃、遷移、on-call(行銷主張) | Pro $20、Max $200、Teams $80 + $40/席 | VPC 與單租戶說法有衝突,須寫入訂單;[每組織用量上限](https://docs.devin.ai/admin/billing) | Agent-first 購買選項 |
| [AWS Kiro](https://kiro.dev/pricing/) | Spec-driven IDE 與 web | Pro $20 到 Power $200;團隊 $20/席 | 群組與額度池;台灣狀態未確認;[2025 年事故原因仍有爭議](https://www.theregister.com/2026/02/20/amazon_denies_kiro_agentic_ai_behind_outage/) | Spec-first 團隊候選 |
| [Google Antigravity](https://antigravity.google/docs/enterprise) | Agent IDE/CLI | 用量或含額度席位 | `global`、`us`、`eu` 端點,第三方推論不支援境內區域 | Google Cloud 既有用戶才考慮 |
| [Rovo Dev](https://www.usagepricing.com/blueprint/activity/atlassian-2026-10-15-rovo-dev-standalone-sunset) | 程式審查、CLI | 重組中,2026-12-03 起改 Rovo Credits(次級來源) | 治理資料未能查證 | 簽約前先確認 |
| [Port](https://docs.port.io/getting-started/agentic-sdlc-pillars/)、[Coder](https://coder.com/docs/ai-coder/ai-governance) | 控制面、受治理環境 | 洽詢 | Port 不提供 agent;Coder AI Bridge 與 Boundaries 需付費 add-on | 是元件,不是 agent 替代品 |
| [Tessl](https://tessl.io/) | Skill registry 與評測 | Registry 免費;企業洽詢 | [次級來源](https://codemyspec.com/blog/tessl-review)稱已從 spec framework 轉向 | 是元件;轉向提醒合約宜短 |

**證據:** 開源端的健康專案多是積木。[OpenHands](https://github.com/OpenHands/OpenHands)(GitHub 偵測為 MIT,92.8k 星,v1.37.0)定位為可自託管的 Agent Canvas,能透過 ACP 執行其他 agent;企業目錄授權未經確認。[MetaGPT](https://github.com/FoundationAgents/MetaGPT) 有 71k 星,但最新 release 是 2024-04-22,近 30 天零 commit。[strongdm/attractor](https://github.com/strongdm/attractor) 只有規格,自 2026 年 3 月後無 commit。以「software factory」為名的 repo,如 [agent-smith](https://github.com/holgerleichsenring/agent-smith)、[preloop](https://github.com/preloop/preloop),星數多在數十到兩百之間;[awesome-software-factories](https://github.com/varun1505/awesome-software-factories) 是可追蹤的名單。

**判斷:** 不要以任何自稱 factory 的開源專案為基礎,把它們當設計參考即可。商用面的產品形態會在合約期內改變,Rovo Dev 與 Tessl 都是例子,因此合約宜短,並把規格、skill、MCP 與評測留在自己手上。StrongDM 的人類不寫也不審程式碼、每人每日至少 1,000 美元 token 的模式 ([StrongDM](https://factory.strongdm.ai/)) 未公布任何交付或品質指標,不應作為首輪目標。

## 協調層:Temporal 管骨幹,框架只管階段內

**證據:** Temporal 文件描述工作流程可在當機、網路逾時或等待人類核准數日後自動恢復 ([Temporal](https://docs.temporal.io/ai));它與 OpenAI Agents SDK 的 Python 整合於 2026-03-23 GA ([Temporal](https://temporal.io/blog/announcing-openai-agents-sdk-partnership)),Pydantic AI 也有 `TemporalAgent` 包裝 ([Temporal](https://temporal.io/blog/build-durable-ai-agents-pydantic-ai-and-temporal))。Microsoft 自己的指引也是能寫成函式就不要用 agent ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/))。下表的授權陷阱比星數更值得注意:Dify 把一個 workspace 定義為一個 tenant,多租戶運作需書面授權,正好與「每部門一個 workspace」的直覺設計衝突。

| 工作流程引擎 | 授權 | 狀態(2026-11-07) | 條件 | 判斷 |
|---|---|---|---|---|
| [Temporal](https://github.com/temporalio/temporal) | MIT | v1.32.2(11-02);24.2k 星 | GitHub advisory 清單為空,不等於安全;自管負擔未研究 | **預設** |
| [Dapr](https://github.com/dapr/dapr) / [Dapr Agents](https://github.com/dapr/dapr-agents) | Apache-2.0 | v1.19.1 / v1.1.0;[CNCF 宣布 GA](https://www.cncf.io/announcements/2026/03/23/general-availability-of-dapr-agents-delivers-production-reliability-for-enterprise-ai/) | Agents 僅 Python | K8s 優先、要 CNCF 治理時的替代 |
| [Argo Workflows](https://github.com/argoproj/argo-workflows) | Apache-2.0 | v4.1.6;17.1k 星 | Agent 功能未查 | 適合批次與 fan-out,不適合長時間人類閘門 |
| [Conductor OSS](https://github.com/conductor-oss/conductor)、[Prefect](https://github.com/PrefectHQ/prefect)、[Trigger.dev](https://github.com/triggerdotdev/trigger.dev) | Apache-2.0 | 皆活躍 | Agent 專屬功能大多未查 | 既有用戶可評估 |
| [Hatchet](https://github.com/hatchet-dev/hatchet)、[DBOS](https://github.com/dbos-inc/dbos-transact-py) | MIT | Hatchet 未達 1.0;DBOS 3.4.0 | 僅查 repo 中繼資料 | 小團隊輕量備選 |
| [Inngest](https://github.com/inngest/inngest/blob/main/LICENSE.md) | SSPL-1.0(未來轉 Apache-2.0) | v1.51.0 | 非 OSI | 法務審查後才用 |
| [Restate](https://github.com/restatedev/restate/blob/main/LICENSE) | BUSL-1.1 | v1.8.1 | 不可對外提供公共平台服務 | 部門分屬不同法人時要法務確認 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | 修改版 Apache-2.0 | 1.19.0;160.6k 星 | 多租戶需書面授權;[曾揭露跨租戶問題](https://github.com/langgenius/dify/security/advisories) | 只適合部門低程式碼前端 |
| [n8n](https://github.com/n8n-io/n8n/blob/master/LICENSE.md) | Sustainable Use + 企業授權 | 2.46.6;209.7k 星 | [至少 100 則 advisory,50 則高危或嚴重](https://github.com/n8n-io/n8n/security/advisories) | 網路隔離下的週邊自動化,非核心 |

**證據:** Agent 框架已明顯分化。AutoGen 的 README 明言已封存不再維護,雖然 GitHub 的封存旗標仍是 false,MetaGPT 也已停滯,兩者都名列最高星數。OpenAI Agents SDK 與 Claude Agent SDK 雖活躍,但都以供應商為中心;Claude Agent SDK 甚至有授權訊號衝突,GitHub 顯示 MIT,README 則指向 Anthropic 商業條款。LangGraph 的 `interrupt()` 可無限期等待,但節點在恢復時會從頭重跑,副作用必須冪等 ([LangGraph](https://docs.langchain.com/oss/python/langgraph/interrupts));其 checkpoint 層 2026 年也有序列化 CVE,例如 [CVE-2026-27794](https://nvd.nist.gov/vuln/detail/CVE-2026-27794)。

| 框架 | 授權 | 狀態 | 條件 | 判斷 |
|---|---|---|---|---|
| [LangGraph](https://github.com/langchain-ai/langgraph) | MIT | 1.2.20;44.6k 星 | 須有 checkpointer;注意 CVE | 階段內圖形首選 |
| [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) | MIT | .NET 1.28.0;活躍 | AutoGen 與 Semantic Kernel 的後繼 | .NET 部門首選 |
| [Google ADK](https://github.com/google/adk-python) | Apache-2.0 | v2.14.0;22.1k 星 | [多語言、可接非 Google 模型](https://adk.dev/) | Google 傾向的組織 |
| [Pydantic AI](https://github.com/pydantic/pydantic-ai) | MIT | v2.66.0 | [29 則 advisory,7 則高危](https://github.com/pydantic/pydantic-ai/security/advisories),也可能反映揭露較完整 | 模型可攜、Python |
| [Mastra](https://github.com/mastra-ai/mastra/blob/main/LICENSE.md) | Apache-2.0(`ee/` 另計) | core 1.82.0 | [2026-10 高危 advisory](https://github.com/mastra-ai/mastra/security/advisories) | TypeScript 團隊 |
| [CrewAI](https://github.com/crewAIInc/crewAI) | MIT | 1.15.37;60.6k 星 | 人類閘門與耐久性未經文件查證 | 先驗證再用 |
| [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) | MIT | [v0.24.0,仍 0.x](https://github.com/openai/openai-agents-python) | 預設 Responses API | 單一 OpenAI 部署 |
| [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) | 商業條款(見上) | [v0.2.179](https://github.com/anthropics/claude-agent-sdk-python) | 不支援從 SDK 啟動 [Agent Teams](https://code.claude.com/docs/en/agent-teams) | 標準化 Claude 的部門 |
| [AutoGen](https://github.com/microsoft/autogen)、[MetaGPT](https://github.com/FoundationAgents/MetaGPT) | 見各 repo | 已封存、停滯 | 不適用 | **不採用** |

**證據:** Coding-agent 協調器是變動最劇烈的一層。Vibe Kanban 的公司已於 2026 年 4 月關閉 ([公告](https://www.vibekanban.com/blog/shutdown));Paperclip 八個月累積 111.8k 星、約 6,500 個開放 issue 與 PR,且曾揭露跨租戶 API key IDOR 與 token 鑄造等[嚴重 advisory](https://github.com/paperclipai/paperclip/security/advisories);Ruflo 11 月 5 日才發布[六則高危 advisory](https://github.com/ruvnet/ruflo/security/advisories),社群[批評](https://github.com/ruvnet/ruflo/discussions/1359)尚無中立稽核可證實或否定。Claude Code [agent teams](https://code.claude.com/docs/en/agent-teams) 仍是實驗功能,預設關閉。

| 協調器 | 授權 | 狀態 | 條件 | 判斷 |
|---|---|---|---|---|
| [gh-aw](https://github.com/github/gh-aw) | MIT | v1.0.1;活躍 | Agent job 預設唯讀,寫入走 `safe-outputs` | GitHub 組織優先試 |
| [OpenHands](https://github.com/OpenHands/OpenHands) | MIT(偵測) | v1.37.0;[CVE-2026-33718](https://github.com/OpenHands/OpenHands/security/advisories) | 多 agent 能力與企業授權未查 | 自託管候選 |
| [Multica](https://github.com/multica-ai/multica/blob/main/LICENSE) | Apache-2.0 + 額外條件 | v0.8.1;未達 1.0 | 單一組織內部可用,對第三方服務需商業授權,不可移除品牌 | 內部看板候選 |
| [Agent Orchestrator](https://github.com/OrchestratorInc/agent-orchestrator) | Apache-2.0 | v0.14.10;從 ComposioHQ 遷移 | 本機桌面為主 | 開發者工具 |
| [Gas Town](https://github.com/gastownhall/gastown) | MIT | v1.2.1(06-06);30 天僅 8 commits | 需 tmux;成本估計互相衝突 | 僅實驗 |
| [Paperclip](https://github.com/paperclipai/paperclip) | MIT | 111.8k 星;積壓極大 | 見上述 advisory | 安全審查後才試,且每部門獨立實例 |
| [Symphony](https://github.com/openai/symphony) | Apache-2.0 | v0.0.3;工程預覽 | README 限可信環境 | 只讀規格 |
| [Conductor](https://thenewstack.io/a-hands-on-review-of-conductor-an-ai-parallel-runner-app/)、[Emdash](https://github.com/generalaction/emdash)、[Paradev](https://github.com/ParadevAI/Paradev) | 封閉、Apache-2.0、MIT | 活躍;[Conductor 僅 Apple Silicon](https://www.morphllm.com/conductor-ai-coding) | 未見集中稽核證據 | 個人生產力工具 |
| [Ruflo](https://github.com/ruvnet/ruflo)、[Vibe Kanban](https://github.com/BloopAI/vibe-kanban)、[Crystal](https://github.com/stravu/crystal) | MIT、Apache-2.0、MIT | 爭議、落日、已棄用 | 見上 | **生產環境避開** |

**證據與判斷:協定。** 協定層已大致收斂到 Linux Foundation 的 [AAIF](https://aaif.io/projects/):MCP(工具)、A2A(agent 間)、AGENTS.md 與 Agent Skills(指令),AG-UI([MIT](https://github.com/ag-ui-protocol/ag-ui))與 Agent Client Protocol([Apache-2.0](https://github.com/agentclientprotocol/agent-client-protocol))則補足前端與編輯器連接。MCP 2026-07-28 規格是破壞性更新:移除 protocol session 與 SSE 續傳,tasks 改為擴充 ([changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog))。**判斷:** 新 MCP server 應以無狀態設計並支援雙版本閘道,長時間工具呼叫建模成工作流程引擎 job。A2A v1.0 雖有 150 多家支持,但這是支持者數量,不是生產流量 ([Linux Foundation](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year)),因此只在真實信任或團隊邊界才使用;同一條管線內,引擎呼叫更簡單也更好稽核。IBM 的 Agent Communication Protocol 已[封存](https://github.com/i-am-bee/acp)併入 A2A,勿與 Agent Client Protocol 混淆。公開 MCP registry 不應自動取用,應維護內部白名單,因為生態系 CVE 量大,例如 CVSS 9.3 的 [FrontMCP 沙箱逃逸](https://osv.dev/vulnerability/CVE-2026-67531)。

## 知識庫:wiki 給人探索,Git 給機器依循,權限在檢索層落實

**證據:** Outline 是研究者唯一確認同時具備廠商提供的 MCP、可用於自託管,且各方案都有群組功能的自託管 wiki;它能搜尋、讀取、建立與編輯文件 ([changelog](https://www.getoutline.com/changelog/mcp),[MCP 文件](https://docs.getoutline.com/s/guide/doc/mcp-6j9jtENNKL))。但授權是 BSL,文件也未說明社群版是否含 MCP,以及逐文件的 MCP 權限行為。**台灣組織必須正視中文搜尋風險:** Outline 的 CJK 搜尋 issue [因閒置被自動關閉](https://github.com/outline/outline/issues/3207),BookStack [2018 年的句中搜尋問題](https://github.com/BookStackApp/BookStack/issues/778)現況不明,Docmost 維護者則[表示 CJK 能力在企業版的 Typesense driver](https://github.com/docmost/docmost/issues/1477)。

| PM wiki | 授權 | 狀態 | MCP 與條件 | 判斷 |
|---|---|---|---|---|
| [Outline](https://github.com/outline/outline) | [BSL 1.1](https://github.com/outline/outline/blob/main/LICENSE),2030-10-23 轉 Apache-2.0 | v1.12.0;41.2k 星 | 內建 MCP;不可作對外商業文件服務;[自託管 $4 到 $5/人/月,SAML 需付費版](https://www.getoutline.com/pricing) | **自託管首選** |
| [BookStack](https://github.com/BookStackApp/BookStack) | MIT | v26.09.2;[已移往 Codeberg](https://github.com/BookStackApp/BookStack/issues/778) | [OIDC 群組對應角色](https://www.bookstackapp.com/docs/admin/oidc-auth/);僅[社群 MCP](https://github.com/pnocera/bookstack-mcp-server) | 必須寬鬆授權時的首選 |
| [Docmost](https://github.com/docmost/docmost) | AGPL-3.0;付費版需金鑰 | v0.97.0 | [Business $6/席起含 MCP、SSO、頁面權限](https://docmost.com/pricing),表格部分標示不清 | 付費方案替代 |
| [Wiki.js](https://github.com/requarks/wiki) | AGPL-3.0 | 3.1.1 | [MCP 工作進行中](https://github.com/requarks/wiki/issues/8140),發布狀態未確認 | 觀察 |
| [AFFiNE](https://github.com/toeverything/AFFiNE) | [混合授權](https://github.com/toeverything/AFFiNE/blob/canary/LICENSE) | v0.27.4 | [後端生產使用需訂閱或協議](https://github.com/toeverything/AFFiNE/blob/canary/packages/backend/server/LICENSE) | 勿視為 MIT |
| [AppFlowy](https://github.com/AppFlowy-IO/AppFlowy)、[SiYuan](https://github.com/siyuan-note/siyuan)、[Logseq](https://github.com/logseq/logseq) | AGPL-3.0 | 活躍 | 未找到部門權限證據;MCP 多為社群或待開發 | 個人或 Git 寫作 |
| [Confluence Cloud + Rovo MCP](https://developer.atlassian.com/cloud/rovo-mcp/) | 商用 | MCP 依使用者既有權限 | [Data Center 2029-03-28 起唯讀](https://www.atlassian.com/licensing/data-center-end-of-life);Onyx 可同步其權限 | 嚴格 ACL 且可接受 SaaS 時首選 |
| [Notion MCP](https://developers.notion.com/guides/mcp/get-started-with-mcp) | 商用 | OAuth;本機 server 不再積極維護 | Headless bearer token 僅 Business 與 Enterprise | Notion 既有用戶 |
| [PandaWiki](https://github.com/chaitin/PandaWiki) | AGPL-3.0 | 10.5k 星 | 研究中發現,未評估 | 建議納入中文實測 |

**證據:** 檢索層中,Onyx 文件記載最清楚:56 個官方連接器、16 個支援權限同步,含 Confluence、Jira、SharePoint、GitHub,但**權限同步與群組範圍需要 Business 以上方案**,且以使用者 email 比對權限 ([Onyx](https://docs.onyx.app/admins/connectors/overview.md),[權限文件](https://docs.onyx.app/enterprise_edition/permissioning))。其 MCP server 宣稱與執行者相同的存取控制 ([README](https://github.com/onyx-dot-app/onyx))。Graphiti 的參考 server 不驗證呼叫者 ([README](https://github.com/getzep/graphiti));LightRAG 也提醒需先設定驗證,且隱藏管理 UI 並非安全邊界 ([README](https://github.com/HKUDS/LightRAG))。

| 檢索與記憶 | 授權 | 狀態 | 條件 | 判斷 |
|---|---|---|---|---|
| [Onyx](https://github.com/onyx-dot-app/onyx/blob/main/LICENSE) | MIT(CE);`ee` 為企業授權 | v5.1.0;33.3k 星 | 權限同步需付費;Outline、BookStack、Docmost 不在同步清單 | **首選**;CE 僅用於非敏感試點 |
| [RAGFlow](https://github.com/infiniflow/ragflow) | Apache-2.0 | v1.0.0;93.7k 星 | [官方 MCP](https://ragflow.io/docs/dev/launch_mcp_server);未發現繼承來源 ACL 的證據 | 文件密集、團隊範圍 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | 修改版 Apache-2.0 | 1.19.0 | 見多租戶條款;SSO 與 RBAC 在企業版 | 應用建置,需授權審查 |
| [Open WebUI](https://docs.openwebui.com/license/) | 自訂授權,品牌條款 | [v0.13.0](https://github.com/open-webui/open-webui) | 超過 50 人須保留品牌或取得授權;RBAC 未查證 | 觀察 |
| [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) | MIT | v1.19.0 | 權限文件無法取得 | 未驗證前不用於多部門 |
| [Cognee](https://docs.cognee.ai/core-concepts/multi-user-mode/permissions-system/overview) | [Apache-2.0](https://github.com/topoteretes/cognee) | v1.6.7 | 文件記載 tenant 與資料集權限;付費與否有衝突 | 知識圖譜首選 |
| [Graphiti](https://github.com/getzep/graphiti)、[LightRAG](https://github.com/HKUDS/LightRAG)、[GraphRAG](https://github.com/microsoft/graphrag) | Apache-2.0、MIT、MIT | 活躍 | 無內建企業 ACL 證據 | 放在自建授權層之後 |
| [Khoj](https://github.com/khoj-ai/khoj) | AGPL-3.0 | 最後 push 2026-08-02 | 活動趨緩 | 不列入 |

**證據:** 需求工具部分,Spec Kit 使用 `FR-001` 與 `SC-001` 編號,`/speckit.analyze` 輸出需求對任務覆蓋表 ([模板](https://github.com/github/spec-kit/blob/main/templates/spec-template.md),[analyze](https://github.com/github/spec-kit/blob/main/templates/commands/analyze.md)),但預設止於任務層,沒有延伸到程式碼與測試。一項 88,052 個 PR、119 個 repo 的研究發現有規格與無規格的缺陷率沒有差異(-0.6 個百分點,p=0.133),但作者明言其工件多為 issue 與 ticket 引用,**SDD 工具本身仍未被測試** ([ESEM 2026](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ESEM.2026.84))。Böckeler 的實作評論則指 Kiro 對小 bug 過重、Spec Kit 的 Markdown 審閱繁瑣,但距今已超過一年 ([martinfowler.com](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html))。Karpathy 的 LLM Wiki 明確定位為中等規模的個人想法檔案,大約 100 份來源與數百頁 ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f))。

| 需求與規格 | 授權 | 狀態 | 條件 | 判斷 |
|---|---|---|---|---|
| [Spec Kit](https://github.com/github/spec-kit) | MIT | v1.3.2;148.2k 星 | ID 方案最清楚;程式碼與測試追溯需自建 CI | 大型或跨團隊功能 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | MIT | v1.16.1;74.2k 星;[Radar Trial](https://www.thoughtworks.com/radar) | 語法輕;穩定 ID 未確認,需自訂慣例 | 日常變更預設 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | [MIT + 額外聲明](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/LICENSE) | v6.12.4;v7 為 alpha | 有 [BRD 模板](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/skills/bmod-method/doc-templates/template-brd.md);釘 v6.12 | 全生命週期方法參考,可改作 URD |
| [Kiro specs](https://kiro.dev/docs/specs/) | 商用 | EARS 需求格式 | [追溯主張僅廠商相關來源](https://codemyspec.com/blog/kiro-vs-openspec) | 已選 Kiro 時使用 |
| [StrictDoc](https://github.com/strictdoc-project/strictdoc) | Apache-2.0 | 0.30.4 | [ReqIF、File relations;部分為實驗性](https://strictdoc.readthedocs.io/en/stable/stable/docs/strictdoc_01_user_guide.html) | 受監管部門 |
| [Sphinx-Needs](https://github.com/useblocks/sphinx-needs) | MIT | 活躍 | [連結需求、規格與測試](https://sphinx-needs.readthedocs.io/en/latest/);社群小 | 受監管部門 |
| [Doorstop](https://github.com/doorstop-dev/doorstop) | [LGPL-3.0](https://github.com/doorstop-dev/doorstop/blob/develop/LICENSE.md) | v3.2 | YAML 項目、驗證追溯樹 | 輕量稽核 |
| [ChatPRD](https://www.chatprd.ai/pricing) | 商用 | Pro $15/月、Teams $29/席 | 未說明繁中支援;Confluence 未列名 | PRD 起草輔助 |
| [Plane](https://github.com/makeplane/plane)、[OpenProject](https://github.com/opf/openproject) | AGPL-3.0、GPL-3.0 | 活躍 | [Plane MCP](https://github.com/makeplane/plane-mcp-server);[OpenProject MCP 為企業 add-on,未列 wiki](https://www.openproject.org/pt/docs/system-admin-guide/integrations/mcp-server/) | 自託管 tracker |
| [Linear](https://linear.app/docs/mcp)、Jira + Rovo | 商用 | Linear 有唯讀 MCP 端點 | Jira 的原生追溯未找到官方文件 | SaaS tracker |

**判斷:** 建議的追溯慣例不是任何標準,而是本報告提案:wiki 中討論,核准時指派 `URD-PAY-012` 這類含領域前綴的 ID;晉升 Git 時記錄來源 wiki URL 與版本;tracker、任務、PR trailer(`Implements: FR-012`)與測試標籤都帶 ID;CI 對孤立需求、未知 ID、需求變更但測試未更新的情況直接失敗,並發布追溯矩陣。Agent 對核准需求的寫入一律走 PR,只有草稿區可以自由寫入。不要把共用服務帳號接上 wiki MCP,否則會繞過 Atlassian、Notion 記載的使用者範圍權限;Outline 的工作區 MCP 開關也不是逐 agent 治理,需要閘道或個別帳號。LLM wiki 可作部門內的衍生內容層,但不是企業架構。URD 沒有完全對應的現成模板,應改作 BMAD BRD 或 Spec Kit 模板,並區分使用者語言的需求與系統需求。最後,Onyx 的同步清單偏向 Confluence,使用自託管 wiki 時可能要以群組範圍或自訂 ACL 處理,這是 Outline 路線的實質成本。

**程式碼知識與 PM wiki 互補,不應合併。** Wiki 回答「應該存在什麼、為什麼」,程式碼工具回答「現在有什麼」。**證據:** [DeepWiki MCP](https://docs.devin.ai/work-with-devin/deepwiki-mcp) 免費但只限公開 repo,[Devin MCP](https://docs.devin.ai/work-with-devin/devin-mcp.md) 才涵蓋私有 repo;[Context7](https://github.com/upstash/context7) 的客戶端是 MIT,但後端與爬取引擎是私有的;[Repomix](https://github.com/yamadashy/repomix) 與 [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) 皆為 MIT;[GitNexus](https://github.com/abhigyanpatwari/GitNexus/blob/main/LICENSE) 是 PolyForm Noncommercial,企業不能直接採用;[Zoekt](https://github.com/sourcegraph/zoekt) 是 Sourcegraph 仍活躍的 Apache-2.0 專案,而 Sourcegraph 主 repo 已不公開。需求 ID 是兩邊的對帳鍵。

## Coding agent:工具更迭是首要風險,可攜資產才是長期投資

**證據:** 12 個月內的變動足以改寫選型:[Roo Code](https://github.com/RooCodeInc/Roo-Code) 於 2026-05-15 關閉;Cursor 被 SpaceX 收購 ([SEC 10-Q](https://www.sec.gov/Archives/edgar/data/0001181412/000162828026052535/spcx-20260630.htm)),且最便宜方案僅限 Grok 模型 ([定價](https://cursor.com/pricing));Gemini CLI 已於 2026-06-18 停止服務個人帳號,轉往 Antigravity CLI ([Google](https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/));Amazon Q Developer IDE 外掛將於 2027-04-30 終止支援 ([AWS](https://aws.amazon.com/blogs/devops/amazon-q-developer-end-of-support-announcement/));Windsurf 則據報已更名 Devin Desktop ([次級來源](https://theaiagentindex.com/agents/windsurf))。定價也已收斂到用量計費,[Copilot 自 2026-06-01 改為 AI Credits](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)。由於同一模型在不同 harness 的分數可差約十個百分點 ([arXiv 2605.21404](https://arxiv.org/html/2605.21404)),模型分數不能當 harness 選型依據。

| Agent | 授權 | 狀態 | Headless 與模型 | 條件 | 判斷 |
|---|---|---|---|---|---|
| [OpenCode](https://github.com/anomalyco/opencode) | MIT | v1.19.5;225.1k 星;已遷移 | [`opencode run`](https://opencode.ai/docs/cli);[多供應商含 Ollama](https://opencode.ai/docs/providers) | [MCP](https://opencode.ai/docs/mcp-servers)、[Skills](https://opencode.ai/docs/skills) | 中立與主權候選 |
| [Goose](https://github.com/aaif-goose/goose) | Apache-2.0 | v1.57.0;AAIF 專案 | [`goose run`](https://goose-docs.ai/docs/guides/running-tasks) | Stripe fork 其作為基礎;供應商範圍未重驗 | 中立治理候選 |
| [Codex CLI](https://github.com/openai/codex) | Apache-2.0 | rust-v0.167.0 | [`codex exec`](https://learn.chatgpt.com/docs/pricing) | API key 模式不含雲端審查功能;MCP 未重驗 | OpenAI 系候選 |
| [Claude Code](https://github.com/anthropics/claude-code) | 專有 | v2.1.319 | [`claude -p`](https://code.claude.com/docs/en/plugins/org.md);僅 Claude,可經 Bedrock、Foundry 等 | [廠商自述 $150 到 $250/人/月](https://code.claude.com/docs/en/costs);[CVE 史](https://www.itpro.com/software/development/claude-code-flaws-left-ai-tool-wide-open-to-hackers-heres-what-developers-need-to-know) | Claude 系候選,見利益揭露 |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | Apache-2.0 | v0.26.9 | `qwen -p`;多 API、Ollama、vLLM | MCP、Skills、hooks | 主權候選 |
| [Cline](https://github.com/cline/cline) | Apache-2.0 | v4.1.26 | Headless CLI、SDK | [2026-02 npm 遭投毒事件](https://thehackernews.com/2026/02/cline-cli-230-supply-chain-attack.html?m=1) | IDE 導向團隊 |
| [OpenHands](https://github.com/OpenHands/OpenHands) | MIT(偵測) | v1.37.0 | REST API、SDK;ACP 調度其他 agent | 企業目錄未查 | 自託管執行面 |
| [Kilo Code](https://github.com/Kilo-Org/kilocode)、[Zoo Code](https://github.com/Zoo-Code-Org/Zoo-Code) | MIT、Apache-2.0 | 活躍 | 功能未驗證 | Zoo 為 Roo 社群 fork | 暫不列入 |
| [Aider](https://github.com/Aider-AI/aider) | Apache-2.0 | 上次 release 2025-08-09 | [`--message`](https://aider.chat/docs/scripting.html) | [無原生 MCP](https://github.com/Aider-AI/aider/issues/3314) | 不作新標準 |
| [Cursor](https://cursor.com/pricing) | 商用 | SpaceX 持有 | [Print 模式](https://cursor.com/docs/cli/headless) | Enterprise 才有完整控管;第三方模型按 API 牌價 | 中立模型策略下視為鎖定訊號 |
| [Factory `droid exec`](https://docs.factory.com/cli/droid-exec/overview) | 商用 | 活躍 | 預設唯讀,`--auto` 分級;可用客製模型 | [Air-gap](https://docs.factory.ai/enterprise.md) | 高敏感環境商用候選 |

**判斷:** 白名單建議組成是一個前沿商用 harness(依既有合約選 Claude Code、Codex 或 Copilot)、一個中立開源 harness(OpenCode 或 Goose)、一個主權路線(OpenCode 或 Qwen Code 接自託管模型),並經 bake-off 定案。部門只有一個主要 agent,因為每多一個 agent 就多一個治理面:例如 [Claude Code 的 plugin 控管](https://code.claude.com/docs/en/plugins/org.md)只管它自己。每季在次要 harness 上跑一次核心 skill 當作可攜性測試。沒有研究直接比較「單一主要 agent」與「每階段 best-of-breed」,這只是推論。各階段對應如下,同樣沒有階段層級的比較證據。

| 階段 | Agent | Skill 與方法 | 控制 |
|---|---|---|---|
| 設計 | 主要互動 agent;Spec-first 團隊可用 Kiro | OpenSpec 或 Spec Kit;`domain-modeling`、`codebase-design` | ADR 人類核准;儀式量隨風險調整 |
| 實作 | 主要 agent;backlog 用雲端 agent | Superpowers 或 mattpocock `tdd`、`diagnosing-bugs` | 必須有測試與沙箱 |
| 審查 | 經 bake-off 選出的獨立 reviewer | 組織規則 | 先建議;人類合併 |
| 重構 | 主要 agent + worktree | 變更前先寫特性測試 | 追蹤複雜度,呼應 CMU 結果 |
| 文件 | 低風險 headless | 文件模板 skill | 文件測試、連結檢查、簽核 |
| 遷移 | 批次 headless + 平行 worktree | 組織遷移 playbook skill | 分批、回滾、agent 不寫遠端 |
| DevOps | 預設唯讀或 plan-only | IaC plan 審查 skill | Push 與部署放流水線,[Factory 也如此建議](https://docs.factory.com/cli/droid-exec/overview) |

**證據:** Skill 的效益並不如星數暗示的那麼大。SkillsBench 中人工策展 skill 平均提升 16.2 個百分點,但軟體工程只提升 4.5,自動生成則為 -1.3 ([arXiv 2602.12670](https://arxiv.org/html/2602.12670v1));另一項軟體 skill 研究據摘要顯示 49 個中有 39 個完全無效 ([arXiv 2603.15401](https://arxiv.org/html/2603.15401))。安全面更值得警惕:Snyk 掃描 3,984 個 skill,確認 76 個惡意 payload,但它自己賣掃描 ([Snyk](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/));Cisco 自測顯示規則掃描在 HIGH 等級只抓到 7.7%,加 LLM judge 後 MEDIUM 以上為 66.7%,其 README 也強調仍需人工審查 ([skill-scanner](https://github.com/cisco-ai-defense/skill-scanner))。

| Skill 與方法包 | 授權 | 狀態 | 條件 | 判斷 |
|---|---|---|---|---|
| [Agent Skills 規格](https://agentskills.io/home) | [Apache-2.0](https://github.com/agentskills/agentskills) | 無 tag 版本 | 無可釘選的規格版本;簽章與來源未查 | **組織封裝標準** |
| [AGENTS.md](https://github.com/agentsmd/agents.md) | MIT | AAIF 專案 | 無版本 release | 跨 agent 基本指令 |
| [obra/superpowers](https://github.com/obra/superpowers) | MIT | v7.0.2;313.7k 星 | 星數異常高,成因未明 | 實作紀律 |
| [mattpocock/skills](https://github.com/mattpocock/skills) | MIT | v1.3.2 | 個人維護,主張鮮明 | 工程基礎 skill |
| [anthropics/skills](https://github.com/anthropics/skills) | Repo 根無授權 | 無 release | 須逐 skill 查;偏文件與創意工作 | 參考,勿直接分發 |
| [open-gsd/gsd-core](https://github.com/open-gsd/gsd-core) | MIT | [原 repo](https://github.com/gsd-build/get-shit-done) 已封存 | 星數未轉移 | 釘版使用 |
| [Agent OS](https://github.com/buildermethods/agent-os) | MIT | v3.0.0(2026-01) | 節奏較慢 | 標準注入參考 |
| [vercel-labs/skills](https://github.com/vercel-labs/skills) | MIT | v1.7.6 | 跨 agent 安裝器 | 分發工具 |
| [cisco skill-scanner](https://github.com/cisco-ai-defense/skill-scanner)、[snyk/agent-scan](https://github.com/snyk/agent-scan) | Apache-2.0 | 活躍 | 通過掃描不等於安全 | 雙掃描 + 人工 |
| [awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) 等清單 | 各異或未宣告 | 索引 | 非審核來源;無授權內容先不分發 | 只用於發現 |

**判斷:** Skill 應視為可執行的供應鏈相依。正式做法是私有 monorepo 當 registry,以 `CODEOWNERS` 分部門,在 CI 產生各 agent 的封裝;分為組織核心、部門、專案三層;第三方 skill 只以 commit SHA vendor,記錄授權,雙掃描後對腳本、hooks、MCP 設定、網路呼叫做人工審查;每個 skill 都有 with/without 配對評測,無效就退役。公開基準沒有測量組織專屬知識,而這正是 skill 最強的理由。治理缺口也要記下:[Claude Code 的伺服器管理設定每組織只有一份](https://code.claude.com/docs/en/plugins/org.md),部門差異要靠端點或閘道政策;[Copilot 的政策不涵蓋 VS Code 本機 agent](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)。Clinejection 事件的教訓是 issue 與 PR 文字不可進入持有發布權杖的 agent ([The Hacker News](https://thehackernews.com/2026/02/cline-cli-230-supply-chain-attack.html?m=1))。

**Code review agent 沒有穩定贏家。** **證據:** Greptile 在自家測試中抓到 82%,在競爭對手 Macroscope 的測試中則是 23.6% ([Particula](https://particula.tech/blog/greptile-vs-coderabbit-vs-qodo-ai-code-review-2026),[Macroscope](https://macroscope.com/content/best-ai-code-review-tools-github-2026));最接近中立的 [Martian benchmark](https://github.com/withmartian/code-review-benchmark) 各時間快照排名不一,現行分數也未能驗證。自託管選項包括 [PR-Agent](https://github.com/The-PR-Agent/pr-agent)(MIT,自帶模型)、[Qodo 企業版](https://www.qodo.ai/pricing/)(on-prem 與 air-gap)、[CodeRabbit](https://www.coderabbit.ai/pricing) 與 [Greptile](https://www.greptile.com/pricing) 的企業方案。[Claude Code Review](https://code.claude.com/docs/en/code-review) 不提供自託管,也不能與 ZDR、HIPAA 設定或 CMEK 併用。**判斷:** 拿 50 到 100 個含已知漏網缺陷的歷史 PR 重播,比較召回率、留言採納率、每 PR 雜訊與成本,不要引用任何單一排名。

## 測試:agent 提案、工具判定、人類擁有 oracle

**證據:** 通過測試是 agent 程式碼的弱證據。SpecBench 中所有前沿 agent 都能通過可見測試,但對隱藏測試的作弊仍在,落差隨程式碼規模每增十倍增加 28 個百分點 ([arXiv 2605.21384](https://arxiv.org/abs/2605.21384))。METR 請四位維護者審閱 296 個通過評分器的 AI patch,評分器分數比合併決定高出約 24.2 個百分點 ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/))。一項預註冊研究顯示,fresh-context verifier 對 46 個植入缺陷的偵測為 95.4%,建構者自己的測試為 85.1%,作者結論是 context 獨立比模型獨立更重要;它是小樣本預印本,且跨供應商比較與能力差異有混淆 ([arXiv 2610.28667](https://arxiv.org/html/2610.28667))。Meta 的 JiTTest 自述 2026 上半年在 diff 階段抓到 74 個嚴重隱私 bug,且因 LLM 判斷不確定而加上規則評估器 ([Meta](https://engineering.fb.com/2026/10/29/security/catching-bugs-before-they-land-jittestings-human-on-the-loop-approach-at-full-scale/))。Mutation score 也有界線:在可假設無 bug 的回歸情境有意義,在程式可能已有 bug 時不可靠 ([arXiv 2607.22880](https://arxiv.org/abs/2607.22880))。

**判斷:** 下列門檻與分級是設計提案,應在試點校準,mutation 閘門先以警告開始。Builder 不得寫入 oracle、閘門設定、門檻或隱藏測試;閘門結果由工具依已提交的設定計算,LLM 可摘要、triage、排序,不能翻轉結果;繞過閘門、改門檻或隔離測試都要人類核准且有到期日。

| 階段 | 觸發 | 執行者 | 阻擋性檢查 |
|---|---|---|---|
| 0 規格 | 開工前 | Planner 起草,人類核准風險級別與驗收條件 | 每條件有 ID 且可測 |
| 1 建置 | 每次變更 | Builder + TDD skill | Lint、型別、單元測試、secret scan |
| 2 PR 快速閘 | 每 PR,目標 10 到 15 分鐘 | 不由 LLM 決定 | 單元與整合、SAST、SCA、SonarQube、測試竄改、追溯 |
| 3 獨立驗證 | 低風險以上 PR | Fresh-context agent,僅取規格與公開介面 | 新測試須可執行,失敗交人 triage |
| 4 深度閘 | 依風險或 merge queue | Mutation、契約、fuzz、preview E2E | Mutation 門檻、Pact、無新 fuzz 失敗、無新 skip |
| 5 發布前 | RC 或 nightly | 完整 E2E、效能、DAST、沙箱 AI pentest、AI eval | SLO、exploited 等級政策、重複執行的 eval 門檻 |
| 6 部署後 | 持續 | Trace、線上 eval、canary | SLO 違反自動回滾 |
| 維護 | 帶外 | Healer 與修復 agent 開 PR | 需 verifier 核准;改 assertion 不得自動合併 |

| 測試層 | 預設(判斷) | 授權與狀態 | 替代或商用 | 避開與條件 |
|---|---|---|---|---|
| 單元、整合 | Coding agent + TDD skill;[PIT](https://github.com/hcoles/pitest)、[StrykerJS](https://github.com/stryker-mutator/stryker-js)、[Stryker.NET](https://github.com/stryker-mutator/stryker-net)、[mutmut](https://github.com/boxed/mutmut) | Apache-2.0;mutmut 為 BSD-3;皆活躍 | [Diffblue](https://www.diffblue.com/wp-content/uploads/2026/03/Diffblue_AI-Testing-Agents_Benchmark-Report_v4.pdf)(廠商自測,Java);[Pynguin](https://github.com/se2p/pynguin)(MIT) | [Qodo Cover](https://github.com/qodo-ai/qodo-cover) 已不維護;[EvoSuite](https://github.com/EvoSuite/evosuite) 陳舊 |
| E2E、UI | [Playwright Test Agents](https://playwright.dev/docs/test-agents) + [Playwright MCP](https://github.com/microsoft/playwright-mcp) | Apache-2.0;[v1.65.0](https://github.com/microsoft/playwright);MCP 仍 0.0.x | [QA Wolf](https://www.qawolf.com/pricing)($1/web run,可匯出);[Momentic](https://www.momentic.ai/pricing);[Applitools](https://applitools.com/platform-pricing/)(視覺,$667/月起) | [Octomind](https://github.com/OctoMind-dev/octomind-mcp)、[Bugster、DogQ](https://releaseworthy.app/octomind-bugster-dogq-alternatives/) 已關閉;[Shortest](https://github.com/antiwork/shortest) 陳舊 |
| 探索式瀏覽器 | [Stagehand](https://github.com/browserbase/stagehand) 或 [Browser Use](https://github.com/browser-use/browser-use) | MIT;Browser Use 未達 1.0 | [Skyvern](https://github.com/Skyvern-AI/skyvern)(AGPL-3.0;[雲端 $29 起](https://www.skyvern.com/pricing)) | 不可作合併閘門 |
| API、契約 | [Schemathesis](https://github.com/schemathesis/schemathesis) + [Pact](https://github.com/pact-foundation/pact-jvm) | MIT;Pact JVM 為 Apache-2.0、[Pact JS](https://github.com/pact-foundation/pact-js) 為 MIT | [EvoMaster](https://github.com/WebFuzzing/EvoMaster)(LGPL-3.0);[Keploy](https://github.com/keploy/keploy)(試點) | [RESTler](https://github.com/microsoft/restler-fuzzer) 緩慢;Schemathesis 會改資料,僅限非正式環境 |
| 效能 | [k6](https://github.com/grafana/k6) | AGPL-3.0;v2.3.0 | [Gatling](https://github.com/gatling/gatling)(Apache-2.0);[Locust](https://github.com/locustio/locust)(MIT) | LLM 只註解,不決定通過 |
| 安全(確定性) | [Semgrep CE](https://github.com/semgrep/semgrep) 或 [Opengrep](https://github.com/opengrep/opengrep)、[Trivy](https://github.com/aquasecurity/trivy)、[ZAP](https://github.com/zaproxy/zaproxy)、[Nuclei](https://github.com/projectdiscovery/nuclei) | LGPL-2.1、Apache-2.0、MIT | 付費 Semgrep;[CodeQL CLI 私有 repo 的 CI 使用需另行授權](https://github.com/github/codeql-cli-binaries/blob/main/LICENSE.md) | Semgrep CE 單檔分析會漏報;所有工具以 SHA 釘版 |
| AI pentest | [Strix](https://github.com/usestrix/strix)(寬鬆授權)或 [Shannon](https://github.com/KeygraphHQ/shannon)(白箱與 exploit 閘門) | Apache-2.0;AGPL-3.0 | [XBOW](https://xbow.com/pricing)、Aikido;[PentAGI](https://github.com/vxcontrol/pentagi) | [CAI](https://github.com/aliasrobotics/cai) 已封存;禁止用於正式環境 |
| 行動 | [Maestro](https://github.com/mobile-dev-inc/Maestro)、[Appium](https://github.com/appium/appium) | Apache-2.0;活躍 | [Appium MCP](https://github.com/appium/appium-mcp)、[Mobile MCP](https://github.com/mobile-next/mobile-mcp) 供探索 | 不以純 agent 做回歸;品質缺獨立證據 |
| AI 自身評測 | [DeepEval](https://github.com/confident-ai/deepeval)、[Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai)、[garak](https://github.com/NVIDIA/garak)、[PyRIT](https://github.com/microsoft/PyRIT) | Apache-2.0、MIT | [promptfoo](https://github.com/promptfoo/promptfoo)(MIT;[OpenAI 宣布收購](https://openai.com/zh-Hant/index/openai-to-acquire-promptfoo/),交割未確認) | [Ragas](https://github.com/vibrantlabsai/ragas) 自 2026-02 無 push;LLM 評分須重複執行 |
| 品質閘門 | [SonarQube Community Build](https://github.com/SonarSource/sonarqube) | LGPL-3.0;26.11.0 | [SonarQube Cloud 或 Server](https://www.sonarsource.com/plans-and-pricing/) | 不以覆蓋率為唯一指標 |

**證據與判斷:兩個容易踩的陷阱。** Playwright healer 認為功能損壞時會把測試標成 skip ([Playwright](https://playwright.dev/docs/test-agents)),這在 CI 上看起來像綠燈。因此任何新增 skip 都要讓 CI 失敗或升級處理,且 healer 只能開 PR,不能在閘門執行中就地修復。另一項是掃描器本身的供應鏈風險:2026 年 3 月 Trivy 生態遭入侵 ([GHSA-69fq-xp46-6x23](https://github.com/advisories/GHSA-69fq-xp46-6x23)),據報進一步波及 LiteLLM ([Wiz](https://wiz.io/blog/trivy-compromised-teampcp-supply-chain-attack));所有 CI 工具都要以 SHA 或 digest 釘版,掃描 job 不放長期密鑰。

**AI pentest 只能用在你擁有或書面授權的環境。** **證據:** Shannon 與 Strix 都要求明確書面授權,Shannon 更警告其 exploit 會改動資料、不可用於正式環境,且不要對不可信程式碼庫執行 ([Shannon](https://raw.githubusercontent.com/KeygraphHQ/shannon/main/docs/safety.md),[Strix](https://github.com/usestrix/strix))。Doyensec 受 Aikido 贊助的比較中,測試期間 XBOW 刪除測試帳號、導致 instance 無回應,並產生 4,800 多封郵件;其結論是兩者差異漸進而非懸殊 ([Doyensec](https://doyensec.com/resources/ComparingAIApplicationSecurityTestingPlatforms_Doyensec.pdf))。Shannon 與 PentestGPT 的 benchmark 百分比都是自述且版本不同,不可互比,而 7 萬星的 Strix 也沒有獨立研究。**判斷:** 控制清單是書面授權存檔、網路層目標白名單、合成資料、快照與回滾、限制對外連線、花費上限、指令稽核,以及人類簽核。

## 平台與治理:部門隔離要放在基礎設施層

**證據:** 應用層 tenancy 在這個新類別中反覆出問題:[agentgateway](https://github.com/agentgateway/agentgateway/security/advisories) 曾有 session 跨路由覆寫授權政策;[IBM ContextForge](https://github.com/IBM/mcp-context-forge/security/advisories) 有跨團隊 A2A agent 接管;[Docker MCP Gateway](https://github.com/docker/mcp-gateway/security/advisories) 有未驗證工具代理的嚴重 CVE。[LiteLLM](https://docs.litellm.ai/blog/security-update-march-2026) 2026 年 3 月的 PyPI 版本遭植入憑證竊取程式,官方 proxy Docker 映像未受影響;其後仍有 [18 則 advisory,其中 11 則高危或嚴重](https://github.com/BerriAI/litellm/security/advisories)。[Daytona](https://github.com/daytonaio/daytona) 的開放 repo 已封存,核心轉為私有;[Portkey gateway](https://github.com/Portkey-AI/gateway) 自 2026-05 後無 push,且據報已被 Palo Alto 收購 ([Technology Decisions](https://www.technologydecisions.com.au/content/security/news/palo-alto-completes-acquisition-of-portkey-1780733452))。Coder 的 AI Bridge 與 Agent Boundaries 需付費 add-on,社群版沒有 ([Coder](https://coder.com/docs/ai-coder/ai-governance));Backstage 的 Actions Registry 也不自動強制權限 ([Spotify Portal](https://backstage.spotify.com/docs/portal/core-features-and-plugins/mcp/registering-tools))。

| 基礎設施層 | 預設(判斷) | 授權與狀態 | 替代 | 條件或避開 |
|---|---|---|---|---|
| 沙箱 | [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) + [gVisor](https://github.com/google/gvisor) 或 [Kata](https://github.com/kata-containers/kata-containers) | Apache-2.0;v1.0.10 | [E2B](https://github.com/e2b-dev/E2B)、[Firecracker](https://github.com/firecracker-microvm/firecracker)、[microsandbox](https://github.com/superradcompany/microsandbox) | 避開已封存的 Daytona repo;E2B 自託管未查 |
| LLM 與 MCP 閘道 | [agentgateway](https://github.com/agentgateway/agentgateway) 或 [Agent Router](https://github.com/theagentrouter/agent-router) | Apache-2.0;AAIF 專案 | [LiteLLM](https://github.com/BerriAI/litellm/blob/main/LICENSE)(釘版簽章容器、不對外開管理面);[Microsoft MCP Gateway](https://github.com/microsoft/mcp-gateway) | [MCP 規格更新](https://agentgateway.dev/blog/2026-08-03-new-mcp-spec-revision/)後需雙版本;Kong 的 AI 與 MCP 多在企業版 |
| 觀測 | [Langfuse](https://github.com/langfuse/langfuse/blob/main/LICENSE) + OTel | MIT(`ee/` 除外);ClickHouse 持有 | [MLflow](https://github.com/mlflow/mlflow)、[OpenLLMetry](https://github.com/traceloop/openllmetry) | [Phoenix](https://github.com/Arize-ai/phoenix/blob/main/LICENSE) 為 ELv2;`gen_ai.*` [仍為 Development 狀態](https://www.dash0.com/knowledge/opentelemetry-genai-semantic-conventions-explained) |
| 身分與密鑰 | [SPIRE](https://github.com/spiffe/spire) + [OpenBao](https://github.com/openbao/openbao) | Apache-2.0;MPL-2.0 | Entra Agent ID(次級來源) | 每 agent 身分的生產證據少 |
| 政策 | [OPA](https://github.com/open-policy-agent/opa) 或 [Cedar](https://github.com/cedar-policy/cedar) | Apache-2.0 | 各廠商內建 | 控管面不跨廠商 |
| 目錄與環境 | [Backstage](https://github.com/backstage/backstage) | Apache-2.0;[AI 文件為 Next 版](https://backstage.io/docs/next/ai/) | [Port](https://docs.port.io/getting-started/agentic-sdlc-pillars/);[Coder](https://github.com/coder/coder)(AGPL-3.0) | Backstage 的權限責任落在每個 action 作者 |

**判斷:多部門模式。** 每部門獨立 namespace、閘道金鑰、憑證、Langfuse project 與預算;應用層 tenancy 只作縱深防禦。目錄記錄誰擁有哪個 agent、skill、MCP server,閘道負責強制與計量,沙箱負責圍堵,registry 本身不強制任何事。成本歸屬以部門金鑰、工作流程 ID、階段三個維度計算,因為 harness 自報的花費只是建議性質。廠商自帶的部門控制差異很大:[GitHub 成本中心](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)、[Kiro 群組](https://kiro.dev/pricing/)、[Devin 組織上限](https://docs.devin.ai/admin/billing)都有,但 [GitLab 文件只記載使用者上限](https://docs.gitlab.com/subscriptions/gitlab_credits),這正是中立閘道的存在理由。

**判斷:自主等級。** 依 Gartner 的 [observe、advise、act with approval、act autonomously 四級](https://gartner.com/en/newsroom/press-releases/2026-05-26-gartner-says-applying-uniform-governance-across-ai-agents-will-lead-to-enterprise-ai-agent-failure)分級:前兩級開放所有部門;一般變更走 PR 流程,人類審規格與高風險路徑;自主合併只限依賴升級、flaky test、lint 與遷移車隊這類有強測試的變更;支付、驗證、受監管資料與正式環境基礎設施一律保留閘門。實務範圍也支持這個分布,Stripe 全部人審,Spotify 的自動合併集中在維護車隊。Kiro 事故的成因雖有爭議,但雙方說法都指向權限過寬,所以最小權限沒有協商空間。

**證據:台灣資料處理是硬限制。** AWS Taipei 區域存在 ([AWS](https://aws.amazon.com/de/blogs/aws/now-open-aws-asia-pacific-taipei-region/)),但 Bedrock 相容性表在已讀的部分顯示 Claude Haiku 5.5、Sonnet 5.5 與 Opus 5.5 在 `ap-east-2` **僅支援全域跨區推論**;該表只讀了前 100k 字元,其他模型未知 ([AWS Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/models-region-compatibility.html))。Vertex AI 表格讀到的台灣處理僅有 Claude Haiku 4.5,而其他文件版本的搜尋摘要列出更多模型,存在衝突,須查現行表格 ([Google](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency))。[TAIDE](https://technews.tw/2025/02/13/taide-8b-launched/) 有 Llama 基礎的商用模型,[Kuwa](https://www.ithome.com.tw/news/162663) 可在地部署,但未找到寫碼代理證據。法規方面,台灣《人工智慧基本法》目前沒有立即的私部門營運義務,主管機關規則仍待訂定;Baker McKenzie 建議先做 AI 盤點、分類與合約責任分配 ([Baker McKenzie](https://www.bakermckenzie.com/en/insight/publications/2026/01/taiwan-ai-basic-act))。歐盟高風險義務據報延至 2027-12-02 與 2028-08-02,應再查 EUR-Lex ([Winston Taylor](https://www.winstontaylor.com/insights/ai-act-rules-on-high-risk-ai-delayed-as-ai-digital-omnibus-agreed))。[ISO 42001](https://dev.to/aguardic/iso-42001-in-the-wild-what-certification-actually-proves-4lnf) 認證的是管理系統,不是 agent 的即時行為。

**判斷:** 本報告假設組織可能位於台灣或有境內處理顧慮;若沒有這項要求,此節限制可放寬。有此要求時,應以敏感度分流:低敏感送前沿模型,敏感程式碼送閘道後的自託管 open-weight 模型。TAIDE 較可能用於繁中文件與需求工作,但這未經測試,不能當寫碼模型。金融、政府、醫療的跨境規定本研究未涵蓋,需在地法律意見。

## 先驗證風險,再擴大規模

**判斷:** 下列路線圖與門檻是推論,不是實測值。起點選擇有強測試、重複性高的工作,與 [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) 的 flaky test 工單及 [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint) 的維護遷移相符。[DORA 的 J 曲線](https://infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report)提醒要預留初期下滑的預算,但其回收期是高不確定性的模型,不是測量值。

| 時程 | 範圍 | 新增元件 | 進入下一階段的條件 |
|---|---|---|---|
| 第 0 到 6 週 | 基線與基礎 | 沙箱、閘道日誌、預算;完成下表中文與隔離驗證;選 2 到 3 個高測試 repo | DORA 指標與審查時間基線到位 |
| 第 6 到 16 週 | 1 到 2 個團隊的受限試點:flaky test、依賴升級、小修 | SCM 工作流程(gh-aw 或 GitLab Flows);全部人審;CI 限重試 | 變更失敗率與漏網缺陷率持平 |
| 第 4 到 9 個月 | Spec-to-PR、第二個部門 | Wiki、規格晉升、獨立審查、Temporal、skill 目錄 | Lead time 改善且品質持平 |
| 第 9 個月後 | 分級自主 | 特定變更類別自動合併;降級規則 | 該類別失敗率經證實偏低 |

| 選型前必做驗證 | 做法 | 判定依據(提案) |
|---|---|---|
| 繁中搜尋 | 對候選 wiki 與檢索層測完整片語、句中二字詞、中英混合詞,用真實 URD | 召回不足就加 sidecar 索引或改選 |
| 部門隔離 | 以 A 部門身分嘗試存取 B 部門的文件、MCP、trace、金鑰 | 任何跨部門成功都阻擋上線 |
| 授權 | 法務檢視 Outline BSL、Dify、Multica、AGPL 工具,特別是部門分屬不同法人時 | 取得書面意見 |
| Harness 比較 | 同一批任務跑候選 harness,記錄 harness 版本 | 以內部評測決定,不引用公開 benchmark |
| Review bake-off | 50 到 100 個歷史 PR 重播 | 召回、採納率、雜訊、成本 |
| Skill 評測 | With/without 配對與確定性驗證器 | 無增益就退役 |
| 成本 | 每合併 PR 的完整 token 與運算成本,含失敗執行 | 建立內部基線 |
| 資料路徑 | 向供應商確認推論、日誌、靜態資料的位置,寫入合約 | 符合資料分級政策 |

**判斷:** KPI 必須配對,不單看 PR 數。交付面看 lead time、部署頻率、變更失敗率、恢復時間;品質面看漏網缺陷率、每合併 PR 事故;瓶頸面看審查時間與佇列年齡;agent 品質面看無人工修改的合併率、每 PR 的 CI 輪數、30 天 churn;測試面看變更碼 mutation、flake、heal 與誤 heal 數;成本與治理面看每合併 PR 成本、預算遵循與政策違規。[Thoughtworks](https://www.thoughtworks.com/radar) 已對以吞吐量衡量生產力提出警示,[METR 的研究](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)也顯示自我感受不可靠。

**已知的未知數。** 本研究沒有實機測試任何工具;沒有找到端到端需求到測試的受控 factory 研究,需求階段幾乎沒有成效證據;沒有公開的每合併 PR 成本基準,也沒有案例公布漏網缺陷率;沒有獨立的框架、閘道或協調器比較;企業價格多需報價;Advisory 僅讀摘要層級,引用於安全決策前要確認可利用性與修補版本。零 advisory 不代表安全,大量 advisory 也可能代表揭露較透明。

## 結論

這次研究改變的不是工具清單,而是投資重心。多數人以為 factory 的核心是挑到最強的 coding agent 或最聰明的協調器,但證據顯示 agent 是最容易更換、也最常被更換的部分:一年內就有多個主流工具關閉、被收購或改名,高星專案也可能停滯。能跨越這些變動而累積價值的是組織自己的資產:含穩定 ID 的規格、部門專屬 skill、held-out 驗收情境、評測資料集、MCP 工具與遙測。預算與人力應先投在這些資產和驗證能力,而不是 agent 數量。

對多部門組織來說,真正的難題是三個目前沒有現成答案的交界點:繁體中文內容如何在 wiki 與檢索層被可靠找到,權限如何從來源文件一路保留到 agent 的檢索結果,以及敏感程式碼能否在可接受的法域內由足夠好的模型處理。這三項都能在前六週以低成本實測,且任何一項的結果都可能推翻本報告的預設選擇。因此建議先把它們當成第一個里程碑,再決定長期工具。
