---
title: "Git 操作筆記"

author: Aryido

date: 2025-04-30T13:53:19+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:

comment: false

reward: false
---

<!--BODY-->

> 用 Git 來做版本管理時，偶爾會需要撤銷某些操作，但對於已經推上遠端 Github 或 Gitlab 且該分支已經有多人協作的時候，其實有蠻多需要注意的事情 ; 再來是 git 其實是有蠻多功能和一些命令的，有時不常用的話也會忘記或者錯估使用場景。那開始筆記之前先把一些名詞定義好，一般來說 Git 的操作會涉及到幾個 **區域** ：
>
> - **硬碟區 disk** : 檔案存放的一般資料夾，也被稱為 **workspace**
> - **暫存區 staging** : 保存 `git add` 紀錄的地方，也被稱為 **index**
> - **本地端 local** : 保存 `git commit` 紀錄的地方，也被稱為 **repository**
> - **遠端 remote** : 保存 `git push` 的倉儲，如 gitjub、gitlab 等等
>
> 而各個之間狀態變化的簡單關係如下圖所示 :
> {{< image classes="fancybox fig-100" src="/images/cicd/git-control-flow.jpg" >}}

<!--more-->

---

# 比較整理

## 另外一種形式的撤消 commit -  git revert

`git revert` 和 `git reset` 的不同點是 `git revert` 是會**增加一個反操作的 commit** :

{{< image classes="fancybox fig-100" src="/images/cicd/revert.jpg" >}}

那麼和 `reset` 這個相對直觀的命令相比 `revert` 有什麼好處呢 ?

> `revert` 可以撤銷中間任意一個 commit

比如下圖有一個 `Change` commit 後又有一個 `Change2` commit ， 假設這個 `Change` commit 的 id 是 `70a2..`:
{{< image classes="fancybox fig-100" src="/images/cicd/revert-2.jpg" >}}
若用 `git revert 70a2` 這樣最終得到的結果，就相當於只做了一個 `Change2` ，而這個效果是我們用 `reset` 沒有辦法輕鬆地達到的。

還有一個更重要的好處，是我們 push 到了遠端之後，由於**對於公有分支 commit 是只能往前走，不可以刪除和更改的**，故不可以進行 `git reset` 只能使用 `git revert`

{{< alert danger >}}
如果我們修改的分支是除了自己之外沒有任何其他人用，就可以用 `git reset` 把某些歷史 commit 直接砍掉，但注意這個時候想同步到遠端的話必須使用 `git push -f` 強制推送更改，因為遠端的 歷史 commit 可能和自己本地端不一樣。

對於「公有分支」絕對不應該使用 `git push -f` ; 但是如果這是「個人分支」的話其實是可以使用 `git push -f`。雖然是這樣說，也是有公司希望就算是自己的 branch 也完全不要使用 `git push -f` 的，這個建議都先問一下比較好。

{{< /alert >}}

## 「直接 Merge」 VS 「Rebase 後才 Merge」

假設情境（合併前）

```
      D---E  master
     /
A---B---C---F  origin/master

```

#### 直接使用 merge

```
      D--------E
     /          \
A---B---C---F----G   master, origin/master

# G 為 merge commit，記錄了 D–E 和 F 的合併狀態。

```

#### 使用 rebase 後才 merge

```
A---B---C---F---D'---E'   master, origin/master

# D' 和 E' 是 D 和 E 的變更內容重新套用在 F 之後的結果，commit SHA 不同。
```

{{< alert warning >}}

注意都是在新的 branch 上使用 rebase ，而不是在 main/master 上使用 rebase

{{< /alert >}}


{{< image classes="fancybox fig-100" src="/images/cicd/rebase-merge.jpg" >}}

| 項目             | `merge`                                 | `rebase 後才 merge`                               |
| ---------------- | --------------------------------------- | -------------------------------------- |
| **歷史紀錄**     | 保留完整分支脈絡，會多一個 merge commit | 重寫歷史，變成線性結構                 |
| **Commit SHA**   | 不變                                    | 會改變（如 D → D'）                    |
| **衝突處理次數** | 只需處理一次                            | 每個 commit 都可能衝突，要多次處理     |
| **操作難度**     | 相對簡單                                | 需要每步都手動確認，再來 `git rebase --continue`             |
| **推薦使用時機** | 大範圍修改且預期有大 conflict           | 想保持線性紀錄                         |

## 偶爾出現的 origin 是什麼

在 Git 中，`origin` 是預設的遠端名稱，代表 clone 時的來源。加上 origin 是為了明確指定要從哪個 remote 抓取資料，因為可能有多個遠端如 origin、upstream 等 :

```bash
# 用來查看目前專案所設定的 remote repository
git remote -v

# Terminal:
# origin  git@github.com:Aryido/aryido.github.io.git (fetch)
# origin  git@github.com:Aryido/aryido.github.io.git (push)
```

那什麼情況下會有多個遠端倉庫呢？

- #### **同步多個平台的倉庫，例如 GitHub 和 GitLab**
- #### **Fork 開源專案：origin 與 upstream**

