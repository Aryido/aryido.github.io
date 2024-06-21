---
title: "VSCode 基本 layout 介紹"

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
> 整合式開發環境 Integrated Development Environment 簡稱為 IDE ，是一種程式開發軟體，通常只針對特定語言，且有應用程式生命週期管理 ALM 功能 ; 另一方面文字編輯器 Text Editor 通常只提供如複製、剪下、貼上、搜尋、取代等文字操作功能，但現在兩個已經沒有區分的這麼明顯，原因就是 VSCode 的出現很大了融合了兩者。 VSCode 其定位採用核心是 <**Text Editor + code understanding + debug**>，並以**檔案夾**作為專案管理，保持輕量與高性能，但還是提供強大的支援語法。
> {{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-positioning.jpg" >}}
與許多程式編輯器一樣，VSCode 也採用通用 layout，設計理念是希望可以提供簡單直覺的工具，讓使用者可以直觀的選取檔案並閱覽、編輯檔案，本文章首先討論 Workbench 的定義和介紹 vscode layout 各個名稱。
 

<!--more-->

---

VS Code 具有簡單直觀的佈局，其 User Interface 可分為五個主要區域：

{{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-layout.jpg" >}}

# Workbench
VSCode 的整個介面統稱為 Workbench(工作臺)，即為上圖整個畫面，包含以下 UI 元件: 
- Activity Bar
- Side Bar
- Panel
- Editor Group
- Status Bar 

這五個部分在 VSCode 的 extension api 裡，同屬於 Workbench 底下。透過 extension api，我們可以任意擴展這些 workbench 底下的元件，當我們要在 vscode 的相關 setting 檔設置跟這些元件相關的個人配置時，也是要在 workbench 的 namespace 底下配置。

# (A) Activity Bar
通常位於最左側，也可以更改位置。上面可以放多個 icon 按鈕，從上到下的 icon 分別會呈現的 View 視圖為：
> - Explorer
> - Search
> - Source Control(GIT)
> - Run and Debug
> - Extension

通常 Activity Bar 的 icon 按鈕會和 SideBar 聯動，點擊不同按鈕可以切換不同 view 至 SideBar 內 ; 再次點擊已開啟 view 的 icon 按鈕會關閉 SideBar。關於 icon 按鈕，也有提供 context-specific indicators 功能，用個清楚的例子解釋，例如說 Source Control(GIT) ，會顯示出藍色的小數字表示更改檔案的數量在 icon 按鈕上，這個就是 context-specific indicators。


# (B) Side Bar
有分成 Primary Side Bar 和 Secondary Side Bar，都是用來顯示 view 的，而最常用的 view 大概就是 Explorer，可把專案的檔案以 tree 的方式呈現。使用 ```⌥⌘B``` 顯示 Secondary Side Bar，並把 view icon 拖動到 Secondary Side Bar，就可以顯示該 view 了。


# (C) Editor
Editor 是用來呈現和編輯 file 檔案的主要區域。可以根據需求垂直、水平、打開任意數量的 Editor ，這個群體統稱為 Editor Group。
{{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-editor-group.jpg" >}}
內部還有一些小組件 ： 
- ##### Breadcrumbs

    Editor 的頂部有一個 navigation bar，如下圖紅色圈起來部分，會顯示檔路徑，稱為 **Breadcrumbs**。如果當前 file 對其語言有支援，還可以更快速地 navigate 到指定 code 位置。
    {{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-breadcrumbs.jpg" >}}

- ##### Tabs
    每當打開一個文檔時，預設都會新增加一個 tab ，如下圖紅色圈選處，可幫助在不同 file 之間快速切換。當 tab 太多時，在 tab 和 Breadcrumb 之間還會有個 scroll bar 可以拖動，雖然還蠻不明顯的。
    {{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-tabs.jpg" >}}
    {{< alert info >}}切換 tab 還可以使用 `Command + Option + 方向鍵`{{< /alert >}}
    
    在 Explorer 中**點擊一次**打開檔案，會發現 tab 的名稱是 italics 斜體字，代表為預覽模式，稱為 **Preview mode** ，如下圖顯示。這時在選擇其他檔案時，會重複使用現有 Tab 並覆蓋，而不會開啟新的 Tab。
    {{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-preview-tab.jpg" >}}
    如果只是想要簡單且快速的看一下特定幾個檔案的內容，這個設定並不會產生新分頁，後續就不用另外再關閉分頁。若要從 Preview mode 換成正常 tab :
    - 1. 可以**點擊兩次**檔案來創建完全新的 Tab
    - 2. 開始編輯（任意輸入/刪除檔案內文字觸發編輯

# (D) Panels
通常位於 Editor 區域下面，主要有四個：
- **Problems Panel**: 顯示當前 folder 下 code 的所有問題和警告，比如語法錯誤、格式問題、拼寫錯誤等，可以通過這個 Panel 瀏覽這些問題並且訪問對應的檔案。
- **Output Panel**: 會把 CLI 和 Extension 的運行狀態和結果輸出到 Output Panel。比如說用內置的 Source Control 來管理 code，其每個在 Source Control 上的 UI 操作，大概都能在 Output Panel 裡看到這個操作對應的 Git CLi命令行以及它的運行結果。故即使意外發生了，依然可以通過此 Panel 找到問題。
- **Debug Console**: 結合 debug 模式使用。
- **Terminal**: 可選擇各式終端環境例如 zsh、bash 等等。

# (E) Status Bar
是最下方的藍色條，用於呈現各種狀態。例如可用於呈現 project 的資訊、 activity 狀態的 editor 的資訊、正在編輯的 file 的資訊。例如說當前 cursor 所在的 line、column 位置，selected 字數，編輯檔案的 coding 語言...等等。

# 其他
### Minimap
{{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-minimap.jpg" >}}
Minimap 就是一個 code 大綱（code outline）可以快速 navigate 到指定區域，通常顯示在編輯器的右側。
{{< alert info >}}
如果說覺得它有些佔畫面的話，可以使用 `Command/control + Shift + P` 叫出 Command Palette 然後使用 `Toggle Minimap`，可以開啟或關閉它。
{{< /alert >}}


### Command Palette
palette 發音是 ```/ˈpæl.ət/```，可使用 `⇧⌘P` 來打開，從這裡可以搜尋到幾乎所有 VSCode 功能 CLI。
{{< image classes="fancybox fig-100" src="/images/plugin-extension/vscode-command-palette.jpg" >}}


---

### 參考資料

- [VSCode Office - User Interface](https://code.visualstudio.com/docs/getstarted/userinterface)


- [Day02 | VSCode使用者介面概覽](https://ithelp.ithome.com.tw/m/users/20108634/ironman/3815)

- [tommymsw/vscode-1](https://github.com/tommymsw/vscode-1)