# 輸入組件：右鍵選單

使用者正在關注的頁面上,\
開發者可利用右鍵功能選單新增自訂擴件功能,\
並讓使用者與其互動.(圖片、連結、選取文字等)

## 右鍵選單能做什麼事

- 取得使用者正在瀏覽或是他所關注的元素,\
  如圖片或連結網址, 當前頁面網址, 選取文字等.
- 可以有子項目或使用分割線群組你的項目
- 可以在特定元素下才能使用擴件選單項目,\
  如圖片、影片

## 應用場景

- 字典 : 文字選取並進行中英查詢
- 圖片蒐集 : 使用者當前指標的圖片網址, 加入蒐集冊
- 筆記蒐集 : 使用者所選的文字片段, 蒐集到線上筆記本
- 連結分析 : 使用者當前指標的連結是否為惡意網址

## 安裝檔

```
{
  "manifest_version": 2,
  "name": "My Context Menu Extension",
  "description": "My Context Menu Extension",
  "version": "2.0",
  "permissions": [
    "contextMenus"
  ],
  "icons": {
    "16": "icon.png"
  },
  "background": {
    "scripts": [
      "event.js"
    ],
    "persistent": true
  }
  ...
}
```

- 需有 contextMenus 權限
- icon 宣告方式, 與頁面按鈕、瀏覽器按鈕的 icon 不同
- 提醒：如果不把常駐(persistent)設為 true,\
  要同時注意擴件字串指定使用項目 id 不可重複,\
  以及開發上要注意腳本重複載入,\
  導致選單元素重複生成問題.

## 創建選單元素

以下方法為創件元素方法, 一次創建一組.\
createProperties 為項目屬性, callback 為創建完成後回調.

```
chrome.contextMenus.create(createProperties, callback);
```

創建元素範例

```
let normal = chrome.contextMenus.create({
  "title": "使用者選擇:'%s'",
  "id": "item1-1",
  "type": "normal",
  "contexts": [
    "all",
    "page",
    "frame",
    "selection",
    "link",
    "editable",
    "image",
    "video",
    "audio",
    "launcher",
    "browser_action",
    "page_action"
  ],
  "documentUrlPatterns": [
    "https://*.google.com.tw/foo*bar"
  ],
  "targetUrlPatterns": [],
  "enabled": true,
  "onClick": function(info, tab) { ... },
  "parentId": "item1",
  "checked": false
});
```

屬性說明

- type 選單的格式, 必填
  - normal
  - checkbox
  - radio
  - separator
- id 識別字母, 在同一個擴件必須唯一, 必填
- title 項目標題, 可使用%S將取得使用者右鍵選中之文字
- context 那些特定項目會啟用此項目
- documentUrlPattern 那些網域會啟用此項目, 使用匹配表達式
- targetUrlPattern 那些網域會啟用此項目, 使用匹配表達式,\
  並支援相對路徑(主要用於具備 src 屬性類型資源檔)
- enabled 啟用或禁用此項目
- onclick 項目被點擊的回調方式
- parentId 巢狀項目的父層項目
- checked 用於radio/checkbox, 可指定預設是否勾選

## 監聽點擊的項目

- 在創建項目時, 在事件上綁定監聽方法

```
function listenOnclickEvent(info, tab)
{
  console.log('Id:' + info.menuItemId);
  console.log('Url:' + info.pageUrl);
  console.log('Text:' + (info.selectionText ? info.selectionText : ''));
  console.log('HoverSrc:' + (info.srcUrl ? info.srcUrl : ''));
  console.log('HoverUrl:' + (info.linkUrl ? info.linkUrl : ''));
  console.log('Iframe:' + (info.frameUrl ? info.frameUrl : ''));
}

let item = chrome.contextMenus.create({
  "title": "你選擇了%s",
  "context": ['all'],
  "onclick": listenOnclickEvent
});
```

- 或是, 在創建項目這個動作上綁定閉包,\
  但是必須要透過閉包內實作操作邏輯,\
  以辨別操作是否是當前開發的插件.

```
chrome.contextMenus.onClicked.addListener((info, tab) => {
  console.log("Id: %s", info.menuItemId);
  console.log("SelectId: %s", info.selectionText);
  console.log("Url: %s", tab.url);
});
```

## checkbox 與 radio 項目

右鍵選單可創建複選型(checkbox)/單選型項目(radio),
這兩組項目較其它右鍵選單有以下差異

- 可以 checked 設定預設值
- 點擊時, 事件回調會多接收到以下屬性
  - checked : 目前狀態
  - wasChecked : 最後的狀態

## 範例

範例包含以下示範

- 所有類型右鍵選單
- 朝狀目錄
- 取得背景資訊, 透過 console.log 顯示

manifest.json

```
{
  "manifest_version": 2,
  "name": "My Context Menu Extension 1",
  "description": "My Context Menu Extension 1",
  "version": "1.0",
  "permissions": [
    "contextMenus"
  ],
  "icons": {
    "16": "icon.png"
  },
  "background": {
    "scripts": [
      "event.js"
    ],
    "persistent": true
  }
}
```

event.js

```
function listenOnclickEvent(info, tab)
{
  console.log('選項ID : ' + info.menuItemId);
  console.log('父層選項ID : ' + info.parentMenuItemId);
  console.log('當前網址:' + info.pageUrl);

  if (info.srcUrl !== undefined)
    console.log('資源網址:' + info.srcUrl);

  if (info.linkUrl !== undefined)
    console.log('連結網址:' + info.linkUrl);
}  

function createMenus() {
  let parent = chrome.contextMenus.create({
    "title": "標準項目",
    "type": "normal",
    "contexts": ['all'],
    "onclick": listenOnclickEvent
  });

  let options = {
    select: chrome.contextMenus.create({  
      "title": "選取的文字: %s",  
      "type": "normal",
      "contexts": ['all'],      
      "parentId": parent,
      "onclick": listenOnclickEvent
    }),
    checkbox: chrome.contextMenus.create({
      "title": "checkbox",
      "type": "checkbox",
      "contexts": ['all'],
      "parentId": parent,
      "onclick": listenOnclickEvent
    }),
    separator1: chrome.contextMenus.create({
      "title": "separator",  
      "type": "separator",  
      "contexts": ['all'],  
      "parentId": parent 
    }),
    radio1: chrome.contextMenus.create({  
      "title": "radio 1",  
      "type": "radio",  
      "contexts": ['all'],  
      "parentId": parent,  
      "onclick": listenOnclickEvent
    }),
    radio2: chrome.contextMenus.create({  
      "title": "radio 2",  
      "type": "radio",  
      "contexts": ['all'],  
      "parentId": parent,  
      "onclick": listenOnclickEvent
    }),
    separator2: chrome.contextMenus.create({
      "title": "separator",  
      "type": "separator",  
      "contexts": ['all'],  
      "parentId": parent 
    }),
  };

  console.log(options);
}

createMenus();
```

  
