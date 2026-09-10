# Taylor Agent — V1 規格書（草案）

> 文件版本：draft-0.1 ‧ 日期：2026-09-10
> 狀態：**草案。** 本文件以圓桌決議 [AGORA-001](001-taylor-core-scope.md) 為 V1 範圍的規範來源，以《Taylor Agent 產品基準》34 節為願景來源。多個章節在 **Gate 1（Brunel 能力稽核）** 與 **Gate 2（使用者／並行需求探測）** 完成前無法定稿，已於各處明確標示。
> 本文件不修改 AGORA-001。AGORA-001 為已結案 ADR，具規範效力；本文件為其下游產物。

---

## 0. 文件狀態與讀法

| 標記 | 意義 |
| --- | --- |
| **[規範]** | 來自 AGORA-001 的已裁決事項，實作必須遵守 |
| **[願景]** | 來自產品基準，方向性描述，尚未工程化 |
| **[待 Gate 1]** | 需 Brunel 能力稽核結果才能定稿 |
| **[待 Gate 2]** | 需使用者／需求探測結果才能定稿 |
| **[待 PO 凍結]** | 等待 Product Owner 單獨裁決 |
| **[未提供]** | 目前無資料，且非上述任一閘可產出 |

閱讀順序建議：§1 → §4（V1 範圍，規範層）→ §15（兩道閘）→ 其餘章節。

---

## 1. 目的與範圍

### 1.1 產品一句話

Taylor Agent 是一個 **Windows 桌面 Coding Agent 控制平面**：以 Session 作為人機互動單位、以 Task Board 管理非同步工作、以 Scheduled Task 處理自動排程，由軟體邏輯統一管理 Agent 執行、Git Worktree、安全權限、長程上下文與工作歷史。Agent 的實際執行委派給執行層 Brunel；Taylor 不設 Manager LLM，orchestration 由程式邏輯負責。**[願景]**

### 1.2 本規格涵蓋

Taylor Agent **V1（首個正式版本）** 的範圍、資料模型、執行與整合架構、安全與權限、排程、復原、治理規則、互動需求與非功能需求，以及動工前必須通過的兩道閘。

### 1.3 本規格不涵蓋

- Brunel 內部設計（見 Brunel 專案 `docs/spec.md`）。
- Pi 內部設計（見 https://github.com/earendil-works/pi）。
- Palladio 設計系統內部（見 Palladio 專案）。
- V1 之後的功能（Recurring 全叢集、Retrieval 子系統、多工作並行 N>1、Event Trigger、Cloud/Remote Execution、Multi-repo 等；見 §4.3、產品基準 §30）。
- 線框圖（wireframes）與具體畫面設計 —— Product Owner 尚未開始，見 §13。

---

## 2. 引用文件

| 代號 | 文件 | 角色 |
| --- | --- | --- |
| ADR-AGORA-001 | `docs/001-taylor-core-scope.md`（本專案內副本） | **V1 範圍的規範來源** |
| BASELINE | 《Taylor Agent 產品基準》34 節 | 願景來源；對照表見附錄 A |
| BRUNEL | https://github.com/bext1998/brunel ‧ 本機 `D:\AgentCoding\Brunel` | 執行層；目前 Alpha 1 實作中 |
| PI | https://github.com/earendil-works/pi | Brunel 委派的 model-facing agent runtime |
| PALLADIO | https://bext1998.github.io/palladio-design-language-system/ ‧ 本機 `D:\AgentCoding\PalladioDesignLanguageSystem` | 設計語言與 design-token 系統；Foundation 已完成，元件進行中 |

---

## 3. 產品概述（願景層）

### 3.1 三種工作型態 **[願景]**

| 型態 | 說明 | V1 狀態 |
| --- | --- | --- |
| **Session Mode** | 人 ↔ Agent 的持續對話工作（寫程式、debug、重構、查 repo、跑測試、Git 操作、修正方向、查看 tool calls、延續歷史 session）。不需要建立 Task。 | V1 納入 |
| **Task Board Mode** | 把工作交給 Agent 自主執行，使用者不需在場。固定三欄看板。 | V1 納入（單一並行，見 §4） |
| **Scheduled Task Mode** | 由 Scheduler 自動觸發 Task 執行。非第四套系統。 | V1 僅 One-time，見 §8 |

### 3.2 核心原則 **[願景]**

- Taylor 管理工作；Brunel 負責 Agent 執行；軟體邏輯負責 orchestration；不讓 Manager LLM 控制整個系統。
- 技術可行 ≠ 產品需要（Feature Freeze，見 §12）。
- Work-level Parallelism 為架構設計前提，但非「Manager Agent 管理多個 Agent」。

### 3.3 目標使用者、商業模式、成功指標

**[未提供 / 待 Gate 2]** —— 產品基準 34 節未定義目標使用者、使用者的現行工具、換用動機、商業模式、KPI、市場規模。圓桌三輪一致將此列為第一優先缺口。Gate 2 需求探測（§15.2）的產出將填入本節。在此之前，本規格的範圍判斷以「最小可用、可恢復閉環」為準，不以使用者規模或市場論證為依據。

---

## 4. V1 範圍（規範層 —— 來自 AGORA-001）

### 4.1 V1 納入 **[規範]**

