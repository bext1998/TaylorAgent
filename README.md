# Taylor Agent

Taylor Agent 是 Windows 桌面 Agent 工作控制平面：以 Session、Task Board 與 Scheduled Task 管理工作，並由程式邏輯統一處理 Run 生命週期、Git Worktree、安全權限、復原點與工作歷史；實際 coding 執行委派給 Taylor Core（codename: Brunel）。

## V1 範圍

- Session Mode：持續的對話式 coding 工作。
- Task Board：待辦、執行中、待人工核准三欄工作流。
- 單一自主 Run：V1 並行上限固定為 1。
- Git Worktree：變更程式碼的 Run 使用獨立 Worktree 與 Git checkpoint。
- Permission Policy Engine：Allow／Ask／Deny/Confirm，未知工具預設拒絕。
- Completion Evidence 與人工 Review。

V1 不實作 Manager LLM、Subagent Framework、自製 Agent Runtime、完整 Recurring 排程、N>1 工作並行或自主部署／合併。

## 架構

Taylor 採 Wails v2 桌面應用：Go control service 是唯一的 OS capability broker 與 process supervisor；WebView 只呈現狀態並送出使用者意圖。Agent 執行預設委派給 Taylor Core（codename: Brunel），其內部由 Go Host + Pi Agent 構成。Taylor Core 整合須通過 Gate 1 稽核後才能成為 V1 自主 Run 閉環的基礎。

## 重要規則

- 所有 Git Worktree 必須位於 `D:\AgentCoding\.codex\worktrees\TaylorAgent`。
- Worktree 分支名稱必須為 `maze/YYYY-MM-DD-short-hash`；隨機短雜湊後不得附加其他字樣。
- V1 的破壞性操作由程式層安全政策裁決，Agent 不得自行提權。
- GitHub Issue 使用 spec-to-issues；專案沒有 CI 時，Issue 不加入額外驗收條件、spec revision 或驗證碼。

## 文件

- [V1 規格](docs/taylor-agent-v1-spec.md)
- [V1 範圍決議（AGORA-001）](docs/001-taylor-core-scope.md)
- [專案工作流設定](MAZE_PROJECT.md)
- [下一步行動](NEXT_ACTION.md)
- [有效決策索引](DECISIONS.md)

## 動工前 Gate

1. Gate 1：稽核 Taylor Core（Brunel）的 policy hook、最小 Run lifecycle、取消、Steering、IPC、秘密保護與受控網路能力。
2. Gate 2：蒐集目標使用者、One-time／Recurring 排程與 N>1 並行需求。

## License

[MIT](LICENSE)
