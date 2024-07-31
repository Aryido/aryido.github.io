---
title: "Yeoman - 專案模板產生器"

author: Aryido

date: 2024-05-21T23:21:38+08:00

thumbnailImage: "/images/others/yeoman-logo.jpg"

categories:
  - plugin-extension

tags:
  - vscode

comment: false

reward: false
---

<!--BODY-->

> 當要建立新專案時，都會需要決定一些關於基礎的架構或開發規劃的事項例如 :
>
> - 設計 folder structure
> - 打包編譯工具的選擇
> - IDE 預設環境配置；IDE plugin 或 extension 的安裝
> - CI/CD、IaC 設定或工具選擇
>
> 每次建立專案時都會參考之前的架構規劃，然後手動複製架構雛形，重新設定參數，其實不太方便... 這時就可以引入 Scaffold 這樣的**模坂**概念，來直接生成**專案的骨架**。本質用意是把那些重複地創建專案基礎結構、專案規格流程取代掉，實現 **DRY (Don’t Repeat Yourself) 原則**。而 **Yeoman** 就是一個著名 scaffolding generator tool，微軟官方維護 Visual Studio Code extensions，就是裡面非常著名的例子。

<!--more-->

---

Yeoman 是一個 Node.js 專案，雖然是用 Javascript 寫成的，但卻不限定所產生的專案語言，可以用來創建: Python、Java 等等各式語言的專案。可以到 Yeoman 官網的 [Discovering Generators page](https://yeoman.io/generators/) 去尋找所需要的範本。有非常多類型的應用。

# 安裝

#### [安装 nodejs](/posts/develop/homebrew/)

參考上述連結的**官方多版本 formula 安裝**，有教學安裝 Nodejs 和版本更換。

#### 安装 yo 和 Generator

Generate 並不隨 yeoman 安裝的，需要我們依照不同的應用框架的需求，自行安裝。故要去 [Discovering Generators page](https://yeoman.io/generators/) 尋找所需要的範本，要安裝的 generator 名稱對應為 : **generator-{Name}**，其中 Name 為 yeoman generators 官網搜尋頁面中，列表顯示名稱。以下用 VSCode extension 來當安裝範例:

```
npm install --global yo generator-code

yo code
```

如果不想 global 安裝 Yeoman 和需要的 Generator，可以使用 npx 來創建模板，**目前強烈推薦使用這方法安裝**:

```
npx --package yo --package generator-code -- yo code
```

---

# 使用心得

會知道 Yeoman 這個工具的原因，是因為有接觸到 [vscode extension](https://code.visualstudio.com/api/get-started/your-first-extension) 的開發，也去了 Yeoman 官網看到許多已經寫好的模板，下載次數也蠻多的，例如: [jhipster](https://www.jhipster.tech/) 為 Spring Boot + Angular/React/Vue 模板；自己也好奇有沒有雲端架構相關的，也有發現 [az-terra-module](https://github.com/Azure/generator-az-terra-module) 模板，且看起來還是官方出的。 另外網路上也有人分享自製 yo 的 Generator 相關教學，感覺應該是還不錯的工具，**但實際使用後發現有蠻多問題的...**

### jhipster

雖然看起來是非常有名，且是有很多人下載過的專案模板，但我卻遇到非常多問題，例如:

> - The punycode module is deprecated.
> - this.log.debug is not a function

雖然有一個很漂亮的的官網，內容似乎也很豐富，但使用時遇到 nodejs 升版棄用 punycode 導致錯誤；以及目前還不太確定的 `this.log.debug is not a function` 問題... 上述問題有些似乎存在有點久了，但都還沒修好，故目前我無法使用 jhipster 來造出模板，無法確定是否真的不錯用...

### az-terra-module

一開始也是不能使用，但經過降板 nodejs 為 20 LTS 避免了 punycode 棄用問題，就順利地創建專案骨架了。進一步去觀看內容，有發現一些文件 :

{{< image classes="fancybox fig-100" src="/images/others/yoeman-azure.jpg" >}}

稍微查一下，前面兩個是 ruby on rails 相關常用檔案；然後有 Go 的包管理依賴檔案，是對應 test.go 用到的模組，以上檔案的使用，對於一個好的 azure terraform module 是必要的嗎我也抱持懷疑；而且 **folder-structure** 算是好的嗎我也抱持觀望，我甚至有去看 azure 官方其他 module 的架構寫法，似乎也和這個 Scaffold 不太一樣...

### 結論

目前我看 yoeman 前面幾名下載量的範本，我自己實際使用體驗其實都不太好，而且 yoeman 官方似乎也沒有很積極去修正，在思考是不是退流行的指標...

另外由於 terraform 架構我有實際經驗，並好好思考分析過，故以 azure terraform module 為範例來想: 所謂的 Scaffold 到底要怎麼定義，大家才覺得好用呢? 工具的選擇，甚麼才是業界最能接受的標準呢? 像是 Makefile 我可以接受，但 Rakefile 真的不好嗎? gopkg.toml 和 go.mod 要如何選擇呢? terraform 的測試要用 terratest 嗎還是別的? 是要把雲端資源都寫入一個 main.tf 嗎還是接受一些拆分? 有時真的不好給出結論...

> 整體來說，目前 **yoeman 我只會當成專為 vscode plugin 專案建置 Scaffold 的工具，並不會考慮來建造其他類型專案...**
> 本來有想說好像沒有 gcp terraform 架構的 Scaffold ，網路上也很多教學製作，說不定可以來嘗試寫! 但考慮到現在 yo 常發生奇怪的錯誤，以及思考所謂架構的通用性後，寫 gcp terraform Scaffold 這個優先序會擺很後面吧 ~

---

### 參考資料

- [YEOMAN Office](https://yeoman.io/contributing/)

- [用 yeoman 打造自己的项目脚手架](https://greenfavo.github.io/blog/docs/03.html)

- [使用 Yeoman 開發一個自動化專案](https://medium.com/@danielhu95/use-yeoman-code-generator-to-make-your-life-easier-6d76695e5a37/)

- [dream_fly_here](https://www.cnblogs.com/dreamFromHere/p/3511319.html)