1. **Session Mode**：對話式 coding 工作。
2. **Task Board**：三欄看板（待辦／執行中／已完成待審），「拖曳到執行中」＝啟動命令，不另跳 Start／Confirm 對話框。
3. **手動 Task → Run → Worktree → Brunel → Completion Evidence → 人工 Review** 的最小可恢復閉環。
4. **自主 Run 並行硬上限 = 1**（Interactive Session 可與 1 個背景 Task Run 並存）。放寬至 2 須先有 Gate 2 需求證據 + 隔離／取消／資源可重現證明，且**不可由 Agent 或一般設定調高**。
5. **Permission Policy Engine（程式層）**：Allow／Ask／Deny/Confirm 四檔；Protected Paths；未分類工具預設 Deny；Agent 不得自行提權；權限綁定於 Run／專案預設而非永久全域。破壞性操作凍結 denylist 見 §7.2。
6. **單一 Checkpoint 機制**：Task Run 必在 Worktree；以該 Worktree 的 Git 可復原點作為 Checkpoint。不做 stash／file-snapshot 子系統，Agent 不得自呼 `git reset --hard` 復原。見 §10。
7. **Cancel**：Run 可取消；Run 行程消失不得卡在 Running（標記 Interrupted／Failed）。
8. **Completion Evidence + 第三欄人工核准**（Approve／Request Changes）。
9. **Steering**：見 §9（採 V2-x，Gate 1 驗收通過即納入 V1）。
10. **One-time Schedule**：見 §8（採 V1-c，In-app 定時器 + 動工前凍結條件，否則退場為「僅資料模型欄位」）。
11. **§34 Feature Freeze 升級為強制門檻**：見 §12。
12. **Ask 互動契約**：見 §7.4。
13. **Session Mode 獨立政策剖面 + 變更前復原點**：見 §7.5（細節 [待 PO 凍結]）。
14. **導覽**：Sessions + Task Board 兩個一級入口（「Review」不作獨立入口）。

### 4.2 V1 延後（資料模型可保留鉤子，功能不實作） **[規範]**

| 項目 | 說明 | 恢復／重議條件 |
| --- | --- | --- |
| 完整 Recurring Scheduled Task | Task Definition／Run 分離、雙卡看板呈現、跨 Run Task Memory、Completion Policy 三層 | Gate 2 需求探測顯示需要；**須另立新議題與新投票**，不由 Gate 結果自動核准 |
| Retrieval pipeline / Universal-style Memory | §18／§19／§20 的 Taylor 側歷史組織、語意檢索、跨 Task 偏好推論 | 觀察到具體失敗（Active Context 爆掉且 Brunel compaction 無法召回）；另立議題 |
| 自動 Retry / Pause / Resume | 執行中 Agent 的可恢復暫停與自動重試 | Gate 1 證明 runtime 契約 + 冪等分類；另立議題 |
| N>1 Work-level Parallelism | 多個自主 Run 併發 | Gate 2 需求證據 + Gate 1 隔離／取消／資源證明 |
| Resource Governor 優先權階級 | Interactive/Manual/Scheduled 三級優先與 per-mode worker 配額 | 本機實際被並行跑飽後再做 |
| 獨立 Review Inbox | 集中的注意力收件匣 | **並行上限開放至 ≥3 或 Scheduled Task 進入 V1 後**，以使用者實證「從 Board 找不到待辦」為升級證據 |
| Task Dependencies | `BLOCKED_BY` 關係 | 後續階段；V1 Board UI 不預留依賴視覺化 |
| Event Trigger（GitHub Issue/PR、CI failure、Webhook、Git push） | 事件觸發排程 | 後續；資料模型保留 Generic Trigger（Manual/Schedule/Event） |

### 4.3 V1 移除（不進 Taylor Core） **[規範 / 願景]**

依產品基準 §29：Subagent Framework、Agent Swarm、Manager/Planner Agent、固定 Agent Team、Taylor 自製 Model Provider／Agent Loop／Tool Runtime／Skill Runtime／Subagent Runtime、Skill/Agent Marketplace、Voice Control、Persona System、Image Generation、自製 Browser Agent／Computer Use／MCP 生態、大量內建第三方 Integration、Autonomous Production Deployment、Autonomous Merge。

額外（AGORA-001）：自製 Checkpoint/snapshot runtime、「接近 Full Access」作為 V1 目標表述。

---

## 5. 核心概念與資料模型

### 5.1 實體 **[規範：概念；待 PO 凍結：欄位與狀態機]**

| 實體 | 定義 | V1 說明 |
| --- | --- | --- |
| **Session** | 使用者與 Agent 的一段互動 | 可獨立存在（`Task = null`, `Run = null`）；也是所有背景 Run 的人工介入介面（Needs Input 時點入即進該 Run 的 Session） |
| **Task** | 需要持續管理、追蹤或排程的工作 | 由使用者建立，或由 Session 轉換而來（抽取 Goal／Relevant context／Constraints／Repository／Important decisions／Source Session reference，不塞整份 Session） |
| **Run** | Agent 對某個 Task 的一次實際執行 | Immutable record；擁有自己的 Worktree identity、base revision、Brunel session 參照、Evidence、Checkpoint 參照、狀態時間軸。Retry／Request Changes **預設建立新 Run**，原 Run 永不覆寫 |
| **Trigger** | Run 的觸發方式 | Generic：`Manual` / `Schedule` / `Event`。V1 僅 `Manual`；`Schedule` 依 §8 結果決定是否可執行；`Event` 僅保留型別 |

關係：`Task 1—N Run`，`Run 1—1 Session`（每個 Run 可對應自己的 Brunel/Agent Session）。普通 Session 可無 Task 無 Run。

### 5.2 Run 狀態機 **[待 PO 凍結 / 待 Gate 1]**

候選狀態（round-2 各席共同提及，尚未定稿）：`Queued/Ready` → `Starting` → `Running` → {`Waiting Permission`（非終止）, `Needs Input`（非終止）} → `Cancelling` → 終止態之一：`Completed (pending review)` / `Cancelled` / `Failed` / `Interrupted`。

必須在動工前凍結：每個狀態的**擁有者**、允許的**轉換**、**終止態**清單、以及 runtime crash／Taylor process crash／機器重啟／App 關閉 各自的 recovery policy（多席主張統一為 `Interrupted`，除少數可安全重連情境）。`Waiting Permission` / `Needs Input` 為**非終止**狀態，不得因顯示在第三欄而被視為已完成、釋放 runtime、觸發新 Run 或視為已核准（AGORA-001 D4 修正 A）。

### 5.3 每 Run 結構化 Handoff **[規範]**

V1 不做記憶子系統。Run 之間只透過有限、可追溯的 handoff schema 延續：`goal`、`constraints`、`base revision`、`outcome`、`outstanding issues`、`known exceptions`、`decisions`、`last successful state`、`artifact references`。不攜帶完整聊天紀錄；context 重建的來源必須可列出。

### 5.4 持久化 **[待 Gate 1]**

