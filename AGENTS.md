## Hard Constraints

## Hard Constraints
- **環境限制**：忽略程式碼警告（Warnings）；除非使用者明確要求，否則禁止執行任何 `pnpm` 指令。
- **Skills 聲明**：回覆中若使用專案設定的 skills，僅需列出名稱，無須解釋其作用。
- **思考與權衡**：編碼前主動釐清模糊需求並提出設計權衡（Trade-offs），不擅自做預設假設。
- **極簡實作**：遵循偏好的 KISS 原則。僅撰寫解決問題的最小代碼，拒絕任何未要求的抽象、功能、設定或過度設計。
- **溝通風格**：回覆採用平實、高通用的詞彙，不使用流行術語（Buzzwords）。
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
