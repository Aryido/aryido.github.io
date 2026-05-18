---
title: "Git Worktree 筆記"

author: Aryido

date: 2025-05-22T10:11:23+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->

> 軟體開發有許多的已經定義的管理工作流程，例如說 Git Flow、Github Flow、GitLab Flow 等等，而這些共通點是都**以 Branch 來區分**不同階段的開發狀態 ; 在研發過程中，也經常需要在不同的 Branch 之間切換來處理不同的任務。這時 `git worktree` 就很大程度派上了用場，它可以讓**同一個 Git Repository 同時擁有多個工作目錄**，**且每個目錄綁定不同 Branch**，各個工作樹之間是互相獨立的。
>
> 這樣就不用在「 code 寫到一半 」時，遇到臨時需要切換到不同的分支作業，卻因為「 正在更動的檔案 」與「 目標分支上的檔案」有衝突而要另外處理，故 `git worktree` 很適合拿來處理緊急 hotfix 或者另外的 code-review。

<!--more-->

---

如果臨時要切換到其他分支去做事情，可能會遇到什麼問題呢？首先 Local 上可能有不少修改到一半的檔案，若切換的 branch 都**剛好沒有衝突的話，是可以直接切過去的**。但多數情況下，「 正在更動的檔案 」與「 目標分支上的檔案 」蠻高機率有衝突的，並且為了維持切換 brach 後不要有不相干的檔案，我們都會需要先做 :

- `git commit`
- `git stash -u` (加上 u 是為了也把未追蹤的檔案也一起暫存起來)

選擇上面兩操作中的其中一個後，才能順利切換分支。

{{< alert warning >}}
若有衝突的話，切換 branch 時 git 會跳出類似下列這樣的錯誤警告訊息：

```
error: Your local changes to the following files would be overwritten by checkout:
        <檔案名稱>
Please commit your changes or stash them before you switch branches.
Aborting
```

{{< /alert >}}

承上知道，大部分都需要多加上 `commit` 或是 `stash` 的步驟，才能切換分支，而且切回來後還需要:

- `git reset -soft <commit>`
- `git stash pop`

才能恢復原狀，其實有點麻煩，因為：

- 使用 commit 的話，之後回來要查看 `git log` 。看一下這個 WIP commit 的上一個 Commit Hash 值，才能知道怎麼 reset 回去

- 考慮到可能在其他地方也做了 `git stash` 所以要用 `git stash list` 來查是哪一個 stash 是上次離開前的狀態，然後記下 index ，下 `git stash apply {index}` 回到之前的狀態

一切麻煩根源，都是因為在**同一個工作目錄下的衝突修正**，在更以前不懂 git 的時候，曾經使用很醜的方式來解決：`git clone` 多個 repo，但會造成 :

- Disk 空間的浪費
- 若 Repo 很大，那麼每次 git clone 都要花很多時間，且個別都要 clone
- 也要每個 Repo 都另外同步

這樣的做法實在不太好...那 Best practice 解決方式是使用 `git worktree` ，可以在一個「本地儲存庫」(Local Repository) 下管理多個 Working Tree (工作目錄)，重點是 **Working Tree 是共用 .git 的** ，大幅節省了磁碟空間。

---

# 常用操作

Git V2.5 之後新增了 `git worktree` 常用使令如下:

```bash
# git 會 clone <source_branch> 至 <folder_path> 這個新的資料夾
# 並創建一個新 branch 會叫做 <new_branch_name>
git worktree add <folder_path> -b <new_branch_name>  <source_branch>

# 查看目前所有的工作樹
git worktree list

# 刪除
git worktree remove <folder_path>
# 刪除工作目錄中如果有一些尚未 Commit 的檔案，這個命令就會失敗
# 要強制刪除工作目錄，可以加上 -f 參數：
git worktree remove -f <folder_path>

# 若有手動刪除了工作目錄可以透過 prune 來清除 Git 無法管理的工作目錄
git worktree prune

# 工作目錄改名
git worktree move ../MyRepo-master ../MyRepo-master2

```

{{< alert info >}}
如果不手動進行 prune ， Git 大約每三個月會自動整理一次工作目錄
{{< /alert >}}

### 情境 : 假如在 「dev-1」 branch 開發到一半，突然要去修一個線上 error，但不想動到目前未完成的變更。

因為是線上環境，所以 source branch 會以 main branch 為主，而且約要創建一個新 branch 來做修正，故需要 加上 `-b` 參數：

```bash
# 在上層目錄建立一個 hotfix-xxxx-error 工作區資料夾
# 新增一個基於 main branch 的新 branch，這個 branch 叫 fix/xxxx-error
git worktree add -b fix/xxxx-error ../hotfix-xxxx-error main

```

### 情境 : 假如是 code-review ，想爲這個要 code review 的 branch 建立新的工作目錄，方便看 code

```bash
git worktree add <folder_path> <existing-branch>

# 也可以使用 commit hash
git worktree add <folder_path> <commit hash>

```

---

# 注意事項

建議習慣：

- 開新 worktree 前，先想好目錄資料夾命名（例如 `../hotfix/xxxx-xxx`、`../code-review/xxxx-xxxx`）
- 任務結束就 `git worktree remove`，避免殘留太多目錄
- 定期 `git worktree list` 檢查目前綁定狀態

使用 Git Worktree 最需要注意的就是：**不能同時有多個 workspace，使用同一個 Branch**，Git 會阻止這種操作，避免 index 與工作目錄狀態衝突。所以可以使用 Git detached HEAD ，例如:

```
git checkout origin/main
```

HEAD 通常會指向當前分支，而當前分支通常會指向該分支頂端的 commit (也就是該分支最新的 commit)，上面 checkout 的方式，會讓 HEAD 不是指向分支，而是指向某個 commit，而這個狀態的 HEAD 就被稱為「detached HEAD」，可以避免 Git worktree 綁到同一 branch。

### 參考資料

- [git-worktree 簡單介紹與使用](https://louis383.medium.com/git-worktree-%E7%B0%A1%E5%96%AE%E4%BB%8B%E7%B4%B9%E8%88%87%E4%BD%BF%E7%94%A8-876897c797bf)

- [使用 git worktree 管理一個本地儲存庫下的多個工作目錄副本](https://blog.miniasp.com/post/2023/10/29/git-worktree-manage-multiple-working-directories)

- [Git 官方文件：git worktree](https://git-scm.com/docs/git-worktree)

- [Git Worktree：有效管理多個分支和工作目錄的秘訣](https://notes.boshkuo.com/docs/DevTools/Git/git-worktree)

- [Git Worktree 是什麼？一個專案同時跑多個 AI Agent 的秘訣](https://israynotarray.com/git/20260121/2025687513/)

- [【Git】Git Worktree 讓你切換分支更方便](https://jimhuang.dev/git/worktree/)
