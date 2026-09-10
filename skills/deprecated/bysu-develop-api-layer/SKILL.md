---
name: bysu-develop-api-layer
description: Use when the user wants to implement frontend API request functions, define TypeScript types for requests/responses, or refine structures based on raw example JSON data.
---

# Develop API Layer

此技能用於引導前端 API 請求函式與型別的開發。確保開發路徑高度可預測，並嚴格遵循「兩端先行，後造橋 (Delayed Normalization)」策略。

## 核心引導詞 (Leading Words)

- **Skeleton-first** (骨架優先)：先建立可呼叫的 API 函式架構與空殼型別（Response 以 `unknown` 暫代），以維持程式碼可編譯，此階段不做任何欄位猜測。
- **Raw-only** (純原始資料)：API 請求層僅處理參數與發送/接收請求，**禁止**在此層實作任何資料轉換（Normalization）或狀態連動邏輯。
- **Example-driven** (範例驅動)：取得 API 實體 JSON 資料後才進行型別補齊，以實體資料為單一事實來源。

## 執行步驟

### 步驟 1：建立 API 請求骨架 (**Skeleton-first** + **Raw-only**)

1. **掃描專案慣例**：尋找專案內現有的 API 請求工具（如 Axios 封裝或 native fetch）並對齊其寫作風格、命名規則與 Import Path Alias。
2. **實作請求骨架**：建立或更新 API 呼叫函式，參數定義為具體型別，而 Response 則定義為 `unknown`。確保完全無 Normalization 邏輯。

> [!IMPORTANT]
> **Completion Criterion**: 
> - 所有的 API 請求函式皆已使用 named export 導出。
> - 所有 API 請求的 Response 型別均暫代為 `unknown`（無任何欄位猜測）。
> - 專案可成功編譯，無任何 TypeScript 錯誤。

### 步驟 2：基於範例資料補齊型別 (**Example-driven**)

1. **定義實體結構**：取得 API 實體 JSON 結構（從對話或文件中），以此結構為唯一依據。
2. **補齊型別**：將步驟 1 中暫代的 `unknown` 改為具體 TypeScript `type`。
3. **容錯與限制**：
   - 欄位除非確定必有，否則一律標記為可選（`?`）以提高容錯。
   - 遵照專案風格：使用 `type` 定義、單行註解（`/** 註解 */`），且**禁止**使用 `any` 與 `as`。

> [!IMPORTANT]
> **Completion Criterion**:
> - 所有的 `unknown` 皆已被替換為對應原始資料（Raw Data）的具體型別。
> - 所有型別均使用 `type` 定義，不使用 `interface`。
> - 無任何型別斷言（`as`）或 `any`。
> - 專案成功編譯無 TS 錯誤。

---

## 參考模板

關於 Axios/Fetch 請求骨架與型別補齊的具體實作範例，請閱讀 [references/api-playbook.md](references/api-playbook.md)。
