---
name: bysu-run-web-feature-workflow
description: 依 UI 設計稿、需求文件與 API 文件規劃、續作或完成一個 Web App 功能，判斷目前所處階段並串接輸入檢查、元件階層、BDD、資料流、UI 合約、Adapter 與實作驗證。單一階段已有明確指定時，改用對應的階段 skill。
---

# Web 功能開發流程

以功能為單位管理設計產出，並讓程式碼成為 UI 型別、元件介面與 Adapter 規則的唯一來源。

## 判斷工作範圍

1. 先確認使用者提供的 `docs/` 目錄與其相對資料夾結構；設計檔應落在該目錄下與 `api/`、`requirements/`、`ui/` 同層的位置。
2. 找出本次可用的 UI 設計稿或 Wireframe、需求文件與 API 文件。
3. 檢查既有設計檔與程式碼，不以「檔案存在」直接視為該階段已完成；確認內容仍符合目前素材。
4. 若使用者只要求一個階段，呼叫該階段 skill，完成後停止。
5. 若使用者要求完整流程或續作，從最早缺少、過期或不一致的階段開始。

## 階段路由

依下列順序執行；前置條件未成立時，不跳到後續階段：

0. `$bysu-review-feature-inputs`：確認輸入素材、功能範圍、缺口與衝突。
1. `$bysu-map-component-hierarchy`：產生 Component Hierarchy。
2. `$bysu-specify-ui-use-cases`：產生 BDD use cases。
3. `$bysu-design-component-data-flow`：結合元件階層與 use cases，產生資料流。
4. `$bysu-define-ui-contracts`：在功能程式碼中定義 UI Schema 與元件介面。
5. `$bysu-design-api-ui-adapter`：在功能程式碼中實作 API 與 UI 的 Adapter 及測試。
6. `$bysu-implement-web-feature`：完成 UI、串接 Adapter，並依 use cases 驗證。

階段 1 與 2 都通過階段 0 後，彼此沒有前後依賴。階段 4 與 5 的合約確認後，UI 與 Adapter 的實作工作可以平行，但不可讓 API 格式進入 UI 元件。

## 推進與回退規則

- 將會改變畫面、互動、資料責任或公開合約的待釐清事項視為阻斷問題；列出問題並等待決策，不自行補寫需求。
- 非阻斷缺口可明確記錄假設與影響後繼續，但不得把假設寫成已確認事實。
- 實作時發現素材不一致，回到最早受影響的階段更新，再重新檢查後續產出。
- 只有需求、畫面結構或資料責任改變時，才更新設計文件。
- 不建立跨功能的大型規格，也不建立重複描述 TypeScript 型別、Props、Callback、Adapter 轉換或 mock data 的 Markdown。

## 完成條件

完整流程完成時應有：

```text
docs/
├── api/
├── requirements/
├── ui/
├── input-review.md
├── component-hierarchy.md
├── use-cases.md
├── data-flow.md
└── ...
```

若需要按功能分群，仍可在 `docs/` 之下建立相對子資料夾並保持同層結構；核心是相對於使用者提供的 `docs/` 目錄，而不是固定的 `specs/` 位置。

第 4–6 階段的成果放在實際功能程式碼與測試旁，不額外產生交付說明。
