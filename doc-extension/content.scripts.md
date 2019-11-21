# 內容腳本

內容腳本是網頁中所運行的 javascript 文件,\
他們可以透過內容腳本的注入來取得、修改網頁內容或監聽網頁中的事件.

## 內容腳本的限制

- 只能存取以下 API ：
  - extension
    - getURL
    - inIncognitoContext
    - lastError
    - onRequest
    - sendRequest
  - i18n
  - runtime
    - connect
    - getManifest
    - getURL
    - id
    - onConnect
    - onMessage
    - sendMessage
  - storage
- 不能存取 extension 裡其他類型腳本組件與方法
- 不能存取定義在網頁中其他 js 中的變數與方法

內容腳本無法直接跟完整 chrome.*APIs 溝通,\
但可透過 messaging API 與事件腳本、彈出視窗腳本溝通

## 內容腳本可做什麼

- 操作瀏覽中網頁的 DOM 物件、CSS
- 透過 messaging API 接收來自網頁腳本訊息
- 可設定插入瀏覽中網頁條件
- 除了安裝檔中指定網址外, 也可以由腳本動態插入內容腳本.

以下為內容腳本 manifest.json 範例

````
"content_scripts": [  
  {  
    "matches": [
      "http://www.google.com/*"
    ],  
    "css": ["mystyles_A.css"],
    "js": [
      "jquery.js",
      "myscript_A.js"
    ]  
  }
]
````

## content_script 允許屬性

- matches : 符合的網址
- exclude_matches : 排除的網址
- match_about_blank : 是否要在以下狀況插入內容腳本
  - about:blank
  - about:srcdoc(*)
- css : 插入的樣式檔
- js : 插入的腳本
- run_at : 插入內容腳本的時機
  - document_start
  - document_end
  - document_idle
- all_frames : 是否要在頁面內的 iframe 插入內容腳本
- include_globs : 符合的網址, 模擬 Greasemonkey 的 @include 關鍵字(*)
- exclude_globs : 排除的網址, 模擬 Greasemonkey 的 @include 關鍵字(*)

```
about:srcdoc
參考：https://www.w3schools.com/tags/att_iframe_srcdoc.asp
```

```
Greasemonkey
參考：https://zh.wikipedia.org/wiki/Greasemonkey
```

## 動態插入腳本

如果你希望內容腳本在使用者點擊瀏覽器按鈕才動態載入,\
你可以使用以下方法.

```
chrome.tabs.insertCSS(integer tabId, object details, function callback)
chrome.tabs.executeScript(integer tabId, object details, function callback)
```

> 你的腳本需要經由特定事件(如點擊 page action 按鈕),\
> 使用程式載入 css / js 就會非常有用

## 範例： make_page_red

manifest.json

```
{
  "name": "Page Redder",
  "description": "Make the current page red",
  "version": "2.0",
  "permissions": [
     "activeTab"
  ],
  "background": {
    "scripts": [
      "background.js"
    ],
    "persistent": false
  },
  "browser_action": {
    "default_title": "Make this page red"
  },
  "manifest_version": 2
}
```

background.js

```
chrome.browserAction.onClicked.addListener(function (tab) {
  console.log('Turning ' + tab.url + ' red!');
  chrome.tabs.executeScript(null, {file: "content_script.js"});
});
```

content_script.js

```
document.body.style.backgroundColor = "red";
```

## 網頁腳本與內容腳本

網頁腳本與內容腳本兩者是完全隔離的,\
因此開發人員可以放心使用自己定義的方法與變數,\
不用擔心與網頁腳本衝突.

網頁腳本

```
# index.html
<!doctype html>
<html>
<body>
  <button id='myButton'>點我</button>
  <script>
    var button = document.getElementById('myButton');
    button.addEventListener('click', () => {
      alert('Hello, This is a dog.');
    }, false);
  </script>
</body>
</html>
```

內容腳本

```
# background.js
chrome.browserAction.onClicked.addListener(function(tab) {
  chrome.tabs.executeScript(null, {file: "content_script.js"});
});

# content_script.js
var button = document.getElementById('myButton');
button.addEventListener('click', () => {
  alert('Hello, This is a cat.');
}, false);
```

## 安全性提議

- 跨站腳本惡意攻擊
  - 內容腳本使用 XMLHttpRequest 載入部分 html, 建議為 innerText
  - 使用 https
  - 利用 JSON.parse 讓 data 變成不可運行的腳本
- 內容腳本使用來自擴充目錄底下檔案
