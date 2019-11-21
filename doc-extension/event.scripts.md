# 事件腳本與背景頁面

事件腳本相當於 extension 的 controller,\
所有的事件均由事件腳本處理.

## 事件腳本可做什麼

- 監聽來自輸入組件的事件.\
  (點擊瀏覽器或頁面按鈕, 快速鍵輸入, 網址列輸入等)

- 監聽來自 extension 自身的事件.\
  (onMessage, onConnect, onInstalled, onUpdateAvailable等)

- 經由 messaging API, 間接與內容腳本溝通並控制網頁內容.

- 監聽來自瀏覽器的事件

  - 頁籤新增, 移除與更新\
    (chrome.tabs - onCreated, onUpdate, onRemoved 等)

  - 瀏覽器通知\
    (chrome.alarms - onAlarm)

  - 瀏覽器 localStorage 變更\
    (chrome.storage - onChanged)

  - 書籤的創造與移除\
    (chrome.bookmarks - onCreated, onRemoved, onChanged, onImportBegan, onImportEnded等)

  - 記錄新增與移除\
    (chrome.history - onVisited, onVisitRemoved)

## 事件腳本與背景頁面

在 manifest.json 內宣告事件腳本,\
chrome 會產生一個看不見的頁面載入事件腳本.

可在擴件 > 詳細資訊 > 查看檢視模式 查看背景頁面

## 背景頁面狀態

事件腳本只有在需要時才會保持運行,\
閒置時 chrome 會暫停他.

## 事件腳本的生命週期

以下為可能觸發頁面載入動作.

- 應用程式第一次安裝或更新到新版本.\
  (onInstalled, onUpdateAvailable)

- 事件頁面監聽某事件觸發.\
  (onMessage, onConnect)

- 內容腳本或其他擴展程序發送訊息.\
  (onMessageExternal)

- 擴展程序內的其他視圖調用 runtime.getBackgroundPage

背景頁面在視圖(view)與發送信息端口(script)關閉後,\
才會卸載背景頁面,\
保持空閒一段時間,\
將會觸發 runtime.onSuspend 事件,\
如果有操作時卸載將取消,\
並產生 runtime.onSuspendCanceled 事件.

以下為背景事件 manifest.json 範例

````
"background" : {  
    "scripts" : [
      "event_script.js",
      "another_event_script.js"
    ],  
    "persistent" : false #腳本持續運作, 但不建議
}
````

## 事件腳本類型

- persistent background pages : 永遠保持開啟.
- events pages : 需要時才會開啟.