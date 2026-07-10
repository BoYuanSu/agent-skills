# Template DOM Hierarchy Reference

## Table of Contents

- Input Checklist
- Style Class Skeleton Heuristics
- DOM Hierarchy Heuristics
- Props Mapping Heuristics
- Template Skeleton Pattern
- Anti-Patterns
- Prompt Starters

## Input Checklist

在動手前先確認：

- 是否為「單一元件」任務。
- 是否提供可用的視覺輸入（截圖、線框或清楚描述）。
- SFC 是否已含 `<script setup>` 與 `<style>`。
- props 是否已在 script 中定義。
- 任務是否要求完成 template 結構並在 style 中補齊 class skeleton 進行 scoped 隔離。
- Vue SFC 通用命名與綁定風格是否依 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）` 執行。

## Style Class Skeleton Heuristics

- 先鎖定 root class，再展開 nested class（命名與 selector 規範遵循 `AGENTS.md`，以做出 name scoped 隔離）。
- 只建立會在 template 使用到的 class，不預先建立未使用 class。
- 每個 selector 只保留空 block，不填任何 CSS 屬性。
- 若 template class 變動，style skeleton 必須同步更新。

## DOM Hierarchy Heuristics

- 先定義單一 root，再拆主要區塊。
- 優先使用語意化標籤，避免全部使用 `div`。
- 對齊視覺分區，不過度追求像素級節點一比一。
- 一個節點若沒有結構、語意、或指令需求，不要建立它。
- 先保留穩定框架，再補局部細節，避免一次性堆疊過深。

## Props Mapping Heuristics

- `string/number/boolean`:
  - 對應文字或顯示狀態。
  - 避免把所有可選值都包 `v-if`，只在空值會造成錯誤結構時使用。
  - 若需要純 UI 互動狀態（如點擊切換顯示），允許在 `<script setup>` 新增對應 local state（如 `ref(false)`），但不修改 props。
- URL/資源欄位:
  - 綁定到 `src` 或 `href`。
  - 需要替代文案時，優先使用既有 props；沒有就使用保守靜態值。
- `array`:
  - 使用 `v-for` 產生重複節點。
  - `:key` 優先使用項目的穩定識別欄位；若缺少識別欄位，退回索引並標註風險。
- `object`:
  - 僅讀取必要欄位，不整包展開到模板。

## Template Skeleton Pattern

```vue
<template>
  <article class="ComponentName">
    <header class="ComponentName__header">
      <img v-if="iconUrl" class="ComponentName__icon" :src="iconUrl" alt="" />
      <h3 class="ComponentName__title">{{ title }}</h3>
      <p v-if="subtitle" class="ComponentName__subtitle">{{ subtitle }}</p>
    </header>

    <section class="ComponentName__content">
      <ul v-if="items?.length" class="ComponentName__list">
        <li v-for="(item, index) in items" :key="item.id ?? index" class="ComponentName__listItem">
          {{ item.label }}
        </li>
      </ul>
    </section>

    <footer class="ComponentName__footer">
      <button class="ComponentName__action">Action</button>
    </footer>
  </article>
</template>

<style lang="scss" scoped>
.ComponentName {
  .ComponentName__header {
  }
  .ComponentName__icon {
  }
  .ComponentName__title {
  }
  .ComponentName__subtitle {
  }
  .ComponentName__content {
  }
  .ComponentName__list {
  }
  .ComponentName__listItem {
  }
  .ComponentName__footer {
  }
  .ComponentName__action {
  }
}
</style>
```

使用此骨架時：

- 替換成實際區塊名稱與 props。
- 保持 script 中的核心 Props/Emits 契約不變，僅在必要時允許新增與純 UI 互動相關的 local state；style 只更新 nested class skeleton。
- 若輸入 template 有既有 root class，優先沿用。

## Anti-Patterns

- 在 script setup 中撰寫複雜的資料轉換或商業邏輯（僅允許定義與純 UI 互動相關的 local state 變數）。
- 在 `style` 區塊填入 CSS 屬性（此 skill 只允許空 block）。
- 在 template 中呼叫未定義函式或不存在變數。
- 為了看起來完整而加入虛構 props 或虛構資料結構。
- 無依據地引入子元件、slot、或複雜互動行為。

## Prompt Starters

- "根據這張畫面和我的 props-only Vue SFC，完成 template DOM 階層與對應的 style nested class skeleton。"
- "保持 script 核心 Props/Emits 契約（僅必要時加 UI 互動變數），不寫 CSS 屬性，只把 style class skeleton 和 template 結構補齊。"
- "這是一個單一元件，不要拆子元件，只做語意化 DOM 結構。"
