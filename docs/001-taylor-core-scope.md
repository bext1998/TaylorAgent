# AGORA-001：Taylor Agent 產品基準的 V1 範圍收斂

> 狀態：**CLOSED。** 由 Product Owner（Chair）於 2026-09-10 定稿。四題裁決：V1 採 V1-c；V2 採 V2-x（含 Chair 對 Brunel 的評估，見 D2）；V3 逐條照收並加 fail-closed 裁定；V4 批准 14 項並採納修正 A、B。撤回條件經 Chair 確認。第 7 節四段完整，已複製為 `agora/decisions/001-taylor-core-scope.md`。

## 票型彙總

| 題 | 選項與票數 | 結果 |
| --- | --- | --- |
| V1 One-time Schedule | V1-c ×3（product-strategist、platform-architect、ux-designer）／V1-a ×2（systems-architect、safety-engineer）／V1-d ×1（devils-advocate）／V1-b ×0 | **V1-c 相對多數**；另 3 席均把 V1-c 或其條件列為「最低可接受條件」 |
| V2 Steering 定位 | V2-x ×4（product-strategist、platform-architect、safety-engineer、ux-designer）／V2-y ×2（systems-architect、devils-advocate） | **V2-x 多數** |
| V3 denylist 定稿 | 逐條照收 ×6 | **一致通過**（附一則跨席落地前提，見下） |
| V4 整包 V1 範圍 | 整包批准 as-is ×4（product-strategist、platform-architect、ux-designer、devils-advocate）／批准但有例外 ×2（systems-architect 對第 12 項、safety-engineer 對第 6 項） | **通過**，建議採納兩則修正 |

## Context

議題 slug `taylor-core-scope`（AGORA-001）。標的為 Taylor Agent 產品基準 34 節（`source-product-baseline.md`）。Product Owner 的目的：以六席對抗抵銷單一 AI 對話的方向漂移，並強制收斂一個體量龐大的專案的 V1 範圍。

決策標準（motion）：範圍是否顯著收斂；每個保留項對得上 §34 九判準之一且為現有流程必要能力；已識別矛盾都有處置（解決／延後／記錄殘餘風險）；不追求全體共識、保留分歧與 trade-off。

程序：round 1（各自陳述）→ round 2（交叉辯論）→ 投票（V1–V4 四題）。max_rounds = 2，已達上限。六席模型：`moonshotai/kimi-k3`（product-strategist）、`openai/gpt-5.6-terra`（systems-architect）、`google/gemini-3.7-flash`（platform-architect）、`openai/gpt-5.6-sol`（safety-engineer，round-2 中途由 Product Owner 自 `x-ai/grok-4.6` 切換）、`z-ai/glm-5.3`（ux-designer）、`deepseek/deepseek-v4-pro-0813`（devils-advocate）。Chair 於議程中裁定 §31 桌面框架維持 Wails v2（Wails v3 beta 不採用、不採用 Electron），六席接受。

回合檔案為決策證據，不具規範效力；本 ADR 結案後方具規範效力。

## Decision

### D1 — One-time Schedule：採 V1-c（In-app 定時器 + 動工前凍結無人值守條件，否則自動退場 V1-a）

- V1 實作「僅 App 開啟期間生效」的一次性延遲觸發；不引入 Windows Task Scheduler、系統服務或常駐工作列。
- **動工前必須凍結並寫入 V1 規格的條件（合併三席清單）：** 僅 App 開啟時觸發；App 關閉／睡眠造成 missed trigger 時不靜默補跑，以「Missed — 重新排程？」卡片呈現；同一 Task 已有 active Run 時不啟動；受 cap = 1；Ask 必停且逾時／關窗 = no-op；一律進第三欄人工 Review；無 Auto-complete；無 Auto-retry；V3 遠端不可逆類仍 Hard Block；UI 明示「關閉 App 不會執行」；missed-trigger 卡片的呈現語意隨上述條件一併定稿。
- **上述任一條件無法在動工前凍結 → 自動退場為 V1-a**（只保留 `Trigger` 資料模型型別欄位，V1 不執行任何 schedule trigger）。
- 無論結果為 V1-c 或退場 V1-a，資料模型均保留 `Trigger` 型別欄位；Gate 2 需求探測題目必須包含排程需求，且 One-time 與 Recurring 分開詢問。

