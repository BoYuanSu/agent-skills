# API Request Playbook

本手冊提供 API 請求骨架與型別定義的具體程式碼範例。所有 TypeScript 範例均遵循本專案的程式碼風格規範。

---

## 1. 第一階段：API 請求與型別骨架

此階段應專注於建立可呼叫的函式結構。型別先以 `unknown` 暫代，避免猜測後端欄位細節。

### Axios 實作範例

```ts
import { request } from '@/utils/request';

type GetUserParams = {
  id: string;
};

type GetUserResponse = unknown;

/** 取得使用者資料 */
const getUser = (params: GetUserParams) => {
  return request.get<GetUserResponse>('/api/v1/user', { params });
};

export { getUser };
```

### Fetch 實作範例

```ts
type GetUserParams = {
  id: string;
};

type GetUserResponse = unknown;

/** 取得使用者資料 */
const getUser = async (params: GetUserParams) => {
  const response = await fetch(`/api/v1/user?id=${params.id}`);
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  const data: GetUserResponse = await response.json();
  return data;
};

export { getUser };
```

---

## 2. 第二階段：依實體資料 (Example Data) 補齊型別

取得後端 response 的 JSON 範例後，再將 `GetUserResponse` 由 `unknown` 補齊為具體型別。欄位應盡量標記為可選 (`?`)，提高容錯率。

```ts
type UserDetail = {
  id: number;
  name: string;
  email?: string;
  status: 'active' | 'inactive';
};

type GetUserResponse = {
  user?: UserDetail;
};

export type { UserDetail, GetUserResponse };
```
