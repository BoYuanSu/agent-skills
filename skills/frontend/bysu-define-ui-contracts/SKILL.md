---
name: bysu-define-ui-contracts
description: 在 Web App 功能開發的第 4 階段，依 Component Hierarchy、BDD use cases 與資料流，在功能程式碼中定義 API 無關的 UI Schema、Vue Props、Callback 或 Emits 合約及必要 mock data。不建立重複的 Markdown 規格。
---

# 定義 UI Schema 與元件介面

## 前置資料

讀取該功能的 `component-hierarchy.md`、`use-cases.md`、`data-flow.md`，並檢查現有功能程式碼與專案慣例。若程式碼放置位置無法由現有結構判定，先向使用者確認。

## 執行方式

1. 從畫面顯示與互動需要出發定義 UI Schema；名稱與型別表達 UI 意義，不沿用 API response 的偶然欄位結構。
2. 依資料流為各元件定義最小 Props Data 與 Callback／Emits 合約。
3. 區分必要與可選資料，只有畫面確實允許缺值時才使用 optional。
4. 對載入、空資料、錯誤、權限與互動狀態使用足以避免矛盾狀態的最小表示方式。
5. 必要的 mock data 放在功能程式碼附近，型別必須符合 UI Schema。
6. 直接修改 TypeScript／Vue 檔；不要新增 UI Schema、Props 或 Callback 的 Markdown 副本。

只建立本階段所需的型別與合約。除非使用者同時要求，不串接 API，也不完成整個 UI。

## 檢查

- UI 元件只依賴 UI Schema，不匯入 API response 型別。
- Props 與事件能對應 `data-flow.md` 的向下資料與向上事件。
- use cases 所需的 UI 狀態都能被合約表達。
- mock data 不引入正式介面之外的欄位。
- 執行與變更風險相稱的型別或測試檢查；依專案限制選擇命令。
