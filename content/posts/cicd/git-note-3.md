---
title: "Git - 簡單周邊工具、實用網站介紹"

author: Aryido

date: 2025-05-24T07:04:47+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->
> 在軟體開發的日常中，Git 可以說已經是比肩「電腦」一樣基礎的存在了！ 因此相應其周邊生態也非常豐富，有些擴展工具或實用網站蠻直得注意一下的，這邊文章就簡單記錄一下：

<!--more-->

---

# .gitignore 範本
專案內總是會有一些不需要進到 Git 的檔案，故每次開發一個新專案，都會需要 review `.gitignore` 檔案，這裏介紹一個網頁，可以讓很便利的快速產生專案所需的 `.gitignore` 檔，就是 [gitignore.io](https://www.toptal.com/developers/gitignore)，它有收集各種不同程式語言、開發框架、開發工具所需的 `.gitignore` 檔案範本。

###  gitignore 設定規則
規則非常多，只記錄一些：
- **忽略所有 B 資料夾中附檔名是 .txt 的檔案**: `B/*.txt`
- **以驚嘆號(！)來例外某些檔案**: `!important.log`，代表不忽略 important.log
- **忽略所有以某個字樣開頭的檔案與目錄**: `image*`，忽略所有以 image 開頭命名的檔案與目錄


{{< alert warning >}}
`.gitignore` 中可以寫註解，不過如果要寫註解，「不要」寫在跟檔名同一行，
否則 Git 可能會把你想寫的註解視為檔名，導致忽略失效
{{< /alert >}}

- **忽略目錄底下的 「所有內容」**： `dist/ `、 `.vscode/ ` 
- **只忽略「根目錄」的目錄或檔案**： `/Test`，代表只會忽略根目錄 `/Test`內的檔案，不忽略例如 `Vue/Test`，因為這個 Test 隸屬於 Vue 子目錄下

### 設定全域 .gitignore
可以直接設定 global 的 `.gitignore`，這樣就不需要每個專案都要重新設定 `.gitignore`。這可以藉由調整 git config 來達成 :
```bash
git config --global core.excludesfile "~/.gitignore_global"
```
或是直接在 `.gitconfig` 檔內的 [core] 底下加上：
```ini
[core]
	excludesfile = ~/.gitignore_global 
```

### 如何忽略已經被版控的檔案？
**已經被版控的檔案會在會導致 .gitignore 無法忽略**，要如何再次忽略呢，操作如下：
```bash
# 首先要把要忽略的檔案夾到 .gitignore 內

# 補充一下，單純 rm 會刪掉檔案
# 但加上 --cached 是告訴 git 不要把檔案刪除，只是刪除「索引」
git rm --cached 檔名 

# 將「移除索引」的狀態 Commit 儲存起來
git commit -m "<Message>"
```

{{< alert warning >}}
`git rm --cached` 移出檔案，並不會影響之前的 commit 
{{< /alert >}}


---

# Git Submodule
在一個稱主專案(super project)的 Git repository中，再嵌入另一個 Git repository，這個被嵌入的舊稱為子模組 (submodule)，且兩者的提交紀錄完全獨立。就以目前我的筆記網站，就有使用到 Submodule，因為 hugo-themes 和文章是分成不同的 Git Repository。

```bash
git submodule status
# 如果「有submodule」則會列出所有子模組的 Commit ID、路徑和當前分支 ; 
# 如果「沒有submodule」就什麼輸出空白
```
另外submodule 資訊一定會記錄在專案根目錄下的 `.gitmodules` 中，如果有看到了 `.gitmodules`，就代表這個專案擁有子模組，可以 `cat` 查看詳細的配置內容，當然如果有多個子模組時，這個檔案會有多個。

```bash
# 想把一個現有的 repo 引進來成為 submodule
git submodule add <遠端倉庫URL> <放置的目錄路徑>

# Clone 一個含有子模組的專案
git clone <主專案URL>
cd <主專案目錄>
git submodule init    
git submodule update
# 或者
git clone <主專案URL>
git submodule update --init --recursive
# 一步到位，最推薦
git clone --recurse <主專案URL>


# 當遠端的 submodule有新版本
git submodule update --remote --merge


# 移除 submodule
git submodule deinit -f <folder>
# 從 Git 索引移除子模組目錄
git rm -rf <folder>
# 提交變更
git commit -m "xxx"
# 清理 .git 目錄中的殘餘檔案（可選，但建議執行）
rm -rf .git/modules/<folder>
```

{{< alert warning >}}
主專案的 git ，不會追蹤 submodule 變更的詳細內容
{{< /alert >}}


### 設定全域 Git Config submodule
當有其他新的人要 git clone 有 submodule 主專案後，他們必須記得執行 `git submodule update --init --recursive`，否則專案會因為 submodule 而造成啟動失敗，這是蠻常見的問題。

為了避免每次 pull 主專案後都要手動更新 submodule，其實可以設定 global 甚至 system config：
```bash
git config --global submodule.recurse true
```

---

# Git Large File Storage (LFS)
Git 並不是為了儲存大型檔案而設計的，原因是其儲存設計是快照而不是差異比對，故試想一張 1MB 的圖片只要改動了 10 次，那 Git 概念上是會儲存 10 個快照到儲存庫當中，其實蠻浪費儲存空間的。 但大型檔案現在儲存在 git 內的需求是存在的，如圖片、影片、音樂、和現在最流行的大型語言模型的 model，
所以就有 Git 的擴充工具：  **Git Large File Storage** 

{{< alert danger >}}
GitHub 限制單一檔案 100 MB 的限制
```bash
remote: warning: Large files detected.
remote: error: File large_file is 123.00 MB; this exceeds GitHub's file size limit of 100 MB
```
目前免費的流量是 1GB / month 
{{< /alert >}}

Git Large File Storage (LFS)的運作方式是大檔案，以「文字指標」的形式儲存在 Git 中；而檔案的實際內容則儲存在「遠端伺服器」上。

{{< alert success >}}
因為 Git LFS 會將大型檔案儲存在額外的伺服器上，所以在 CI/CD 流程上會需要額外的設定來下載這些 LFS 檔案，可以參考
- [download](https://github.com/riceball-tw/web-dong-blog/blob/b2e8eb9de34653370707baab1e15933caa816029/.github/workflows/pages-deployment.yml#L50)
- [cache](https://github.com/riceball-tw/web-dong-blog/blob/b2e8eb9de34653370707baab1e15933caa816029/.github/workflows/pages-deployment.yml#L41)
{{< /alert >}}


``` bash
# 安裝
brew install git-lfs
git lfs install

# 列出當前被 LFS 管理的檔案
git lfs track # 目前所有 tracked 的大型檔案
git lfs ls-files # 查看狀態

# 選擇希望 Git LFS 管理的檔案
# 或者是編輯 `.gitattributes`，此檔要被提交到 Git 儲存庫中
git lfs track "*.jpg" 

# 刪除被 LFS 管理的檔案
git lfs untrack <file>

```

{{< alert info >}}
如果是使用 GitHub 來託管，那麼只需像平常一樣 commit 和 push 檔案即可，GitHub 會替你處理好後續的事情，在 GitHub 上預覽 LFS 檔案時也會有標註
{{< /alert >}}


若有已經 Commit 的歷史紀錄中已經有大型文存在，且在 main branch 了，可能會需要強制刪除之前的 Commit，會使用到 `--force` 會需要團隊討論，可以[參考](https://hackmd.io/@ppp300a/git-github/%2FZ2lh6kccQJqS9Zsp2Vcueg)。


---

### 參考資料

- [如何設定 git ignore 命令並自動下載所需的 .gitignore 範本](https://blog.miniasp.com/post/2020/05/24/Setup-git-ignore-alias-to-download-gitignore-templates)


- [實用小工具 - gitignore.io 介紹](https://www.gss.com.tw/blog/%E5%AF%A6%E7%94%A8%E5%B0%8F%E5%B7%A5%E5%85%B7-gitignore-io-%E4%BB%8B%E7%B4%B9)

- [一起來學 Git 吧！(12) - 設定 .gitignore](https://ithelp.ithome.com.tw/articles/10329189)

- [專案中的專案：git submodule 的協作魔法與陷阱](https://vocus.cc/article/68afe88dfd8978000117efec)

- [submodule](https://blog.csdn.net/toopoo/article/details/104225592)

- [Git Large File Storage (LFS)](https://git-lfs.com/)

- [如何使用 Git LFS](https://www.webdong.dev/zh-tw/post/how-i-use-git-lfs-to-manage-large-git-files/)