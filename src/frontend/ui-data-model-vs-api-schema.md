# 前端資料設計常犯的錯誤：直接拿 API Schema 作為 UI 資料設計的基礎

前端拿到 API 回應後，直接將它傳進元件、用回應欄位渲染畫面，是很常見的做法。它在畫面單純時看似有效；但只要需求加入搜尋、篩選、批次操作、語系或多個資料來源，元件很容易被 API 的欄位名稱、資料結構與特殊規則綁住。

問題不在 API Schema 本身，而在於它回答的是「後端如何傳輸資料」，不是「使用者如何操作畫面」。把兩者視為同一件事，會讓 UI 隨著 API 的變動而變動。

較穩定的做法是：**先依使用者互動定義 UI 資料模型，再在資料進出畫面的邊界轉換 API Schema。**

## 一個常見情境：通知偏好設定

假設要做一個通知偏好設定頁面，使用者可以：

- 搜尋通知項目；
- 篩選已開啟或已關閉的項目；
- 切換單一項目；
- 勾選多個項目後，一次開啟或關閉。

後端可能提供兩份資料。第一份是可設定項目的目錄，第二份則是使用者的設定：

```js
// 目錄 API：系統有哪些通知項目
const channelsResponse = [
  { code: 'order', display_name: '訂單通知' },
  { code: 'promotion', display_name: '優惠活動' },
  { code: 'security', display_name: '帳號安全' },
];

// 設定 API：使用者關閉了哪些通知項目
const preferenceResponse = {
  disabled_channels: ['promotion'],
};
```

若元件直接使用這些 API Schema，它就得理解下列規則：

- 顯示名稱是 `display_name`；
- 識別值是 `code`；
- 項目不在 `disabled_channels` 中，才表示開啟；
- 更新設定時，請求欄位可能又是另一種命名。

這些規則散落在 JSX、搜尋邏輯、開關事件與批次操作中後，元件便不再只負責 UI。當 API 將 `display_name` 改成 `name`，或從「禁用清單」改成「啟用清單」時，連原本不該受影響的互動程式都要修改。

## API 資料與 UI 資料回答不同問題

API Schema 適合描述傳輸需求，例如欄位命名、資料壓縮方式或後端的設定表示法；UI 資料模型則應直接描述畫面所需的狀態與操作。

| 問題 | API Schema 的答案 | UI 資料模型需要的答案 |
| --- | --- | --- |
| 項目叫什麼？ | `display_name` | `name` |
| 如何識別項目？ | `code` | `id` |
| 項目是否開啟？ | 不在 `disabled_channels` 中 | `enabled: true / false` |
| 可否被搜尋、篩選、選取？ | 通常沒有直接表示 | 可直接由 `name`、`enabled`、`selectedIds` 處理 |

因此，這個頁面適合採用下面的 UI 模型：

```js
[
  { id: 'order', name: '訂單通知', enabled: true },
  { id: 'promotion', name: '優惠活動', enabled: false },
  { id: 'security', name: '帳號安全', enabled: true },
]
```

`id`、`name` 與 `enabled` 都是使用者互動直接需要的資料：顯示名稱、搜尋、開關狀態、React 的 `key`，以及批次操作的選取目標。這個模型不必長得像任何一份 API 回應。

## 先在邊界轉換 API Schema

將 API Schema 轉為 UI 模型的工作，集中在一個純函式中：

```js
function toNotificationItems(channels, preferences) {
  const disabledChannels = new Set(preferences.disabled_channels || []);

  return channels.map((channel) => ({
    id: channel.code,
    name: channel.display_name,
    enabled: !disabledChannels.has(channel.code),
  }));
}
```

這層轉換吸收了欄位差異與「禁用清單」的特殊語意。函式輸出後，畫面不再需要知道 `code`、`display_name` 或 `disabled_channels` 的存在。

它也是純函式，因此可獨立測試幾個重要情況：沒有設定資料時全部開啟、設定包含不存在的項目、API 回傳空目錄，以及後端欄位改名後的轉換結果。

## 元件只處理使用者互動

以下元件只接收 UI 模型，並處理搜尋、篩選、選取與切換。它不關心資料來自幾個 API，也不關心後端怎麼表示關閉狀態。

