# PM 需求分析類 Repo 比較

查核日期：2026-11-10。星數、授權與最後推送日期取自 GitHub API；功能描述取自各專案 README，**沒有安裝或實測**。本文由 Anthropic 的模型撰寫，多數專案預設搭配 Claude Code，篇幅不代表排名。

## 結論

PM 這一段的開源專案比工廠整體多，但分成三種，**沒有一個同時做到「知識庫、URD 產出、需求追溯」**：

1. **PM skill 套件**：最多也最成熟，把需求訪談、PRD、使用者故事寫成可重複使用的 skill。適合當 URD 助理的內容來源。
2. **需求追溯工具**：把需求 ID 連到程式碼、測試與證據。這正是 URD 交給後續 agent 時最需要、也最常缺的一段。
3. **PM 應用程式**：有介面的需求訪談或 PRD 工具，但多數很新、星數少，只適合參考設計。

建議的組合（我的判斷）：

| 用途 | 建議參考 | 理由 |
|---|---|---|
| URD 訪談與撰寫 skill | [product-on-purpose/pm-skills](https://github.com/product-on-purpose/pm-skills) | Apache-2.0、68 個 skill 涵蓋完整生命週期、有範例輸出與 CI 檢查 |
| 需求追溯與核准 | [Agile-V/agile_v_skills](https://github.com/Agile-V/agile_v_skills) | 型別化追溯、獨立驗證、人類核准點；授權為 CC-BY-SA-4.0 要確認 |
| 規格與程式碼偏移檢查 | [mori-shin-x/artgraph](https://github.com/mori-shin-x/artgraph) | 以需求 ID 連結規格、程式碼和測試，不用 LLM，結果可重現 |
| 業務需求與驗收項目 | [mk3008/alder](https://github.com/mk3008/alder) | 以業務語言寫需求並作為唯一正本，連結驗收項目，可把訪談紀錄結構化 |

這些都能和 WeKnora、RAGFlow 或 Onyx 搭配：知識平台負責「查得到」，skill 負責「寫得好」，追溯工具負責「接得上」。

## 一、PM skill 套件

| 專案 | 星數 | 授權 | 最後推送 | 重點 | 注意 |
|---|---|---|---|---|---|
| [product-on-purpose/pm-skills](https://github.com/product-on-purpose/pm-skills) | 793 | Apache-2.0 | 2026-11-03 | 68 個 skill、6 個子 agent、200 多個範例輸出；可由訪談整理接到 PRD；最新 v2.35.0（10-30） | 配套 MCP server 自 2026-05-04 起進入維護模式 |
| [mohitagw15856/pm-claude-skills](https://github.com/mohitagw15856/pm-claude-skills) | 1,536 | MIT | 2026-11-09 | 號稱 1,310 個 skill，含訪談整理、PRD 範本、利害關係人更新；**有繁中 README** | 數量龐大，品質要抽查 |
| [deanpeters/Product-Manager-Skills](https://github.com/deanpeters/Product-Manager-Skills) | 7,937 | **CC BY-NC-SA 4.0** | 2026-10-16 | 77 個 skill，含訪談準備、使用者故事與 Gherkin 驗收條件 | **禁止商業使用**，企業內部採用要先問法務 |
| [franklinxkk/ai-delivery-spec](https://github.com/franklinxkk/ai-delivery-spec) | 36 | MIT | 2026-11-01 | 中文 PM skill：判斷需求價值、追問細節、產出需求與原型 | 簡體中文、領域資料以中國情境為主 |

## 二、需求追溯工具

| 專案 | 星數 | 授權 | 最後推送 | 重點 | 注意 |
|---|---|---|---|---|---|
| [Agile-V/agile_v_skills](https://github.com/Agile-V/agile_v_skills) | 55 | CC-BY-SA-4.0 | 2026-10-05 | 從核准的需求到實作、測試、獨立驗證與人類發布決策；40 種證據 schema、AI 影響來源追溯 | CC-BY-SA 用在程式或 skill 上的影響要問法務 |
| [mori-shin-x/artgraph](https://github.com/mori-shin-x/artgraph) | 14 | MIT | 2026-10-05 | 用 `REQ-001`、`@impl` 等標記連結需求、程式碼與測試，偵測哪一邊變了 | 星數少，屬早期專案 |
| [Looted/kibi](https://github.com/Looted/kibi) | 12 | AGPL-3.0 | 2026-11-09 | 需求編譯與強制層，有 MCP；會先訪談需求存在哪裡（Jira、wiki、規格） | AGPL、星數少 |
| [rtmx-ai/rtmx](https://github.com/rtmx-ai/rtmx) | 36 | Apache-2.0 | 2026-11-09 | 需求追溯矩陣 CLI 與 MCP，Git 中的 CSV 可離線使用 | 團隊即時同步功能是其商用服務 |
| [mk3008/alder](https://github.com/mk3008/alder) | 21 | MIT | 2026-11-09 | 業務設計與驗收項目，由訪談紀錄產生草稿，有檢視器 | 星數少，屬早期專案 |

這一類最值得借鏡的是 **artgraph 的做法**：在規格寫 `REQ-001`，在程式碼標 `@impl REQ-001`，在測試標對應 ID，再由確定性工具檢查。它和我們先前提的 `Implements: FR-012` commit trailer 是同一個思路，可以一起考慮。

## 三、PM 應用程式

| 專案 | 星數 | 授權 | 最後推送 | 重點 | 注意 |
|---|---|---|---|---|---|
| [TomMaSS/PRDforge](https://github.com/TomMaSS/PRDforge) | 9 | 未標示 | 2026-03-29 | PRD 拆成章節存在 PostgreSQL，33 個 MCP 工具；有版本、相依和評論 | 授權不明，七個多月沒更新 |
| [boxdate/ReqPipeline](https://github.com/boxdate/ReqPipeline) | 3 | MIT | 2026-03-26 | USDM、EARS、BDD 格式的需求編輯器，用 **Ollama 本地模型**檢查邏輯矛盾 | 只有一次提交，範例模型是 Qwen |
| [lephus/ba-workspace](https://github.com/lephus/ba-workspace) | 6 | Apache-2.0 | 2026-03-28 | 把訪談和利害關係人對話轉成可追溯的知識圖 | 綁定 Google Gemini |
| [Tomato-love-potato/PMAgent](https://github.com/Tomato-love-potato/PMAgent) | 2 | Apache-2.0 | 2026-06-02 | 像真人 PM 一樣追問並更新需求畫像，自動產生 PRD | 簡體中文、早期 |
| [eabait/product-agents](https://github.com/eabait/product-agents) | 2 | 未標示 | 2026-02-10 | PRD agent，有後端 API 和 Next.js 前端 | 實驗性質 |

這類專案大多只有個位數星數，**不建議直接採用**，但有幾個設計值得參考：

- **PRDforge 的章節化：** URD 很長時，按章節讀寫能大幅減少 token。如果 LLM wiki 平台無法做到，可以借用這個做法。
- **ReqPipeline 的格式約束：** 用 EARS 等固定句型寫需求，再由模型檢查矛盾，可提升 URD 可測試性。
- **PMAgent 的需求畫像：** 訪談時逐項標記哪些資訊已確認，能讓 PM 看到缺口。

## 另一個相關專案

[responsibleai/ASSERT](https://github.com/responsibleai/ASSERT)（361 星，MIT）把自然語言規格轉成可執行的評測。它的對象是 AI 系統本身，但思路可用在驗證 URD 助理和 LLM wiki 的品質。

## 對 PM 流程的建議

1. **知識層：** 照原計畫用 WeKnora、RAGFlow 或 Onyx 做 PoC。
2. **URD 助理：** 從 pm-skills 挑訪談整理、PRD、使用者故事等 skill，改寫成公司的 URD 範本與繁中用語，放進私有 skill repo。
3. **格式：** 每條需求用穩定 ID，驗收條件用固定句型（例如 Gherkin 或 EARS）。
4. **追溯：** 先用 artgraph 的標記思路建立需求、程式碼、測試的連結，再評估是否需要 Agile V 那種完整證據鏈。
5. **授權：** 採用前先排除 CC BY-NC-SA 這類禁止商用的套件，CC-BY-SA 和 AGPL 也要先問法務。

## 限制

- 沒有任何專案實測過，skill 品質要用真實 URD 驗證。
- 沒有找到原生支援 Azure DevOps Boards 的 PM 專案；kibi 會詢問 Jira、YouTrack 等來源，但沒提到 Azure DevOps。
- 沒有找到繁體中文為主的 PM 專案；中文專案以簡體與中國情境為主。
- 多數專案在 2026 年才建立，長期維護情況未知。
