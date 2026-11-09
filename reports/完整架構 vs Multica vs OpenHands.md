# 完整架構 vs Multica vs OpenHands

查核日期：2026-11-10。「完整架構」指《Azure DevOps Services 混合式 AI Factory》報告的建議架構，知識層改以 LLM wiki（WeKnora 或 RAGFlow）為主。Multica 與 OpenHands 的資料來自 README、官方文件、授權檔與 GitHub API，**都沒有安裝或實測**。本文由 Anthropic 的模型撰寫，兩個平台都支援 Claude Code，篇幅不代表排名。

## 結論

**三者都無法單獨達到目的；最省工的做法是以 OpenHands 當 agent 執行層，放進我們的架構，而不是整套採用 Multica 或 OpenHands。**

- **完整架構**是唯一能滿足多部門、Azure DevOps、地端模型和稽核要求的方案，但整合工作最多。
- **OpenHands** 的執行層最成熟：可接任何模型（含 vLLM、Ollama）、有 Docker 與 Kubernetes 後端、支援多種 agent。它的開源版缺多人權限，Azure DevOps 整合則在商用的 Enterprise 版。
- **Multica** 上手最快，看板、團隊、成本和審查閘門都現成，但它沒有沙箱、沒有 SSO、不支援 Azure DevOps，自己的看板也會和 Azure Boards 重疊。

| 方案 | 省工程度 | 能否達到目的 | 建議 |
|---|---|---|---|
| 完整架構（自建整合） | 最低 | 能，但要自己做最多 | 長期目標 |
| **完整架構 + OpenHands 執行層** | **中** | **能** | **推薦** |
| OpenHands Enterprise 為主 | 中高（需付費） | 大部分能，但需入站連線且受授權限制 | 預算充足、可接受入站時 |
| Multica 為主 | 最高 | 不能（缺 Azure DevOps、SSO、沙箱） | 只適合小團隊試點 |

## 三者差異