```jsx
import { useMemo, useState } from 'react';

const NotificationPreferences = (props) => {
  const {
    items, // [{ id, name, enabled }]
    onChangeEnabled,
  } = props;

  const [keyword, setKeyword] = useState('');
  const [status, setStatus] = useState('all');
  const [selectedIds, setSelectedIds] = useState(new Set());

  const visibleItems = useMemo(() => {
    const query = keyword.trim().toLowerCase();

    return items.filter((item) => {
      const matchesText = !query || item.name.toLowerCase().includes(query);
      const matchesStatus =
        status === 'all' || (status === 'enabled' ? item.enabled : !item.enabled);
      return matchesText && matchesStatus;
    });
  }, [items, keyword, status]);

  const toggleSelected = (id, checked) => {
    setSelectedIds((previous) => {
      const next = new Set(previous);
      if (checked) next.add(id);
      else next.delete(id);
      return next;
    });
  };

  const changeSelected = (enabled) => {
    onChangeEnabled({
      ids: [...selectedIds],
      enabled,
    });
  };

  return (
    <section>
      <input
        value={keyword}
        placeholder="搜尋通知項目"
        onChange={(event) => setKeyword(event.target.value)}
      />
      <select value={status} onChange={(event) => setStatus(event.target.value)}>
        <option value="all">全部</option>
        <option value="enabled">已開啟</option>
        <option value="disabled">已關閉</option>
      </select>

      <button type="button" onClick={() => changeSelected(true)}>
        開啟已選項目
      </button>
      <button type="button" onClick={() => changeSelected(false)}>
        關閉已選項目
      </button>

      {visibleItems.map((item) => (
        <label key={item.id}>
          <input
            type="checkbox"
            checked={selectedIds.has(item.id)}
            onChange={(event) => toggleSelected(item.id, event.target.checked)}
          />
          <input
            type="checkbox"
            checked={item.enabled}
            onChange={() => onChangeEnabled({
              ids: [item.id],
              enabled: !item.enabled,
            })}
          />
          {item.name}
        </label>
      ))}
    </section>
  );
};
```

這段程式描述的是 UI 的事實：哪些項目可見、哪些項目被選取，以及使用者想將哪些項目切換到什麼狀態。它不應承擔 API 欄位映射或請求格式的判斷。

## 將使用者意圖轉回 API 請求

元件送出的資料是 `{ ids, enabled }`，代表「將這些項目設為開啟或關閉」。容器或資料存取層再決定如何呼叫 API：

```js
async function updateNotificationPreference({ ids, enabled }) {
  if (enabled) {
    return api.enableNotificationChannels({ channel_codes: ids });
  }

  return api.disableNotificationChannels({ channel_codes: ids });
}
```

這能把後端端點、請求欄位與傳輸格式留在邊界。未來 API 改成一個 `PATCH` 端點，或將 `channel_codes` 改名，調整點只會在這個函式，而非每一個 UI 元件。

整個資料流可以簡化為：

```text
目錄 API + 設定 API
        ↓
資料轉換：統一為 { id, name, enabled }
        ↓
UI 互動：搜尋、篩選、選取、切換
        ↓
操作轉換：{ ids, enabled } → API 請求
```

## 設計前先回答四個 UI 問題

在查看 API Schema 前，先釐清畫面必須回答什麼：

| 使用者問題 | UI 模型應提供的資料或行為 |
| --- | --- |
| 我現在看得到哪些項目？ | `visibleItems` |
| 這個項目目前是什麼狀態？ | `enabled` |
| 我選了哪些項目，要做什麼？ | `selectedIds`、`ids`、`enabled` |
| 操作失敗後畫面怎麼處理？ | 重新取得資料、回復本地狀態或顯示錯誤訊息 |

這些答案才是 UI 模型的基礎。API Schema 是實作這些需求的輸入，而不是畫面資料結構的規格。

## 結語

直接使用 API Schema 並不一定立刻出錯；問題通常在需求成長、API 演進或資料來源增加後才浮現。那時候，原本看似省事的做法會讓元件同時處理呈現、互動、欄位映射與傳輸規則。

讓 UI 模型服務使用者互動，讓 API Schema 留在轉換邊界，能讓兩者各自演進。前端不必假裝後端不存在，但也不需要讓每個元件都成為後端資料格式的專家。
