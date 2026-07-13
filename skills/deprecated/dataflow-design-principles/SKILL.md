---
name: dataflow-design-principles
description: 在規劃新功能或重構既有模組時，使用五個資料流核心原則（單向資料流、副作用集中、領域模型優先、Fail-fast 與安全 fallback、最小可行閉環）快速產生可落地的資料流設計。當任務涉及狀態擁有權、API/socket 邊界、協議轉換、錯誤處理與重試、或需要先做最小閉環再擴充時使用。
---

# Dataflow Design Principles

## 核心原則

1. 堅持單向資料流（Read Down, Event Up）。
- 讓資料往下傳，讓事件往上拋。
- 避免子層直接改父層狀態。

2. 集中副作用在邊界層（Effects at the Edge）。
- 把 API、socket、storage、timer、scroll 控制放在 composable 或 service。
- 不在純 UI 元件中直接耦合外部系統。

3. 先定義內部領域模型，再接外部協議（Domain First, Adapter Later）。
- 先定義穩定的內部資料結構。
- 再用 normalize/transform/adapter 吸收 payload 差異。

4. 失敗可預期（Fail-fast + Safe Fallback）。
- 缺 provider、缺必要依賴、契約不符時，立即失敗並明確回報。
- 非關鍵能力可提供 fallback，但需可觀測、可追蹤。

5. 先交付最小可行閉環（KISS）。
- 先完成一條端到端主路徑。
- 僅在重複出現時再抽象或擴充配置。

## 四個設計問題（先答再寫碼）

1. 哪些是「來源狀態」？誰擁有？
- 逐一列出狀態名稱與唯一 owner。

2. 哪些是「衍生狀態」？在哪一層計算？
- 只保留必要來源狀態，衍生值以 computed/selector 計算，不重複存放。

3. 哪些動作會改變狀態？副作用放哪裡？
- 區分純狀態轉移與副作用。
- 指定每個副作用的邊界層位置。

4. 失敗、重試、去重、一致性如何定義？
- 明確寫出錯誤處理策略與可接受的一致性模型（例如最終一致）。
- 明確寫出去重鍵與重試條件。

## 設計流程

1. 定義邊界。
- 定義 `UI`、`Feature Logic`、`Service/Adapter` 三層責任。

2. 建立狀態表。
- 列出 `State`、`Owner`、`Writers`、`Readers`、`TTL/生命週期`。

3. 定義事件與命令。
- 列出所有會改變狀態的動作與觸發來源。
- 指定資料下行路徑與事件上行路徑。

4. 定義協議轉換。
- 在 adapter 層定義 `inbound -> domain -> UI` 與 `UI command -> outbound payload`。

5. 定義失敗模型。
- 指定 fail-fast 條件、fallback 條件、重試策略、去重規則。

6. 收斂成最小閉環。
- 只保留首版必需能力與單一路徑。
- 列出刻意延後的擴充項目。

## 輸出格式（固定）

產出設計時固定使用以下段落：

1. `Boundary`
- 說明三層責任與禁止跨層耦合點。

2. `State Ownership`
- 列出來源狀態與擁有者。
- 列出衍生狀態與計算層。

3. `Flow`
- 列出資料下行、事件上行、命令執行路徑。

4. `Effects & Adapters`
- 列出副作用所在邊界與協議轉換點。

5. `Failure Model`
- 列出 fail-fast、fallback、retry、dedupe、一致性規則。

6. `Minimal Loop`
- 說明首版最小可行閉環與延後項目。

7. `Tradeoffs`
- 明確說明當前選擇與放棄點。

## Guardrails

1. 不要讓同一份狀態有多個可寫入口。
2. 不要在 UI 元件內直接操作外部協議。
3. 不要把外部 payload 直接當 UI 契約。
4. 不要先做多協議、多抽象、多配置。
5. 不要略過失敗與一致性定義。
