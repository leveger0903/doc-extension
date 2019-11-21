# 輸入組件：網址列

網址列(omnibox)允許使用者在網址列,\
輸入特定關鍵字(按下tab)來啟動擴件,\
並透過監聽使用者輸入行為實踐操作邏輯.

## 網址列(omnibox)能做什麼事

- 優化搜尋體驗\
- 提供建議輸入清單並自動完成
- 綁定一些特別的邏輯操作(如 FB 發文, Trello 新增卡片)

## 應用場景

- 字典 : 輸入文字並返回中英查詢
- API查詢：輸入指定 API 名稱時, 並返回查詢

## 安裝檔

```
{
  "name": "My Onmibox Extension",
  "description": "My Onmibox Extension",
  "version": "2.0",
  "omnibox": {
    "keyword": "OI"
  },
  "icons": {
    "16": "icon.png"
  },
  "background": {
    "scripts": [
      "background.js"
    ],
    "persistent": false
  }
  ...
}
```

- omnibox.keyword 為啟動 omnibox 關鍵字
- 啟動關鍵字為 case-insensitive
- 輸入關鍵字啟動方式為
  1. 輸入關鍵字
  2. 按下 tab

## 可用 API

- chrome.omnibox.onInputStarted\
  使用者點下網址列(omnibox)行為時
- chrome.omnibox.onInputChanged\
  網址列有任何變更時
- chrome.omnibox.onInputEntered\
  網址列按下 enter 時
- chrome.omnibox.onInputCancelled\
  網址列按下 esc 時

## 創建建議輸入清單

以下方法為創建建議輸入清單方法,\
呼叫的方法為 suggest,\
注意參數帶入的為一組陣列.

## 範例

manifest.json

```
{
  "name": "My Onmibox Extension",
  "description": "My Onmibox Extension",
  "version": "2.0",
  "omnibox": {
    "keyword": "OI"
  },
  "icons": {
    "16": "icon.png"
  },
  "background": {
    "scripts": [
      "background.js"
    ],
    "persistent": false
  }
  ...
}
```

background.js

```
// 搜尋關鍵字
let descriptions = {
  "a" : ["actions", "alarms", "apps"],  
  "b" : ["background-page", "bookmarks", "browser-action"],  
  "c" : ["commands", "content-script", "context-menu"],  
  "d" : ["dashboard", "declarativeContent"],  
  "e" : ["event-script", "examples"],  
  "h" : ["history"],  
  "i" : ["incognito", "inject"],  
  "m" : ["management page", "manifest", "match pattern", "messaging"],  
  "o" : ["omnibox"],
  "p" : ["page-action", "permissions", "plugins", "popup"],  
  "r" : ["resources panel", "runtime"],  
  "s" : ["sources panel", "store", "storage"],  
  "t" : ["tabs", "themes"],
};

// 搜尋提示
function getSuggestResults(key) {
  let results = [];
  if (descriptions[key] === undefined) {
    return results;
  }

  for (var i = 0; i < descriptions[key].length; i++) {
    results.push({
      content: descriptions[key][i],
      description: "Search '" + descriptions[key][i] + "' on developer.chrome.com",
    });
  }
  return results;  
}  

chrome.omnibox.onInputChanged.addListener((text,suggest) => {  
  suggest(getSuggestResults(text));  
});

// enter 觸發跳開新視窗
chrome.omnibox.onInputEntered.addListener((text, disposition) => {
  chrome.windows.create({
    url: "https://www.google.com/search?q=chrome+extensions+developers+" + text
  });
});
```