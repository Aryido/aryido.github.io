---
title: "快捷鍵筆記簿"

author: Aryido

date: 2025-05-12T20:13:38+08:00

thumbnailImage: "/images/others/hotkey/command.jpg"

categories:
  - develop

comment: false

reward: false
---

<!--BODY-->

> 在現代開發環境中，熟練掌握各種工具的快捷鍵能顯著提升工作效率，本筆記整理了自己常用開發工具的基礎操作與進階快捷鍵，幫助更高效地進行開發工作。**由於自己現在開發主要環境都是使用 mac os，所以快捷鍵會以 mac 的鍵盤配置為主**，筆記會列出的重點工具為 :
>
> - `MacOS Terminal`
> - `Visual Studio Code (VSCode)`
> - `Vim`
>
> 然後可能也會補充一些 `Windows PowerShell` 和 `IntelliJ IDEA`。在日常開發中可有意識的多使用快捷鍵來加深記憶，若遺忘時這裡可以快速查看筆記，鞏固快捷鍵的知識。

<!--more-->

---

# MacOS Terminal

### 游標移動

- ###### 將游標移至 row 最前面： `control 和 a`
- ###### 將游標移至 row 最後面： `control 和 e`
- ###### 單詞為移動單位，快速移動游標: `option 和 左右鍵`

### 刪除輸入的 cli

- ###### 整個 row 刪除: `control 和 u`
- ###### 光標之後全部刪除: `control 和 k`
- ###### 光標之前刪除一個單字: `control 和 w`

### 其他

- ###### 清除終端機螢幕內容，類似 clear 命令: `control 和 l`
- ###### 中斷目前執行的命令或程序: `control 和 c`
- ###### 結束目前的 shell: `control 和 d`

---

# VSCode

### 看 code 時會需要的操作

- ###### 在看 code 時會進入某個方法，之後想要返回之前的地方: `control 和  -`
- ###### 折叠所有 function: `command 和 k 和 數字0`
- ###### 展开所有 function: `command 和 k 和 j`

### 編輯

- ###### 檔案 format: `option 和 shift 和 f`

- ###### 縮排往前 : `command 和 [`
- ###### 縮排往後: `command 和 ]`

- ###### 把某個詞全部都 replace 換掉: `option 和 command 和 f`
- ###### 取消復原: `command 和 shift 和 z`

- ###### 可以垂直 column 選取: `option 和 shift 和 滑鼠拉一下`

  附圖會看起來如下，會發現是以**垂直**的方向來選取，而不是一般的 row 選取:
  {{< image classes="fancybox fig-100" src="/images/others/hotkey/vertical-edit.jpg" >}}


### 查詢

- ###### 使用選取的文字進行搜尋: `command 和 e`
  - ###### 開始搜尋後，尋找下一個: `command 和 g`
  - ###### 開始搜尋後，尋找上一個: `command 和 shift 和 g`

---

# Vim
vim 的操作方式不同於一般編輯器，主要透過模式切換來操作：

  {{< image classes="fancybox fig-100" src="/images/others/hotkey/vim-mode.jpg" >}}

- ###### 全選: `ggVG` (大小寫重要)

  - `gg`：將游標移動到檔案的第一行最上層
  - `V`：進入 Visual Line Mode，開始選取整行
  - `G`：將游標移動到檔案的最後一行， `shift + g` 就會是大寫Ｇ

  完成上述操作後，整個 file 的內容將被選取，此時可以：

  - `y`：複製選取的內容（Yank）
  - `d`：刪除選取的內容（Delete)

- ###### 在游標後貼上複製的文字: `p`

- ###### 全刪除: `:%d`

  - `:`： 進入命令模式。
  - `%`： 表示整個檔案的範圍，等同於 `1,$`，即從第一行到最後一行

- ###### 撤銷上一步操作: `u`

- ###### 整段註解
  - 首先把游標移到整段想註解的開頭
  - `control 和 v`
  - 接下來按[上]/[下] 去選整段想註解的段落
  - `Shift 和 i`
  - 輸入註解的符號例如："#"
  - `Esc`

  就可以看到，整個選取段都會填上註解的符號了。

- ###### 刪除整段註解
  - 首先把游標移到整段想註解的開頭
  - `control 和 v`
  - 接下來按[上]/[下] 去選整段想註解的段落
  - `d`

  就可以看到註解的符號都被刪掉了。


### 搜尋

- ###### 向下搜尋指定的文字: `/文字`
- ###### 向上搜尋指定的文字: `?文字`
- ###### 向下跳一頁: `control 和 f`（forward
- ###### 向上跳一頁: `control 和 b`（backward）
- ###### 向下跳半頁: `control 和 d`（down）
- ###### 向上跳半頁: `control 和 u`（up）

---

### 參考資料

- [Mac 上「終端機」中的鍵盤快速鍵](https://support.apple.com/zh-tw/guide/terminal/trmlshtcts/mac)

- [Mac OS Terminal 几个快捷键](https://www.cnblogs.com/abeen/p/4104158.html)

- [必会！VSCode 最实用的快捷键](https://www.youtube.com/watch?v=diLeGIYL3nQ)

- [31 ! 一些你可能不知道的Tips&Tricks.md](https://github.com/tommymsw/vscode-1/blob/master/31%20!%20%E4%B8%80%E4%BA%9B%E4%BD%A0%E5%8F%AF%E8%83%BD%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84Tips%26Tricks.md)

- [Linux vi/vim](https://www.runoob.com/linux/linux-vim.html)

- [Vim實用小技巧教學，如何整段加上#或移除 | 適用各式Linux系統 | 挨踢實驗室](https://www.youtube.com/watch?v=k3TnS8O8_m8)

- [Vim进阶技巧，你知道多少？](https://www.youtube.com/watch?v=l6pIDcMsz8w)