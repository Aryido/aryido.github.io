---
title: "VSCode 架構簡介"

author: Aryido

date: 2024-05-21T23:21:38+08:00

thumbnailImage: "/images/plugin-extension/vscode-logo.jpg"

categories:
  - plugin-extension

tags:
  - vscode

comment: false

reward: false
---

<!--BODY-->

> Visual Studio Code 簡稱 VSCode ，是微軟 2015 年開源的輕量化 code editor ，基於 Electron 並用自家雲端的編輯器 Monaco Editor 作為其底層開發，支援多語言及平台且使用 TypeScript 來進行編寫，也提供了強大的外掛程式拓展機制給人加強功能。VSCode 的開發是由 Eclipse 之父 Erich Gamma 領導（ Erich 也是《設計模式》作者之一），在 2019 年的 Stack Overflow 開發者研究中，其獲選最受歡迎的 code editor 有 `50.7%` 的使用率。
> {{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-context-overview.jpg" >}}

<!--more-->

---

# 由來

Visual Studio Code 的由來可先追溯到在大概 2011 年，廣泛用於微軟內部及外部一些 Web 產品的編輯器 Monaco Editor，後來 Monaco 變成一個內部產品代號，並於 2013 年推出成為一個上線的網頁編輯器產品 Visual Studio Online，其介面與較老版本的 VSCode 一樣，最後 Visual Studio Online 也做了些改善並搬到了**桌面端**，就變成我們現今的 VSCode 。

承前考古不難想像 Monaco Editor 和 VSCode 有一大部分的程式都是共用的，實際上也是如此的，在 VSCode source code 也可發現 monaco-editor-core 基本成為 VSCode 的基石 ; 最後甚至在 editor 交互以及 UI 上例如佈局、輸出面板、終端...等等，兩個也幾乎是一樣的。

{{< alert info >}}
隨著 VSCode 的生態逐漸成熟， Visual Studio Online 也回頭參考 VSCode ，並於 2019 年短暫的運營了一段時間以後，改名叫做 GitHub Codespaces。
{{< /alert >}}

由於 **VSCode 基於 Electron 框架構建**，故接下來簡介一下 Electron。 

# Electron

Electron 原名 Atom-Shell，是 Github 為 Atom editor 編寫的開源框架，它將 Chromium 與 Node.js 融合，讓開發者能用使用 HTML 來實現 UI ，並用 Node.js API 來存取文件系統。

> 為什麼會使用 Electron 這個框架的呢？ 因為單純只用 js 是做不了桌面應用的，一般來說 Brower 並不提供對 local file 訪問的能力，而 VSCode 作為一款**桌面編輯器**，是一定需要對本地文件進行開發的。

在 web 頁面裡呼叫系統底層的 API 處理底層 GUI 資源是非常危險的，容易導致安全問題，為了保護本地文件的安全性，和與 filesystem 交互的能力，故利用 Nodejs 實現本地文件系統的訪問功能。 Chromium 基於 HTML、CSS、JS 可以做出好看的 UI 介面互動，是已經很成熟的 Web UI 技術 ; 再加上 Nodejs 實現 filesystem 和基礎通信功能，故 VSCode 選擇已經完成此實現的框架 **< Electron >** 來發作為開發基礎。

{{< image classes="fancybox fig-100" src="/images/plugin-extension/electron-architecture.jpg" >}}

- Chromium: 使用 web 技術實現 ui 界面，可以簡單理解 chromium 是開源且無需安裝可直接使用的 beta 體驗版 chrome
- Nodejs: 用產生桌面文件系統來存取本地檔案，以及網路功能
- Native-API: 採用 C/C++ 語言編寫，用來調用操作系統 API，可以理解是對 Nodejs 的擴展

{{< alert success >}}
Electron 擁有 Node 運行環境，賦予了使用者與系統底層互動的能力，依靠 Chromium 提供基於 Web 技術（HTML、CSS、JS）的介面互動支持，另外還具有一些平台特性，例如桌面通知等
{{< /alert >}}
簡單來說 ：

- Monaco 基於 Brower 框架開發
- VSCode 基於 Electron 框架開發

---

# VSCode 架構

讓 VSCode 高效快速的原因，還有因為 VSCode 採用多進程架構，將耗時的任務分到多個進程中，有效的保證了主進程的回應速度。

{{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-architecture.jpg" >}}

API 設計上 Electron 是 multi-process 的:

- ##### **1 個 Main Process** :

每一個 Electron 應用，只會產生一個主進程，通過執行 package.json 中的 main 所指向的檔做為 VSCode 的入口，主要負責**管理編輯器生命週期**、**功能表註冊**、**進程間通信（IPC Server）**，可存取 Node API 和 Native API。啟動 VSCode 的時候，進程會首先讀取各種配置資訊和歷史記錄，然後啟動一個瀏覽器視窗來顯示編輯器的 UI。 Main Process 會一直關注 UI 的狀態，當所有 UI 被關閉的時候，整個編輯器退出。

- ##### **多個 Renderer Process** :

由於 electron 使用 Chromium 來顯示 UI ，故 Chromium 多進程架構也會被用到。 Main Process 會通過 Chromium API 創建 Renderer Process，而 workbench 內的每一個 UI 元件會對應一個 Renderer Process，其主要功能是存取 Browser API 和一部分 Node API、Native API。

Renderer Process 一般只有負責 UI 相關的，如果想要在 UI 上執行 GUI 操作，相應的 Renderer Process 必須與 Main Process 進行通信，VSCode 內部提供了通訊機制如: IPC 進程通訊模組、RPC、Shared Process。

{{< alert warning >}}
Workbench 是指包含以下 UI 元件的整體 Visual Studio Code UI：Title Bar、Activity Bar、Side Bar、Panel、Editor Group、Status Bar
{{< /alert >}}

- ##### **多個 Extension Host** :

Extension Host 也是獨立的 Process，用於運行其它 extension，並可以存取一些 VSCode 的 API 供 extension 去使用。 由於 VSCode 考慮到 Extension 可能會影響啓動性能和 IDE 自身的穩定性，所以通過**進程隔離**來解決這個問題，每個 Extension 都運行在一個自己的 NodeJS 環境中，彼此間由 Shared Process 進行通信。

這樣設計最主要的目的就是避免複雜的 Extension 阻塞 UI 的回應，但是將 Extension Host 放在其他的 Process 也有很明顯的缺點: 

> 因為不是直接屬於 UI Process，所以沒有辦法直接訪問 DOM tree，故想要即時高效的改變 UI 會比較困難，也比較難客製化 VSCode 外觀。

VSCode 開發團隊，也蠻常為了優化 VSCode 而頻繁更改 UI Dom，所以將 UI 定製能力限制起來也方便他們做事情。


- ##### **其他的進程** :

  - Debug Process： 它不屬於 Extension Host 這個 Process ，而是在每次 debug 的時候由 UI 單獨新開一個進程

  - Search Process： 搜索是密集型任務，單獨佔用一個進程，保證主視窗的效率

{{< image classes="fancybox fig-100" src="/images/plugin-extension/processes.jpg" >}}

---

### 參考資料

- [VSCode Office](https://code.visualstudio.com/docs)

- [VSCode 技术揭秘](https://juejin.cn/post/6844903981559316488)

- [Erich Gamma at SpringOne Platform 2017](https://www.youtube.com/watch?v=Vs3AGfeuNKU)

- [Monaco Editor](https://juejin.cn/post/7146457023415058468)

- [vscode 插件原理淺析與實戰](https://www.readfog.com/a/1669679579178045440)
