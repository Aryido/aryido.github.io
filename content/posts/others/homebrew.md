---
title: "Homebrew 介紹和常用操作"

author: Aryido

date: 2024-05-20T19:38:28+08:00

thumbnailImage: "/images/others/homebrew-logo.jpg"

categories:
- develop

comment: false

reward: false

---

<!--BODY-->
> Homebrew 是一個廣泛使用在 MAC 上的**套件管理工具**，可以安裝一些 Mac App Store 上沒有的軟體，其操作十分方便，可以簡化軟體安裝的過程，是個很有名的**非官方工具**，由 Max Howell 以 Git 和 Ruby 為基底寫成，並通過 GitHub 維護，為 2012 年 GitHub 上擁有最多新貢獻者的專案。對於其作者也有個有趣的軼事：Max Howell 曾應聘過 Google 的職位，面試失敗之後在 Twitter 上發文章 :
> - Google: 90% of our engineers use the software you wrote (Homebrew), but you can't invert a binary tree on a whiteboard so f*** off.
> 
> 因此在網上引發了面試白板題的討論。
<!--more-->

---


就如同 Homebrew 官網所說：```The Missing Package Manager for macOS```，一開始 macOS 並沒有很好的套件管理工具， 官方 App Store 經常讓人詬病！ Apple 對於 Mac App Store 的經營似乎不如 iOS App Store 用心。對比一下 [Linux 系統套件管理工具](/posts/linux/apt-yum-apk/)例如： Ubuntu, Debian : ```apt-get```；CentOS : ```yum``` 等等，Mac 在軟體真的不太友善。故隨後就推出 Homebrew 補足了不足 ，可對套件做統一控管，以及利用指令做到批量管理和安裝。 

# [Homebrew 名詞簡介](https://github.com/Homebrew/brew/blob/master/docs/Formula-Cookbook.md#homebrew-terminology)
homebrew 又大概把安裝的套件分為兩類：
- 以 Command Line 方式操作，我們就單純稱為 Homebrew 
- 對於一些圖形化介面的應用程式套件，我們就稱為 Homebrew-cask

{{< alert warning >}}
以上只是大概分類的概念，像是 gcp 雲端的 gcloud，明明感覺是 CLI 工具，但還是歸類在 Homebrew-cask 內。
{{< /alert >}}

Homebrew 比喻起來就是在 Home 「家裡」 Brew「釀酒」，所以一個套件就相當於是一個酒的配方，故 Homebrew 將套件稱為 formula ，而實際上 formula 是指從 source code 中構建的 Homebrew package 的定義，且 formula 都是 Ruby script。

當釀好酒後，需要 Keg「桶子」 來存放，實際上桶子就是標示了 formula 版本訊息 ; 而許多桶子就要有一個 rack「架子」來放各種 Keg「桶子」；再來 Cellar「地窖」內有許多的 racks「架子」。

最後要將 Keg 中的酒倒出來，會用到 Tap：
- Tap 當名詞時是指酒閥，是指由 formulae 所形成的 Git 版本庫 
- Tap 當動詞時是指在本機 clone 這個 Git 版本庫


# 操作指令
### 管理 Homebrew 本身的 CLI
```
# 查看 homebrew 安裝路徑
brew --prefix

# 查看brew配置
brew config

# 清除 homebrew 在安裝套件過程遺留的暫存檔
brew cleanup

# 更新 Homebrew 本身，以及已經安裝的所有套件的資訊，會列出有異動的 formula
brew update

```

### 管理套件 CLI
```
brew search <formula>      
brew install <formula>     
brew uninstall <formula>

# 查詢指定套件的資訊，如果是已經安裝的，會連同安裝路徑與檔案大小等訊息也出現
brew info <formula>

# 列出指定軟體的依賴關係
brew deps <formula> --tree

# 列出過時，需要更新的套件 ;
brew outdated
# 從 brew info 可以看到有些套件會自行管理升級(auto-update)
# 一般來說 brew 是不會自己去 track 套件的升級資訊，故可能版本會比較舊
brew outdated --greedy 

# 更新套件到最新版本，如果有依賴關係也會一併升級
brew upgrade
brew upgrade <formula>

```

# Homebrew 安装指定版本

### 官方多版本 formula 安裝
例如說使用 ```brew search node```，可以看到 ```node@18``` 和 ```node@20``` 等等，這裡也附上在安裝 nodejs 時，可以選擇版本的方式:
```
brew install node
node -v # newest version ex:v21.7.3
brew install node@20
brew unlink node
brew link --overwrite node@20
node -v # v20.12.2 LTS
```

### Formula Git 歷史版本安裝
brew 也可以安裝第三方的庫，除了自己的來源庫，還允許別人的來源庫添加進來，適用於安裝不在 homebrew 的第三方套件，會增加 homebrew 的 formula
```
brew tap <tap-name> 
brew untap <tap-name>        

# 查看 tap 詳細資訊
brew tap-info <tap-name>

# 列出所有 tap-repositories
brew tap                      
```
以 hugo 為例子，可以先上網找一下其 [ruby-script](https://github.com/Homebrew/homebrew-core/blob/e84fce57d1e9a2b44bfba8ee5a289ce4665f827f/Formula/h/hugo.rb) 訊息

```
# Tapping homebrew/core is no longer typically necessary. 
# Add --force if you are sure you need it for contributing to Homebrew.
brew tap homebrew/core --force

# 查看 homebrew/core 的路徑訊息
brew tap-info homebrew/core

# 切換到 homebrew/core 的倉庫路徑下。找到需要 Checkout 至的 Commit 版本。
# 怎麼找 Commit 呢？透過搜尋 Git log
git log -p -- Formula/hugo.rb | grep -e ^commit -e 'url "http'

# git checkout 到要的 commit，然後設定 HOMEBREW_NO_AUTO_UPDATE=1 
# 此環境變數是為了關閉 brew update 被執行，可使用 brew config 來查是否被成功設置
HOMEBREW_NO_AUTO_UPDATE=1 brew install hugo

```


---

### 參考資料

- [Homebrew Office](https://brew.sh/)

- [[Mac] Homebrew 與 Homebrew-Cask —— 更快速、簡潔、優雅地管理你的 Mac 軟體套件](https://www.onejar99.com/mac-homebrew-homebrew-cask-mac/)

- [Homebrew 安装指定版本的三种方式](https://shockerli.net/post/homebrew-install-formula-specific-version/)

- [Homebrew 簡介](https://mt116.blogspot.com/2017/11/homebrew.html)

- [macOS 软件推荐之 mac 开箱之后最先安装的 3 款软件（第0篇--Homebrew）](https://www.youtube.com/watch?v=oB6XIVEU_OE&t=1245s)

- [Homebrew (2) - Homebrew 常用與隱藏指令](https://note.koko.guru/posts/homebrew-useful-command)

- [Homebrew 筆記](https://pjchender.dev/app/homebrew/)