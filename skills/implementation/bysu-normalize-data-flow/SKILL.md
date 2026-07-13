---
name: bysu-normalize-data-flow
description: Use when completing the front-end data flow using normalization and sewing (造橋與系統縫合). Triggered when the user wants to connect real APIs to UI components, map raw data to view models, or replace mock data with composables.
---

# 正規化資料流 - 造橋與系統縫合 (bysu-normalize-data-flow)

本技能旨在指引 Agent 實作前端分層架構中的資料正規化層，將請求層（API 原始資料）與呈現層（UI 展示元件）無縫整合。

## 核心概念 (Leading Words)

- **KISS-prune (KISS 剪裁)**：評估頁面複雜度。若為結構簡單、無複雜資料分流或多重 UI 呈現需求的頁面，直接在 API 請求後傳遞原始資料（Raw Data）給呈現元件，省略造橋，提早結束本技能以避免過度設計。
- **Diff (落差分析)**：比對呈現層定義的 UI View Model（通常在 Props 中）與請求層回傳的 API Raw Data，條列欄位命名、資料格式（如 timestamp 與日期字串）、分流或統計的結構落差。
- **Bridge (造橋)**：實作純粹的轉譯函式（Pure Mapping Functions）與專屬 Composable。以單一 `ref` 保存 API 原始資料，並一律透過 Vue **`computed`** 衍生輸出 UI 所需的 View Model，維合單向資料流。
- **Sew (系統縫合)**：在頁面的 Container 元件中注入 Bridge Composable 取代 Mock Data，並將真實資料流與狀態（`loading`、`error`）傳遞綁定至展示元件的 Props 與事件。

---

## 工作流程

1. **落差分析 (Diff Analysis)**
   - 動作：讀取 UI 元件的 Props 介面與 API Response 型別，逐一分析並列出兩端資料欄位的轉換落差。
   - **完成標準**：在回覆中輸出「落差分析對照表」，明確標示每一個需要轉換的欄位。

2. **設計與撰寫轉譯函式 (Mapping Functions)**
   - 動作：撰寫純粹 (Pure) 無副作用的 Mapping 函式（輸入為 API Response 型別，輸出為 UI View Model 型別）。需使用 `?.` 或 `??` 預設值進行空值與異常防禦。
   - **完成標準**：所有 Mapping 函式均有明確的輸入與輸出 TS 型別定義，且無任何 Side Effects。

3. **實作業務邏輯 Composable (The Bridge)**
   - 動作：於 `views/<page>/hooks/` 建立 `use{FeatureName}.ts`。在 Composable 內使用單一 `ref` 完整保存 API Raw Data，並使用 `computed` 呼叫 Mapping 函式產生衍生資料。同時輸出狀態旗標（`loading`、`error`）與 `refetch` 方法。
   - **完成標準**：Composable 中僅包含單一儲存 API 原始資料的 `ref`，所有資料轉譯一律透過 `computed` 衍生輸出，禁止手動寫入額外狀態。

4. **系統縫合與驗證 (Sewing)**
   - 動作：修改 Container 元件，改為調用上述 Composable 以取代 Mock Data。將 Composable 輸出的 reactive 狀態綁定至展示元件。
   - **完成標準**：輸出 Container 元件修改的 Diffs，且 TypeScript 編譯器檢查無型別報錯。

---

## 行為約束 (Guardrails)

- 展示元件（Presentational Component）僅透過 props 接收 View Model 並用 `$emit` 觸發事件。API 請求與轉譯邏輯必須完全移出展示元件。
- 轉譯函式必須是純粹（Pure）且無副作用的轉譯邏輯（禁止發送 fetch、操作 DOM 或修改外部 state）。
- API 原始資料存於單一 `ref`，轉換結果一律使用 `computed` 衍生輸出，以維護單向資料流與響應性。
- Composable 應保持只做 named export，檔名為 `use{FeatureName}.ts`。
- 詳細範例與最佳實踐請參考 [normalization-playbook.md](references/normalization-playbook.md)。
