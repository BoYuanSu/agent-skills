---
name: bysu-run-web-feature-workflow
description: 依 UI 設計稿、需求文件與 API 文件規劃、續作或完成一個 Web App 功能，判斷目前所處階段並串接輸入檢查、元件階層、BDD、資料流、UI 合約、Adapter 與實作驗證。單一階段已有明確指定時，改用對應的階段 skill。
user-invocable: true
disable-model-invocation: true
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

這一段是「真正的執行入口」，不是只描述流程。每個階段都必須直接呼叫對應的 skill 來完成，不得在本文件中直接寫出內容或跳過技能指令；前置條件未成立時，不得進入後續階段。

依下列順序執行，並以「呼叫對應 skill」為實際動作：

0. 直接呼叫 `/bysu-review-feature-inputs`：確認輸入素材、功能範圍、缺口與衝突，並在此階段完成後再進入下一步。
1. 直接呼叫 `/bysu-map-component-hierarchy`：產生 Component Hierarchy，並以該 skill 的輸出作為後續設計基礎。
2. 直接呼叫 `/bysu-specify-ui-use-cases`：產生 BDD use cases，並讓 use cases 成為後續資料流與驗證依據。
3. 直接呼叫 `/bysu-design-component-data-flow`：結合元件階層與 use cases，產生資料流，且後續不再在本層重複補列資料責任。
4. 直接呼叫 `/bysu-define-ui-contracts`：在功能程式碼中定義 UI Schema 與元件介面，確保 UI 與程式合約已確立後再做 Adapter。
5. 直接呼叫 `/bysu-design-api-ui-adapter`：在功能程式碼中實作 API 與 UI 的 Adapter 及測試，並以合約為界線限制 API 資料進入 UI。
6. 直接呼叫 `/bysu-implement-web-feature`：完成 UI、串接 Adapter，並依 use cases 驗證最終功能行為。

階段 1 與 2 都通過階段 0 後，彼此沒有前後依賴。階段 4 與 5 的合約確認後，UI 與 Adapter 的實作工作可以平行，但不可讓 API 格式進入 UI 元件。若執行時沒有依照上述 skill 入口逐步呼叫，表示流程未依規範完成，必須回到最早缺失的階段重新處理。

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

若需要按功能分群，仍可在 `docs/` 之下建立相對子資料夾並保持同層結構；核心是相對於使用者提供的 `docs/` 目錄。

第 4–6 階段的成果放在實際功能程式碼與測試旁，不額外產生交付說明。
