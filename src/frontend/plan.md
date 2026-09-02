這是開發 Web App 的素材

- UI 設計稿 / Wireframe
- 需求文件
- API 文件

## 建議 Workflow

### 0. 確認輸入素材

- 對照 UI 設計稿／Wireframe、需求文件與 API 文件。
- 標記素材間的缺口或衝突，例如：畫面需要的欄位未出現在 API、互動規則未寫明，或 API 狀態無法對應畫面狀態。

產出：已確認的畫面範圍、互動範圍與待釐清事項。

### 1. 建立畫面結構

- 將 UI 設計稿設計成 Component Hierarchy Skeleton。
- 區分頁面、區塊、可重用元件與純展示元件；此階段只描述結構，不先決定 API 細節。

產出：Component Hierarchy。

### 2. 定義使用者互動

- 將需求文件轉為以 UI 互動行為為導向的 BDD use case。
- 針對每個 use case 描述前置條件、使用者操作、畫面結果，以及載入、空資料、錯誤與權限等狀態。

產出：BDD use cases。

### 3. 設計前端資料流

- 將 BDD use cases 與 Component Hierarchy 結合，設計以 Component Hierarchy 為語彙的簡易資料流。
- 明確標示資料由哪個元件持有、向下傳遞的資料，以及向上回傳的使用者事件。

產出：Component Hierarchy 對應的資料流描述。

### 4. 定義 UI 資料與元件介面

- 設計 UI 元件所需的資料 Schema。
- 依資料流定義每個元件的 Props Data 與 Callback 介面。
- 讓元件只依賴 UI Schema，不直接依賴 API 回傳格式。

產出：UI Schema 與元件 Props／Callback 合約。

### 5. 設計 API 與 UI 的 Adapter

- 根據 API 文件與 UI Schema，定義 API 與 UI 的 Adapter。
- 處理欄位轉換、預設值、資料缺漏、錯誤格式與必要的請求參數組裝。
- Adapter 的輸出固定符合 UI Schema，避免 API 格式滲入 UI 元件。

產出：Adapter 介面與轉換規則。

### 6. 實作與整合

- 實作 UI 元件：先依 Props／Callback 合約搭配 mock data 完成互動與狀態呈現。
- 實作 API 與 UI 的 Adapter：串接實際 API，並以 Adapter 結果提供資料給 UI。
- 依 BDD use cases 驗證主要流程與各種 UI 狀態；若發現需求、畫面或 API 不一致，回到對應設計階段調整。

產出：可運作的 UI、Adapter，以及已驗證的 use cases。

## 依賴關係

`輸入素材 → Component Hierarchy + BDD use cases → 資料流 → UI Schema／元件介面 → Adapter 設計 → UI 與 Adapter 實作 → BDD 驗證`

其中「UI 元件實作」與「Adapter 實作」在第 4、5 步的合約確認後可平行進行；UI 元件先使用 mock data，可降低等待 API 串接的時間。
