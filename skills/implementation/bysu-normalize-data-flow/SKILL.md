---
name: bysu-normalize-data-flow
description: 適用於前端分層架構中的正規化層與資料流整合。引導 Agent 分析 API 原始資料與 UI 理想 View Model 的結構落差，實作資料轉譯 (Mapping) 與 Composable 邏輯，並將真實資料流注入 UI 容器元件完成系統縫合。
---

# 正規化資料流 - 造橋與系統縫合 (bysu-normalize-data-flow)

## 概覽
本技能旨在落實「兩端先行，延後造橋」中的 **「最後造橋（資料正規化與縫合）」** 階段。在前端的呈現層（UI 元件已使用 Mock 渲染）與資料請求層（API 請求函式與型別已就緒）均打通後，指引 Agent 比對兩端資料結構的落差，設計並實作轉譯邏輯 (Mapping)，透過 Vue Composable/Hook 將真實 API 資料轉換為符合 UI 預期 View Model 的標準格式，並注入容器元件中完成系統縫合。

> [!IMPORTANT]
> **簡單頁面的 KISS 剪裁原則 (KISS Rule for Simple Pages)**：
> 在開始本技能前，Agent 必須評估頁面複雜度。若目標頁面結構簡單、無複雜資料分流或多重 UI 呈現需求：
> - **省略資料正規化層**：不要建立額外的轉譯 Composable。
> - **直接傳遞資料**：直接在 API 請求完成後，將 Response 原始資料（Raw Data）傳遞給呈現元件進行渲染。
> - **此技能應提早結束**，避免過度設計 (Over-engineering)。

---

## 單獨執行模式 (Skill-Only)

- 當使用者明確呼叫 `bysu-normalize-data-flow` 時，只執行本技能。
- 專注於 **「轉譯邏輯」與「資料流縫合」**，不重構無關的 UI 樣式、不隨意拆分或修改已定義好的 UI 元件結構與 API 請求層，除非發現根本的型別相容性問題。
- 未被明確要求前，禁止新增無關的外圍全域控制（如全域同步、計時器、視覺開關等），除非是串接必須的狀態連動。

---

## 工作流程

1. **落差分析 (Diff Analysis) 🌟**
   - 讀取呈現層定義的 View Model 型別（通常位於 UI 元件 we 的 Props 介面或 Composable 簽名中）。
   - 讀取請求層定義的 API Response Raw Data 型別。
   - 分析並列出兩端資料結構的落差（例如：欄位命名不一致、資料格式需轉譯如 Unix timestamp 轉日期字串、需要將單一 API 資料分流至多個 UI 區塊、需要進行過濾或加總等）。

2. **設計與撰寫轉譯函式 (Mapping Functions)**
   - 撰寫純粹 (Pure)、無副作用的 Mapping 函式（如 `mapApiToViewModel`）。
   - 嚴格對齊兩端型別：輸入為 API Response 型別，輸出為 UI View Model / Props 型別。
   - 做好空值與異常防禦（使用 `?.` 或 `??` 預設值），確保 API 資料缺少部分欄位時畫面不會崩潰。

3. **實作業務邏輯 Composable (The Bridge) 🌟**
   - 建立專屬的 Composable (例如 `useProductList.ts`，放置於 `views/<page>/hooks/` 內)。
   - **保存原始資料**：在 Composable 內使用 `ref` 完整保存從請求層取得的 API 原始資料 (Raw Data)。
   - **使用衍生資料 (Derived Data)**：利用 Vue **`computed`** 調用 Mapping 函式，將 API 原始資料響應式地轉譯為 UI 所需的 View Model，**禁止在 API 成功時手動寫入另一個 ref 狀態**。
   - 同時輸出狀態旗標，如 `loading`、`error`，以及 `refetch` 方法。

4. **系統縫合與驗證 (Sewing)**
   - 修改頁面的容器元件 (Container Component)。
   - 移除原先寫死的 Mock Data，改為調用上述實作的 Composable。
   - 將 Composable 輸出的 reactive 狀態、讀取狀態與事件綁定至子元件的 Props/Emits。
   - 驗證畫面是否能使用真實 API 資料正確渲染，且無型別報錯。

---

## 輸出格式

執行本技能後，應依序輸出以下內容：
1. **落差分析對照表**：說明 API Raw Data 與 UI View Model 的欄位對應與轉換邏輯。
2. **Mapping Functions & Composable 程式碼**：符合 TS 規範與專案約定的 Hook 程式碼。
3. **Container 元件修改 Diffs**：展示如何將 Composable 注入到容器元件中取代 Mock Data 的代碼異動。
4. **確認提問**：說明已完成縫合，請使用者進行 API 連線與渲染驗證。

---

## 程式碼規範與防呆 (Guardrails)

### TS 與 Vue 規範
- 必須遵循專案根目錄 `AGENTS.md` 規範與個人寫作風格：
  - TS 偏好單行註解 `/** 這是單行註解 */`。
  - TS 偏好 Arrow Function，且非必要不使用 `any` 與 `as`。
  - TS 偏好 type 來定義型別，只有 Component Props 使用 interface。
  - 偏好單一 named export。
- Composable 應保持只做 named export，檔名為 `use{FeatureName}.ts`。

### 嚴格禁令與邊界
- ❌ **嚴禁在 UI 呈現元件（Presentational Component / 展示元件）內部直接撰寫資料轉換邏輯或發送 API 請求**。
- ❌ **嚴禁在 Mapping 函式中引入 Side Effect**（例如發送 fetch、操作 DOM、修改外部 state 等）。
- ❌ **嚴禁在簡單頁面中套用此正規化流程**，請時刻遵守 KISS 原則。
- ❌ **嚴禁在 API 請求成功後手動將轉譯結果寫入另一獨立的 `ref`**，必須一律使用 `computed` 作為衍生資料 (Derived Data) 轉譯輸出，以維護單向資料流與響應性。

詳細範例與最佳實踐請參考 [references/normalization-playbook.md](references/normalization-playbook.md)。
