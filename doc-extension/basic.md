# Chrome Extenstion 組成

以下為 google extension 組成

- 輸入組件 | input component
- 腳本組件 | script component
- 彈出視窗組件 | popup component
- 安裝檔 | manifest component

## 輸入組件

chrome 提供與 user 互動用元素,\
不一定是實體(如shortcut-key),\
以下為幾種類型與圖示

- browser-action 瀏覽器按鈕
- page-action 頁面按鈕
- shortcut-key 快捷鍵
- context-menu-item 右鍵功能選單
- omnibox-input 網址列輸入
- content-ui 網頁內容 UI

## 腳本組件

chrome extension 運作邏輯,\
以下為不同時機使用的腳本組件

- event scripts | 事件腳本\
  在安裝檔中定義, 通常以背景頁面中執行, 為主要的 extension 邏輯.

- popup scripts | 彈出視窗腳本\
  負責彈出視窗裡的操作邏輯.

- content scripts | 內容腳本\
  在安裝檔中定義, 作為 extension 與網頁溝通橋樑, 通常被注入網頁中.

不同腳本均有獨立的 scope, 三者相互溝通的方式為 messaging API.

## 彈出視窗組件

彈出視窗是 browser-action | page-action 特有的介面,\
user 操作可在彈出視窗完成,\
並且不用離開既有的網頁操作畫面,\
因此可以當作一個獨立組件來看待.

- 不能修改彈出視窗外觀, 但可指定 popup html 內容.

關於 popup script
- 負責彈出視窗組件的腳本
- 只有彈出視窗時才能仔入並且運行
- popup script 不能直接放置於 popup html 裡, 只能以資源方式讓 html 載入.\
  \<script src=\'popup.js\'\>\<\/script\>
- 允許多個 popup script 存在
- 除了可以使用於 chrome extension api 外, 還能像一般網頁 script 去操作 popup html 內的 dom 元素.

## 安裝檔

安裝檔為擴件安裝依據,\
記載許多重要定義,\
以下為安裝檔範例.

```
{  
  "name": "My Extension",  
  "version": "2.1",  
  "description": "Gets information from Google.",  
  "icons": { "128": "icon_128.png" },  
  "background": {  
    "persistent": false,  
    "scripts": ["bg.js"]
  },  
  "permissions": [
    "http://*.google.com/", 
    "https://*.google.com/"
  ],  
  "browser_action": {
    "default_title": "",  
    "default_icon": "icon_19.png",  
    "default_popup": "popup.html"
  },  
  "content_scripts": [
    {  
        "matches" : ["*://*/*"],  
        "js" : ["content.js"],  
        "css" : ["content.css"]  
    }  
   ]  
}
```

## 擴充套件與網頁執行階段

我們可以把擴件當作一個獨立網站,\
他與使用者當前載入的頁面完全是兩個獨立個體,\
兩者之間只能藉由注入內容腳本溝通(跨網域).

以下為三個腳本分別所屬執行階段與延伸議題:

### 事件腳本 | event scripts

- 描述擴充功能的執行階段
- 長時間在背景執行,\
  擔任中央控制器角色
- 除了 event scripts 外,\
  其他腳本無法長時間運作,\
  因此我們需要仰賴 event scripts 實作功能邏輯

### 彈出視窗腳本 | popup scripts

- 描述擴充功能的執行階段
- 只有在 popup 開啟狀態下才會載入彈出視窗腳本
- 部分事件如套件安裝 / 移除時,\
  彈出腳本監聽不到事件.

### 內容腳本 | content scripts

- 描述網頁環境的執行階段
- 因為被視為使用者瀏覽網頁的一部分,\
  可存取 API 資源非常有限
- 只能透過 messaging API 間接使用擴件完整功能
- 可操作、維護使用者瀏覽的網頁,\
  這是另外兩者無法辦到的