---
name: bysu-implement-web-feature
description: 在 Web App 功能開發的第 6 階段，依既定 UI 合約、Adapter 與 BDD use cases 實作及整合 Vue UI，先用 mock data 完成互動狀態，再串接實際 API，最後驗證主要流程與例外狀態。
---

# 實作與整合 Web 功能

## 前置資料

確認 Component Hierarchy、BDD use cases、資料流、UI Schema／元件介面及 Adapter 都已完成且彼此一致。若缺少合約，不在實作中自行決定，回到對應階段補齊。

## 執行順序

1. 依 Component Hierarchy 與 Props／Callback 合約建立或完成 UI 元件。
2. 先以符合 UI Schema 的 mock data 驗證主要互動與載入、空資料、錯誤、權限等適用狀態。
3. 串接 API client 與 Adapter，只把 Adapter 輸出的 UI Schema 提供給 UI 元件。
4. 依 `use-cases.md` 逐項驗證可觀察行為，補上能提供實質信心的元件、整合或端對端測試。
5. 發現需求、畫面、資料責任或 API 契約不一致時，停止擴寫實作，回到最早受影響的設計階段修正後再繼續。

## 邊界

- 維持既有 DOM、事件與條件渲染，除非設計產出明確要求變更。
- 只實作 use cases 與必要狀態要求的內容，不加入未要求的抽象、設定或附加功能。
- UI 元件不得直接讀取或判斷原始 API response。
- mock data、Adapter 規則與型別留在功能程式碼及測試附近，不建立重複 Markdown。
- 不額外建立交付說明文件。

## 完成條件

- 主要 use cases 及適用的載入、空資料、錯誤與權限狀態均已驗證。
- UI 只使用 UI Schema，API 差異集中於 Adapter。
- 相關測試與專案允許的檢查通過；依環境規則忽略不影響結果的警告。
- 實際行為與四份功能設計文件一致。