### D2 — Steering：採 V2-x（Brunel 稽核通過即可納入 V1），以 V2-y 支持者提出的驗收條件為納入門檻

- Steering 訊息注入列為 **Gate 1 必測項**。
- **Chair 評估：** Brunel 以 Pi Agent 為基礎，Pi 已具備 Steering 能力，故 Brunel 預期支援 Steering、Gate 1 此項預期通過、Steering 預期進入 V1。以下六項驗收條件仍須於 Gate 1 逐項確認；任一未通過則按降級路徑處理，且不阻塞 V1 交付。
- 納入 V1 的門檻為以下全部成立（任一不滿足即「未通過」，採全有或全無判定，曖昧 = 未通過）：runtime 發出 accepted／applied／rejected 明確事件；訊息只在明示安全點生效；已開始的破壞性工具不被 UI 誤示為已取消；每次請求帶 `command_id`，可回報 rejected／timeout／runtime death；crash／重連後不重複套用；Cancel 不依賴 Agent 自願配合。
- 未通過 → 降級為 Cancel + 結構化 handoff（goal／constraints／已做決策／base revision）+ 開新 Run，且**不阻塞 V1 交付**。
- 納入 V1 時，UI 與對外敘事不得暗示即時生效：steering 訊息顯示「已送出，將於安全點套用」狀態，不顯示為已生效；不與 Pause／Resume 共用或暗示無損續跑。
- 若 Product Owner 改採 V2-y：Steering 明確非 V1 交付，列為「Gate 1 通過後第一批恢復清單」第一項，其餘同上。

### D3 — 破壞性操作凍結 denylist：逐條照收 `vote.md` V3 定稿表（六類 Hard Block + 三項 carve-out + journal 分開記錄）

- 六類語意 Hard Block、Block 不可被 Agent／Run 級一次授權／「Full Access」偏好／Schedule 覆蓋；使用者須離開 Agent 流程手動執行；Taylor 經明確 UI 啟動、對已記錄 checkpoint 的 recovery primitive 不等同 Agent 取得這些工具權限。
- carve-out：`git reset --hard`／`clean -fdx` 於主 workspace 或未知 cwd 的 Agent 原始命令 Block，僅 Taylor Recovery primitive 於 disposable worktree 可執行；授權 worktree 內大量刪除須 Confirm + 執行前 checkpoint + canonical path manifest + blast-radius 上限，出界或穿越 symlink 立即 Block。
- **跨席落地前提（一致提出）：** 第 6 類「秘密外洩與未授權資料傳輸」及一般 exfiltration 的可執行性，取決於 V1 是否提供足以判定 network destination 與 outbound data class 的受控工具。**裁定：V1 不給 Agent 不受限的網路 shell；工具層無法判定目的地與資料類別時 fail closed。** 此項列 Gate 1 必測。

### D4 — 整包 V1 範圍：批准 `vote.md` V4 的 14 項，並採納兩則修正

**修正 A（第 6 項，safety-engineer 提，ux-designer round-2 同指）：** `Needs Input`／`Waiting Permission` 為非終止 Run 狀態，卡片留在第二欄（執行中）並置頂；僅 `Failed／Interrupted`／`Needs Review`／`Completed` 進第三欄並依注意優先度排序。不得因顯示位置把仍持有 runtime／worktree 的 Run 標為 Completed、釋放 runtime、觸發新 Run、開始 Retry 或視為已核准。核准仍只存在 Run detail inline card。仍不建立獨立 Review Inbox、維持單一真相來源；升級為獨立入口的觸發條件寫入本決議（見 Consequences）。

**修正 B（第 12 項，systems-architect 提）：** 「Embedded Portable Node.js」與 Job Object `KILL_ON_JOB_CLOSE` 不作為已驗證、不可替換的產品承諾；改述為「由 Gate 1 驗證的可攜 Node distribution 與 child-process containment 策略」。Gate 1 失敗時可改用等效且同樣可觀察、可終止、可打包的實作，不得因此自製 runtime 或阻塞整包。第 12 項的可觀測契約部分（versioned／bounded／backpressured framed stdio、每則事件帶 run id + sequence id、unknown schema fail-closed、journal reconcile、無 runtime identity 的 Running 標 Interrupted）仍必須保留。

