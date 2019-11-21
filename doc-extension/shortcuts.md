# 輸入組件：快速鍵

當快速鍵被觸發時,\
可以利用 Command API 處理對應操作.

## 快速鍵能做什麼事

- 讓使用者更方便使用擴件, 提升使用經驗.
- 允許綁定 chrome 提供預設操作(快速啟動瀏覽器, 快速啟動頁面按鈕)
- 可覆蓋原有 chrome 定義的預設操作, 並客製化操作邏輯.

## 快速鍵規範

- 一個擴件允許有多組快速鍵設定
- 一個快速鍵設定最多只能由四個鍵組成
- 支援按鍵
  - A ~ Z
  - 0 ~ 9
  - , .
  - Home, End, PageUp, PageDown, Insert, Delete
  - 方向鍵
  - 多媒體鍵
- 所有組合鍵必須包含 Ctrl 或 Alt 其中一組
- 不允許使用 Ctrl + Alt 組合鍵, 
  但可 Ctrl + Shift 或 Alt + Shift 組合鍵
- Mac 的 Ctrl 會轉換成 Command 鍵, 
  如要正確使用 Ctrl 請使用 MacCtrl
- 快速鍵有優先等級, 
  如是視窗管理的快速鍵高於擴鍵快速指令, 
  無法覆蓋.

## 安裝檔

以下為快速鍵範例

```
{
  ...
  "commands": {
    "switch-icon": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "MacCtrl+Shift+Y"
      },
      "description": "切換Action的Icon"
    },
    "_execute_browser_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y",
        "chromeos": "Ctrl+Shift+U",
        "linux": "Ctrl+Shift+J"
      }
    },
    "_execute_page_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+E",
        "windows": "Alt+Shift+P",
        "mac": "Alt+Shift+P"
      }
    }
  }
  ...
}
```

- 快速鍵允許 key/value 成對宣告
- 屬性 global 可設定你的快捷觸發是所有視窗(true), 或當前的視窗(false), 但 ChromeOS 目前不允許.
- 使用 suggested_key 可指定特定作業系統.

## 監聽命令的觸發

運用 onCommand 監聽事件,\
callback 接受快速鍵名稱的回傳值去做對應的動作.

```
chrome.commands.onCommand.addListener((command) => {
  console.log('Command:' + command);
});
```

你也可以透過 getAll 取得你的擴件所有快速設定.

```
chrome.commands.getAll((command) => {
  console.log('Command:' + command);
});
```

## 內建操作

_作為保留前綴,\
保留給擴件作為內建操作,\
開發者定義快速鍵使用保留前綴相關組合會發生錯誤.

```
{
  ...
  "commands": {
    ...
    "_execute_browser_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y",
        "chromeos": "Ctrl+Shift+U",
        "linux": "Ctrl+Shift+J"
      }
    },
    "_execute_page_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+E",
        "windows": "Alt+Shift+P",
        "mac": "Alt+Shift+P"
      }
    }
    ...
  }
  ...
}
```

## 關於 _execute_browser_action 與 _execute_page_action

_execute_browser_action 與 _execute_page_action 只會觸發預定的彈出視窗腳本,\
不提供任何可處理的事件(如 onClicked),\
如果有些邏輯必須與彈出視窗一起載入,\
可考慮在彈出視窗腳本內透過 onDomReady 事件處理.

當沒有設定 _execute_browser_action 與 _execute_page_action,\
它會自動觸發 browserAction.onClicked 與 browserAction.onClicked 所屬事件.

```
1. 腳本內容無法直接使用 Command API,
   但可利用 Chrome 的 Messaging API 讓事件腳本收到快速鍵觸發後,
   傳遞訊息給內容腳本.

2. 只要有指定彈出視窗,
   browserAction.onClicked 與 browserAction.onClicked 所屬事件都會就不會被觸發.
```

## 使用者快捷鍵管理介面

輸入 chrome://extensions/configureCommands,\
便可呼叫快速鍵管理界面,\
使用者可覆蓋安裝檔原始設定,\
重複宣告則會覆蓋前一組所設定的快捷鍵.

## 範例： 利用快速鍵改變背景顏色

manifest.json

```
{
  "manifest_version": 2,
  "name": "My Shortcut Extension 1",
  "description": "My Shortcut Extension 1",
  "version": "1.0",
  "page_action": {
    "default_title": "switch bg color",
    "default_icon": "icon.png"
  },
  "permissions": [
    "tabs",
    "activeTab",
    "declarativeContent"
  ],
  "commands": {
    "switch-bg-color": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "MacCtrl+Shift+Y"
      },
      "description": "switch bg color"
    }
  },
  "background": {
    "scripts": ["event.js"],
    "persistent": false
  }
}
```

event.js

```
let toggleBg = false;
let color = 'white';
chrome.commands.onCommand.addListener((command) => {
  console.log('Command:', command);

  switch (command) {
    case 'switch-bg-color':
      color = toggleBg === false ? 'white' : 'gray';
      console.log('Color:', color);
      chrome.tabs.executeScript({
        code: 'document.body.style.backgroundColor="' + color +'"'
      });
      toggleBg = !toggleBg;
      break;

    default:
  }
});
```