| 面向 | 完整架構 | Multica | OpenHands |
|---|---|---|---|
| **定位** | 多部門 AI 軟體工廠 | 把 agent 當隊友的工作看板 | 自架的 agent 控制中心與執行環境 |
| **授權** | 各元件各自授權 | [Apache-2.0 加附加條件](https://github.com/multica-ai/multica/blob/main/LICENSE)：組織內部使用（含多工作區）不需商業授權；對外提供服務需要 | 核心 MIT；[Enterprise 是 PolyForm Free Trial](https://github.com/OpenHands/enterprise)，無商業授權時每年限用 30 天 |
| **版本** | — | v0.8.1（2026-11-06），1.0 前 | v1.37.0（2026-11-06） |
| **開發商** | — | Index Labs（香港） | OpenHands（原 All Hands AI） |
| **工作來源** | Azure Boards | **自己的看板**，Git 支援 GitHub、GitLab、Gitea、Forgejo | 自動化可接 GitHub、Linear、Slack；Azure DevOps 只在 Enterprise |
| **Azure DevOps** | 自建輪詢 dispatcher 與 Build Validation | **不支援**，[整合需求 #3573](https://github.com/multica-ai/multica/issues/3573) 仍開著 | [Enterprise 支援 Services](https://docs.openhands.dev/enterprise/integrations/azure-devops)，需入站可達的 URL 與 Entra 應用程式 |
| **協調方式** | 確定性流程（Pipelines，之後可加 Temporal） | 指派 issue，agent 自行處理；有重試、逾時、審查閘門 | 對話與自動化（排程或 webhook） |
| **Agent 支援** | OpenCode、Goose 等 CLI | Claude Code、Codex、Cursor、Copilot、OpenCode 等 | OpenHands 自家 agent，加 Claude Code、Codex、Gemini、OpenCode 等 ACP agent |
| **地端模型** | vLLM 或 SGLang，依資料分級路由 | 取決於所用的 agent CLI | [任何 LiteLLM 支援的模型](https://docs.openhands.dev/openhands/usage/llms/llms)，含 vLLM、Ollama |
| **沙箱** | agent-sandbox 加 Kata 或 gVisor | **沒有**，[官方要求把每次執行視為未隔離](https://multica.ai/docs/security-model) | Docker、VM、Kubernetes 後端；不用沙箱時可存取整個檔案系統 |
| **多部門與權限** | 基礎設施層隔離 | 工作區、owner/admin/member、可限制成員能用哪些 agent | 開源版只有 API key；多人權限未在開源文件中看到 |
| **SSO** | Entra ID | **沒有**；登入用 email 驗證碼，[OIDC](https://github.com/multica-ai/multica/issues/1014) 和 [Azure AD](https://github.com/multica-ai/multica/issues/5423) 需求仍開著 | 開源版靠 API key 加前置代理（例如 OAuth proxy） |
| **知識層** | LLM wiki 加程式碼索引 | Projects 可附 repo 和文件、Skills | Skills、MCP；沒有知識庫 |
| **測試 agent** | 獨立 verifier、Playwright Test Agents、確定性工具 | 沒有專門機制，依 agent 和 skill | 沒有專門機制，依 agent 和 skill |
| **可觀測與成本** | Langfuse、閘道日誌、部門預算 | 執行紀錄可重播，依 agent 和 issue 統計成本 | 對話紀錄；成本統計未確認 |
| **遙測** | 自行控制 | 預設每日傳送匿名部署資料，`DO_NOT_TRACK=1` 可關閉 | 未查 |
| **資安紀錄** | 依元件 | 沒有公開 advisory，但開放 issue 很多，不能據此視為安全 | 2026-03 有一個高嚴重度的命令注入 advisory |

## 兩個平台缺少的功能

| 我們需要的 | Multica | OpenHands 開源版 | OpenHands Enterprise |
|---|---|---|---|
| Azure Boards 與 Azure Repos 整合 | 缺 | 缺 | 有（Services），需入站 |
| 不開入站連線 | 可（daemon 主動連線） | 可（自行觸發） | 缺（webhook 需入站） |
| Entra ID 單一登入 | 缺 | 缺 | 未查 |
| 部門隔離與 RBAC | 部分（工作區、角色） | 缺 | 未查 |
| agent 沙箱 | **缺** | 有 | 有 |
| 依資料分級的模型路由 | 缺 | 部分（可設多組模型） | 部分 |
| 確定性品質閘門 | 部分（審查閘門） | 缺，要靠 CI | 缺，要靠 CI |
| 獨立的測試 verifier | 缺 | 缺 | 缺 |
| LLM wiki 與需求追溯 | 缺 | 缺 | 缺 |
| 稽核紀錄 | 部分（執行紀錄） | 部分（對話紀錄） | 未查 |
| 部門預算控管 | 部分（成本統計） | 缺 | 未查 |

## 推薦方案：完整架構 + OpenHands 執行層

```
Azure Boards / Repos（雲端）
  ↓ 地端 dispatcher 輪詢
Azure Pipelines（自託管 agent pool，確定性流程與閘門）
  ↓
OpenHands agent server（Docker 或 Kubernetes 後端）
  ├─ OpenHands agent 或 OpenCode（經 ACP）
  ├─ 模型：LiteLLM 相容端點 → 地端 vLLM 或核准的雲端
  └─ 知識：WeKnora 或 RAGFlow 的 MCP
  ↓
PR → Build Validation（Playwright、Semgrep、mutation、fresh-context verifier）→ 人類合併
```

**省下的工作（我的判斷）：**

- 不必自建 agent 執行環境、沙箱後端和多 agent 切換。
- 模型設定沿用 OpenHands 的 LiteLLM 介面，方便切換地端與雲端。
- 自動化與 webhook 機制可沿用。

**仍要自建：**

- Azure DevOps dispatcher：輪詢 Boards、觸發 Pipelines、回寫結果。
- 部門隔離：每部門一個 agent server，或放在 Kubernetes 命名空間，前面加身分代理。
- 品質閘門與 verifier，放在 Build Validation。
- 知識層與需求追溯。

## 各自缺點

### 完整架構（自建）

- 整合工作最多，我先前估計平台團隊要 5 到 7 人。
- 由多個開源元件拼成，升級和相容性要自己管。
- 沒有現成介面，PM 和工程師的操作體驗要另外設計。

### Multica

- **沒有沙箱：** agent 以 daemon 使用者的完整權限執行，能讀取該使用者家目錄裡的憑證並連外。多部門使用前一定要另外加容器或 VM 隔離。
- **不支援 Azure DevOps：** 只支援 GitHub、GitLab、Gitea、Forgejo。
- **看板重疊：** 它有自己的 issue 看板，會和 Azure Boards 形成兩套工作來源。
- **沒有 SSO：** 登入靠 email 驗證碼；寄信預設使用 Resend 雲端服務，不設定就要從後端日誌複製驗證碼。
- **成熟度：** 1.0 前；遙測預設開啟。
- **來源：** 開發商是香港公司，如果受台灣公部門相關規範約束，要先確認。

### OpenHands

- **Azure DevOps 整合要付費：** Enterprise 是 PolyForm Free Trial，無商業授權時每年只能用 30 天，價格不公開。
- **需要入站連線：** Enterprise 的 Azure DevOps 整合靠 service hook 打到你的 URL，和我們「不開入站」的設計衝突。
- **開源版沒有多人權限：** 只有 API key，要靠前置代理或每部門獨立部署來隔離。
- **不用沙箱時風險高：** 官方警告 agent server 可存取檔案系統、執行指令和連網，必須用 Docker、VM 或 Kubernetes 後端。
- **產品轉型中：** 現在定位為 Agent Canvas，介面和文件變動快。

## 決定前要驗證

1. OpenHands 開源版能否由我們的 dispatcher 或 Pipelines 觸發，不經 Enterprise 整合。先前的研究筆記把這點列為未測。
2. OpenHands 經 ACP 使用 OpenCode 接地端 vLLM 的穩定性。
3. 每部門一個 agent server 的資源與維運成本。
4. 如果考慮 OpenHands Enterprise：價格、能否離線授權、能否避免入站連線。
