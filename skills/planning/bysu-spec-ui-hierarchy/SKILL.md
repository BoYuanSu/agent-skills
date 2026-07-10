---
name: bysu-spec-ui-hierarchy
description: 根據功能需求與 UI 設計，進行 UI 優先的元件拆分，並定義最理想、最易渲染的 View Model 數據契約，產出符合規範的 Vue SFC 與 Composable 骨架。當需要規劃前端 UI 結構、定義元件間職責邊界，並建立 Mock Data 結構時使用。
---

# UI 優先 - 元件樹與理想資料契約設計 (bysu-spec-ui-hierarchy)

## 概覽
本技能旨在落實「兩端先行，延後造橋」的 **「UI 先行 (UI-First)」** 理念。在開發初期完全解耦後端 API 結構，專注於 UI 呈現與互動需求。逆推設計出對 UI 渲染最友善、最直覺的資料結構（即 **View Model / Mock Data**），並以此結構建立元件階層樹與 Mock 資料契約，輸出 Vue SFC 及 Composable 骨架。

## 單獨執行模式 (Skill-Only)

- 當使用者明確呼叫 `bysu-spec-ui-hierarchy` 時，預設僅執行本技能之分析與交付物。
- **採兩階段交付 (Review Gate) 決策流程**：
  - **Phase 1 (Review Required)**：僅輸出 `Hierarchy Tree` (Mermaid `graph TD`) 與 `View Model Mock Data (JSON)` 供使用者審查。必須以一句話明確詢問（例如：「請確認此元件樹與 View Model 結構是否可進入 Phase 2 元件骨架輸出？」）。
  - **Phase 2 (After Approval / Direct Mode)**：當使用者確認或在批次/非互動（或明確指示直接產出）模式下，輸出完整內容（包含骨架檔案與實作計畫）。
  - 在未取得 Phase 1 確認前（且需要審核時），**禁止產生或修改任何程式碼檔案**。
- 未被明確要求前，禁止進入功能實作、真實 API 串接、資料轉換邏輯或跨檔案重構。

## 工作流程

1. **專案與需求範疇釐清**
   - 識別目標框架、路由模型、狀態管理規範與裝置斷點。
   
2. **UI 拆解與元件樹規劃**
   - 從路由／頁面層級的容器 (Container) 開始拆解。
   - 拆分為功能區段 (Sections)、可重用區塊 (Components) 與展示型葉節點 (Leaf Components)。
   - 控制階層深度（通常 3-6 層），避免過度切出微型元件。

3. **設計理想的 View Model (Mock Data 結構) 🌟**
   - **核心原則**：完全不考慮 API 結構，從 UI 渲染的便利性出發，定義最理想的 JSON 數據結構。
   - 確定各區塊渲染所需之標準化欄位與型別。
   - 設計出符合此結構的 View Model Mock Data (JSON 範例)。

4. **元件契約與邊界定義**
   - 定義各元件的 Props 介面（優先傳遞 View Model 或 primitive values，避免 prop-drilling）。
   - 定義對外拋出的事件 (Emits / Callbacks)。
   - 規劃狀態擁有權 (State Ownership)，決定 local 狀態與上提狀態。

5. **建置計畫與骨架產出**
   - 提供由外殼至葉節點的建置順序。
   - 為各元件產出標準化 Vue SFC 骨架，以及 Composable 邏輯/Hook 骨架（僅含型別簽名，不含邏輯）。

## 輸出格式

### Phase 1：僅提供設計提案 (Review Gate)
1. **Hierarchy Tree**：Mermaid `graph TD` 呈現元件階層。
2. **View Model Mock Data**：以 JSON 格式提供 UI 理想的數據結構與 Mock 範例。
3. **確認提問**：在結尾附上一句明確的確認詢問。

### Phase 2：完整交付內容 (確認後或 Direct Mode)
請依以下順序輸出：
1. **Hierarchy Tree & View Model Mock Data** (同 Phase 1 內容)。
2. **Component Contract Table**：
   | 元件名稱 | 父元件 | 子元件 | 擁有狀態 | 關鍵 Props | 發出事件 (Emits) | 依賴 Composables | 備註 |
   | -------- | ------ | ------ | -------- | ---------- | ---------------- | ---------------- | ---- |
3. **Component Scaffold Files**：符合下述 SFC 規則的代碼骨架。
4. **Composable Scaffold Files** (選用)：符合下述 Composable 規則的 `.ts` 骨架。
5. **Implementation Sequence**：編號建置順序。
6. **Risk Notes & Tradeoffs**：列出可能變動點與可簡化選項。

---

## 程式碼骨架規則

### Vue SFC 骨架規則
- 必須符合根目錄 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）` 規範。
- 預設包含 `<template>`、`<script setup lang="ts">`、`<style scoped lang="scss">` 三個區塊。
- 必須宣告 `{ComponentName}Props` 介面，並使用 `withDefaults(defineProps<{ComponentName}Props>(), { ... })`。
- 對展示型葉節點，優先使用 primitive-based props (string, number, boolean)；對容器/區段組件，可適時放寬，允許使用 domain-specific object (如 View Model 物件)。
- `<template>` 僅保留一個 root element，class 為元件名稱（如 `.DemoComponent`），內部放置元件名稱文字。
- `<style scoped lang="scss">` 僅保留 root class style block，不填寫任何 CSS 屬性。
- **嚴禁加入 API 呼叫、watch/computed、實作 emit 邏輯、或跨檔案依賴**。

### Composable 骨架規則
- 檔名採 `use{FeatureName}.ts`，只做 Named Export。
- 定義傳入參數與回傳型別，不實現內部邏輯，僅回傳 Mock 資料或空實作，確保骨架可編譯。

## 品質規則與防呆
- **不要為單次使用提早抽象**。
- **不把後端協定欄位直接當 UI 契約**：UI 元件僅消費最理想的 View Model。
- 骨架輸出應「最小可編譯」，避免任何超出模板需求的預先實作。
- 若專案為 KISS 簡裁架構（簡單頁面），則直接省略 Normalization 考量，合併 State & Request，但仍需維持 UI 與資料獲取邏輯解耦。

詳細範本與反模式檢查請參考 [references/component-hierarchy-reference.md](references/component-hierarchy-reference.md)。
