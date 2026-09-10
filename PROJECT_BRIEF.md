# Taylor Agent — 專案說明

> 建立日期：2026-09-11
> 最後更新：2026-09-11

---

## 一句話說明

Taylor Agent 是以 Windows 桌面應用管理 Coding Agent 工作、Git Worktree 與安全權限的控制平面。

## 核心問題

Coding Agent 的 Session、背景 Task、排程、復原點與權限判定常分散在不同工具。Taylor Agent 以程式邏輯統一管理工作生命週期，並把 Agent 執行委派給 Brunel。

## 技術棧

- **語言**：Go、TypeScript
- **框架 / 主要套件**：Wails v2、Brunel、Pi
- **資料存儲**：待 Gate 與 Product Owner 凍結；SQLite 為候選方案
- **目標平台**：Windows x64、PowerShell 7、WebView

## Coding Agent 工具

- **主要工具**：Codex
- **備用工具**：未設定

## 相關文件

- 規格書：[docs/taylor-agent-v1-spec.md](docs/taylor-agent-v1-spec.md)
- 下一步：[NEXT_ACTION.md](NEXT_ACTION.md)
- 決策紀錄：[DECISIONS.md](DECISIONS.md)

## 重要限制

- V1 自主 Run 並行上限為 1。
- Agent 執行由 Brunel 負責；Taylor 不實作 Manager LLM 或自製 Agent Runtime。
- Permission Policy Engine 採 Allow／Ask／Deny/Confirm，未知工具預設拒絕。
- 變更程式碼的 Run 必須在獨立 Worktree，且 Worktree 必須位於指定集中目錄。
