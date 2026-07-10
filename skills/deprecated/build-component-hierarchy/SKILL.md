---
name: build-component-hierarchy
description: 根據功能需求、使用者流程與線框圖建立並驗證前端元件階層，並輸出符合專案規範的 Vue SFC 骨架元件。當 Codex 需要將 UI 拆解為父子元件樹、定義責任邊界，並直接產出可落地的元件檔案骨架時使用。
---

# 建立元件階層

## 概覽
將產品需求轉換為清晰的元件樹，並為每個節點定義責任。最終輸出以「Vue 元件骨架」與「Composable 邏輯骨架」為主，讓開發可在一致模板上填入實際內容。

## 單獨執行模式（Skill-Only）

- 當使用者明確點名 `$build-component-hierarchy` 時，預設只執行本技能交付物。
- 採兩階段交付（review gate）決策流程：
  - 在互動式對話中，預設採兩階段交付：第 1 階段只輸出 hierarchy tree（Mermaid `graph TD`）供使用者 review；第 2 階段於使用者確認後再輸出其餘內容（scaffold/contract 等）。
  - 在非互動式批次任務，或使用者已在 Prompt 中明確要求直接產出/不需等待確認時，應跳過 review 門檻，一次輸出 Phase 1 與 Phase 2 內容。
  - 在未確認 hierarchy tree 前（且需要兩階段審核時），禁止產生或修改任何程式碼檔案。
- 未被明確要求前，禁止進入功能實作、互動邏輯、跨檔案串接與額外重構。
- 若需求超出本技能邊界，先完成本技能交付，再以一句話提出可選下一步，不自動延伸執行。

## 工作流程
1. 釐清範圍與限制。
   - 識別目標框架、路由模型、狀態管理函式庫、設計系統規範與裝置斷點。
   - 識別非功能性限制，例如效能目標、SSR/CSR 需求、無障礙要求與分析事件。

2. 從最上層外殼到葉節點拆解 UI。
   - 先從路由／頁面層級的容器開始。
   - 將每個容器拆成功能區段。
   - 再把區段拆成可重用區塊與展示型葉節點。
   - 階層深度維持在可實務落地的範圍（通常 3-6 層），避免過早切出微型元件。

3. 為每個元件分配責任與邊界。
   - 定義每個元件的 props 介面與元件定位（容器／展示）。
   - 定義輸入 props 與對外發出的事件／callback（僅描述，不在骨架內實作）。
   - 定義資料抓取位置，以及 loading/error 的歸屬。
   - 定義商業邏輯與呈現邏輯的分界（評估是否需抽離成 Composable/Hook）。

4. 區分可重用元件與功能專屬元件。
   - 僅在至少有兩個合理重用場景時，才將元件提升為 shared/common。
   - 功能專屬抽象應保留在區域內，以降低耦合。
   - 優先採用組合（composition），而非繼承式抽象。

5. 定義具體的實作計畫。
   - 提供由外殼到葉節點的建置順序。
   - 標示跨元件的相依關係與阻塞點。
   - 為每個元件產出標準化 Vue SFC 骨架（只含最小必要結構），以及對應的 Composable 邏輯骨架。

6. 在定稿前驗證元件階層。
   - 檢查每個節點是否符合單一職責。
   - 檢查 prop 流向是否正確，避免循環依賴。
   - 檢查狀態放置是否能降低 prop-drilling，並避免隱性全域耦合。
   - 檢查命名一致性與目錄結構可行性。

## 輸出格式
在互動式 Skill-Only 且無跳過指示時，預設分為兩階段：
1. `Phase 1 (Review Required)`：僅輸出 `Hierarchy Tree`（Mermaid `graph TD`），最後加一句明確確認問題（例如：`請確認這個 hierarchy tree 是否可進入下一階段？`）。
2. `Phase 2 (After Approval / Direct Mode)`：在使用者確認後（或在直接產出模式下），依以下順序輸出完整內容。

Phase 2 請依以下順序輸出，以維持一致性：
1. 以 Mermaid `graph TD` 格式呈現 `Hierarchy Tree`。
2. 提供 `Component Contract Table`，包含元件名稱、父元件、子元件、擁有狀態、關鍵 props、發出事件、依賴的 Composables 與備註。
3. 提供 `Component Scaffold Files`：每個元件輸出一個 Vue SFC 骨架，格式必須符合以下規則。
4. 提供 `Composable Scaffold Files`（可選）：若定義了邏輯抽離的 hooks，輸出其 TypeScript 骨架（`.ts` 檔，僅含簽名，不含邏輯）。
5. 提供編號式的 `Implementation Sequence` 建置計畫。
6. 提供 `Risk Notes`，列出可能高變動點與可簡化選項。

### Component Scaffold Files 規則
- Vue SFC 通用風格（`script setup`、props/template 綁定、emits 型別、class 命名）一律遵循根目錄 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。
- 必須宣告 `{ComponentName}Props` 介面；先定義介面即可，不需填入完整欄位。
- props 設計偏好：對專案架構的展示層（Leaf 元件、純展示組件），優先使用 primitive-based（例如 `string`、`number`、`boolean`）；但對於容器或區段組件，可適時放寬，允許使用 domain-specific object 傳遞，以避免嚴重的 prop-drilling。
- `<template>` 僅保留 root element，class 名稱為元件名（例如 `.ExampleComponent`），不實作實際內容。
- `<style lang="scss" scoped>` 僅保留 root class style block，不填具體樣式。
- 不要加入 API 呼叫、watch/computed、emit 實作、slot 細節或子元件組裝。

### Composable Scaffold Files 規則
- 檔案名稱採 `use{FeatureName}.ts`。
- 輸出對應的 TypeScript 骨架，定義 export 函數、必要的傳入參數型別與回傳型別。
- 不實現內部具體邏輯，僅回傳 mock 資料或空實作，確保骨架為最小可編譯狀態。

當功能屬於中高複雜度時，請使用 [references/component-hierarchy-reference.md](references/component-hierarchy-reference.md) 中的範本與反模式檢查。

## 品質規則
- 保持元件高內聚；將橫切關注點移到 hooks/composables/services。
- 最小化可變共享狀態。
- 明確定義非同步邊界。
- 讓展示型元件維持無副作用。
- 對核心元件，優先使用明確介面，而非透傳 `...rest` 模式。
- 骨架輸出應「最小可編譯」，避免任何超出模板需求的預先實作。
