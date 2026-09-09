---
name: bysu-design-component-data-flow
description: 在 Web App 功能開發的第 3 階段，結合 Component Hierarchy 與 BDD use cases，描述資料由哪個元件持有、向下傳遞的資料、向上回傳的事件，以及本地狀態與 API 往返。此 skill 不定義 TypeScript 型別。
---

# 設計元件資料流

## 前置資料

讀取 `specs/{FeatureName}/component-hierarchy.md` 與 `specs/{FeatureName}/use-cases.md`。兩者若不完整或使用不同的畫面範圍，先回到對應階段修正。

## 執行方式

1. 沿用 Component Hierarchy 中的元件名稱。
2. 逐一檢查 use case，找出需要持有的 API 資料、本地 UI 狀態與使用者事件。
3. 將狀態放在能完成該流程的最低共同擁有者；只有多個分支需要共享時才提高持有位置。
4. 標示父元件向子元件傳遞的資料，以及子元件向上回傳的使用者事件。
5. 有 API 往返時，標示觸發元件、請求、結果回到何處，以及載入、成功、空資料與錯誤如何改變畫面。
6. 保持概念層級，不在此階段定義欄位型別、Props 語法、事件簽章、store 實作或 Adapter 轉換。

## 產出

建立或更新 `specs/{FeatureName}/data-flow.md`，內容包括：

- 每項資料或狀態的擁有元件與理由。
- 向下資料與向上事件的對照。
- 以 Mermaid flowchart 表達的主要互動與 API／本地狀態流向。
- use case 與資料流的對應，讓主要情境沒有遺漏。

## 完成條件

- 每個主要 use case 都能沿著元件與資料流完成。
- 資料只有一個明確擁有者，沒有無理由的重複狀態。
- API 格式尚未滲入 UI 元件語彙，且下一階段可據此定義 UI Schema 與介面。
