# 前端「兩端先行，後造橋」架構與 OpenSpec 整合指南

本文件說明如何將前端的**「兩端先行，後造橋」**開發策略，與 **OpenSpec (OpenAPI Specification) 構件化工作流**進行深度整合，建立一套高效且具備型別安全的 AI 協作流程。

## 核心設計理念

在實作複雜的單向資料流時，我們不急於在初期就決定資料轉換（Normalization）的細節。相反地，我們利用 OpenSpec 隨時迭代構件的特性，採用**延遲決策**的開發策略：

1. **先定義兩端**：先寫死畫面理想的資料結構（UI 呈現層）與串接後端 API 原始回傳（資料請求層）。
2. **最後再造橋**：當兩端都打通後，再針對 API 回傳與 UI 預期資料的落差，實作轉譯橋樑（正規化層）與輔助的全域狀態同步。

---

## 前端 4+1 分層實作順序

1. **呈現層 (Presentation Layer)**：
   - 專注於 UI 渲染與視覺。使用標準化假資料 (Mock Data) 餵給 UI 元件，確保 UI 開發不被後端阻塞。
2. **請求層 (Request Layer)**：
   - 處理 API 請求參數（Payload）與取得 API 原始回應。
3. **正規化層 (Normalization Layer) [造橋點]**：
   - 實作資料轉換邏輯，將 API 原始資料收斂、分流、轉換為呈現層所需的標準格式。
4. **全域同步與外圍控制 (Global Sync & Extras)**：
   - 處理計時器、視覺開關（不觸發 API 的純 UI 設定）、期號同步觸發 refetch 等全域聯動。

---

## OpenSpec 整合協作流程

以下是將上述實作順序融入 OpenSpec 五大指令的具體協作步驟：

### 階段 1：探索與釐清兩端格式
* **指令**：`/opsx:explore` (AI Chat)
* **目的**：與 AI 共同討論 UI 區塊劃分與預期資料結構，以及 API 請求參數。
* **關鍵對話指引**：
  > 「我們採用兩端先行的策略。請釐清 UI 呈現層的標準資料格式，以及 API 請求層的 Payload。**此時先不要規劃資料正規化 (Normalization) 的轉譯邏輯，將其留空。**」

### 階段 2：產出兩端規劃與任務
* **指令**：`/opsx:propose <change-name>` (AI Chat)
* **產出構件**：
  * `proposal.md`：記錄本功能意圖與範圍。
  * `specs/` (Delta specs)：定義 UI 元件預期行為與 API 規格 (標記為 `ADDED` 或 `MODIFIED`)。
  * `design.md`：記錄兩端架構（視圖呈現層與資料請求層的初步設計）。
  * `tasks.md`：**特意只包含**「用假資料實作 UI 渲染」與「串接 API 取得 Raw Data」的任務。

### 階段 3：兩端平行實作
* **指令**：`/opsx:apply` 或對話推進 (AI Chat)
* **目的**：打通 UI 元件（餵假資料）與 API 請求。
* **原則**：在此階段若發現規格落差，隨時手動/對話更新 `design.md` 或 `specs/` 後繼續。

### 階段 4：正式「造橋」與規格迭代
* **指令**：在對話中直接指示 AI 更新規格。例如：
  > 「現在 UI 與 API 均已接通。我們需要實作資料正規化層，請分析兩者格式落差，並更新 `design.md` 與 `tasks.md`。」
* **產出異動**：
  * `design.md`：新增 API 原始資料與 UI 格式之間的 **Mapping 轉譯規則**。
  * `tasks.md`：追加實作正規化橋樑（例如 `useNormalizedTableData`）與第四階段全域同步（例如 `useIssueHistorySync`）的待辦清單。
* **執行**：再次運行 `/opsx:apply` 完成後續造橋與狀態連動程式碼。

### 階段 5：驗證與歸檔
* **指令**：
  * `/opsx:verify` (AI Chat / Terminal)：驗證實作是否符合 `specs/` 描述的行為。
  * `/opsx:archive` (AI Chat / Terminal)：將 `ADDED`/`MODIFIED` 的 Delta specs 合併回專案根目錄的 `specs/` 中，完成歸檔。

---

## 協作狀態對照表

| 實作階段 | 開發重點 | OpenSpec 指令/操作 | 構件 (Artifacts) 狀態 |
| :--- | :--- | :--- | :--- |
| **1. 探索期** | 定義 UI 理想格式 & API Payload | `/opsx:explore` | 無（僅於對話中釐清概念，暫不寫檔） |
| **2. 提案期** | 建立初步計畫，延遲正規化決策 | `/opsx:propose` | 產生 `proposal`, `specs`, `design`, `tasks` (僅含兩端任務) |
| **3. 實作期 1** | 打通 UI 呈現與 API 串接 | `/opsx:apply` | 完成 `tasks.md` 中的初步兩端開發 |
| **4. 迭代/造橋**| 實作轉譯邏輯與全域控制 | 對話指示 AI 更新構件，再 `/opsx:apply` | `design.md` 補上 Mapping 規則；`tasks.md` 追加造橋任務並實作 |
| **5. 歸檔** | 驗證功能並更新全案規格 | `/opsx:verify` -> `/opsx:archive` | Delta specs 合併至全案主 `specs/`；變更資料夾封存 |
