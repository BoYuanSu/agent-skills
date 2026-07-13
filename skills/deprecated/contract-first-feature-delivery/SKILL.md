---
name: contract-first-feature-delivery
description: 在規劃新功能、重構既有模組、或需要決定「先做 API/Socket 還是先做 UI」時，使用契約優先的五步交付法：先定成功條件，再定內外部契約，接著以依賴適配與 UI 並行實作，最後在容器層整合並做最小端到端驗證。當任務涉及跨層資料流、狀態擁有權、元件與服務邊界、或需要避免過度設計時使用。
---

# Contract First Feature Delivery

## 方法總覽（五步）

1. 定義成功條件。
- 先寫 `Must Have`、`Out of Scope`、`Done Definition`。
- 只保留本次要交付的最小價值，不預先加入未要求功能。

2. 定義契約。
- 先定義三種契約：`外部契約`（任何跨邊界依賴，如 API/Socket/SDK/檔案格式/Event）、`內部契約`（Domain Model）、`UI 契約`（props/emits/command）。
- 先定名稱與型別，再寫實作。
- 若任務依賴 runtime 才能確認的外部契約，先做「契約探詢」：
  - 是否存在可用的關聯鍵、狀態語義或版本欄位？
  - 契約是否已在 runtime 驗證可用？
  - 若未驗證，本輪採「保留契約概念」還是「暫時策略」？

3. 進行雙軌實作。
- 依賴軌：實作 adapter/provider/composable，吸收外部系統差異。
- UI 軌：實作純 UI 元件，只吃契約輸入，只輸出事件，不直接碰外部依賴。

4. 在容器層整合。
- 在 page/container 組裝資料流與事件流。
- 讓資料下行（props/context），事件上行（emit/callback）。
- 把副作用集中在邊界層，避免散落於 presentational 元件。

5. 執行最小端到端驗證。
- 先驗證 1-2 條核心路徑（高風險、高價值）。
- 確認主流程可閉環後，再補邊角情境與抽象。

## 每步必交付輸出

1. 成功條件輸出
- `Must Have` 清單（最多 3-5 項）。
- `Out of Scope` 清單。
- 一句 `完成判準`。

2. 契約輸出
- 外部契約：必要欄位、方向（inbound/outbound）、錯誤欄位。
- 內部契約：穩定 model，不直接暴露原始 payload。
- UI 契約：最小 props/emits/commands。

3. 雙軌輸出
- 依賴軌：最小可運作 adapter（必要時先 fake/stub）。
- UI 軌：可渲染骨架與互動事件。

4. 整合輸出
- 一張簡化資料流：`Input -> Transform -> UI` 與 `UI Action -> Command -> Effect`。
- 一個容器整合點（單一來源組裝依賴）。

5. 驗證輸出
- 核心流程驗證結果（成功/失敗、原因）。
- 下一步增量一項（只選一項）。

## 順序決策規則

1. 外部依賴高度不穩定時，先固化內部契約，再做 adapter。
2. UI 需求高度不確定時，先做低保真 UI 契約與事件模型，再補視覺細節。
3. 兩端都不穩定時，先做「契約 + 最小假資料」打通主流程，再替換真實依賴。
4. 需要快速交付時，優先跑通單一路徑，不先做多型別抽象與高可配置化。
5. 契約未經 runtime 驗證時，不把假設語義或欄位寫死在主流程；先放在可替換邊界（hook/interface/adapter）。

## 標準回覆格式

輸出設計或實作建議時，固定使用以下段落：

1. `Success Criteria`
- 說明 Must Have、Out of Scope、Done Definition。

2. `Contracts`
- 說明外部契約、內部契約、UI 契約。
- 額外標示：
  - `Contract Confirmed`: yes/no
  - `Runtime Verified`: yes/no
  - `Fallback Strategy`: 若未驗證，本輪採用的降級策略

3. `Build Order`
- 說明雙軌如何並行，以及先後關係。

4. `Integration Plan`
- 說明容器層組裝與資料/事件路徑。

5. `Validation Plan`
- 說明最小 E2E 驗證與驗收條件。

6. `Tradeoffs`
- 說明刻意延後的複雜度與觸發條件。

## Guardrails

1. 不要在契約未定時直接展開大量程式碼。
2. 不要讓 UI 元件直接耦合 API/Socket 細節。
3. 不要把外部 payload 直接當內部或 UI 契約。
4. 不要為單次場景預先建立多層抽象。
5. 不要同時引入新功能、重構、最佳化三件事。
6. 不要跳過最小端到端驗證。
