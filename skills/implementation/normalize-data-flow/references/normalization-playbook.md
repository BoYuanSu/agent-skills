# 資料正規化與系統縫合實戰指南 (Normalization Playbook)

本手冊提供資料正規化層 (Normalization Layer) 實作的標準範例。我們以一個「產品列表與數據看板 (Product Directory with Stats)」為例，展示如何將後端 API 原始資料轉換並縫合至 UI 理想 View Model。

---

## 1. 兩端原始型別定義 (兩端先行)

### UI 呈現層預期的最理想 View Model (由 spec-ui-hierarchy 定義)
UI 需要的資料最為扁平、易於渲染，包含產品列表與統計卡片：

```typescript
/** 呈現層產品項目 View Model */
type ProductViewModel = {
  id: string;
  name: string;
  priceText: string; /** 格式化好的價格，如 NT$ 1,200 */
  stockStatus: 'in-stock' | 'low-stock' | 'out-of-stock';
  updatedAtText: string; /** 格式化好的時間字串 */
};

/** 統計卡片 View Model */
type SummaryStatsViewModel = {
  totalCount: number;
  outOfStockCount: number;
  averagePriceText: string;
};

/** 呈現層所期待的完整頁面資料結構 */
type ProductDirectoryViewModel = {
  products: ProductViewModel[];
  stats: SummaryStatsViewModel;
};

export type { ProductViewModel, SummaryStatsViewModel, ProductDirectoryViewModel };
```

### 資料請求層取得的 API Raw Response (由 develop-api-layer 定義)
後端 API 回傳的原始資料通常是未格式化的數值、Unix 毫秒時間戳，且結構沒有依據 UI 看板特別區分：

```typescript
type ApiProductItem = {
  product_id: number;
  title: string;
  unit_price: number;
  inventory_quantity: number;
  updated_timestamp: number; /** Unix timestamp */
  is_active: boolean;
};

type ApiGetProductsResponse = {
  status: string;
  data: ApiProductItem[];
};

export type { ApiProductItem, ApiGetProductsResponse };
```

---

## 2. 撰寫資料轉譯函式 (Mapping Functions)

在邊界層實作純粹的轉譯邏輯，負責將 `ApiGetProductsResponse` 轉換成 `ProductDirectoryViewModel`。

```typescript
import type { ApiGetProductsResponse, ApiProductItem } from '../api/types';
import type { ProductDirectoryViewModel, ProductViewModel, SummaryStatsViewModel } from './types';

/** 格式化日期：Unix Timestamp -> YYYY-MM-DD */
const formatDate = (timestamp: number) => {
  const date = new Date(timestamp);
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;
};

/** 格式化貨幣：1200 -> NT$ 1,200 */
const formatCurrency = (amount: number) => 
  `NT$ ${amount.toLocaleString('zh-TW')}`;

/** 計算庫存狀態 */
const resolveStockStatus = (quantity: number): ProductViewModel['stockStatus'] => {
  if (quantity <= 0) return 'out-of-stock';
  if (quantity <= 5) return 'low-stock';
  return 'in-stock';
};

/** 轉譯單一產品項目 */
const mapProduct = (item: ApiProductItem): ProductViewModel => ({
  id: String(item.product_id),
  name: item.title ?? '未命名商品',
  priceText: formatCurrency(item.unit_price ?? 0),
  stockStatus: resolveStockStatus(item.inventory_quantity ?? 0),
  updatedAtText: formatDate(item.updated_timestamp ?? Date.now())
});

/** 計算並轉換統計數據 */
const computeStats = (items: ApiProductItem[]): SummaryStatsViewModel => {
  const totalCount = items.length;
  const outOfStockCount = items.filter(item => (item.inventory_quantity ?? 0) <= 0).length;
  const totalPrice = items.reduce((sum, item) => sum + (item.unit_price ?? 0), 0);
  const averagePrice = totalCount > 0 ? totalPrice / totalCount : 0;

  return {
    totalCount,
    outOfStockCount,
    averagePriceText: formatCurrency(Math.round(averagePrice))
  };
};

/** 主資料流轉譯 Entry Point */
const mapApiToViewModel = (response: ApiGetProductsResponse): ProductDirectoryViewModel => {
  const rawItems = response?.data ?? [];
  
  /** 僅過濾出 active 的商品進行渲染 */
  const activeItems = rawItems.filter(item => item.is_active);

  return {
    products: activeItems.map(mapProduct),
    stats: computeStats(activeItems)
  };
};

export { mapApiToViewModel };
```

---

## 3. 實作業務邏輯 Composable (The Bridge)

將 API 請求與轉譯邏輯封裝在專屬的 Hook 內。本例以 Vue 3 搭配基礎 fetch 示意，保留 Refetch 與 Loading 狀態，且**完整保存 API 原始資料並透過 computed 轉譯為衍生資料**：

