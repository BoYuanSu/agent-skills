---
name: develop-api-layer
description: 適用於前端 API 請求層 (Request Layer) 的開發。此技能引導 Agent 建立前端 API 呼叫函式、定義輸入參數與輸出回應型別，並能依據範例資料 (Example Data) 補齊與收斂 TypeScript 型別定義。
---

# Develop API Layer

此技能用於引導前端 API 請求函式與型別的開發。目標是快速建立 API 呼叫骨架，並根據範例資料補齊 TypeScript 型別，不在第一輪過度猜測欄位細節，同時保留融入 TanStack Query、統一 Request 封裝等狀態與請求機制的彈性。

## 執行流程

開發過程分為兩個主要階段：**先建骨架（不猜細節）** 與 **吃範例資料補型別**。

### 第一階段：對齊專案慣例與建立骨架

1. **對齊專案慣例：** 
   - 掃描既有 API 模組或請求工具實作方式（例如使用統一的 Axios instance 或是 native fetch 封裝）。
   - 優先沿用既有命名規則、註解語氣、import path alias 以及 TypeScript 型別風格。

2. **建立呼叫骨架：**
   - 建立或更新 API 呼叫函式：傳入 parameters 或 payload。
   - 新增型別定義：先用最小可編譯的空殼型別（例如 `unknown` 或 `Record<string, unknown>`）暫代 Response。

### 第二階段：依實體資料 (Example Data) 補齊型別

1. **處理範例資料：**
   - 若使用者提供實體資料 (example data)，不論是貼在對話中還是註記在型別定義檔內，直接將其作為單一事實來源進行型別定義。
   - 依範例資料拆分出核心的 TypeScript `type` 定義。

2. **寬鬆化與容錯：**
   - 欄位型別盡量寬鬆或保留可選（`?`），避免因為後端回傳結構輕微變動而崩潰。
   - 只補已知欄位；不確定或非必要的欄位先不假設。

---

## 決策規則

1. **KISS 原則優先：** 保持 API 請求層的純粹性，只負責處理請求與定義型別，不介入畫面渲染或多餘的架構分層。
2. **不猜測型別：** 使用者尚未提供 example data 時，不主動臆測完整 response 欄位，優先交付可呼叫的 API 骨架。
3. **型別寬容度：** 產出的型別欄位除非確定必有，否則應使用 `?` 標記為 optional，以提高容錯。

---

## 參考模板

關於 Axios/Fetch 請求骨架與型別補齊的具體實作範例，請閱讀 [references/api-playbook.md](references/api-playbook.md)。