**其餘第 1–5、7–11、13–14 項照 `vote.md` V4 原文批准。** 多席明列為承重、增刪其餘項時不得動搖：第 5 項（cap = 1、放寬須雙證據、不可由 Agent 或一般設定調高）、第 9 項（§34 強制門檻）、第 10 項（兩道 go/no-go 閘）、第 14 項（Ask 逾時／關窗 = no-op、不自動 Allow）。

## Consequences

### 預期結果

- V1 範圍相對 34 節基準顯著收斂為「人啟動、單一自主 Run、強制 Review、程式層邊界、誠實降級」的最小可恢復閉環。
- 移出 V1：完整 Recurring 叢集、Retrieval／Task Memory／偏好子系統、Retry／Pause／Resume、獨立 Review Inbox、自製 Checkpoint runtime、Resource Governor 優先權分層、N>1 並行、「接近 Full Access」表述。
- §14 Brunel 由「無條件固定」改為「首選 execution adapter，須通過整合原型」。
- §34 Feature Freeze 升級為強制門檻：每個 V1 保留項須同時（a）對應九判準之一（b）指名一條具體現有使用流程並說明移除後於哪一步失效；抽象對應不通過；例外僅由 Product Owner 於 ADR 核准；PR 只需 trace 到已通過門檻的保留項。

### 代價與殘餘風險（各席 Risks 彙整）

- **V1 automation 故事變薄**：V1-c 條件凍結不成即退 V1-a；cap = 1 可能被體感為「序列 queue」（緩解：待辦欄卡片顯示「排隊中（第 N 位）」，Cancel 排隊任務零成本）。若 Gate 2 顯示使用者要的就是自動化，差異化延後——接受為殘餘風險，探測閘置於動工前而非交付後。
- **denylist 摩擦**：force push、`reset --hard` 等回到人手，離開 Agent 流程手動執行；合法工作流會較常遇 Confirm（緩解：V1 驗收含一次確認頻率的可用性檢查，資料來源為 policy journal）。
- **denylist 第 6 類可能落空**：若受控網路工具未到位，程式層無法可靠執行；已裁定 fail closed、不給不受限網路 shell，並列 Gate 1 必測；仍殘留「一般建置工具的隱藏網路副作用」。
- **Embedded Node 供應鏈面**：安裝包自帶 Node runtime 增加版本／簽章／更新面，若無固定版本 + hash／簽章驗證 + 受控更新，Brunel sidecar 可成能力繞過點——列 Gate 1，不交給 Agent 判斷。
- **Gate 延遲**：Gate 1 若延期，UX 狀態模型與多項契約設計懸置——接受為殘餘風險，閘門本身是防更大範圍錯誤的機制。
- **V2-x 曖昧稽核**：Brunel 部分支援時有納入壓力——以全有或全無判定、六項驗收條件硬性把關，曖昧 = 未通過。

### 後續動作（動工前）

1. 凍結 D1 的 One-time 條件文件（含 missed-trigger 卡片呈現語意）；凍不成則 V1 規格直接寫 V1-a。
2. 撰寫 Gate 1 稽核測試規格，至少涵蓋：工具執行前政策攔截 hook（第一項）、normalized event stream、cooperative cancellation + 可強制終止的獨立行程樹、Steering 六項驗收條件、versioning／schema handshake、compaction ownership、秘密不經 IPC log 洩漏、可攜 Node distribution 與 containment 策略、denylist 第 6 類所需的受控網路工具與出網 policy 判定、Evidence（§23）tests／build 欄位的資料來源（Brunel 事件 vs Taylor 程式層檢驗）。
3. 撰寫 Gate 2 需求探測問卷：目標使用者、目前工作流、是否需要無人值守／One-time／Recurring／N>1／集中注意力入口、可接受何種 Review；One-time 與 Recurring 分開問。
4. 凍結第 13 項 Session 政策剖面：Session 是否預設使用主 workspace 或可選 Taylor-owned worktree；變更前復原點採何種 Git 基線；如何處理既有 dirty state。
5. 記錄 Review Inbox 升級為獨立入口的觸發條件：**並行上限開放至 ≥3 或 Scheduled Task 進入 V1 後，以使用者實證的「從 Board 找不到待辦」為升級證據**（採 ux-designer round-2 條件）。
6. §34 強制門檻的 PR 審查機制：每個保留項附使用流程、九判準對應、替代方案與 Product Owner 裁定；缺 trace 即退回。

### 撤回條件

