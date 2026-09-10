---
name: bysu-specify-ui-use-cases
description: 在 Web App 功能開發的第 2 階段，將已確認的需求轉成以 UI 行為為主的 Given／When／Then use cases，涵蓋正常、載入、空資料、錯誤與權限狀態。產出是需求規格，不是自動化測試碼。
---

# 定義 UI BDD Use Cases

## 前置資料

讀取相對於 `docs/` 目錄的 `input-review.md`、`requirements/` 及必要的 UI 素材。若互動範圍仍有阻斷問題，先回到 `$bysu-review-feature-inputs`。

## 執行方式

1. 以使用者目標或可觀察行為分組，不以 API endpoint 分組。
2. 每個情境使用 Given／When／Then，分別寫清楚前置條件、使用者操作與可觀察的畫面結果。
3. 對每個主要流程檢查正常、載入、空資料、錯誤與權限狀態；只寫適用且有素材依據的狀態。
4. 包含重試、取消、重複送出、輸入驗證等行為時，明確寫出觸發條件與畫面結果。
5. 素材沒有定義的行為列為待釐清事項，不將常見做法當成需求。

## 產出

建立或更新與 `api/`、`requirements/`、`ui/` 同層的 `use-cases.md`。每個 use case 應有穩定、簡短的識別名稱，並以 Gherkin 風格情境呈現，例如：

```gherkin
Scenario: 使用者成功載入資料
  Given 使用者可查看此功能
  When 使用者開啟功能頁面
  Then 畫面顯示已取得的資料
```

可在情境前補充來源或範圍，但不要加入元件實作、API 呼叫順序或測試框架語法。

## 完成條件

- 主要互動都有可觀察、可驗證的結果。
- 適用的載入、空資料、錯誤與權限狀態均有覆蓋。
- 情境之間沒有互相矛盾；未決問題沒有被寫成既定行為。
