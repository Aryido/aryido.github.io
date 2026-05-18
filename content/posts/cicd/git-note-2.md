---
title: "Git - 情境範例和命令補充"

author: Aryido

date: 2025-05-01T17:53:19+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->

> 軟體工程師每天都會用 Git 進行版控，雖然是經常使用，但有一些差異可能還真的沒有好好想過，所以前面有文章補充了一些[名詞解釋和功能比較](https://aryido.github.io/posts/cicd/git-note-1/)。Git 使用會遇到這是各樣的情形，雖然大部分都千篇一律，但有一些實際面對到的情況，是蠻值得記錄下來的！ 所以這篇文章就筆記一下做一些 CLI 操作補充，另外也把一些踩坑的筆記統整。若以後有想到類似情境，就直接來參考這篇文章。

<!--more-->

---

# Branch 命令補充

不論是開發新功能、修復 Bug，還是整理合併功能，都離不開 `git branch`，自己最常使用 `git branch -a` 來同時列出 local 與 remote 的所有分支 （目前所在的分支前面會標註 \* 號並顯示綠色），除此之外還可以：

```bash
git branch # 只有 local 分支
git branch -r  # 只有 remote 分支
git branch -vv # 列出 local 分支的同時，顯示最後一次 Commit Message

# 修改 branch name，要使用 -m 這個參數
git branch -m <new-name> # 如果「正位於」該分支上
git branch -m <old-name> <new-name> # 如果「在其他分支」上

# 刪除本地分支，要使用 -d(D) 這個參數
git branch -d <branch-name> # 如果分支的尚未合併，Git 會拒絕刪除
git branch -D <branch-name> # 強制刪除
# 刪除本地分支並不會影響遠端分支，還要發送刪除命令到遠端
git push origin --delete <branch-name>

# 推送新分支並建立追蹤關係 (Upstream)
git push -u origin <branch-name>
git push --set-upstream origin <branch-name>

# 如果其他人推了一個新分支到 GitHub，但自己本地看不到：
# 1. 先同步遠端的最新分支清單
git fetch origin
# 2. 切換過去，Git 會自動在本地建立同名分支並建立追蹤，筆記一下 switch
git switch <branch-name>

# 清理失效的遠端追蹤分支 (Prune)
# 如果 remote 的分支已經被網頁端（PR 合併後）刪除了，但你本地執行 git branch -a 依然看得到 remotes/origin/xxx
git fetch --prune origin
```

---

# 情境

### 假設我對 file 進行了一些修改，但我發現對 file 的修改是錯誤或不必要的，所以我想測銷掉對 file 的修改

```bash
# 沒有 `git add` 也沒有 `git commit`
git checkout <changed_file>
git restore <changed_file>
# 以上兩個操作等價
# 由於 checkout 還有切換 branch 的功能，新版本 git 想把這兩件事分開所以有了 restore
# 但我個人沒用過 restore，就簡單筆記一下

# 使用了 `git add` 但還沒 `git commit`， 反悔了想取消 `git add`
git reset <changed_file>
git restore --staged < changed_file >
# 上面的操作兩個等價且安全，只會把文件從 staging 移出，會保留在 workspace 上的修改

# 如果就是想測銷掉所有改動，且直接恢復到原始狀態，可以使用
git checkout HEAD < changed_file >
# HEAD 代表 git 內最近一次的 commit

```

### 使用了 `git commit` 產生了一個叫 `change` 的 commit 了，但是想取消這個 commit

假設在初始狀態下 disk、staging、local、remote 這四個是保持同步的，再來假設初始狀態下已經有一個 commit 叫做 `init` ，然後又 commit 一個叫 `change` 的 commit，如果要取消 `change` 這個 commit 的話 :

```bash
# 以下兩個等價，reset 的預設就是 --mixed
git reset HEAD~1
git reset --mixed HEAD~1

```

為什麼上面做法就可以取消 `change` 這個 commit 呢？

- 依照上述情境，目前 HEAD 是在 `change` 這個 commit
- 波浪線 `~` 代表「往前」
- 數字和波浪線的配合的意思，例如 : `~1` 代表往前一個 ; `~3` 代表往前三個

  {{< alert info >}}

- `HEAD^`：代表上一個 commit，等價於 `HEAD~1`
- `HEAD^^`：代表上上個提交 commit，等價於 `HEAD~2`
- `HEAD` 和 `@` 是等價的，故上面也可以換成 `＠^`、`＠^^`、`@~2` 等等
  {{< /alert >}}

所以在本範例 `HEAD~1` 會表示 `init` 這個 commit ，為 `change` 這個 commit 的前一個 commit。故運行完上述命令之後 :

- workspace 的文件不會變，還是修改過後的樣子
- **staging 紀錄沒了**
  {{< alert success >}}
  workspace 還是修改過後的樣子但， staging 內沒有 add 的檔案，這就是 `--mixed` 的作用
  {{< /alert >}}

- 當然 `change` 這個在 git local 的 commit 沒了

{{< image classes="fancybox fig-100" src="/images/cicd/commit-reset-mixed.jpg" >}}

{{< alert warning >}}

- `git reset --soft HEAD~1`

  soft 模式也會保留 workspace 上的修改，且 staging 內還是有 add 的檔案。

- `git reset --hard HEAD~1`

  如果就是想測銷掉所有修改包括 local, staging, workspace 可以使用 hard，但請謹慎

{{< /alert >}}

### 整理自己的 commit

通常會是遇到下面狀況後，會選用 `git rebase` 整理 commit，讓別人看 code 可以輕鬆一些:

- 改寫 commit 的內容或訊息（edit / reword）
- 刪除錯誤的 commit（drop）
- 合併多個 commit 成一個（squash）

可以配合 `-i` 進入 interactive 模式，再來是選擇要更改的 commit 範圍，可以搭配 `~` 來表示範圍，例如說最近的 8 次 commits：

```bash
git rebase -i HEAD~8
```

接下來會進入 vim 編輯畫面，例如 :

```vim
pick 2a72a71 Add file1
pick 747f10f Add file2
pick 3b1ca36 Add file3
pick bb4c16c Add file4
pick 3ebea07 Add file5
```

按照**從上到下**順序，把這些 commit 串起來的。比較簡單的例如

- **變更 commit 的順序**，如附例子可以把 `pick 3b1ca36 Add file3` 移到最下面，儲存退出後順序就變成是 `1 -> 2 -> 4 -> 5 -> 3` 了

- **刪除 commit** 的話，就把前面的 「`pick` 指令改成 `drop`」

- **修改 commit message** 的話，把 「`pick` 改成 `reword`」，儲存離開之後，git 會執行套用，當執行到改成 reword commit 時會自動打開 vim 讓你改 message

  {{< alert warning >}}
  `git rebase -i` 編輯的這個畫面中，是這筆 commit 的原本 message，這段是給人類看的，例如把 `Add file1` 改成 `Add file new`，進行 rebase，不會套用改的訊息
  {{< /alert >}}

- **合併 commit** 的話，譬如說想把 `Add file3` 跟 `Add file4` 合併，那就把 Add file4 的「`pick` 改成 `squash`」，會把上一個 commit 合併一起

### 想把目前的分支狀態，復原成跟遠端分支一樣的狀態

有時候發覺自己目前的 branch 改壞了有太多問題，想直接從遠端的穩定版本「砍掉重練 ; 或者本地分支落後遠端太多，且完全不在乎本地目前的修改，想要快速同步遠端乾淨分支，這時就可以使用 :

```bash
git reset --hard origin/<BRANCH_NAME>
```

{{< alert warning >}}
當然要特別注意使用 `--hard` 代表**所有尚未儲存的修改（Uncommitted changes），都會被直接刪除**
{{< /alert >}}

### Git Branch 命名大小寫不一樣而導致的問題

先來說明情境，首先有創建了一個 branch 叫 `Feature/test`，這個 branch 也被推送制遠端倉庫了。這時候又創建一個 branch 叫 `feature/test2`，但是當使用 `git push --set-upstream origin feature/test2` 打算推到遠端倉庫時，卻發生：

> `fatal: feature/test2 cannot be resolved to branch`

{{< alert info >}}
首先有個先備知識點，Windows 系統和 Mac 系統是**不區分大小寫**的 ; Linux 系統則是**區分大小寫的**，然後本人使用的 MAC 電腦。 Git 在預設情況下不區分大小寫，**但可以透過修改 git config 來改為區分大小寫**

```bash
git config --get core.ignorecase
# true
git config core.ignorecase false
git config --get core.ignorecase
# false
```

但其實不推薦修改 git 大小寫預設的配置
{{< /alert >}}

接下來分析一下 root-cause :

我們知道 Git 儲存分支資訊是在 `.git/refs/heads/`， 而建立 `Feature/test` branch 時會建立一個名為 `Feature` 的大寫資料夾，裡面放一個叫 `test` 的 bob object ，接下來我們把這個 branch 推到遠端倉庫。

同樣地，在建立 `feature/test2` branch 時，照理來說要建立一個名為 `feature` 的小寫資料夾，但在 Git 在打算建立一個 feature 小寫資料夾時，對 MAC 作業系統來說，因為大小寫不敏感（Case-insensitivity），所以 MAC 作業系統會認為已經有存在資料夾了，故會直接使用 `Feature` 大寫資料夾，所以會直接在 `Feature` 大寫資料夾內，建立 `test2` 的 bob object。

其實這個時候本地的 ref 的狀態就有些問題了，明明 branch 是叫做 `feature/test2` ，但是 `.git/refs/heads/` 卻把引用放到 `Feature/test2`，這邊就埋下一個引爆點...，而推送 `feature/test2` branch 到遠端時就發生問題了，那 git 發生錯誤的地方，看起來是在[source code 這裏](https://github.com/git/git/blob/59ff4886a579f4bc91e976fe18590b9ae02c7a08/refs.c#L402)，主要就是 `strcmp(buf, rest)` 這邊的檢查：

> - buf 是 `git push --set-upstream origin feature/test2` 推上去的路徑，也就是 `feature/test2`
> - rest 是 refs 指向的 `Feature/test2`

兩個字串檢查後發現不一致，所以出現 fatal。

我的想法是偏向「**branch 名稱應該要統一都小寫**」，所以我的解決做法會是：

```bash
# `Feature/test` branch 改名成 `feature/test` branch
# 但在 macOS/Windows 上，如果只改大小寫， Git 會因為檔案系統不區分大小寫而報錯「分支已存在
# 要用兩步改名法：

git branch -m Feature/test Feature/test-back
git branch -m Feature/test-back feature/test

# 接下來去 refs/heads，會看到 Feature folder
# 以防萬一可以先備份，之後直接改名成 feature
mv Feature feature

# 之前 --set-upstream 會失敗的，現在會成功
git push --set-upstream origin feature/test2
```

{{< alert info >}}

大小寫問題有蠻多類似的坑，其實還蠻容易踩到的，例如說

- 有人 branch 命名 `Modify/xxx-xxx`，然後推送上遠端倉庫了，剛好你從來沒這樣命名過，所以本地 refs 都沒有 `Modify` 資料夾，然後你正常 pull 專案，默默地就拿到 `Modify` 這個 branch 了，所以 refs 默默產生 `Modify` 資料夾。直到某天自己命名了一個 branch 叫 `modify/yyy-yyy`，然後想推上遠端時，會發現怎麼都推不上去！？

- 不只是 branch 大小寫，**檔案大小寫不同**，也會踩到坑，可以參考下面兩個連結
  - [Git 大小寫不敏感導致提交文件衝突問題解決](https://chegva.com/6025.html)
  - [一下子敏感, 一下子不敏感, Git 你搞得我好亂啊](https://tn710617.github.io/zh-tw/gitSensitiveness/)

{{< /alert >}}

要避免以上問題，最好一開始組織或個人，一開始就做好**統一規範**來防止類似事情發生：

- 一個專案，如果參與的開發者們有使用不同的作業系統，那同一個資料夾內不要有同檔名，但大小寫不同的檔案 ; 另外 branch 也不要大小寫混用
- 如果是獨自開發，最好讓 Git config case-sensitiveness 的設定和作業系統是一致的

---

### 參考資料

- [十分钟学会常用 git 撤销操作，全面掌握 git 的时光机](https://www.youtube.com/watch?v=ol7CMoJuAvI)

- [送 PR 前，使用 Git rebase 來整理你的 commit 吧！](https://www.google.com/search?q=git+rebase+-i+HEAD~8&oq=git+rebase+-i+HEAD~8&gs_lcrp=EgRlZGdlKgYIABBFGDsyBggAEEUYOzIGCAEQRRhAMgYIAhBFGEDSAQc1NDlqMGoxqAIAsAIA&sourceid=chrome&ie=UTF-8)