當從 GitHub 上 fork 一個開源專案時，預設的遠端名稱為 origin 會指向自己的 fork。但為了同步原始專案的更新，可以新增一個名為 upstream 的遠端，指向原始專案的倉庫。這樣可以方便地從原始專案拉取更新，然後是把更改推送到自己的 fork。

## fetch & pull 差異

- #### fetch

```bash
# 從預設 remote（origin）抓取所有分支更新
git fetch
git fetch origin

# 從指定的 remote（如 origin）抓下特定 <branch> 更新
git fetch origin <branch>

# 對於所有，有設定的 remote ，抓取所有分支的更新，例如同時抓 origin 和 upstream 的所有 branch 更新
git fetch --all
```

**特別注意 `git fetch` 只是下載所需的資料，不會合併和更新任何的檔案**

- #### pull

```bash
# 預設會根據 origin 更新本地的所有 branch
git fetch
# 預設會根據 origin 只更新本地的 master
git fetch origin master

# 但以上兩個做完，在 master 上的資料都還沒更新，只有把資料下載下來而已

# 現在才手動把更新合併進來
git merge origin/master

---

# 等同於上面兩步：fetch + merge
git pull

```

## origin master 還是 origin/master ? 何時使用斜線 /

當從 remote 抓取分支時，Git 會在本地建立對應的遠端追蹤分支，其命名格式為： `<remote>/<branch>`

例如： `origin/master` 表示遠端 origin 的 master 分支在本地的追蹤分支:

```bash
# 會使用斜線 / , 表示將遠端 origin 的 master 分支合併到當前本地分支
git merge origin/master

```

在某些命令中，你需要分別指定遠端名稱和分支名稱作為參數，此時不使用斜線 /：

```bash
# 表示從遠端 origin 抓取 master 分支的更新
git fetch origin master
```

## reflog
reflog 是縮寫，全稱是 reference logs
- `git log`: commits 的紀錄
- `git reflog`: 使用者在 git 上的操作紀錄

使用場景： 例如用了 git reset 語法，將版本還原到太前面的版本，用 git log 上看不到那些比較新的 commit ，此時想回復到原來狀態該怎麼辦？此時可以用 `git reflog` 指令，它會詳細顯示你每個指令的 SHA-1 值。

---

# 情境

## 假設我對 file 進行了一些修改，但我發現對 file 的修改是錯誤或不必要的，所以我想測銷掉對 file 的修改

```bash
# 沒有 `git add` 也沒有 `git commit`
git checkout < changed_file >
git restore < changed_file >
# 以上兩個操作等價，由於 checkout 還有切換 branch 的功能，新版本 git 想把這兩件事分開所以有了 restore，但我個人沒用過 restore，就簡單筆記一下。

# 使用了 `git add` 但還沒 `git commit`， 反悔了想取消 `git add`
git reset < changed_file >
git restore --staged < changed_file >
# 上面的操作兩個等價且安全，只會把文件從 staging 移出，會保留在 workspace 上的修改。

# 但如果就是想測銷掉且直接恢復到原始狀態，可以使用
git checkout HEAD < changed_file >
# HEAD 代表 git 內最近一次的 commit

```

---

在初始狀態下 disk, staging, local, remote 這四個是保持同步的，再來假設初始狀態下已經有一個 commit 叫做 `init` :

## 使用了 `git commit` 產生了一個叫 `change` 的 commit 了，但是想取消

```bash
# 以下兩個等價，reset 的預設就是 --mixed
git reset HEAD~1
git reset --mixed HEAD~1

```

- 由於產生了新的 commit ，故這裡 HEAD 代表 `change` 這個 commit
- 波浪線 `~` 代表「前面的」
- 數字和波浪線的配合，例如 : `~1` 代表往前一個 ; `~3` 代表往前三個
  
  {{< alert info >}}
- `HEAD^`：代表上一個 commit，等價於 HEAD~1
- `HEAD^^`：代表上上個提交 commit，等價於 HEAD~2

{{< /alert >}}

所以在本範例 `HEAD~1` 會表示 `init` 這個 commit ，為 `change` 這個 commit 的前一個 commit。故運行完上述命令之後 :

- workspace 的文件不會變，還是修改過後的樣子
- staging 的紀錄也沒了，如果要推上去還要重新再 `git add` 才可以
- `change` 這個在 git local 的 commit 沒了

{{< image classes="fancybox fig-100" src="/images/cicd/commit-reset-mixed.jpg" >}}

{{< alert warning >}}

- `git reset --soft HEAD~1`

  soft 模式也會保留 workspace 上的修改，只是不會把 staging 的紀錄也給拿掉

- `git reset --hard HEAD~1`

  如果就是想測銷掉所有修改包括 local, staging, workspace 可以使用 hard，但請謹慎

{{< /alert >}}


---

### 參考資料

- [十分钟学会常用 git 撤销操作，全面掌握 git 的时光机](https://www.youtube.com/watch?v=ol7CMoJuAvI)

- [git 撤销修改](https://www.cnblogs.com/tomorrow0/p/13876637.html)

- [Git 課程學習筆記-ep4](https://medium.com/jordanttcdesign/git-%E8%AA%B2%E7%A8%8B%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-ep4-790f010a7fa3)

- [git-revert 你真的会用吗？](https://www.youtube.com/watch?v=KHkTF3MlGG0)
