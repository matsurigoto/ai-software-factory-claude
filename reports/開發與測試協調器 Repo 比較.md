# 開發與測試協調器 Repo 比較

查核日期：2026-11-10。範圍只看開發與測試階段的「協調器 + 多 agent」，不含 PM，也不限 Azure DevOps。星數、授權與最後推送日期取自 GitHub API；功能描述取自各專案 README，資安紀錄沿用 2026-11-07 的研究筆記。**沒有安裝或實測任何專案。** 本文由 Anthropic 的模型撰寫，多數專案預設支援 Claude Code，篇幅不代表排名。

## 結論

**這一段有相當多成熟度不一的類似專案，比 PM 和完整工廠都多。** 它們大致分四類，和我們建議的架構對照如下：

| 我們的設計 | 最接近的專案 | 差異 |
|---|---|---|
| 確定性流程當骨幹，agent 是節點 | [Archon](https://github.com/coleam00/Archon)、[zeroshot](https://github.com/the-open-engine/zeroshot) | 用 YAML 或 JSON 定義流程，不是 Temporal |
| Temporal 支撐長時間流程，agent 在隔離容器執行 | [Aixle Flow](https://github.com/AixleHQ/flow) | 設計最接近，但只有 35 星 |
| 工作看板派工給 agent，人類審查 | [Multica](https://github.com/multica-ai/multica)、[OpenHands Agent Canvas](https://github.com/OpenHands/OpenHands)、[Symphony](https://github.com/openai/symphony) | 多數以 GitHub 或 Linear 為工作來源 |
| 寫程式與驗證分開，閘門由程式碼決定 | [addyosmani/factory](https://github.com/addyosmani/factory)、[Gas Town](https://github.com/gastownhall/gastown) 的合併佇列 | 驗證能力依各專案而異 |

如果現在要挑幾個深入評估（我的判斷）：

1. **[Archon](https://github.com/coleam00/Archon)：** 23,866 星、MIT、可自架，SQLite 或 PostgreSQL。用 YAML 寫流程，例如「實作直到測試通過」由指令結果判定，而不是由模型自己說完成。最符合「流程由程式碼控制」的原則。
2. **[OpenHands](https://github.com/OpenHands/OpenHands)：** 93,036 星、MIT，現在定位為自架的 agent 控制中心，可在本機、Docker、VM 或雲端後端執行 OpenHands、Claude Code、Codex、OpenCode 等 agent。社群與文件最成熟。
3. **[Aixle Flow](https://github.com/AixleHQ/flow)：** 架構幾乎就是我們的 L3 加 L4：看板派工、每個 agent 在獨立容器、Temporal 負責長時間流程、團隊看得到每步的 token 與成本。缺點是太新太小，適合參考設計，不適合直接依賴。

## 一、團隊用的協調平台（伺服器端）

| 專案 | 星數 | 授權 | 最後推送 | 重點 | 注意 |
|---|---|---|---|---|---|
| [coleam00/Archon](https://github.com/coleam00/Archon) | 23,866 | MIT | 11-09 | YAML 流程、內建 SDLC 套件從 issue 到合併 PR、自架 | 以 GitHub 為主要工作來源 |
| [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | 93,036 | MIT（核心） | 11-09 | 自架控制中心，支援多種 agent 與後端，有自動化 | Enterprise 功能另有授權，先前查到有試用限制 |
| [multica-ai/multica](https://github.com/multica-ai/multica) | 55,412 | 原始碼可見，GitHub 未判定 | 11-09 | 像指派給同事一樣把 issue 指派給 agent，回報進度與阻礙，可自架 | 先前查到授權在 Apache 上附加條件，內部多工作區可用 |
| [AixleHQ/flow](https://github.com/AixleHQ/flow) | 35 | Apache-2.0 | 11-09 | 團隊看板、容器隔離、**Temporal**、成本可視化 | 2026-08 才建立 |
| [paperclipai/paperclip](https://github.com/paperclipai/paperclip) | 112,546 | MIT | 11-09 | 以「公司」方式管理 agent：組織圖、預算、核准、技能 | **先前查到有嚴重的跨租戶漏洞**；範圍不只軟體開發 |
| [awslabs/cli-agent-orchestrator](https://github.com/awslabs/cli-agent-orchestrator) | 1,532 | Apache-2.0 | 11-09 | 本機伺服器協調 Claude Code、Kiro、Codex 等，有 Web UI 與 MCP | 偏單人或小組使用 |
| [openai/symphony](https://github.com/openai/symphony) | 27,988 | Apache-2.0 | 09-15 | 監看 Linear 看板並派 agent，附 CI、審查與操作影片作為工作證明 | 官方標明是給可信環境的工程預覽 |

## 二、開發與驗證流程引擎

這一類在上一份《GitHub 類似架構參考比較》已詳細比較，這裡只列重點：

| 專案 | 星數 | 授權 | 重點 |
|---|---|---|---|
| [the-open-engine/zeroshot](https://github.com/the-open-engine/zeroshot) | 2,131 | MIT | JSON 定義流程圖，reviewer 判定決定下一步；範例流程要求獨立審查、E2E 和 CI 全過才合併 |
| [Agent-Field/SWE-AF](https://github.com/Agent-Field/SWE-AF) | 1,126 | Apache-2.0 | PM、架構、開發、審查、測試角色，平行 worktree，有 verifier |
| [addyosmani/factory](https://github.com/addyosmani/factory) | 333 | MIT | 確定性閘門、乾淨的 verifier、審查佇列上限 |
| [disler/super-simple-software-factory](https://github.com/disler/super-simple-software-factory) | 1,038 | MIT | Python 控制流程；刻意不設測試 agent，因為跑測試是確定性指令 |
| [cortexapps/engrams](https://github.com/cortexapps/engrams) | 63 | AGPL-3.0 | 自架的 agent microVM 執行層 |

## 三、個人或桌面用的平行 agent 工具

適合工程師個人提高產能，但不是團隊層級的協調器：

| 專案 | 星數 | 授權 | 重點 | 注意 |
|---|---|---|---|---|
| [OrchestratorInc/agent-orchestrator](https://github.com/OrchestratorInc/agent-orchestrator) | 14,344 | Apache-2.0 | 主 agent 規劃，每個工作開一個 worktree 給 worker，依 PR、CI、審查狀態排序 | 桌面與手機 App |
| [ParadevAI/Paradev](https://github.com/ParadevAI/Paradev) | 16,140 | MIT | 每個工作一個 worktree，內建審查、PR 和瀏覽器預覽 | macOS 桌面 App |
| [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) | 11,298 | Apache-2.0 | 統一操作十多種 agent harness，可讓一個 agent 審另一個 | 雲端沙箱選項偏 CoreWeave 等服務 |
| [gastownhall/gastown](https://github.com/gastownhall/gastown) | 18,746 | MIT | 用 git 保存工作狀態；合併佇列先跑驗證閘門再合併 | 概念多，學習成本高 |
| [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 40,712 | MIT | Claude Code 的多 agent 團隊模式 | 綁定 Claude Code |
| [ruvnet/ruflo](https://github.com/ruvnet/ruflo) | 81,330 | MIT | Claude Code 與 Codex 的 swarm 與記憶 | **先前查到 2026-11-05 有高嚴重度安全公告** |

## 四、測試 agent

專門的測試 agent 專案多半很小。較可用的仍是先前報告提過的工具：

| 專案 | 星數 | 授權 | 重點 |
|---|---|---|---|
| [proffesor-for-testing/agentic-qe](https://github.com/proffesor-for-testing/agentic-qe) | 524 | MIT | QA agent 平台，可接 Ollama 本地模型與 Azure OpenAI；效果為自述 |
| [zachblume/autospec](https://github.com/zachblume/autospec) | 68 | MIT | 自動探索網站，產生並執行 Playwright 測試；最後更新 2026-05 |
| Playwright Test Agents | — | Apache-2.0 | planner、generator、healer 三種 agent，仍是我們的 E2E 預設 |

搜尋結果裡還有許多「autonomous QA agent」專案，但幾乎都是 0 到 10 星的課程或比賽作品，不建議參考。

## 值得借鏡的設計

1. **完成由指令決定（Archon）：** `until_bash: "bun run test"`，測試通過才算完成，不讓模型自行宣告。
2. **合併佇列先驗證（Gas Town）：** 完成的工作批次進入合併佇列，跑驗證閘門後才合併。
3. **工作證明（Symphony）：** agent 交付時附 CI 結果、審查回饋和操作影片，人類據此決定。
4. **成本可視化（Aixle Flow）：** 每個步驟的 token 與成本都看得到，方便做預算管控。
5. **依狀態排序待辦（Agent Orchestrator）：** 依 PR、CI、審查狀態自動排序，需要人處理的排最上面。
6. **測試是指令不是 agent（super-simple-software-factory）：** 能用確定性指令完成的事，不交給 agent。

## 對我們架構的意義

- **協調器不一定要從零自建。** Archon 或 OpenHands 可以當起點，再加上我們需要的部門隔離、模型路由與 Azure DevOps 整合。
- **Temporal 的方向有旁證。** Aixle Flow 走的是同樣的設計，只是專案還小。
- **多數平台以 GitHub 或 Linear 為工作來源。** 回到 Azure DevOps 時，要評估各平台加入新工作來源的難度，這部分沒有查。
- **團隊層級的權限、稽核、多部門隔離仍是普遍缺口。** 少數有這類功能的平台（例如 Paperclip）又有跨租戶漏洞紀錄，所以這一層還是要放在基礎設施層處理。

## 限制

- 所有專案都沒有實測，功能來自 README。
- 星數成長很快，不代表生產環境成熟度；多數專案在 2025 到 2026 年才建立。
- 資安狀況只沿用先前對部分專案的查核，其餘專案沒有查 advisory。
- 沒有查各平台是否支援地端模型與 Azure DevOps。