Taylor 側需持久化：Run lifecycle、runtime/tool event 摘要、failure reason、completion evidence、baseline reference、policy decision journal（見 §7.6）。儲存後端 [待 PO 凍結]（候選：SQLite）。Brunel 側 session 以 Brunel 自己的 `events.jsonl` 為準（Pi session 停用）——Taylor 讀取契約待 Gate 1。

---

## 6. 執行層與 Brunel 整合

### 6.1 已知的 Brunel 現況（截至 2026-09-08，Alpha 1 實作中）

- **Brunel 為 Go 專案**（Windows x64、PowerShell 7、`CGO_ENABLED=0` 靜態編譯），非 Node.js。
- 依 **ADR-002**，Brunel 把 model-facing agent runtime 委派給 **Pi**（`pi --mode rpc`）。Pi 需要 Node.js/npm 與 Git for Windows（Brunel 已文件化的安裝依賴）。
- Brunel 的 **8 個工具留在 Go**，透過 `taylor-tools.ts` extension 暴露給 Pi。`internal/pirpc` **禁止**送出 `bash` RPC command（INV-9）。
- **已合併的核心**：`internal/workspace`（junction／symlink／絕對路徑逃逸攔截）、`internal/exec` + PowerShell **Job Object 執行器**、`internal/filetools`（全檔 SHA-256 + `expected_hash` 前置條件 + 精確 hunk 套用 + 暫存檔／鎖內重驗／原子換檔）、`internal/safety`（**單一安全決策入口 `Gate.Decide`**、`Risk`／`ApprovalPrompt`／`Approver`、readonly 確定性拒絕、無 TTY `E_APPROVAL_REQUIRED_NO_TTY` 快速失敗、`run_powershell` 對六類代表危險命令的字串／token 分類）、`internal/pirpc` + `internal/redact`（provider/model 透傳、憑證環境變數注入、錯誤轉譯）。
- **尚未接上閉環**（Brunel issue #4）：工具接線、TUI/Approver 實作、Pi 子行程啟動／生命週期／RPC event 轉譯（#9）。
- 憑證：僅 Windows Credential Manager，啟動 Pi 子行程時經環境變數注入並與 provider 綁定。
- Provider/model 範圍 = 使用者當下安裝的 Pi 版本所支援的範圍，Brunel 不承諾特定清單。

### 6.2 Taylor ↔ Brunel 拓撲 **[規範：分層；待 Gate 1：介面細節]**

```
Wails WebView（UI；不直接持有 filesystem / process / credential / network 權限）
   │  Wails bindings（只呈現與送出使用者意圖）
Go control service（Taylor 後端）
   │  · 唯一 OS capability broker 與 process supervisor
   │  · 持久化 Run state 與 command/event journal
   │  · Permission Policy 執行點
   │  versioned / bounded / backpressured framed 訊息協定
   │  （每則事件帶 run id + sequence id；unknown schema fail-closed）
每個 Run 一個 brunel.exe 子行程
   │  Brunel（Go）→ pi --mode rpc 子行程（Pi + Node）
worktree / 授權工具
```

- UI **不**直接管理 Brunel/Pi 的 PID，**不**以畫面存活推論 Run 存活；Go control service 是唯一的 process ownership 與狀態裁決者。
- 子行程樹以 **Windows Job Object** 綁定，確保 Cancel、Go 服務 crash 或 App 關閉時可清理整棵行程樹（Brunel 已具備 PowerShell Job Object 執行器；Taylor 端對 `brunel.exe` 的 Job Object 包裹 [待 Gate 1] 驗證）。
- 重開 App 時對 journal 做 reconcile；無 runtime identity 的 `Running` 標為 `Interrupted`。

### 6.3 §14 Brunel 的定位 **[規範]**

Brunel 為 Taylor V1 的**首選 execution adapter，須通過整合原型（Gate 1）**，而非無條件固定依賴。Taylor 只依賴自己定義的 adapter contract；Brunel 能否滿足該契約是 Gate 1 的結論。Gate 1 的 policy hook 或最小 Run lifecycle 不成立 → V1 的自主 Run 閉環**不得動工**，回到閘前重新收斂，**不得自製完整 tool runtime**。其餘不支援項 → 對應功能從 V1 移除或以已說明的降級語意取代。

### 6.4 Node/Pi 的打包與 containment **[規範（修正 B）/ 待 Gate 1]**

「可攜 Node distribution 與 child-process containment 策略」由 Gate 1 驗證，**不作為已驗證、不可替換的產品承諾**。Gate 1 失敗時可改用等效且同樣可觀察、可終止、可打包的實作，不得因此自製 runtime 或阻塞整包。可觀測契約部分（framed stdio、run id + sequence id、unknown schema fail-closed、journal reconcile、無 runtime identity 即 Interrupted）必須保留。

備註：由於 Pi 是 Brunel 的子行程依賴，Node 執行環境的打包**主要是 Brunel 的責任範圍**；Taylor 端是否需要另外內嵌一份 Node runtime [待 Gate 1] 依 Brunel 的分發形式（NPM 依賴模組 vs 獨立 CLI binary）決定。

---

## 7. 安全與權限

### 7.1 原則 **[規範]**

「低風險流暢、高風險停、未知拒絕。」安全機制不依賴 LLM 判斷，也不靠 Agent 自我約束；程式層具有可執行的安全政策。「接近 Full Access」不作為 V1 目標表述。Full Access 若日後作為使用者選項，**不得覆蓋** Deny 類（保護路徑、提權 Block、workspace 外破壞）。

裁決流：`Agent Intent → Tool Request → Policy Engine → Risk Classification → Allow / Ask(Confirm) / Deny(Block) → Execution → Audit / Recovery`。

### 7.2 破壞性操作凍結 denylist **[規範 —— AGORA-001 D3，逐條照收]**

以下六類為**程式層 hard Block**，以「語意操作類別」判定（**非命令字串比對**）。Block 不可被 Agent、Run 級一次授權、Schedule 或「Full Access」偏好覆蓋；使用者若要執行須離開 Agent 流程手動處理。Taylor 經明確 UI 啟動、對已記錄 checkpoint 的 recovery primitive 不等同 Agent 取得這些工具權限。

