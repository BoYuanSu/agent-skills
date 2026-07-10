# API Request Playbook (Fetch / Axios)

## 1) 檔案落點
配合專案既有架構，將 API 請求與型別定義放置於適當的目錄中，優先沿用該模組或專案中已建立好的 API 定義檔。

## 2) 第一階段骨架模板
只建立 API 呼叫骨架，型別暫時使用寬鬆定義（例如 `unknown` 或 `Record<string, unknown>`），避免在第一階段花時間猜測 API 欄位細節。

### Axios 範例

```ts
import { request } from '@/utils/request';

type GetFeatureParams = {
  id: string;
};

type GetFeatureResponse = unknown;

/** 取得特定功能資料 */
const getFeature = (params: GetFeatureParams) => {
  return request.get<GetFeatureResponse>('/api/v1/feature', { params });
};

export { getFeature };
```

### Fetch 範例

```ts
type GetFeatureParams = {
  id: string;
};

type GetFeatureResponse = unknown;

/** 取得特定功能資料 */
const getFeature = async (params: GetFeatureParams) => {
  const response = await fetch(`/api/v1/feature?id=${params.id}`);
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  const data: GetFeatureResponse = await response.json();
  return data;
};

export { getFeature };
```

## 3) 第二階段型別補齊模板
當使用者提供 API response 的實體資料 (example data) 後，依據結構補齊 TypeScript 型別定義。

### TypeScript Type 方案

```ts
type FeatureItem = {
  id: number;
  name: string;
  status: 'active' | 'inactive';
};

type GetFeatureResponse = {
  data: FeatureItem[];
  total: number;
};

export type { FeatureItem, GetFeatureResponse };
```

## 4) 驗證與檢查
- 確保新增或修改 the API 模組能通過 TypeScript 型別檢查。
- 專案若是 Vue 專案或 React 專案，應確保 API 模組不包含 UI 邏輯。

