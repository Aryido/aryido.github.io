---
title: "Language Server Protocol"

author: Aryido

date: 2024-05-21T23:21:38+08:00

thumbnailImage: "/images/plugin-extension/vscode-logo.jpg"

categories:
  - plugin-extension

comment: false

reward: false
---

<!--BODY-->

> Language Server Protocol 簡稱 LSP ，是微軟於 2016 製定的 Protocol 協定，專門用來輔助 Visual Studio Code 開發用的，目標是讓 Code-Editor 能便利地支援更多的程式語言。設計理念是把**語言撰寫領域模型如：自動補全、引用定義、類型檢查器**等等，這些提供輔助功能的部分拆出去用「公定的介面」來做溝通，給各自領域的人開發。
> {{< image classes="fancybox fig-100" src="/images/plugin-extension/lsp-nolsp.jpg" >}}
> LSP 專門用於描述 Code-Editor 中，用戶行爲與響應之間**通訊方式**和**傳輸資料結構**，像是 VSCode 的 IntelliSense 提供的 auto-completion 就可以基於這個協定支援更多不同的 coding language。 現在支援 LSP 的 Code-Editor 也不少，除了 VSCode 還有 Eclipse 、 Vim 、 NeoVim 都已經支援了，可以 在 langserver.org 可以看到各個 client 的支援狀況。

<!--more-->

---

實現**每個 Code-Editor** 上的**每個程式語言的輔助功能**如：auto-completion、跳轉或 hover 出現 Doc 解釋功能等等，都是一項花時間心力的工作。傳統上必須對每個開發工具重複這項工作，因為每種程式開發平台都提供不同的 API 來實現相同的功能，而 Language Server Protocol 背後的理念是標準化，把上述這些功能都可以抽象化為一系列的「行為事件」，因此**只要遵循 LSP 協定實作某個語言的特性功能後，Code-Editor 只需要呼叫該語言的 Language Server ，即可實現程式碼提示、定義跳轉、程式碼診斷等功能**，之後就可以重複利用了！

# Language Server Protocol Specification

Language Server Protocol 是使用 JSON-RPC 溝通，故本質上是一種基於 Process 間通訊的協議，依靠這個協議讓 Extension Plugin 和 Language Server 間進行溝通。由於 LSP 是一個「雙工協議」，通訊是雙向都可以發起的，故每個 RPC 事件會需要定義 : **發起方向** 、 **是否需要對方回應**。舉例來說，在 [Language Server Protocol Specification](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/) 定義文件中，找一個 request 來看一下：
{{< image classes="fancybox fig-100" src="/images/plugin-extension/jsonrpc-request-example.jpg" >}}
在文件中看到 title 最右邊尾部，有個藍色小圖示，
有一個從左到右然後回轉的箭頭，這就表示是 **Extension 發起請求，且要求 Language Server 的必須回傳事件**。
{{< alert warning >}}
由於 Language Server Protocol 沒有限制一定要所有功能都有支援，所以有些 Language Server 可能沒有支援特定功能，故具備哪些能力都會在初始化階段告知，以避免後續產生某些無效的功能請求。
{{< /alert >}}

# LSP 通訊流程

Language Server 和 Extension 會持續不斷地進行各式各樣的請求體通訊，其互動一般需要遵循以下生命週期，工作流程如下：

### 初始化 (Initialize)

由於 Extension 和 Language Server 都作為單獨的 Process 運行，故都不會知道彼此目前的狀態，所以第一個 RPC 請求一定是 Extension 傳送 `initialize` 請求，包含一些初始化參數 ; Server 收到請求後開始準備啟動 Language Server，之後會回傳 `initialized` 完成通知給 Extension 表示 Language Server 已經開始運作，同時告知 Extension 目前 Server 具有哪些能力。
{{< alert info >}}
雖然不用每個事件都強制要實現，但當然也有些通知事件是一定要實現的，例如 :

- textDocument/didOpen
- textDocument/didChange
- textDocument/didClose

隸屬於 Text Document Synchronization。
{{< /alert >}}

啟用完成後，Client 和 Language Server 就可以相互溝通了，Client 和 Server 都可以相互發送請求/通知，發送的各種請求有哪些，也是參考 [Language Server Protocol Specification - 3.17](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/)，簡單舉例如有 :
- textDocument/completion(下語法提示快捷鍵，請求取得智慧提示清單)
- textDocument/hover
- ...

### 事件流程

初始化完成之後，接下來舉個實際例子，下圖解釋我們在開發時經常使用`Go to Definition`功能的通訊流程圖：
{{< image classes="fancybox fig-100" src="/images/plugin-extension/lsp-communication.jpg" >}}