| # | Hard Block 類別 | 範圍／例子 |
| --- | --- | --- |
| 1 | 提權與安全控制繞過 | UAC/administrator、`runas`/`sudo` 類提權；關閉或修改防毒／防火牆／Taylor policy／Protected Paths／audit 設定；逃逸 sandbox／job 限制 |
| 2 | 系統目錄與憑證保護區的 mutation 或秘密讀取匯出 | Windows 系統目錄、Taylor runtime/policy/audit 安裝路徑、OS credential store、`~/.ssh`、`~/.aws`、`~/.gnupg`、瀏覽器 profile、全域 git 設定；秘密內容的直接讀取／匯出亦 Block（憑證僅由 broker 使用、不回傳原值） |
| 3 | 授權 workspace／worktree 外的破壞性 filesystem 操作 | 以 canonicalize + symlink/junction 解析後的絕對路徑判定；遞迴刪除、批次覆寫／搬移、改 ACL/owner、格式化。一般的出界「非破壞性」存取不因此自動 Allow，另依 policy Ask/Deny |
| 4 | 改寫遠端 Git 歷史或受保護 ref | `git push --force`、`--force-with-lease`、刪除 remote default/protected branch 或 tag、等價 remote ref rewrite。一般 `git push` 仍至少 Ask |
| 5 | 遠端／生產的不可逆 mutation | production deploy、merge、刪除或覆寫遠端資源／資料、撤銷或輪替憑證／token、發布（npm publish／release）、對外 webhook 觸發、不可撤銷交易 |
| 6 | 秘密外洩與未授權資料傳輸 | 將 credential／token／private key／明定敏感檔送往未授權網路目的地或外部服務。目的地與資料類別須由工具層可判定，**判定不了則 fail closed** |

**carve-out（非 blanket deny）：**

- `git reset --hard` / `git clean -fdx`：在主 workspace 或未知 cwd 的 Agent 原始命令 → Block；僅可由 Taylor Recovery primitive 對已記錄 checkpoint、於 Taylor-owned disposable worktree 執行。
- 授權 worktree 內的大量刪除：須 **Confirm + 執行前 checkpoint + canonical path manifest + blast-radius 上限**；出界或穿越 symlink 立即 Block。

**執行要求：** 先 canonicalize path 與解析 symlink/junction；Git 以結構化參數或受控 wrapper 分類；Shell 腳本若無法可靠分類，不得因命令字串「看起來安全」而 Allow。

**V1 裁定（AGORA-001 D3）：** V1 **不給 Agent 不受限的網路 shell**；工具層無法判定 network destination 與 outbound data class 時 **fail closed**。此項列 Gate 1 必測。

備註（現況）：Brunel `internal/safety` 已對六類代表危險 PowerShell 命令（強制／遞迴刪除、清空內容、大量移動覆寫、git 狀態變更、安裝或更新套件、網路傳輸、背景程序／job、workspace 外絕對路徑）做分類，且工具留在 Go。這與本 denylist **方向一致**；Gate 1 需確認 Brunel 對外暴露的裁決點與事件是否足以讓 Taylor 的 Policy Engine 成為權威裁決者，以及 Pi 在 8 工具以外的行為（若有）如何納管。

### 7.3 風險分類 **[待 PO 凍結]**

- **低風險**（Allow）：repo 內讀取、建立暫存檔、自己 worktree 內的可回復修改、跑測試。
- **中風險**（依 policy Ask/Confirm）：大量修改、安裝依賴、外部網路操作、Git history 變更（本地）、一般 `git push`。
- **高風險**（Confirm 或 Block）：見 §7.2 denylist；破壞性操作一律 Confirm **且**執行前 checkpoint（非盡力而為）。

具體等級由 Product Owner 裁定一張短表；示例不視為既定規格。

### 7.4 Ask 互動契約 **[規範 —— AGORA-001 D4 #14]**

- 背景 Run 的 Ask → 該 Run 卡片進入 `Waiting Permission`（**留在第二欄並置頂**，見 §5.2、§13.2）；核准在 **Run detail inline card**。
- 互動 Session 的 Ask → 對話內的 **inline permission card**。
- **最小資訊集**：發出者／Run、正規化操作、canonical target／network destination、blast radius、為何需要、可否復原 + checkpoint id、將傳出的資料類別、授權作用域。
- **選項**：`Allow once` / `Deny` / `Cancel Run`。V1 高風險 Ask **不提供「永久允許」**。
- **逾時或 UI 關閉 = no-op，不自動 Allow。**
- 不同風險操作**不得**為減少次數而不透明批次核准。

### 7.5 Session Mode 政策剖面 **[規範：需存在；待 PO 凍結：細節]**

Session Mode 與 Task Run 是**兩套政策剖面**。Session 在使用者主工作區操作時，破壞半徑可能大於 Board。V1 要求：

- Session 變更前至少要有可復原點（Git 基線），不得以「Task 有 Worktree」代替 Session 的復原。
- 介面在 Session 進行破壞性變更時明示「這在改你的主工作區」。

**待 PO 凍結：** Session 是否永遠在主工作區、或可選 Taylor-owned worktree；復原點採何種 Git 基線；如何處理使用者既有 dirty state。此裁決影響 `git commit`、大量修改與 checkpoint 的 Allow／Ask 邊界。

### 7.6 稽核（工程紀錄，V1 不做 Audit UI） **[規範]**

Append-only journal，對 **Block / Ask / Allow** 均記錄：policy decision、normalized tool identity、解析後 target（可安全記錄時）、實際 tool outcome。三者（裁決、使用者答覆、實際結果）**分開記錄**。不預設保存可能含秘密的完整工具輸出（保留期限與是否含輸出 [待 PO 凍結]）。

### 7.7 明文禁止 **[規範]**

- 權限一次授予後永久有效 —— 禁止。
- Agent 自行提高權限 —— 禁止。
- 安全規則只存在 Prompt —— 禁止（必須程式層可執行）。

---

## 8. 排程（One-time / Recurring）

### 8.1 One-time Schedule **[規範 —— AGORA-001 D1，採 V1-c]**

V1 實作「**僅 App 開啟期間生效**」的一次性延遲觸發（in-app 定時器；不引入 Windows Task Scheduler、系統服務或常駐工作列）。