- Gate 1 的 policy hook 或最小 Run lifecycle 不成立 → V1 的自主 Run 閉環不得動工，回到閘前重新收斂，**不得自製完整 tool runtime**。
- Gate 1 其餘不支援項 → 對應功能從 V1 移除或以已說明的降級語意取代。
- Gate 2 若顯示使用者需要全自動 Recurring 或 N>1：**探測結論為裁決輸入，不自動推翻本 ADR**；重開 Recurring／並行需另立新議題與新投票（採 product-strategist、systems-architect、devils-advocate 一致主張）。

## Dissent

| 未採納立場 | 提出者 | 未採納原因 |
| --- | --- | --- |
| V1-a（只留 Trigger 欄位，V1 不執行任何 schedule trigger）為**首選** | systems-architect、safety-engineer | 投票 V1-c 得 3 票、V1-a 得 2 票。V1-c 內建「條件凍不成自動退場 V1-a」，V1-a 作為失敗路徑已被保留；兩席的無人值守條件已全數併入 D1 的凍結清單。systems-architect 明示 V1-c（含嚴格限制）為其最低可接受條件；safety-engineer 明示 V1-c 為其 V1-a 落選後的方案。 |
| V1-d（提醒卡，不自動執行） | devils-advocate | 得 1 票。提醒卡與手動建立 Task 的使用者行為幾乎無法區分，無法對「時間到自動啟動後使用者如何應對 Ask／Review」產生行為證據，使 Gate 2 排程探測缺實證輸入。其四條凍結條件已併入 D1；D1 的 V1-a 退場路徑滿足其最低可接受條件（「至少退到 V1-a」）。 |
| V1-b（In-app 定時器，無條件凍結條款） | （round-2 提出者 platform-architect、ux-designer） | 提出者於投票時均改投 V1-c（V1-b + 安全閥），V1-b 得 0 票，已被 V1-c 取代。 |
| V2-y（Steering 明確非 V1 交付，僅列 V1 後第一批恢復項） | systems-architect、devils-advocate | 投票 V2-x 得 4 票、V2-y 得 2 票。兩席提出的六項驗收條件已寫入 D2 作為 V2-x 的納入門檻；D2 明載「Brunel 稽核未通過時，Steering 行為等同 V2-y」。systems-architect 主張「未來 Gate 結果不應自動視為核准、應以新 ADR 重開 scope」已納入撤回條件。 |
| Steering 列為 V1 不可移除底線 | ux-designer（round 1） | 提出者於 round 2 主動撤回，改為「使用者必須能中斷並以最低成本修正方向」為底線、Steering 本身列原型驗證。 |
| 改用 Electron、結束 Wails 懸缺 | platform-architect（round 1） | Chair 於議程中裁定維持 Wails v2（Wails v3 beta 不採用）。提出者接受，round 2 起轉為「Wails v2 前提下的殘餘風險設計」。 |
| V1 自主 Run 並行上限 = 2 | platform-architect（round 1） | 提出者於 round 2 主動收斂為硬上限 1，與 systems-architect／safety-engineer／devils-advocate 一致。 |
| 簡化版 Recurring 進 V1 | ux-designer（round 1） | 提出者於 round 2 撤回，接受完整 Recurring 叢集延後（附「延後決議須載明 Gate 2 探測題目」之條件，已納入 D1）。 |
| V4 第 6 項原文（所有 attention 狀態併入第三欄） | — | 依修正 A 調整：`Needs Input`／`Waiting Permission` 保留為非終止狀態、留在第二欄。safety-engineer 提出、ux-designer round-2 同指語意衝突。不改變「無獨立 Inbox、單一真相來源」的結果。 |
| V4 第 12 項原文（Embedded Portable Node.js 與 `KILL_ON_JOB_CLOSE` 為已驗證產品承諾） | systems-architect | 依修正 B 調整為「Gate 1 驗證的 Node distribution 與 containment 策略」。可觀測契約部分全數保留。 |
| 三席重疊的長程記憶（§18／§19／§20）作為長任務底線 | （round 1 潛在立場） | round 2 systems-architect 撤回 Taylor 側 Retrieval／Task Memory 主張，devils-advocate 註記「此異議已關閉」；全席收斂為僅留每 Run 結構化 handoff。 |

> Dissent 完整性：本表已記錄每個未採納立場、提出者與未採納原因。缺少 Dissent 的決議不得複製至 `agora/decisions/`。
