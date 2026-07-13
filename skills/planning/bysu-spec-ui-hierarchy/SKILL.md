---
name: bysu-spec-ui-hierarchy
description: 適用於 UI 先行 (UI-First) 階段，規劃元件樹與理想資料契約 (View Model Contract)，並生成無狀態 Vue SFC 與 Composable 骨架。
---

# UI 優先 - 元件樹與理想資料契約設計 (bysu-spec-ui-hierarchy)

## 概覽
本技能落實「兩端先行，延後造橋」的 **「UI 先行 (UI-First)」** 理念。在開發初期完全解耦後端 API，專注於 UI 呈現與互動需求，逆推設計出對 UI 渲染最友善的 **理想資料契約 (View Model Contract)**，並以此建立元件階層樹與 Mock 資料，輸出無狀態的 Vue SFC 及 Composable 骨架。

## 雙階段審查決策 (Review Gate)
依據使用者的呼叫情境執行對應階段，未取得 Phase 1 確認前，不建立任何程式碼檔案：
- **Phase 1 (審查提案)**：僅產出 `Hierarchy Tree` 與 `View Model Mock Data`。並於結尾以一句話明確詢問（例如：「請確認元件樹與 View Model 結構是否可進入 Phase 2 骨架輸出？」）。
- **Phase 2 (交付骨架)**：當使用者確認通過，或在非互動模式下，始輸出完整內容（包含骨架檔案與實作序列）。

## 工作流程與完成標準 (Workflow & Completion Criteria)

1. **專案與需求範疇釐清**
   - 識別目標框架、路由模型、狀態管理規範與裝置斷點。
   - *完成標準*：明確表列出目標框架（如 Vue 3 setup）、主要路由架構與 RWD 斷點。
   
2. **UI 拆解與元件樹規劃**
   - 從路由／頁面層級 of 容器 (Container) 開始拆解至功能區段 (Sections) 與展示型葉節點 (Leaf Components)。控制階層深度於 3-6 層內，避免過度切出微型元件。
   - *完成標準*：產出 Mermaid 樹覆蓋所有視覺區段，層級結構清晰合理。

3. **設計理想的 View Model (理想資料契約) 🌟**
   - 完全不考慮後端 API 結構，從 UI 渲染的便利性出發，定義最直覺的資料結構與 Mock Data。
   - *完成標準*：提供完整的 JSON 範例，所有欄位皆已為 UI 轉換完成（如已格式化之顯示標籤，非 API 原始欄位）。

4. **元件契約與邊界定義**
   - 定義各元件的 Props 介面（展示型優先使用 primitive-based props，容器層使用 View Model 物件）與 Emits 事件，並規劃狀態擁有權。
   - *完成標準*：明確定義出各元件的 Props（含型別與 default）與 Emits。

5. **建置計畫與骨架產出**
   - 產出元件骨架檔案，僅包含型別簽名與靜態 DOM，無邏輯實現。
   - *完成標準*：提供由外殼至葉元件的建置順序，且產出的骨架可正常編譯。

## 輸出格式

### Phase 1：僅提供設計提案
1. **Hierarchy Tree**：Mermaid `graph TD` 元件階層樹。
2. **View Model Mock Data**：以 JSON 格式提供理想資料結構與 Mock 範例。
3. **確認提問**：在結尾附上一句明確的 Phase 2 授權詢問。

### Phase 2：完整交付內容
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
- 必須嚴格遵循根目錄 [AGENTS.md](../../../AGENTS.md) 的 `Vue SFC Code Style（Single Source）` 規範。
- 骨架代碼應保持完全靜態與無狀態 (Stateless)，僅包含基礎的 TypeScript 型別宣告與靜態元件標記。
- 優先在展示型葉節點使用 primitive-based props (string, number, boolean)；對容器/區段組件，可使用設計好的 View Model 物件。
- `<template>` 僅保留一個 root element，class 為元件名稱（如 `.DemoComponent`），內部放置元件名稱文字。
- `<style scoped lang="scss">` 僅保留 root class style block，不填寫任何 CSS 屬性。

### Composable 骨架規則
- 檔名採 `use{FeatureName}.ts`，只做 Named Export。
- 定義傳入參數與回傳型別，不實現內部邏輯，僅回傳 Mock 資料或空實作，確保骨架可編譯。

## 品質規則與防呆
- **不要為單次使用提早抽象**。
- **不把後端協定欄位直接當 UI 契約**：UI 元件僅消費最理想的 View Model。
- 骨架輸出應「最小可編譯」，避免任何超出模板需求的預先實作。
- 若專案為 KISS 簡裁架構（簡單頁面），則直接省略 Normalization 考量，合併 State & Request，但仍需維持 UI 與資料獲取邏輯解耦。

詳細範本與反模式檢查請參考 [references/component-hierarchy-reference.md](references/component-hierarchy-reference.md)。
