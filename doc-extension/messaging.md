# 在腳本間訊息傳遞

由於腳本間無法直接溝通,\
因此需要透過事件驅動來處理各種邏輯.

## 一次性請求

訊息發送

- runtime.sendMessage

傳送訊息給內容腳本以外的其他部件,\
甚至可向其他擴件傳遞訊息(但須要附上另一擴件ID).

```
chrome.runtime.sendMessage(string extensionId, any message, object options, function responseCallback)
```

範例

```
```

- tabs.sendMessage

傳送訊息給內容腳本(專屬),\
需要附上 tabID,\
注意不同頁籤無法共享內容腳本.

```
chrome.tabs.sendMessage(integer tabId, any message, function responseCallback)
```

範例

```
```