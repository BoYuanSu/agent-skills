---
name: bysu-build-ui-template
description: 在 Vue 元件 Props 契約就緒後，實作語意化 DOM 階層與 SCSS 空樣式骨架，使其能以 Mock 資料獨立渲染。
---

# UI 優先 - 單一元件 HTML/CSS 骨架 (bysu-build-ui-template)

## 概覽
本技能專注於「兩端先行，延後造橋」中 **「UI 先行 (UI-First)」** 的單一元件骨架開發階段。在 Props/ViewModel 已定義的前提下，建立語意化 DOM 階層 (`<template>`) 與樣式隔離的 nested class 骨架 (`<style>`)，確保 UI 與後端邏輯完全解耦。

## 單獨執行模式 (Skill-Only)
- 當使用者明確呼叫 `bysu-build-ui-template` 時，預設僅執行本技能。
- 專注於 **單一元件** 範疇，若超出本技能邊界，完成本技能後，以單句提出後續建議。

## 工作流程與完成條件

1. **對齊輸入與契約**
   - 讀取畫面視覺（截圖/線框/描述）與現有的 Vue SFC 檔案。
   - *完成條件*：SFC 檔案存在，且 `<script setup>` 中已定義好對應的最理想 Props / ViewModel 結構。

2. **解析畫面並建立語意化 DOM 階層**
   - 識別單一根節點，並劃分主要結構區塊（優先使用 HTML5 語意化標籤，如 `article`, `header`, `section`, `ul`, `li`, `button` 等）。
   - *完成條件*：產出無冗餘 wrapper、層級扁平且具語意化的 DOM 結構。

3. **綁定 Props/ViewModel 到模板**
   - 將文字/屬性綁定至 Props；列表渲染 (`v-for`) 提供穩定 `:key`；可選欄位加上 `v-if` 保護。
   - *完成條件*：所有 Props 與靜態 Mock 資料完全對應並綁定完成，且可選欄位無空節點渲染隱患。

4. **生成空殼樣式 (Skeleton Styling)**
   - 依據根類名，在 `<style scoped lang="scss">` 中建立 nested class selectors 空 blocks。
   - *完成條件*：SCSS 類別結構與 template 完全對應，且所有 block 皆為空 block（無任何 CSS 屬性，如 `color`、`padding`）。

## 程式碼規範與防呆 (Guardrails)

- **單一元件專注**：僅處理當前元件。若視覺上有子元件，以靜態 DOM 結構呈現，不在此階段進行拆分或建立新元件檔案。
- **樣式空殼化**：SCSS 僅做名稱空間隔離（Name Scoping），不填入任何 CSS 屬性。
- **資料解耦**：僅消費傳入的 Props 或 minimal local state (如 `ref(false)`)，不寫入任何真實 API 請求或資料轉換邏輯。
- **SFC 代碼風格**：遵循 [AGENTS.md](file:///Users/takosu/Desktop/projects/boyuansu/agent-skills/AGENTS.md) 的 Vue SFC 規範（採 `Block__elementName` 命名，SCSS 中禁用 `&__` 縮寫）。

*詳細的樣式與 DOM 啟發式規則（Heuristics）請參考 [references/template-dom-hierarchy-reference.md](references/template-dom-hierarchy-reference.md)。*