##### textDocument/didOpen

打開 Document 時， Extension 會偵測到並發通知`textDocument/didOpen`給 Language Server，然後 Document 內容會保存到 Extension 記憶體中，也同步 synchronized 內容給 Language Server。以下是一個範例通知結構，VSCode 打開 main.go 文件 :

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello World go!")
}
```

這時 extension 會發送 jsonrpc 請求：

```json
{
    "jsonrpc": "2.0",
    "method": "textDocument/didOpen",
    "params": {
        "textDocument": {
            "uri": "file:///workspace/main.go",
            "languageId": "go",
            "version": 2,
            "text": "package main\n\nimport (\n\t"fmt"\n)\n\nfunc main() {\n    fmt.Println("Hello World go!")\n}"
        }
    }
}

```

##### textDocument/didChange

再來當我們開始編輯 Document 時， Language Server 會發送編輯產生的 **diff 部分** 而非更新後的整體內容。舉例來說:

>1. 我們在 code 中新增一行 `a`：
>
>   ```go
>   package main
>   import (
>       "fmt"
>   )
>
>   func main() {
>       fmt.Println("Hello World go!")
>   +   a
>   }
>   ```
>
>   Extension 會偵測到並更新文檔的內容，然後發通知 `textDocument/didChange` 給 Language Server：
>
>   ```
>   {
>       "jsonrpc":"2.0",
>       "method":"textDocument/didChange",
>       "params": {
>           "textDocument": {
>               "uri": "file:///workspace/main.go",
>               "version": 37  // 用於確認先後順序
>           },
>           "contentChanges": [{
>               "range": {
>                   "start": {
>                       "line":8,
>                       "character":4
>                   },
>                   "end": {
>                       "line": 8,
>                       "character": 4
>                   }
>               },
>               "rangeLength": 0,
>               "text": "a"
>           }]
>       }
>   }
>   ```
>
>   Language Server 會更新的記憶體中的內容，再來是決定是否產生某些行為，如程式碼診斷發送 `textDocument/publishDiagnostics` 通知回給 Extension。

再用其他例子舉例來說 :

> 2. 假設 user 使用了 `Go to Definition`，則 Extension 會發送 `textDocument/definition`，以下範例是一個具體 `textDocument/definition` request/response json 格式：
>
>    ```
>    ## request
>    {
>        "jsonrpc": "2.0",
>        "id" : 1,
>        "method": "textDocument/definition",  ##  method 表觸發的事件
>        "params": {
>            "textDocument": {
>                "uri": "file:///VSCode/Playgrounds/cpp/use.cpp"
>            },
>            "position": {
>                "line": 3,
>                "character": 12
>            }
>        }
>    }
>
>    ## response
>    {
>        "jsonrpc": "2.0",
>        "id": 1,
>        "result": {
>            "uri": "file:///VSCode/Playgrounds/cpp/provide.cpp",
>            "range": {
>                "start": {
>                    "line": 0,
>                    "character": 4
>                },
>                "end": {
>                    "line": 0,
>                    "character": 11
>                }
>            }
>        }
>    }
>    ```
> 上述通知傳送給 Language Server 後，會回應 responses ，並在`Go to Definition`的使用位置顯示 Result。
> {{< alert success >}}
`Go to Definition` 簡單說法就是所謂**引用跳轉功能**，我們會點選 class、function、variable 等等，看看還有哪些地方有出現相同引用、有哪些地方有呼叫等等。
{{< /alert >}}

##### textDocument/didClose

最後當我們關閉 Document 時，Extension 會偵測到並發通知 `textDocument/didClose` 給 Language Server 表示 Document 已不再在記憶體中。

{{< alert info >}}
由於 Code-Editor 和 Language Server 是兩個進程 process，所以如果 Language Server 掛了，編輯器進程本身也會存在，不用擔心還沒修改好的程式碼因此遺失的問題。
{{< /alert >}}

# 結語

上述範例說明 Extension 使用 LSP 和 Language server 的互動流程，不難發現 LSP 的中立性和抽象性，因為不包含任何 program symbol information。

---

### 參考資料

- [What is the Language Server Protocol?](https://microsoft.github.io/language-server-protocol/overviews/lsp/overview/)

- [Language Server Protocol](https://blog.othree.net/log/2018/07/28/language-server-protocol/)

- [blog/article
  /language-server-protocol.md](https://github.com/Aaaaash/blog/blob/master/article/language-server-protocol.md)

- [理解 Language Server Protocol 的工作原理](https://juejin.cn/post/7051453384645148680)
