---
name: build-template-dom-hierarchy
description: 根據畫面截圖、線框圖或視覺稿，搭配一個只定義 props 的 Vue SFC，分析單一元件 of UI 區塊並建立 `<template>` 的 DOM hierarchy。當任務要求「優先補齊 style nested classname（不寫樣式）並完成 template」、「保留既有 `<script setup>` 核心結構與 Props/Emits 契約不變」時使用，特別適合把設計稿快速轉成可落地的單元件結構與綁定骨架。

---

# Build Template DOM Hierarchy

## Overview

將視覺稿拆成單一 Vue 元件的結構化模板。生成 `<template>` DOM hierarchy，並在 `<style scoped>` 內補齊對應 template 的 nested classname（空 block，不填樣式，以做出 name scoped 隔離）；`<script setup>` 原則上保持核心 Props/Emits 契約不變。Vue SFC 通用命名與綁定風格一律遵循根目錄 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。

在中高複雜度畫面（多區塊、條件渲染、重複列表）下，先讀 [references/template-dom-hierarchy-reference.md](references/template-dom-hierarchy-reference.md) 再輸出。

## 單獨執行模式（Skill-Only）

- 當使用者明確點名 `$build-template-dom-hierarchy` 時，預設只執行本技能工作。
- 只處理單一元件的 `<template>` 與 `<style class skeleton>`，不擴展到其他元件或跨檔案串接。
- 未被明確要求前，禁止新增行為實作、資料流邏輯、API 串接、事件處理函式或額外重構。
- 若需求超出本技能邊界，先完成本技能輸出，再以一句話提出可選下一步，不自動延伸執行。

## Workflow

1. 鎖定輸入與範圍。
   - 讀取畫面（截圖、線框、設計描述）與既有 Vue SFC。
   - 明確限定任務為「單一元件」；不要拆檔，不要生成子元件。
   - 保持輸入 SFC 中的 `<script setup>` 核心結構與 Props/Emits 契約不變，僅在必要時允許新增與純 UI 互動相關的 local state。

2. 解析畫面區塊並建立層級草圖。
   - 先辨識 root container，再拆出主要區塊（header/body/footer 或 left/content/right）。
   - 對每個區塊建立語意化節點（`article`、`header`、`section`、`ul/li`、`button` 等）。
   - 控制深度在可讀且可維護範圍，避免無意義 wrapper。

3. 對應 props 到模板節點。
   - 文字類 props 綁到對應文案節點（`{{ title }}`）。
   - URL 類 props 綁到資源屬性（`:src`、`:href`）。
   - 列表類 props 使用 `v-for`，提供穩定 `:key`。
   - 可選 props 在必要時加上 `v-if` 守門，避免空節點。
   - 不得虛構不存在的 props；若視覺稿需要但 props 未提供，保留靜態占位內容。

4. 生成最小可用 template 與對應的 style nested classname。
   - 生成 `<template>` 結構，並保持 root class 與現有命名風格一致（優先沿用既有元件名或 BEM 區塊）。
   - 以 root class 為基準（例如 `.ComponentName`），在 `<style>` 中補齊對應的 nested class 結構（只新增 class selector 與空 block，不可加入任何 CSS 屬性，用以做出 name scoped 的樣式隔離）。
   - class 命名與 selector 寫法遵循 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。
   - class 清單需覆蓋 template 會使用到的所有結構節點。
   - 只輸出與畫面結構相關的 DOM 與必要指令（`v-if`/`v-for`/基本綁定）。

5. 進行輸出前檢查。
   - 確認 `<script setup>` 僅新增純 UI 顯示控制的 local state（如 `ref`），核心 props/emits 未變動。
   - 確認 `<style>` 只新增/調整 nested class selector，且沒有任何 CSS 屬性（保留空 block 實現 name scoped）。
   - 確認 `<style>` selector 寫法符合 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。
   - 確認模板有單一 root element，且 DOM 階層對應畫面。
   - 確認所有 binding 都來自既有 props、local UI state 或列表迭代變數。
   - 確認沒有引入額外元件、slot 協定、或跨檔案依賴。

## Output Contract

輸出完整 Vue SFC，規則如下：

1. 允許 `<template>` 與 `<style>`（僅 class skeleton）內容與原檔不同。
2. `<script setup>` 應保持核心 Props/Emits 契約不變，僅允許因應 UI 交互加入 minimal local state (如 `ref`)。
3. `<style>` 僅可包含 nested classname 的空 block，不可加入任何樣式宣告；selector 寫法需符合 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。
4. 若輸入原本已有 `<template>`，以新 DOM hierarchy 覆蓋。
5. 若畫面資訊不足，保持保守結構，避免過度猜測複雜業務邏輯。

## Naming Convention

- class 命名與 selector 規範一律遵循 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）`。
- 本技能只負責補齊「本元件實際會用到」的 class skeleton，不預先擴張未使用節點。

## Guardrails

- 不更改 props 介面、型別、預設值與註解。
- 不新增或刪除 `<script>`、`<style>` 區塊。
- 不在 `<style>` 裡撰寫 CSS 屬性（例如 `color`, `padding`, `display`）。
- 不把「視覺上可能是子元件」強行抽成新元件。
- 不預作複雜商業邏輯與 API 實作，僅允許為 UI 互動狀態綁定必要的 minimal handler（如切換顯示狀態的 click event）。
- 不為了對齊視覺而塞入多層無語意空節點。