**動工前必須凍結並寫入 V1 規格的條件（凍結不成 → 自動退場為「僅保留 `Trigger` 資料模型欄位、V1 不執行任何 schedule trigger」）：**

1. 僅 App 開啟時觸發。
2. App 關閉／睡眠造成 missed trigger 時**不靜默補跑**，以「Missed — 重新排程？」卡片呈現。
3. 同一 Task 已有 active Run 時不啟動。
4. 受並行上限 = 1。
5. Ask 必停，且逾時／關窗 = no-op。
6. 一律進第三欄人工 Review（不 Auto-complete）。
7. 無 Auto-retry。
8. §7.2 遠端不可逆類仍 Hard Block。
9. UI 明示「關閉 App 不會執行」。
10. missed-trigger 卡片的呈現語意隨上述條件一併定稿。

**無論結果為 V1-c 或退場：** 資料模型保留 `Trigger` 型別欄位；Gate 2 探測題目**必須包含排程需求**，One-time 與 Recurring **分開詢問**。

**[待 PO 凍結]** 上述十條的凍結文件（含 missed-trigger 卡片呈現語意）。
**[未提供]** In-app 定時器在 Wails v2（Go 後端）下的實作成本尚未重新評估。

### 8.2 Recurring Scheduled Task

V1 **延後**（見 §4.2）。延後決議須載明 Gate 2 的需求探測題目。恢復須另立新議題與新投票。

---

## 9. Steering **[規範 —— AGORA-001 D2，採 V2-x]**

### 9.1 定位

Steering（Agent 執行中由使用者插入訊息、在下一個安全執行點生效，不需 Stop）**不是** V1 不可移除底線。由 **Gate 1** 決定去留。

**Chair 評估：** Brunel 以 Pi 為基礎，Pi 已具備 Steering 能力（`pi --mode rpc`），故 Brunel 預期支援、Gate 1 此項預期通過、Steering 預期進入 V1。下列驗收條件仍須逐項確認。

### 9.2 納入 V1 的門檻（全有或全無；曖昧 = 未通過）

1. runtime 發出 `accepted` / `applied` / `rejected` 明確事件。
2. 訊息只在明示安全點生效。
3. 已開始的破壞性工具**不被 UI 誤示為已取消**。
4. 每次請求帶 `command_id`，可回報 `rejected` / `timeout` / `runtime death`。
5. crash／重連後**不重複套用**。
6. Cancel **不依賴 Agent 自願配合**。

### 9.3 未通過時的降級

修正方向一律 = **Cancel + 結構化 handoff（§5.3）+ 開新 Run**。**不阻塞 V1 交付。**（此即 V2-y 的行為；若 PO 改採 V2-y，Steering 明確非 V1 交付、列為 V1 後第一批恢復清單第一項。）

### 9.4 納入 V1 時的 UI 約束

steering 訊息顯示「**已送出，將於安全點套用**」狀態，**不**顯示為已生效；不與 Pause／Resume 共用或暗示無損續跑。V1 **不**實作「名為暫停實為重跑」的假 Pause。

---

## 10. 復原（Checkpoint / Rollback） **[規範]**

- Permission Policy 是預防；Checkpoint 是復原。
- V1 僅有 **Taylor 建立並記錄的 Git／Worktree 基線**。Rollback 單位 = 一個 Run（回到本 Run 起點）。
- **不做**：可互換的 snapshot abstraction、stash 子系統、file-snapshot runtime、多層 checkpoint 選單。
- Agent **不得**自呼 `git reset --hard` 做復原（見 §7.2 carve-out）。
- Git 基線**無法**回復：未追蹤／ignored 檔、workspace 外副作用（全域 npm、容器、遠端 API）。V1 政策把工作區外與遠端破壞列為 Ask／Block，**不假裝能回滾外部世界**；Evidence 明示 Git rollback 的範圍。

---

## 11. 長程 Context **[規範 / 願景]**

- **原則**：完整工作歷史可持續增長；Active Context 維持在受控範圍。
- V1：以 **Brunel（經 Pi）的 compaction** + 可捲動的封存歷史 + 每 Task 一份結構化 Markdown handoff（§5.3）為限。
- V1 **不做**：語意索引、跨任務偏好推論、模型自行挑選長期內容、Taylor 側 Retrieval 子系統。
- Retrieval 的唯一啟動條件（未來）：觀察到 Active Context 爆掉且 compaction 無法召回。

---

## 12. 治理：Feature Freeze 強制門檻 **[規範 —— AGORA-001 D4 #9]**

§34 由「建議」升級為**強制門檻**。每個要留在 V1（或日後新增）的「可觀察功能或子系統」必須在 spec／PR 前**同時**提供：

1. 對應產品基準 §34 九判準之一（Session 互動 / Task Management / Scheduling·Automation / Work Isolation / Reliability·Recovery / Safety·Governance / Long-running Context / Human Supervision / Parallel Work）；
2. 指名一條**已描述的現有最小使用流程**，並說明移除後該流程在哪一步失效；
3. （UX 附加）能回答「使用者在哪個畫面看到它的狀態」。

抽象對應（只寫「屬於 Reliability」）**不通過**。bug fix、已保留能力的安全修補、adapter 必要實作細節不算新增功能，但不得藉此包裝新 workflow。例外**僅由 Product Owner 在 ADR 核准**。PR 只需 trace 到已通過門檻的保留項。此規則是軟體治理，不是 LLM 自行判斷的 prompt。

---

## 13. UI / 互動需求

> **[待 PO：線框圖]** Product Owner 尚未開始 wireframes。本章只定義**互動約束與必須可見的狀態**（來自圓桌），不含畫面佈局、元件擺放或視覺稿。這些將在 wireframe 階段補上，並受 §12 門檻約束。

### 13.1 視覺語言：Palladio **[規範方向]**