```typescript
import { ref, computed } from 'vue';
import { fetchProductsApi } from '../api/products';
import { mapApiToViewModel } from './mapping';
import type { ApiGetProductsResponse } from '../api/types';
import type { ProductDirectoryViewModel } from './types';

const useProductDirectory = () => {
  /** 保存 API 原始資料 (Raw Data) */
  const rawData = ref<ApiGetProductsResponse | null>(null);
  
  const loading = ref(false);
  const error = ref<unknown>(null);

  const fetchDirectory = async () => {
    loading.value = true;
    error.value = null;
    try {
      /** 僅保存 API 原始資料 */
      rawData.value = await fetchProductsApi();
    } catch (err) {
      error.value = err;
    } finally {
      loading.value = false;
    }
  };

  /** 透過 computed 產生衍生資料 (Derived Data)，整合轉譯邏輯 */
  const data = computed<ProductDirectoryViewModel>(() => {
    if (!rawData.value) {
      return {
        products: [],
        stats: {
          totalCount: 0,
          outOfStockCount: 0,
          averagePriceText: 'NT$ 0'
        }
      };
    }
    return mapApiToViewModel(rawData.value);
  });

  return {
    data,
    loading,
    error,
    refetch: fetchDirectory
  };
};

export { useProductDirectory };
```

---

## 4. 系統縫合 (SFC Container Component)

在容器元件中，使用 `useProductDirectory` 取代原先開發時使用的 Mock Data。

### 縫合前 (Mock Data 渲染版本)
```vue
<script setup lang="ts">
import { ref } from 'vue';
import ProductList from './components/ProductList.vue';
import StatsWidget from './components/StatsWidget.vue';

/** 開發初期寫死的 Mock Data */
const mockStats = ref({
  totalCount: 3,
  outOfStockCount: 1,
  averagePriceText: 'NT$ 150'
});

const mockProducts = ref([
  { id: '1', name: 'Mock商品 A', priceText: 'NT$ 100', stockStatus: 'in-stock', updatedAtText: '2026-07-10' },
  { id: '2', name: 'Mock商品 B', priceText: 'NT$ 200', stockStatus: 'out-of-stock', updatedAtText: '2026-07-10' }
]);
</script>

<template>
  <div class="ProductDirectory">
    <StatsWidget :stats="mockStats" />
    <ProductList :products="mockProducts" />
  </div>
</template>
```

### 縫合後 (真實資料串接版本)
```vue
<script setup lang="ts">
import { onMounted } from 'vue';
import { useProductDirectory } from './hooks/useProductDirectory';
import ProductList from './components/ProductList.vue';
import StatsWidget from './components/StatsWidget.vue';

/** 調用 Composable 取得縫合後的真實資料流與狀態 */
const { data, loading, error, refetch } = useProductDirectory();

onMounted(() => {
  void refetch();
});
</script>

<template>
  <div class="ProductDirectory">
    <div v-if="loading" class="ProductDirectory__loading">載入中...</div>
    <div v-else-if="error" class="ProductDirectory__error">
      載入失敗，<button @click="void refetch()">重試</button>
    </div>
    <template v-else>
      <!-- 直接將 Composable 輸出的標準 View Model 注入子元件 -->
      <StatsWidget :stats="data.stats" />
      <ProductList :products="data.products" />
    </template>
  </div>
</template>

<style scoped lang="scss">
.ProductDirectory {
  .ProductDirectory__loading {}
  .ProductDirectory__error {}
}
</style>
```

---

## 5. 常見反模式與防呆檢查

* ❌ **在 Mapping 函式中發起請求**：
  * *原因*：Mapping 應維持 Pure Function，只負責結構轉換，API 副作用必須由 Composable / Service 層控制。
* ❌ **把 API 的 `ApiProductItem` 直接宣告為 `ProductList` 元件的 Prop**：
  * *原因*：這會使 UI 元件與 API 回傳結構產生強耦合。當後端決定把 `unit_price` 改名為 `price` 時，你必須修改所有底層元件。應該使用 `ProductViewModel` 解耦。
* ❌ **在簡單頁面中硬塞 Mapping 邏輯**：
  * *原因*：如果 API Response 已經是 `[{ id: '1', title: 'A' }]` 且 UI 只需要渲染 title，直接傳遞即可，不需要建立 Mapping 函式與專屬的 Composable。**請維持 KISS 原則。**
* ❌ **在 API 成功後手動將轉譯資料寫入另一層 ref (如 `data.value = map(res)`)**：
  * *原因*：這會破壞響應式單向資料流，且失去了保存 API 原始資料以供後續其他操作 (如回傳 Payload 組裝、除錯或與其他狀態合併) 的彈性。一律應將原始資料存於 `ref`，並用 `computed` 轉譯為衍生資料。
