# Template DOM Hierarchy Reference

## 樣式骨架啟發式規則 (Heuristics)
- **類別對齊**：SCSS nested selectors 結構必須與 `<template>` 中的 class 結構完全一致，便於後續樣式填寫。
- **樣式空殼化**：SCSS block 內只能為空，僅做樣式隔離：
  ```scss
  .ComponentName {
    .ComponentName__header {
    }
  }
  ```
- **互動狀態**：僅互動狀態（如 `active`, `selected`, `disabled`）可使用短 class 搭配：
  ```scss
  .ComponentName__btn {
    &.active {
    }
  }
  ```

## DOM 階層與綁定 Heuristics
- 優先使用 HTML5 語意化標籤，避免 `div` 堆疊。
- 若屬性為非必填（Optional），必須在模板中以 `v-if` 保護，避免渲染出多餘的空節點。
- 列表渲染必須綁定穩定之唯一 ID（例如 `item.id`）。若資料無 ID，退回 index 並標註未來改善點，不在此時虛構 API 欄位。

## 典型範本 (Template Pattern)
符合專案規範的 Vue SFC 骨架範本：

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
    <span class="Demo__title" v-if="title">{{ title }}</span>
    <ul class="Demo__list" v-if="items.length">
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

## 反模式 (Anti-Patterns)
- ❌ **樣式帶有具體屬性**：如在 `.Demo__title` 內寫了 `font-size: 16px;`。
- ❌ **使用 `&__` 縮寫**：在 SCSS 內寫了 `&__title`。
- ❌ **過度設計與提前拆分**：將簡單的列表項目直接拆成一個新的元件檔案。
- ❌ **真實邏輯污染**：在 script 中引入了 `useQuery`、真實的 API 請求或資料 Normalize 邏輯。