- Taylor Agent V1 採用 **Palladio Design Language System** 作為視覺語言與 design-token 來源。
- 取用方式：`palladio/dist/css/palladio.css`（CSS custom properties）或 `palladio/dist/ts/tokens.ts`；AI 代理參考 `palladio/dist/agent-reference.md`。
- 約束：semantic token only；accent 色由 Taylor 明確提供並驗證對比（系統不推導色值）；支援 Compact／Default／Spacious 三密度與 reduced-motion。
- **現況**：Palladio Foundation（token 系統）已完成；**元件庫仍在進行中**。Taylor 取得的是 token，需在其上自建元件，不可假設有完整 UI kit。
- 可及性：遵循 Palladio 可及性契約（A-M1–A-M6）。

### 13.2 Task Board：三欄語意（固定，不擴充） **[規範]**

| 欄 | 語意 |
| --- | --- |
| 待辦 / 定時任務 | 接下來有哪些事情 |
| 執行中任務 | Agent 現在正在做什麼；**含 `Waiting Permission`、`Needs Input` 的 Run（非終止，置頂）** |
| 已完成 / 待核准 | Agent 做完了什麼、需要我看什麼；依注意優先度分組排序：`Failed/Interrupted` > `Needs Review` > `Completed` |

- 「拖曳到執行中」＝明確執行命令，背後 `Create Run → Prepare environment → Create/select Worktree → Start Brunel Session → Agent 自主執行`，**不**再跳 Start／Confirm／Select agent。
- **不**建立獨立 Review Inbox；升級條件見 §4.2。
- 待辦欄卡片在並行上限 = 1 時需顯示「排隊中（第 N 位）」；Cancel 排隊任務零成本。

### 13.3 必須可見的狀態與可用的控制 **[規範]**

使用者必須隨時能知道：Agent 在做什麼、為什麼、目前在哪個狀態、什麼需要我處理、怎麼中止、怎麼恢復、怎麼修改方向、什麼操作有風險。

- **Running**：可點入該 Run 的 Session；可 Cancel。
- **Needs Input**：點入 → 進該 Run 的 Session 回答。
- **失敗**：failure reason 持久化且可理解。
- **Completion Evidence**（第三欄）：至少呈現 —— 改了幾個檔、測試結果（passed/failed 數）、build 結果、git commit、warnings。使用者據此 `Approve` / `Request Changes`。Request Changes → 建立新 Run（或恢復目前 Run）。
- **V1 不做**：Retry／Pause／Resume 的 UI 承諾（見 §4.2）。

### 13.4 Confirmation UX **[規範]**

避免「做一步→跳確認→做一步→跳確認」。依風險分類決定確認點（§7.3）。高風險 Confirm 必含 §7.4 最小資訊集；不可復原者明示「此操作無法復原」。

### 13.5 導覽 **[規範]**

一級入口 = **Sessions + Task Board**（「Review」不作獨立入口）。Task Board 內：Board（含定時）+ History。

### 13.6 待補（wireframe 階段） **[待 PO]**

Task Card 版面與狀態徽章、Ask inline card 版面、Evidence 呈現格式、「Missed — 重新排程？」卡片、Session 破壞性變更警示的呈現形態（banner／狀態列／執行前提示，須避免 modal 迴圈）、排隊狀態呈現、steering「已送出／已套用／被拒」三態呈現、Session → Task 轉換的觸發 UI 與轉換後原 Session 呈現、Notification 範圍（桌面內 toast/badge vs 系統通知）。

---

## 14. 非功能需求

| 項目 | V1 要求 | 狀態 |
| --- | --- | --- |
| 目標平台 | Windows x64（桌面）；PowerShell 7 | **[規範]**（Brunel/Pi 亦同） |
| 桌面框架 | **Wails v2**（Wails v3 beta 不採用；不採用 Electron） | **[規範 —— Chair 裁定]** |
| 前端安全 | WebView 不直接持有 filesystem／process／credential／network 權限；Agent 文字與工具輸出視為不可信資料，呈現不得執行 HTML／script | **[規範]** |
| 並行 | 自主 Run 硬上限 = 1 | **[規範]** |
| 資源治理 | V1 = 單一全域 active-autonomous-run semaphore（值 1），不可由 Agent 繞過。子行程綁 Windows Job Object 並可配置 CPU/記憶體上限 | **[規範：semaphore]** / **[待 PO 凍結：CPU/記憶體數值]** |
| Crash 隔離 | 每個 Run 為獨立 OS 子行程樹；單一 Run 崩潰／記憶體洩漏／死結不得波及桌面視窗或其他 Run | **[規範]** |
| Worktree 生命週期 | 確定性建立／檢驗／分支命名／dirty state 偵測／清理；Windows 檔案鎖定（EBUSY/EPERM）以非同步延遲重試佇列 + `Tombstone` 標記處理，不阻塞 UI；提供手動強制清理 | **[規範方向]** / **[待 Gate 2 benchmark]** Worktree-per-Run vs per-Run branch 成本 |
| Worktree 適用範圍 | 僅「變更程式碼的 Run」建立專屬 worktree；唯讀分析／報告型 Run 可原地執行（明確 repository snapshot／working directory policy） | **[規範]** |
| Git 依賴 | 是否要求系統預裝 Git、或安裝包自帶可攜式 Git | **[待 PO 凍結]** |
| 打包 / 更新機制 | Wails 安裝包；Node/Pi 分發形式與更新 | **[待 Gate 1]**（見 §6.4） |
| 效能目標 | 啟動時間、UI 回應延遲、Run 啟動延遲 | **[未提供]** |
| 供應鏈 | 內嵌／依賴的 Node/Pi/Brunel 需固定版本 + hash／簽章驗證 + 受控更新 | **[待 Gate 1]** |

---

## 15. 兩道動工閘（go / no-go） **[規範 —— AGORA-001 D4 #10]**

兩閘**相互獨立**、可**並行**、**time-box**，產出為小型可丟棄的證據工作。需求存在不能補足 runtime 不可控制；runtime 做得到也不構成產品需要。

### 15.1 Gate 1 — Brunel 能力稽核 + Wails v2 sidecar 原型（硬技術閘）

在**實際的 Brunel 版本**與 **Wails v2 打包／開發路徑**上驗證：

