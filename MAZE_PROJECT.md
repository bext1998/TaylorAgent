# MAZE_PROJECT — Taylor Agent 定位與工作流設定

> 由 `maze-project-init` 建立。Agent 讀取規格前必須先由此取得實際路徑。
> 文件搬移或設定變更時才更新；不得記錄 token、API key、密碼或私密憑證。

## 專案資訊

- 專案名稱：Taylor Agent
- 目標工具：Codex
- 建立日期：2026-09-11

## 文件

- Spec：docs/taylor-agent-v1-spec.md
- Project Brief：PROJECT_BRIEF.md
- Next Action：NEXT_ACTION.md
- Decisions：DECISIONS.md

## 自適應 Guidance

- Default profile：standard
- Model overlay：none
- Host capabilities：Codex、PowerShell、GitHub CLI；子代理依執行環境規則使用。
- Profile escalation evidence：只有發生具體失敗時記錄。

## GitHub

- Repository：https://github.com/bext1998/TaylorAgent
- Issue tracking：enabled
- Spec to Issues：enabled
- Priority label convention：P0、P2、P3、P4、P5
- Category label convention：未設定
- Default assignee policy：bext1998
- Allow label creation：yes

## 備注

- 所有 Git Worktree：`D:\AgentCoding\.codex\worktrees\TaylorAgent`
- Worktree 分支格式：`maze/YYYY-MM-DD-short-hash`；`short-hash` 為隨機值，結尾不得附加其他字樣。
- Issue 不記錄 spec revision 或驗證碼；專案沒有 CI 測試時，不額外加入驗收條件。
