# Taylor Agent — Coding Agent 指令

> 本文件供 Codex 在每個 session 開始時閱讀。

---

## 專案概述

Taylor Agent 是 Windows 桌面 Agent 工作控制平面：以 Session、Task Board 與 Scheduled Task 管理工作；實際 Agent 執行委派給 Taylor Core（codename: Brunel）。

技術棧：Go + Wails v2 + TypeScript/WebView；持久化方案待 Gate 與 Product Owner 凍結。

---

## 工作原則

1. 只實作任務要求的功能，不添加額外功能或重構。
2. 優先編輯現有檔案，只在嚴格必要時建立新檔案。
3. GitHub Issue／PR 與 Git 是工作狀態權威；只有明確 closeout 才重建 `NEXT_ACTION.md`。
4. Git 操作前確認 `pre-commit-checklist.md`。
5. 使用 spec-to-issues 建立 Issue 前，先確認專案是否有 CI 測試；沒有 CI 時，不在 Issue 加入額外驗收條件。
6. Issue 不記錄 spec revision 或驗證碼；預設 Assignee 為 `bext1998`。

## Git Worktree 規則

1. 所有 Git Worktree 必須集中建立於 `D:\AgentCoding\.codex\worktrees\TaylorAgent`，不得使用其他目錄。
2. 建立 Worktree 的分支名稱必須為 `maze/YYYY-MM-DD-short-hash`。
3. `short-hash` 必須是隨機短雜湊值；字尾不得加入任務名稱、使用者名稱或其他字樣。
4. 建立前確認目標目錄與分支不存在，並確認 Worktree 指向正確 repository。

## 下一步

閱讀 `NEXT_ACTION.md` 了解這個 session 的目標。

## 重要文件

| 文件 | 用途 |
|---|---|
| `docs/taylor-agent-v1-spec.md` | V1 功能規格與驗收來源 |
| `NEXT_ACTION.md` | 下一步行動 |
| `DECISIONS.md` | 有效重大決策索引 |

## 禁止行為

- 不得 force push 到 main / master。
- 不得在使用者未確認前 commit 或 push。
- 不得修改 `docs/taylor-agent-v1-spec.md` 的功能範圍，除非使用者明確要求。
- 不得在 `D:\AgentCoding\.codex\worktrees\TaylorAgent` 以外建立 Git Worktree。