- [ ] **工具執行前政策攔截 hook**（第一項）：Taylor 的 Policy Engine 能在工具實際執行前強制裁決；或明確證明採用的窄 proxy 不會重造 agent/tool runtime。**不成立 → 自主 Run 閉環不得動工。**
- [ ] 可由 Go 啟動並識別 `brunel.exe` 子行程；版本／schema handshake。
- [ ] normalized 雙向 event stream；Run／tool 事件具 ID 與順序；unknown schema fail-closed。
- [ ] cooperative cancellation + Windows Job Object 可強制終止整棵 process tree（含 Pi 子行程與其編譯／測試子行程）。
- [ ] runtime crash / UI crash / Taylor process crash / Windows 睡眠 各自的 Run terminal state（多席主張統一為 `Interrupted`，除少數可安全重連情境）。
- [ ] Steering 六項驗收條件（§9.2）—— 可分離子項，失敗只降級 Steering。
- [ ] compaction / session 持久化的 ownership 與 Taylor 讀取契約（Brunel `events.jsonl`）。
- [ ] 秘密不經 IPC log 洩漏。
- [ ] denylist 第 6 類所需的**受控網路工具**與出網 policy 判定能力；若只有任意 shell → V1 不給，且 fail closed。
- [ ] Evidence（§13.3）的 tests／build 欄位資料來源：Brunel／Pi 事件 vs Taylor 程式層驗證。
- [ ] 可攜 Node distribution 與 child-process containment 策略（§6.4）。
- [ ] Pi 版本釘選政策（對照 Brunel OQ-8）。

**失敗處置：** policy hook 或最小 Run lifecycle 不成立 → 閘前重新收斂；其餘不支援項 → 從 V1 移除或以已說明的降級語意取代。**任何情況下不得為通過閘門而自製完整 tool runtime。**

### 15.2 Gate 2 — 使用者／並行需求探測（產品範圍閘）

產出**可反駁的工作流程證據**：

- 目標使用者是誰、目前如何完成「把工作交給 Agent」這件事、為何會換。
- 是否需要：無人值守執行 / One-time schedule / Recurring schedule / 同時多個 Run（N>1）/ 集中注意力入口。
- 可接受何種 Review。
- V1 的發布目標（自用工具 / 內部團隊 / 公開產品）。

**無證據 → 預設維持：** manual trigger、cap = 1、無 Inbox、無 Scheduler。
探測結論為 **Product Owner 裁決輸入**，**不單獨放寬 denylist**，**不自動推翻 AGORA-001**。若顯示需要全自動 Recurring 或 N>1 → 重開需**另立新議題與新投票**。

---

## 16. 未解問題（彙整自 AGORA-001 與圓桌 Open Questions）

1. Session 是否永遠在使用者主工作區操作，或可選 Taylor-owned worktree？（決定 §7.5、§13.6、`git commit`／大量修改／checkpoint 邊界）**[待 PO 凍結]**
2. Run 正式狀態機、轉換擁有者、terminal states、各 crash 情境 recovery policy。**[待 PO 凍結 / 待 Gate 1]**
3. 專案 `.env` 算 credential protected path（僅 broker 使用）還是可 Ask 後由 Agent 直接讀？**[待 PO 凍結]**（安全預設：Agent 不取得原值）
4. V1 是否具備足以判定 network destination 與 outbound data class 的受控工具？若無，denylist 第 6 類為紙上條款。**[待 Gate 1]**
5. Evidence 的 tests／build 欄位資料來源。**[待 Gate 1]**
6. One-time in-app 排程在系統睡眠時的 missed-trigger 恢復語意由誰定稿（Taylor 顯示規格 vs 平台事件）。**[待 PO 凍結]**
7. Windows 上 Worktree-per-Run 的實際成本（磁碟、dependency install、clone 時間）vs per-Run branch。**[待 Gate 2 benchmark]**
8. Brunel 的封裝分發形式（NPM 依賴模組 vs 獨立 CLI binary）→ 決定 IPC 形式與 Node 打包責任。**[待 Gate 1]**
9. 是否強制使用者預裝系統 Git，或安裝包自帶可攜式 Git。**[待 PO 凍結]**
10. 破壞性操作的 V1 凍結 denylist「風險分類短表」由 Product Owner 定稿（§7.3）。**[待 PO 凍結]**
11. Audit journal 保留期限、是否含可能含秘密的工具輸出。**[待 PO 凍結]**
12. 同一 workspace 多 Session 並發寫入要警告還是硬擋？**[待 PO 凍結]**（V1 建議：至少偵測／警告）
13. 資源治理的 CPU／記憶體上限數值。**[待 PO 凍結]**
14. 效能目標（啟動時間、UI 延遲、Run 啟動延遲）。**[未提供]**
15. Gate 1 稽核的 time-box 上限與由誰簽核通過。**[待 PO 凍結]**
16. Gate 1／Gate 2 的實際排程與 Brunel Alpha 進度的相依（Brunel 尚未接上閉環，Gate 1 需 Brunel 到可測狀態）。**[未提供]**

---

## 17. 風險登錄（彙整自圓桌 Risks）

