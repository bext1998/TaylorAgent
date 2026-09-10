# Taylor Agent — 下一步行動

> 僅保留當前有效前線；明確 closeout 時整體重建，不追加歷史。

## 下一個 Session 目標

完成 Gate 1 稽核測試規格，確認 Brunel 能否支援 Taylor V1 的最小 Run 閉環與政策裁決點。

## 行動（最多 3 項）

1. 撰寫 Gate 1 稽核測試規格，涵蓋 policy hook、事件流、取消、Steering、IPC schema、秘密保護與受控網路工具。
2. 凍結 One-time Schedule 條件；無法凍結時依規格退場為僅保留 Trigger 資料模型。
3. 撰寫 Gate 2 需求探測問卷，分開蒐集 One-time、Recurring 與 N>1 並行需求。

## 阻塞與待決策

- Brunel 需完成最小閉環後才能執行 Gate 1 稽核。
- Product Owner 需凍結 Session 政策剖面、風險分類與 One-time Schedule 條件。

## 權威連結

- [Taylor Agent V1 規格](docs/taylor-agent-v1-spec.md)
- [AGORA-001 V1 範圍決議](docs/001-taylor-core-scope.md)
