---
title: "GitHub Flow"

author: Aryido

date: 2025-05-01T12:42:08+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:

comment: false

reward: false
---

<!--BODY-->

> 雖然使用了 Git 作為版本管理工具，但每個人對於分支的認知可能不同，故造成每次 commit 到不同分支之後，要**合併**要回哪個分支可能會有歧義，這時就可參考一些已存在的 Workflow 規範，只要團隊遵守這樣的 branch 的 commit 和 merge 規則，就可以有一致性。每個 Workflow 規範都不太一樣，常見有: Git Flow、**GitHub Flow**、GitLab flow ，主要都是希望就算 Project 越來越大協作人員越來越多，也能有效管理 Git Branch，那這邊會以 **GitHub Flow** 為主要說明，但也會筆記分析了解其他不同策略的優缺點。

<!--more-->

---

提 GitHub Flow 之前先來提 Git Flow 吧！

{{< image classes="fancybox fig-100" src="/images/cicd/workflow/gitflow.jpg" >}}

- **master (main)** : 代表穩定版，主要會在這裡打上 tag 而每個 tag 都要可以執行的
- **develop** : 從 master 出來，是所有 feature 的基礎分支，而新增 feature 分支開發完後再合併回 develop ; 當多個功能都完成後， develop 最後會合併回 master
- **feature** : 開始新增功能的時候，從 develop 分支出來，完成之後會再併回 develop
- **hotfix** : 當 master 有 Bug 時，會緊急產生 hotfix 修復，修完後再合併回 master 、 develop
- **release** : 當認為 develop 功能完成到一個進度時，就可以合併到 release 做上線前的最後測試。測試完成後，release 會同時合併到 master 、 develop

