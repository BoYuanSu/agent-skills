## Hard Constraints

- 專注在使用者提交的任務，不要做沒有相關的事情。
- 如果程式碼有 warning，請**忽略**它們，除非使用者特別要求處理。
- 除非使用者明確要求，禁止執行任何 `pnpm` 指令（包含 `pnpm exec`、`pnpm install`、`pnpm lint`、`pnpm type-check`）。
- 如果你有用到任何專案內設定的 skills 或者工具，請在回覆中明確說明使用了哪些這這個專案下設定的 skills。列出名字就好，不需要解釋它們的作用。
- Think Before Coding
  Do not assume. Do not hide confusion. Surface tradeoffs.

- Simplicity First
  Use the minimum code that solves the problem. Avoid speculative complexity.
  - Do not add features beyond what was asked.
  - Do not add abstractions for single-use code.
  - Do not add configurability that was not requested.
  - Do not add handling for impossible scenarios.
  - Rewrite if implementation is clearly overcomplicated.
  - Vue SFC props 使用規範：詳見下方 `Vue SFC Code Style` block。

  Check: would a senior engineer consider this overengineered? If yes, simplify.

---

## Vue SFC Code Style（Single Source）
- SFC 基本結構
  - 預設使用 `<script setup lang="ts">`。
  - 需要樣式時使用 `<style scoped lang="scss">`。
  - 產生 scaffold 類任務時，預設維持最小三區塊：`<template>`、`<script setup lang="ts">`、`<style scoped lang="scss">`。

- Props 與 Template 綁定
  - 使用 `{ComponentName}Props` 介面搭配 `withDefaults(defineProps<...>(), {})`。
  - 若 `<template>` 可直接引用 props，必須直接使用參數變數（`someProp`），不要寫 `props.someProp`。
  - 禁止為了轉手而宣告重複別名（例如 `const x = computed(() => props.x)`）；只有在需要實際衍生/轉換邏輯時才可使用 computed。
  - Template 事件觸發用 `$emit(...)`，不要使用 `emits`。

- Emits 型別
  - 若需要宣告 emits，使用 call-signature 風格：
    `const emits = defineEmits<{ (e: 'eventName', ...args): void; }>();`

- **樣式與 BEM**：
  - 採 `Block__elementName` 命名。SCSS 內**禁用** `&__element` 縮寫，必須寫完整類名。
  - 僅互動狀態（`active`、`disabled`、`selected`、`loading`、`error`）可使用短 class。

- 模板與結構變更原則
  - 非必要不更動 DOM 階層；若重構 class 命名，需維持既有行為、事件與條件渲染。

### 最小結果範例

```vue
<script setup lang="ts">
interface DemoProps { title?: string; }
const props = withDefaults(defineProps<DemoProps>(), { title: '' });
void props;
const emits = defineEmits<{ (e: 'click'): void }>();
void emits;
</script>
<template>
  <div class="Demo">
    <span class="Demo__title">{{ title }}</span>
    <button class="Demo__btn" :class="{ active }" @click="$emit('click')">OK</button>
  </div>
</template>
<style scoped lang="scss">
.Demo {
.Demo__title {}
.Demo__btn { &.active {} }
}
</style>
```
