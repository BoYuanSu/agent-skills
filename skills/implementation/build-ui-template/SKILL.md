---
name: build-ui-template
description: 根據畫面設計、線框圖或描述，配合已定義 Props (或 View Model) 契約的 Vue SFC，實作單一元件的 <template> DOM 階層與 <style scoped> nested class 骨架，使其能以 Mock 數據獨立正確渲染。
---

# UI 優先 - 單一元件 HTML/CSS 骨架 (build-ui-template)

## 概覽
本技能專注於「兩端先行，延後造橋」中 **「UI 先行 (UI-First)」** 的具體單一元件開發階段。在元件的 Props 或 View Model 契約已定義（由 `spec-ui-hierarchy` 產出）的前提下，將畫面設計轉化為語意化的 HTML (`<template>`) 與樣式隔離的 nested class 骨架 (`<style>`)。

在此階段，元件的 `<script setup>` 僅消費 mock 數據或傳入的 props，**完全不涉及 API 請求與任何資料轉換/轉譯邏輯**，以確保 UI 呈現與後端完全解耦。

## 單獨執行模式 (Skill-Only)

- 當使用者明確呼叫 `build-ui-template` 時，預設只執行本技能。
- 本技能僅處理 **「單一元件」** 的 `<template>` 與 `<style>` 骨架，不擴展到其他元件、不拆分新元件、不涉及跨檔案串接。
- 未被明確要求前，**禁止新增真實行為實作、API 資料流邏輯、外部 API 請求、或轉譯邏輯**。
- 若需求超出本技能邊界，先完成本技能輸出，再以一句話提出可選的下一步。

## 工作流程

1. **對齊輸入與契約**
   - 讀取畫面（截圖、線框、描述）與現有的 Vue SFC 檔案。
   - 確認 `<script setup>` 已定義好對應最理想 View Model 的 Props 結構。
   
2. **解析畫面並建立語意化 DOM 階層**
   - 識別單一根節點 (Root Element)。
   - 拆出主要區塊，優先使用 HTML5 語意化標籤 (`article`, `header`, `section`, `ul`, `li`, `button` 等)，避開無意義的 `div` 堆疊。
   - 控制 DOM 深度，不為了排版而加入過多無意義的 wrapper。

3. **綁定 Props/View Model 到模板**
   - 文字、數字類 Props 綁定至對應的文字節點 (如 `{{ item.displayTitle }}`)。
   - URL/資源類 Props 綁定至 HTML 屬性 (如 `:src`, `:href`)。
   - 列表類 Props 使用 `v-for` 渲染，必須提供穩定的 `:key`（優先使用項目 ID）。
   - 對於非必填/可空屬性，在必要時加上 `v-if` 判斷，避免渲染出空節點。

4. **生成 Style Nested Class 骨架**
   - 基於根類名 (Root Class)，在 `<style scoped lang="scss">` 中補齊 template 中會用到的 nested class selectors。
   - **嚴禁在 style 內填寫任何 CSS 屬性**（例如 `padding`, `color`）。此時只做「結構化 class selector 與空 block 宣告」，以達成 CSS Name Scoped 的樣式隔離。
   - class 命名與 nested 寫法必須符合根目錄 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）` 規範。

5. **輸出前品質自查**
   - 核心 Props/Emits 契約未變動，僅允許因 UI 交互需要新增 minimal local state (如 `ref(false)`)。
   - 樣式檔只含有 nested selectors 空 blocks。
   - 沒有任何 API 請求或資料轉換代碼。

## 輸出格式

直接輸出一個完整且符合規範的 Vue SFC 檔案：
1. `<script setup lang="ts">`：保持核心 Props 契約，視情況加上 local UI 狀態。
2. `<template>`：語意化的 HTML 結構，綁定理想的 Props/ViewModel。
3. `<style scoped lang="scss">`：僅包含 nested classname selectors 的空 blocks。

---

## 程式碼規範與防呆 (Guardrails)

### Vue SFC 規範
- class 命名採 `Block__elementName` (BEM) 格式。
- **禁用 `&__element` 縮寫！** 在 SCSS 中必須寫出完整的類名（例如 `.ComponentName__title` 而不是 `&__title`）。
- 互動狀態（如 `active`, `selected`, `disabled`）可使用短 class 搭配。

### 嚴格禁令
- ❌ **嚴禁在 `<style>` 裡寫入任何 CSS 屬性**。
- ❌ **嚴禁把視覺上像子元件的區塊強行拆成新檔案**（此技能僅處理單一元件）。
- ❌ **嚴禁在 `<template>` 或 `<script setup>` 中寫入真實 API 請求或資料轉換邏輯**。

詳細範本與反模式檢查請參考 [references/template-dom-hierarchy-reference.md](references/template-dom-hierarchy-reference.md)。
