---
name: bysu-map-component-hierarchy
description: 在 Web App 功能開發的第 1 階段，將已確認的 UI 設計稿或 Wireframe 整理成以頁面、區塊、可重用元件與純展示元件組成的 Component Hierarchy。此 skill 不處理 API、資料流或實作細節。
---

# 建立 Component Hierarchy

## 前置資料

讀取同層的 `input-review.md` 與 `ui/` 相關素材；路徑應相對於使用者提供的 `docs/` 目錄，而不是固定的 `specs/{FeatureName}`。若畫面範圍仍有阻斷問題，先回到 `$bysu-review-feature-inputs`。

## 執行方式

1. 依實際畫面辨識 Page、Section、可重用元件與純展示元件。
2. 以畫面責任與可獨立理解的區塊劃分元件，不因單一標籤或容器機械式拆分。
3. 只有在多處具有相同責任與行為時標記為可重用元件；沒有證據時保留在所屬功能內。
4. 保留條件顯示、清單項目與插槽等會影響階層理解的關係，但不加入 Props、事件、API 欄位、狀態管理或檔案路徑。

## 產出

建立或更新與 `api/`、`requirements/`、`ui/` 同層的 `component-hierarchy.md`，內容包括：

- 涵蓋的畫面與必要說明。
- Mermaid 元件樹。
- 各 Page、Section 與元件的單一句責任說明。
- 無法由 UI 素材判定、且會影響階層的待釐清事項。

Mermaid 節點名稱應穩定且可在後續資料流文件中直接沿用。文件只描述結構，不放實作細節。

## 完成條件

- 所有範圍內畫面都能對應到元件樹。
- 每個元件只有清楚的畫面責任，父子關係符合 UI 結構。
- 後續資料流可使用相同元件名稱，不需重新命名或猜測。
