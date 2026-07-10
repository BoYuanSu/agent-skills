# Template DOM Hierarchy Reference

## Table of Contents

- Input Checklist
- Style Class Skeleton Heuristics
- DOM Hierarchy Heuristics
- Props / ViewModel Mapping Heuristics
- Template Skeleton Pattern
- Anti-Patterns

## Input Checklist

在動手實作前先確認：

- 是否專注於「單一元件」任務（不拆分新元件，不擴展至其他元件）。
- 是否提供清楚的畫面視覺輸入或描述。
- Vue SFC 是否已由上一階段 (`bysu-spec-ui-hierarchy`) 定義好 Props 與 View Model 契約。
- Vue SFC 命名與結構是否依據 `AGENTS.md` 的 `Vue SFC Code Style（Single Source）` 執行。

## Style Class Skeleton Heuristics

- 先鎖定 root class，再展開 nested class（例如以 `.ComponentName` 包裹所有 element class selectors）。
- **【重要】禁用 `&__` 縮寫**：在 SCSS 內必須寫出完整類名，例如：
  ```scss
  .ComponentName {
    .ComponentName__header {
    }
  }
  ```
- 每個 selector 只保留空 block，不填任何 CSS 屬性，用以做出 name scoped 的樣式隔離。
- 僅互動狀態（如 `active`, `disabled`, `selected`, `loading`, `error`）可使用短 class 搭配：
  ```scss
  .ComponentName__btn {
    &.active {
    }
  }
  ```

## DOM Hierarchy Heuristics

- 先定義單一 root element，再拆分主要結構區塊（如 header/body/footer）。
- 優先使用語意化 HTML 標籤（如 `article`, `section`, `header`, `footer`, `nav` 等），避免無意義的 `div`。
- 一個節點若沒有結構、樣式、語意、或指令需求，則不需建立，以避免 DOM 層級過深。

## Props / ViewModel Mapping Heuristics

- **純 UI 互動狀態**：若需要切換顯示、展開等純 UI 互動，允許在 `<script setup>` 中新增對應 local state（如 `const isOpen = ref(false)`），但不修改核心 Props/ViewModel 契約。
- **列表渲染 (v-for)**：
  - `:key` 必須綁定至項目之唯一 ID（例如 `item.id`）。若資料結構未包含 ID，退回 index 並標註未來改善點，不在此時虛構 API 欄位。
- **可選欄位**：對可空/可選欄位在 template 使用 `v-if`，避免渲染出空節點。

## Template Skeleton Pattern

符合 `AGENTS.md` 規範的 SFC 骨架範本：

```vue
<script setup lang="ts">
interface DemoProps {
  title?: string;
  items?: Array<{ id: string; displayTitle: string }>;
}
const props = withDefaults(defineProps<DemoProps>(), {
  title: '',
  items: () => []
});
void props;
</script>

<template>
  <div class="Demo">
    <span class="Demo__title">{{ title }}</span>
    <ul class="Demo__list">
      <li v-for="item in items" :key="item.id" class="Demo__item">
        {{ item.displayTitle }}
      </li>
    </ul>
  </div>
</template>

<style scoped lang="scss">
.Demo {
  .Demo__title {
  }
  .Demo__list {
  }
  .Demo__item {
  }
}
</style>
```

## Anti-Patterns

- ❌ **在 SCSS 中使用 `&__element` 縮寫**（例如：`&__title {}`，這違反了 `AGENTS.md` 的規範）。
- ❌ **在 `<style>` 區塊填入具體 CSS 屬性**（例如 `color: red;`，此技能只允許空 block 做隔離）。
- ❌ **在 script 中寫入 API 請求、 TanStack Query 呼叫、或真實的 API 資料轉換邏輯**。
- ❌ **為了看起來完整而虛構 Props 介面或更改 Props 契約**。
- ❌ **引進子元件、複雜 Slot 或非必要的跨檔案依賴**。
