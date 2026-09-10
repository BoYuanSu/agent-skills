---
name: bysu-design-api-ui-adapter
description: 在 Web App 功能開發的第 5 階段，依 API 文件與既有 UI Schema，在功能程式碼中實作 API 到 UI 的 Adapter、必要請求參數轉換及測試，隔離欄位差異、缺漏、預設值與錯誤格式。
---

# 設計並實作 API／UI Adapter

## 前置資料

讀取 `specs/{FeatureName}/sources/api/`、`input-review.md`、`use-cases.md`，以及第 4 階段完成的 UI Schema。檢查專案現有 API client 與錯誤處理邊界，但不要讓既有 API 型別取代 UI Schema。

## 執行方式

1. 明確定義 Adapter 的 API 輸入與 UI Schema 輸出。
2. 實作必要的欄位重新命名、巢狀結構整理、格式轉換與預設值。
3. 對缺漏或無效資料採取明確策略：可安全預設、可略過、轉為可顯示錯誤，或回報阻斷契約問題。不要悄悄捏造業務資料。
4. 若 UI 操作需要送出資料，將 UI 輸入轉成 API 請求參數，並處理文件要求的格式。
5. 將 API 錯誤轉為 UI 流程可辨識的最小錯誤形式；不要把原始 transport response 傳入展示元件。
6. 以測試表達重要轉換，包括正常資料、缺漏資料、邊界值與已知錯誤格式。

轉換規則、型別與測試直接放在功能程式碼旁，不另建 Markdown 說明。除非專案既有架構要求，Adapter 不負責畫面狀態與元件互動。

## 完成條件

- Adapter 的成功輸出固定符合 UI Schema。
- API 欄位名稱與 response 結構不會滲入 UI 元件。
- 請求與回應的必要轉換均有可執行測試。
- API 文件不足以決定的業務轉換已回報，沒有以任意預設掩蓋。