| # | 風險 | 緩解 | 殘餘風險 |
| --- | --- | --- | --- |
| R1 | V1 過瘦、automation 故事變薄（無 Recurring、cap=1、Steering 視 Gate 1） | Gate 2 置於動工前；cap=1 待辦欄顯示「排隊中」；對外敘事誠實（「架構為並行設計，V1 以單一被監督 Run 交付」） | 若 Gate 2 顯示使用者要的就是自動化，差異化延後 |
| R2 | 六席「共識」建在同一組未驗證前提（使用者需求、Brunel 契約）上 | 兩道閘為明確絆線；所有收斂標為「條件收斂」，未驗證件標殘餘風險 | Gate 失敗則整套 V1 範圍需重評 |
| R3 | 假治理：Policy 寫在文件／Prompt，Brunel 無攔截點；或允許任意 Shell | Gate 1 第一項 fail closed；cwd 綁 workspace／worktree；絕對路徑與 symlink 納入政策；未分類 Deny | Brunel hook 能力若不足，本底線落空 → 閘前重新收斂 |
| R4 | Session 主工作區是最大日常殺傷面 | §7.5 獨立政策剖面 + 變更前復原點 + 破壞性變更明示 | 使用者未提交的工作與 Agent 變更纏在一起，git 可復原點不夠乾淨 |
| R5 | denylist 過寬阻礙合法工作 / 過窄產生不可回復事故 | 語意類別而非字串；未知 mutation fail closed；合法高風險操作回到使用者手動執行 | 流暢度下降；一般建置工具的隱藏網路副作用 |
| R6 | Wails v2 Go↔Node(Pi) 邊界：打包、stdio 壅塞、child/orphan 清理、崩潰重連、版本配對 | Gate 1 斷線／UI 關閉／各 crash 測試案例；Go journal 為唯一真相；無 runtime identity 即 Interrupted；Job Object 整棵 kill | 已開始的外部副作用無法撤回 |
| R7 | Embedded Node/Pi 供應鏈與更新面 | 固定版本 + hash／簽章驗證 + 受控更新；列 Gate 1 | 分發形式未定前無法完全評估 |
| R8 | §34 強制門檻退化成標籤遊戲 | 每項附使用流程、九判準對應、替代方案、PO 裁定；PR trace 而非重複作文 | PO 仍可基於產品判斷核准例外（其正式權限） |
| R9 | Gate 延遲拖累動工；需求探測樣本偏差 | 兩閘並行、time-box；缺證據採較小預設 | 可能砍掉其實有價值的 One-time／N>1（比先建出不可控能力可逆） |
| R10 | 「Missed — 重新排程？」等新 UI 狀態流在規格未定稿時被實作期即興發明 | missed-trigger 卡片呈現語意隨 §8.1 十條一併凍結 | wireframe 未開始前，UI 狀態語意仍有懸置 |
| R11 | Steering 曖昧稽核（部分支援）造成納入壓力 | §9.2 全有或全無判定，曖昧 = 未通過 | 若判 V2-y，差異化能力延後一版 |

---

## 附錄 A — 產品基準 34 節 → V1 決議對照

| 基準節 | 主題 | V1 決議 |
| --- | --- | --- |
| §1–§2 | 產品定位、三種工作型態 | 願景保留；Session + Board（cap=1）納入 V1，Scheduled 僅 One-time（§8） |
| §3 | 拖曳即執行 | 納入（§13.2）；「啟動後仍有確認邊界」以 §7 政策落實 |
| §4 | Session/Task/Run 分離 | 納入（§5）；欄位與狀態機 [待 PO 凍結] |
| §5–§6 | Session ↔ Task 轉換 | 概念保留；轉換 UI [待 wireframe] |
| §7–§8 | Recurring 資料模型、雙卡呈現 | 完整 Recurring 延後（§4.2、§8.2） |
| §9 | 第三欄「已完成／待核准」 | 納入（§13.2、§13.3）；含 Evidence + Approve/Request Changes |
| §10 | Completion Policy 三層 | Auto-complete 不進 V1；V1 一律 Review |
| §11 | Work-level Parallelism | 架構前提保留；V1 硬上限 = 1（§4、§14） |
| §12 | 不採 Subagent-Orchestrator | 沿用（§4.3） |
| §13 | Software Orchestration 保留 | 沿用；Queue/Scheduler/Concurrency/State/Retry/Pause/Resume/Cancellation/Dependencies/Review state 由程式邏輯負責，但 V1 只做 cap=1 semaphore + Cancel + Review state |
| §14 | Brunel 角色固定 | 改述為「首選 execution adapter，須通過 Gate 1」（§6.3） |
| §15 | Git Worktree 為核心 | 納入；僅「變更程式碼的 Run」建立專屬 worktree（§14）；Windows 成本 [待 Gate 2] |
| §16 | 安全架構、Policy Engine | 納入（§7）；「接近 Full Access」表述淘汰 |
| §17 | Checkpoint / Rollback | 收斂為 Worktree + Git 單一機制（§10） |
| §18–§20 | 長程 Context、偏好、Task Memory | Retrieval/Task Memory/偏好子系統延後（§4.2、§11）；只留每 Run handoff |
| §21 | Resource Governor | V1 = 單一 semaphore（值 1）；優先權階級延後 |
| §22 | Steering | §9（採 V2-x，Gate 1 驗收） |
| §23 | Completion Evidence | 納入（§13.3） |
| §24 | Review Inbox | 不做獨立入口，併第三欄（§4.2、§13.2） |
| §25 | Task Dependencies | 延後；V1 不預留依賴 UI |
| §26 | Event Trigger | 延後；保留 Generic Trigger 型別 |
| §27 | 桌面主要 Navigation | Sessions + Task Board 兩入口（§13.5） |
| §28 | 三欄心智模型 | 納入且固定（§13.2） |
| §29 | 移除清單 | 沿用（§4.3） |
| §30 | 保留但延後 | 沿用（§4.2） |
| §31 | Windows 桌面端、Wails/Electron | **Wails v2**（Chair 裁定，§14）；Palladio 為設計語言（§13.1）；wireframe 未開始 |
| §32 | 原專案關係、Brunel repo | 另開新專案（本 repo）；Brunel 現況見 §6.1 |
| §33 | 最終核心架構 | 以 §4–§14 收斂版取代；軟標籤「建議保留」項按 §4.2 延後 |
| §34 | Feature Freeze 判斷規則 | 升級為強制門檻（§12） |

---

## 附錄 B — 待辦（動工前）

1. [待 PO] 凍結 §8.1 的 One-time 十條件文件（含 missed-trigger 卡片語意）；凍不成則 §8.1 直接寫「僅資料模型欄位」。
2. [待 PO] 凍結 §7.5 Session 政策剖面（主 workspace vs worktree、復原點基線、dirty state 處理）。
3. [待 PO] 定稿 §7.3 風險分類短表與 §16 各 [待 PO 凍結] 項。
4. [執行] 撰寫 Gate 1 稽核測試規格（§15.1 清單）。
5. [執行] 撰寫 Gate 2 需求探測問卷（§15.2；One-time 與 Recurring 分開問）。
6. [執行] 定義 §12 的 PR 審查機制（每個保留項附使用流程 + 九判準對應 + 替代方案 + PO 裁定）。
7. [待 Brunel] Gate 1 需等 Brunel 接上閉環（issue #4）到可測狀態。
8. [後續] wireframe 階段補 §13.6 清單，逐項過 §12 門檻。