上面介紹完每個分支的說明了，但是目前看起來蠻多文章都有提到，其實不推薦 gitflow 了，[主要原因是 **太強調 develop 導致導致後續許多繁瑣的多餘步驟**](https://blog.hellojcc.tw/the-flaw-of-git-flow/)，看完之章後列出原因如下：

- 既然 develop 是從 master 分出來的，最後也會合併 master ，那為什麼 hotfix / release 還要多作合併 develop 的動作 ？
- develop 是從 master ，完成功能後合併至 master ，然後又從 master 拉出來，這很多餘

因此比較推薦 **GitHub Flow**，它原則單純，只強調一條 master 其餘的 branch 都算是 feature，只要求 branch 命名要有敘述性。現在很多開源專案都是採取 **GitHub Flow** 的規範 :

# Github Flow

{{< image classes="fancybox fig-100" src="/images/cicd/workflow/githubflow.jpg" >}}

Github Flow 就直接進入實操環節吧。有一種非常常見的情況是在我們修改 code 的時候，`main branch` 又更新了。 比如說自己有個 `my-feature` branch 且自己有上了一個新的 commit 叫 `f-commit`。 這時 main branch 更新了又多了 `update` 這個 commit

{{< image classes="fancybox fig-100" src="/images/cicd/workflow/githubflow-exercise.jpg" >}}

那根據 Github Flow 步驟應該是要 :

```bash
# 切換至主分支
git checkout main

# 更新並和遠端同步
git pull origin main

# 切換回自己的功能分支
git checkout my-feature

# 將主分支的最新變更整合到功能分支
# 最重要步驟，要注意是使用 rebase 不是直接 merge main
git rebase main
# 意思是： 先把我的修改先都放到到一邊，然後把 main 最新的修改拿過來
#         接著在最新修改的基礎之上，再把我的 commit 給嘗試弄回去


# 那由於做了 reabase 動作，所以原本 commit 的 SHA 改變
# 故需要 force push 至遠端倉庫，蓋掉自己  my-feature 原有的 commit ;
git push -f origin my-feature

```

{{< image classes="fancybox fig-100" src="/images/cicd/workflow/githubflow-exercise2.jpg" >}}

上述部分也可以簡化成：

```bash
# 在自己開發的 branch 上
git checkout my-feature

git pull --rebase origin main

git push -f origin my-feature

```

- `git pull`：從遠端倉庫 origin 拉取 main 最新的提交，並會整合至 local main 分支內

- `--rebase`：整合至 local main 分支內的策略，是指定使用 rebase 而非 merge 的方式進行合併，會進行的操作如下 :
  - 暫存本地修改: 將 my-feature 分支上，**自從與遠端分支分岔以來的 commit** ，都暫時保存起來
  - 更新本地分支: 將 my-feature 分支起點，重置為與最新遠端 master 分支相同的狀態
  - apply 本地修改： 將先前暫存的 commit ，逐一應用到更新後 my-feature 分支上。

### Conflict

那上面的 rebase 是順利的過程，實際上我們也會遇到有衝突的情況 :

- ###### 執行 git rebase 的過程中，發生了 conflicts，而且「覺得這次 rebase 做錯了、不想繼續」

  ```bash
  git rebase --abort
  # 回到 rebase 前的 commit 狀態，簡單來說就是撤銷 rebase
  ```

  {{< alert warning >}}
  `--abort` 只能在 rebase 進行中 才能使用，如果 rebase 已完成或未開始，執行 `--abort` 會出現錯誤訊息: `fatal: no rebase in progress`
  {{< /alert >}}

- ###### 執行 git rebase 的過程中，發生了 conflicts，「需要去選擇到底要哪一段 code」

  ```bash
  # 想要快速選擇「自己分支」或「對方分支」的內容，可以使用：
  git checkout --ours <file> # 目前分支上的內容
  git checkout --theirs <file> # 要 rebase 或 merge 的來源分支內容
  ```
  上面是可以快速選擇，但其實更常發生需要自己慢慢看並且手動找出需要的 code 的情況，都選好之後：

  ```bash

  # mark 解決衝突的 file，就是加到 staging
  git add <file>

  # 因為是在 rebase 過程中，不需要 commit
  # 直接使用 rebase continue 來合併衝突
  git rebase --continue
  ```

都做完之後就可以發 pull-request/merge-request 了，當 code 都審查 review 完畢之後，蠻多時候會選擇使用 Squash 來 Merge 的 。

{{< alert info >}}
開發完成後會發送一個合併請求，在不同平台名稱是不太一樣的：

- github: **Pull Request**

- gitlab: **Merge Request**

{{< /alert >}}


### Squash

{{< image classes="fancybox fig-100" src="/images/cicd/squash-merge.jpg" >}}

squash and merge 意思是把新分支上面的所有 commit 合併成一個 commit，然後這個 commit 放到 main branch，會這樣做的原因是

- 讓 main branch commit history 儘可能的簡潔
- 讓 main branch 裡面每一個 commit 都是正常工作的

### 最後環境整理

merge 之後一般情況下我們就會把合併的遠端 branch 直接刪掉，但因為自己 local git 還有該 branch，這個時候要

- 切換到 main branch 上
- 然後使用 `git branch -D` 來把 branch 從 Local Git 裡面也刪掉
- 最後再使用一次 `git pull origin main` 把最新的遠端更新給拉到我的 Local

經過了這些操作之後 Local Git 就又和 Remote Git 一模一樣了

---

# Gitlab Flow

GitHub Flow 很簡單，但如果遇到需要拆分多個區域，分作開發、驗證、測試等區域時，按照原本的 Github Flow 無法滿足，因此 GitLab 這家公司，提出了所謂的 GitLab Flow，想多改善 Github flow 的弱點，但依舊保持簡單的優勢

{{< image classes="fancybox fig-100" src="/images/cicd/workflow/gitlabflow.jpg" >}}

如上圖 GitLab Flow 應付不同的開發流程分成 :

- 開發環境為 master
- 生產環境為 production
- 預發分支 pre-production

規範也是按照 Github Flow 的做法，都是**由 feature branch 合併至 master branch**。

當要正式 release 錢，才根據需要 deploy 的環境，例如先要做**預發布測試**，則從 master 合併至 pre-production，同時在 pre-production 搭配 GitLab CI 功能觸發 CI/CD 。

---

# Version

| 名稱                      | 穩定度 | 適用對象     | 說明                       |
| ------------------------- | ------ | ------------ | -------------------------- |
| `nightly`                 | 最低   | 開發者       | 每日自動建構，未經測試     |
| `alpha`                   | 低     | 開發內部測試 | 功能不完整，可能不穩定     |
| `beta`                    | 中     | 外部測試者   | 功能完整，但仍在測試中     |
| `rc`（Release Candidate） | 高     | 接近正式版本 | 若無重大 bug，將轉為正式版 |
| `release`                 | 穩定   | 一般使用者   | 正式上線的穩定版本         |

### 參考資料

- [十分钟学会正确的 github 工作流，和开源作者们使用同一套流程](https://www.youtube.com/watch?v=uj8hjLyEBmU&t=292s)

- [Git 上的三種工作流程](https://medium.com/i-think-so-i-live/git%E4%B8%8A%E7%9A%84%E4%B8%89%E7%A8%AE%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B-10f4f915167e)

- [git flow 實戰經驗談 part1 - 別再讓 gitflow 拖累團隊的開發速度](https://blog.hellojcc.tw/the-flaw-of-git-flow/)

- [完整介紹 Git 分支策略 feat. Git Flow, GitHub Flow, GitLab Flow, TBD](https://www.webdong.dev/zh-tw/post/trunk-based-development/)

- [Git 版本控制系統 - GitHub Flow 工作流程與實際演練](https://awdr74100.github.io/2020-05-11-git-githubflow/)

- [使用 git rebase 避免無謂的 merge](https://ihower.tw/blog/archives/3843)

- [Git merge & rebase 区别和用法](https://www.youtube.com/watch?v=rYQ8uwwOb3M&t=2s)

- [Day 28 - GIT 團隊協作 談 流程管理 02 GitHub Flow](https://ithelp.ithome.com.tw/articles/10228090)
