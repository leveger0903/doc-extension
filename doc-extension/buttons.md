# 輸入組件：瀏覽器按鈕、頁面按鈕

Chrome 允許你在網址列右側設定一組按鈕,\
當使用者點擊 icon 時會展開一組介面與使用者互動,\
根據互動時機不同分為瀏覽器按鈕(Browser Action)與頁面按鈕(Page Action)

## 按鈕的用途

- 作為與使用者互動的元素
- 點擊彈出視窗, 開發者可為此視窗打造使用者介面
- 也可單純當作一個觸發器使用

## 瀏覽器按鈕與頁面按鈕共同點

- 都可設定彈跳視窗
- 引入彈出視窗腳本, 並使用 Chrome 提供的 API
- 使用點擊右上角圖示並互動
  - chrome.browserAction.onClicked
  - chrome.pageAction.onClicked

## 瀏覽器按鈕與頁面按鈕差異點

- 瀏覽器按鈕必須是瀏覽器開啟狀態
- 頁面按鈕需要是符合指定條件才會啟用
- 頁面按鈕過濾條件
  - chrome.declarativeContent API 指定啟用頁面按鈕匹配條件
  - chrome.pageAction.show(tabId) 啟用 pageAction 的圖示
  - chrome.pageAction.hide(integer tabId) 關閉按鈕

備註

```
1. 參考官方 chrome.tabs 功能實作與存取
https://developer.chrome.com/extensions/tabs#type

2. 瀏覽器按鈕與頁面按鈕僅能選擇其一使用
```

## 範例： 瀏覽器按鈕, 跳出新視窗

manifest.json

```
{
  "manifest_version": 2,
  "name": "My Popup Extension",
  "description": "My Popup Extension",
  "version": "1",
  "browser_action": {
    "default_title": "My First Popup Extension",
    "default_icon": {
      "16": "icon.png",
      "24": "icon.png",
      "32": "icon.png"
    },
    "default_popup": "popup.html"
  }
}
```

popup.html

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>POPUP</title>
  <style>
    body { width: 500px; }
  </style>
</head>
<body>
  <h2>Hello, This is POPUP DIALOG.</h2>
  <hr>
  <button type="button" id="button">PRESS ME</button>
  <script type="text/javascript" src="popup.js"></script>
</body>
</html>
```

popup.js

```
document.addEventListener('DOMContentLoaded', (dcl) => {
  let button = document.getElementById('button');
  button.addEventListener('click', (ev) => {
    chrome.windows.create({
      url: 'https://www.google.com'
    });
  });
});
```

## 頁面按鈕應用場景

頁面按鈕啟用前右上角是灰色狀態,\
特定情境下才會啟用,\
如果不是任何頁面都需要有作用時,\
建議使用頁面按鈕,\
反之瀏覽器按鈕.

- 提供 RSS 訂閱功能網站
- 輸入密碼框網頁
- 屏蔽 Facebook 廣告

## 啟用情境

### 情境 1： chrome.pageAction.show(tabId)

manifest.json

```
{
  "manifest_version": 2,
  "name": "My Page Action Extension",
  "description": "My Page Action Extension",
  "version": "1.0",
  "page_action" : {
    "default_title": "頁面按鈕測試",
    "default_icon": "icon.png",
    "default_popup": "popup.html"
  },
  "background": {
    "scripts": [
      "event.js"
    ],
    "persistent": false
  },
  "permissions": [
    "tabs"
  ]
}
```

popup.html

```
<!doctype html>
<html>
<head>
  <title>Page Action Button</title>
</head>
<body>
  <h2>Page Action Button</h2>
</body>
</html>
```

event.js

```
let pattern = [
  '*://www.google.com.tw/*',
  '*://www.google.com/*'
];

// 檢查所有 tab
let queryTabsAndShowPageActions = (obj) => {
  chrome.tabs.query(obj, (tabs) => {
    if (tabs && tabs.length > 0) {
      for (var i = 0; i < tabs.length; i++) {
        // 載入完畢的 tab 上, 使用 chrome.pageAction.show 啟用按鈕
        if (tabs[i].status === 'complete')
          chrome.pageAction.show(tabs[i].id);
      }
    }
  });
}

// 初次載入
chrome.runtime.onInstalled.addListener(() => {
  queryTabsAndShowPageActions({
    active: false,
    currentWindow: true,
    url: pattern,
  });
});

// 當 tab 有更動時檢查
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  queryTabsAndShowPageActions({
    active: true,
    currentWindow: true,
    url: pattern,
  });
});
```

chrome.tabs.query 允許使用者查詢目前開啟所有書籤,
並且可以利用正規表達式(Matches Patterns)過濾書籤.

### 情境 2： 申明式內容 API (DeclarativeContent API)

申明式內容 API 允許利用 URL 與 CSS 選擇器匹配,\
若符合條件則啟動頁面按鈕,\
需要使用 activeTab 權限.


manifest.json

```
{
  "manifest_version": 2,
  "name": "My Page Action Extension 2",
  "description": "My Page Action Extension 2",
  "version": "1.0",
  "page_action" : {
    "default_title": "頁面按鈕測試",
    "default_icon": "icon.png",
    "default_popup": "popup.html"
  },
  "background": {
    "scripts": [
      "event.js"
    ],
    "persistent": false
  },
  "permissions": [
    "tabs",
    "declarativeContent" // 需要有此權限
  ]
}
```

popup.html

```
<!doctype html>
<html>
<head>
  <title>Page Action Button</title>
</head>
<body>
  <h2>Page Action Button</h2>
</body>
</html>
```

event.js

```
/**
 * 1. 定義網址與 CSS 選擇匹配器條件 (PageStateMatcher)
 * 2. 啟動頁面按鈕 (ShowPageAction)
 * 3. 移除舊規則, 新增新規則
 */

let rule = {
  conditions: [
    new chrome.declarativeContent.PageStateMatcher({
      pageUrl: { hostContains: 'www.google.com' },
      css: ['img'] // 頁面要有 img 的標籤
    }),
    new chrome.declarativeContent.PageStateMatcher({
      pageUrl: { hostContains: 'www.google.com.tw' },
      css: ['img'] // 頁面要有 img 的標籤
    })
  ],
  actions: [
    new chrome.declarativeContent.ShowPageAction()
  ]
};

chrome.runtime.onInstalled.addListener(() => {
  chrome.declarativeContent.onPageChanged.removeRules(undefined, () => {
    chrome.declarativeContent.onPageChanged.addRules([
      rule
    ])
  });
});
```