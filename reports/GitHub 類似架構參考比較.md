# GitHub 類似架構參考比較

查核日期：2026-11-10。星數、授權與最後推送日期取自 GitHub API；功能描述取自各專案 README，**沒有安裝或實測**。本文由 Anthropic 的模型撰寫，多數專案預設使用 Claude Code，篇幅不代表排名。

## 結論

GitHub 上沒有和我們建議架構完全相同的現成專案。大部分「AI software factory」專案只覆蓋**工單到 PR** 這一段，並綁定 GitHub Issues；涵蓋 PM 知識庫、URD、LLM wiki 的專案幾乎沒有。

但這些專案在三件事上和我們的設計一致，可以當成方向正確的旁證：

1. **確定性程式碼控制流程，agent 只做節點內的工作。** 例如 [disler/super-simple-software-factory](https://github.com/disler/super-simple-software-factory) 的「Agent proposes, code disposes」，以及 [MostAshraf/ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) 把流程規則從 prompt 移到程式碼。
2. **寫程式和審查分開。** 幾乎所有專案都使用獨立、乾淨 context 的 reviewer 或 verifier。
3. **人類保留需求與合併的決定權。** 多數專案明確要求人類核准計畫或合併。

最值得深入研究的三個（我的判斷）：

| 優先 | 專案 | 原因 |
|---|---|---|
| 1 | [MostAshraf/ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) | 唯一確認內建 **Azure DevOps** provider（`ado_cli`、`ado_mcp`），支援 **Qwen Code**，人類閘門由程式碼判定 |
| 2 | [addyosmani/factory](https://github.com/addyosmani/factory) | 最清楚的參考架構：工單佇列、章程限制、確定性閘門、乾淨的 verifier、審查佇列上限 |
| 3 | [cortexapps/engrams](https://github.com/cortexapps/engrams) | 自架的 agent 執行層，Firecracker microVM 隔離與出站過濾，對應我們的 L6 平台層 |

## 依架構層的對照

| 我們的層 | 對應專案 | 可借鏡之處 | 落差 |
|---|---|---|---|
| L1 PM 知識與 URD | [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)、[Konductor](https://github.com/aws-solutions/konductor)、[coleam00/ai-software-factory](https://github.com/coleam00/ai-software-factory) | 需求訪談、PRD 拆分成工單、驗收情境 | 都是 Git 或聊天中的文件，**沒有 LLM wiki 或多部門知識庫** |
| L2 規格與追溯 | [spec-kitty](https://github.com/spec-kitty/spec-kitty)、[jpsasi/ai_sdlc](https://github.com/jpsasi/ai_sdlc) | 規格、計畫、工作包和審查狀態都存在 repo | spec-kitty 4.x 仍是 RC |
| L3 協調 | [zeroshot](https://github.com/the-open-engine/zeroshot)、[super-simple-software-factory](https://github.com/disler/super-simple-software-factory)、[ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) | JSON 定義流程圖、reviewer 判定決定下一步、SQLite 保存狀態 | 多為單機或單 repo，沒有 Temporal 等級的長時間流程 |
| L4 Agent 執行 | [SWE-AF](https://github.com/Agent-Field/SWE-AF)、[eve 範本](https://github.com/vercel-labs/eve-software-factory-template)、[Last Light](https://github.com/nearform/lastlight) | 角色分工、平行 worktree、各工作站獨立沙箱與工具 | 大多預設雲端模型 |
| L5 驗證 | [agentic-qe](https://github.com/proffesor-for-testing/agentic-qe)、[addyosmani/factory](https://github.com/addyosmani/factory)、[orbi](https://github.com/orbi-build/orbi) | 乾淨的 verifier、證明新測試在沒有實作時會失敗、AI 不可自行合併 | agentic-qe 的效果是自述，未經獨立驗證 |
| L6 平台與治理 | [engrams](https://github.com/cortexapps/engrams)、[chainloop](https://github.com/chainloop-dev/chainloop)、[apexyard](https://github.com/me2resh/apexyard) | microVM 隔離、出站過濾、簽章證據、具名人員核准特定 commit | engrams 以 GKE/EKS 為主，不是地端 |

## 候選專案一覽

| 專案 | 星數 | 授權 | 最後推送 | 類型 | Azure DevOps | 地端或本地模型 |
|---|---|---|---|---|---|---|
| [MostAshraf/ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) | 20 | MIT | 2026-08-27 | Claude Code 與 Qwen Code 外掛 | **有**（provider 模組） | Qwen Code 可接自架模型，未驗證 |
| [addyosmani/factory](https://github.com/addyosmani/factory) | 333 | MIT | 2026-08-21 | 參考架構，無獨立服務 | 無（GitHub Issues） | 未提及 |
| [cortexapps/engrams](https://github.com/cortexapps/engrams) | 63 | AGPL-3.0 | 2026-11-09 | 自架 agent 沙箱平台 | 未提及 | 需自帶 Anthropic、OpenAI 或路由器 |
| [the-open-engine/zeroshot](https://github.com/the-open-engine/zeroshot) | 2,131 | MIT | 2026-11-09 | 多 agent 流程引擎 | 未提及 | README 提到 Ollama，未驗證 |
| [Agent-Field/SWE-AF](https://github.com/Agent-Field/SWE-AF) | 1,126 | Apache-2.0 | 2026-11-08 | 自主工程團隊執行環境 | 未提及 | 可接 OpenAI 相容端點 |
| [vercel-labs/eve-software-factory-template](https://github.com/vercel-labs/eve-software-factory-template) | 1,170 | MIT | 2026-11-02 | 四站式工廠範本 | 無（GitHub、Linear） | 未提及 |
| [disler/super-simple-software-factory](https://github.com/disler/super-simple-software-factory) | 1,038 | MIT | 2026-08-04 | Python 控制流程的 skill | 未提及 | 各 agent 可設模型 |
| [coleam00/ai-software-factory](https://github.com/coleam00/ai-software-factory) | 518 | 未標示 | 2026-11-09 | PRD 到 staging 的工廠 | 無 | 依 Archon 設定 |
| [orbi-build/orbi](https://github.com/orbi-build/orbi) | 248 | AGPL-3.0 | 2026-11-08 | 自架暗工廠 | 無（GitHub Issues 為唯一狀態） | README 提到 Ollama，未驗證 |
| [nearform/lastlight](https://github.com/nearform/lastlight) | 45 | MIT | 2026-11-09 | 自架 repo 維護 agent | 無 | 15 個以上的模型供應商 |
| [aws-solutions/konductor](https://github.com/aws-solutions/konductor) | 11 | Apache-2.0 | 2026-11-09 | Kiro、Claude Code 的角色團隊 | 未提及 | 依 Kiro 或 Claude Code |
| [proffesor-for-testing/agentic-qe](https://github.com/proffesor-for-testing/agentic-qe) | 524 | MIT | 2026-11-09 | QA agent 平台 | 未提及 | **Ollama**、Azure OpenAI |
| [spec-kitty/spec-kitty](https://github.com/spec-kitty/spec-kitty) | 1,716 | MIT | 2026-11-09 | 規格驅動與治理 CLI | 未提及 | 支援 OpenCode 等多種 agent |
| [me2resh/apexyard](https://github.com/me2resh/apexyard) | 543 | MIT | 2026-11-09 | 多專案營運 repo | 未提及 | 預設 Claude Code |
| [tobihagemann/turbo](https://github.com/tobihagemann/turbo) | 419 | MIT | 2026-11-09 | 人類把關的個人工廠 | 未提及 | Claude Code、Codex |
| [jpsasi/ai_sdlc](https://github.com/jpsasi/ai_sdlc) | 0 | 未標示 | 2026-11-08 | Claude Code 外掛 | **有**（`az` CLI、Azure Repos） | 未提及 |
| [chainloop-dev/chainloop](https://github.com/chainloop-dev/chainloop) | 609 | Apache-2.0 | 2026-11-09 | SDLC 證據與政策 | 未提及 | 不適用 |
| [strongdm/attractor](https://github.com/strongdm/attractor) | 1,341 | Apache-2.0 | 2026-03-17 | 只有規格，沒有實作 | 不適用 | 不適用 |
| [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 55,168 | GitHub 未判定 | 2026-11-08 | 需求到實作的方法論 | 不適用 | 不綁模型 |

## 值得借鏡的具體設計

1. **章程限制可接的工作（addyosmani/factory）：** 只讓 agent 接章程允許的小型工作，其他導向規格或問題。對應我們的分級自主。
2. **審查佇列上限（addyosmani/factory）：** 待審 PR 達上限就停止產出。這正好處理先前研究提到的審查瓶頸。
3. **人類閘門由程式碼判定（ai-sdlc-harness）：** Hook 原樣擷取人類回覆，再由確定性程式碼解析，模型不能代替人類核准。
4. **Reviewer 看不到作者的推理（eve 範本）：** 審查者只看推送後的分支。對應我們的 fresh-context verifier。
5. **Holdout 驗收情境（coleam00/ai-software-factory）：** 保留 builder 看不到的驗收情境。可對應 URD 的驗收條件。
6. **只有核准的那個 commit 能合併（orbi、apexyard）：** 防止審查後又被改動。
7. **工廠記憶（eve 範本）：** 每次執行從累積的 repo 筆記開始。可與 LLM wiki 銜接。

## 和 Microsoft 方向的關係

Microsoft 在 [Build 2026 BRK202](https://github.com/microsoft/Build26-BRK202-azure-devops-meets-github-the-path-to-ai-powered-sdlc) 的建議是：**把 repo 從 Azure Repos 搬到 GitHub，保留 Azure Boards 和 Azure Pipelines**，才能取得完整的 agent 功能。這和我們「留在 Azure DevOps Services、agent 放地端」的方向不同。

這不代表要改方向，但它是一個實際的決策點：留在 Azure Repos 就要自建整合層；搬到 GitHub 能用到更多現成工具，例如上面大多數專案，但會增加雲端依賴。

## 限制

- 所有專案都沒有實測，功能來自 README。
- 多數專案很新，最早的也只到 2025 年，生產環境證據很少。
- Azure DevOps 支援只在兩個專案確認，其中 jpsasi/ai_sdlc 是 0 星的新專案。
- 沒有找到同時涵蓋 LLM wiki、多部門權限、Azure DevOps、地端模型的整合專案。這正是我們自建整合層的理由，也是最大的實作風險。
