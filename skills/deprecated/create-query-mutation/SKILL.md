---
name: create-query-mutation
description: 為前端專案建立 API 呼叫 (fetch / axios) 的可落地骨架，並在使用者提供 example data 後補齊 TypeScript 型別定義。適用於需要快速打通 API 呼叫與型別定義的專案。
---

# Create API Request (Fetch / Axios)

## 目標
先完成可呼叫的 API 請求函式結構，再依 example data 將型別由空殼補成完整 TypeScript 型別，不在第一輪過度猜測欄位細節。

補充常見互動：使用者第一輪通常只提供 API endpoint；agent 先建立可呼叫 of API 請求函式骨架。第二輪使用者會把 example data 貼進型別定義檔或對話中，並要求 agent 依該 example data 生成/收斂對應的 TypeScript 型別。

## 單獨執行模式（Skill-Only）

- 當使用者明確點名 `$create-query-mutation`（或更名後之請求技能）時，預設只執行本技能範圍。
- 僅處理 API 請求函式定義與對應 TypeScript 型別定義之相關變更，不受特定目錄結構限制。
- 未被明確要求前，禁止延伸到頁面 UI、元件行為、跨功能重構或非必要架構調整。
- 若需求超出本技能邊界，先完成本技能交付，再以一句話提出可選下一步，不自動延伸執行。

## 執行流程

1. 對齊專案慣例後再動手。
   - 掃描既有 API 模組或請求工具實作方式（例如使用統一的 Axios instance 或是 native fetch 封裝）。
   - 優先沿用既有命名規則、註解語氣、import path alias 以及 TypeScript 型別風格。

2. 第一階段：先建骨架（不猜細節）。
   - 新增或更新 API 呼叫函式：加入 endpoint function，傳入必要的 params 或 payload，並設定基本 request config。
   - 新增型別定義：先用最小可編譯的空殼型別（例如 `unknown` 或 `Record<string, unknown>` 暫代）。

3. 第二輪互動入口：處理貼在型別定義檔的 example data。
   - 若使用者後續把 example data 直接貼在型別定義檔（常見為註解、常數、暫存 JSON），先以該檔內容為單一事實來源。
   - 不要求使用者重貼到其他地方；直接在同檔或同模組內完成型別收斂。

4. 第二階段：吃 example data 補型別。
   - 依 example data 拆分出核心的 TypeScript type/interface。
   - 欄位型別盡量寬鬆或保留可選（optional），避免因為後端回傳結構輕微變動而崩潰。
   - 只補已知欄位；不確定欄位先不假設，必要時保留寬鬆策略。

5. 驗證與回報。
   - 至少執行針對修改檔案的 lint 檢查。
   - 若全專案 type-check 有歷史錯誤，明確標示「與本次修改無直接關聯」。
   - 回報時附上檔案路徑與關鍵函數名稱，讓使用者可直接在頁面上引用。

## 交付格式

1. 先給「已完成的 API 請求函式與型別骨架」摘要。
2. 再給「型別補齊結果」摘要（例如新增的 TypeScript type 結構）。
3. 最後給驗證結果：lint / type-check（若失敗，說明範圍）。

## 決策規則

1. 使用者尚未提供 example data 時，不主動臆測完整 response 欄位。
2. 當現有頁面已可先跑流程，優先交付可呼召骨架，不阻塞在型別完美化。
3. 若專案已經有同 endpoint 的舊實作，優先重用其欄位命名與註解語彙。
4. 若 endpoint 命名與功能有歧義，先保持 minimal API surface，待 example data 再收斂。

## 參考模板
需要快速套版時，讀取 [references/query-mutation-playbook.md](references/query-mutation-playbook.md)。
