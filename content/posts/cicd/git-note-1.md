---
title: "Git - 名詞解釋和功能比較"

author: Aryido

date: 2025-04-30T13:53:19+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->

> 一般來說 Git 的操作會涉及到幾個**區域** ：
>
> - **硬碟區 disk** : 檔案存放的一般資料夾，也會被稱為 **workspace**
> - **暫存區 staging** : 保存 `git add` 紀錄的地方，也被稱為 **index**
> - **本地端 git local** : 保存 `git commit` 紀錄的地方，也被稱為 **repository**
> - **遠端 git remote** : `git push` 的倉儲，如 **Github**、**Gitlab** 等等
>
> 而各個之間狀態變化的簡單關係如下圖所示 :
> {{< image classes="fancybox fig-100" src="/images/cicd/git-control-flow.jpg" >}}
> 以上就是最基本的使用，在開始筆記之前先把一些名詞定義好，

<!--more-->

---

# revert

`git revert` 和 `git reset` 都是 Git 撤消 commit 的一種形式，但是 revert 和 reset 的不同點是 `git revert` 是會**增加一個反操作的 commit** :

{{< image classes="fancybox fig-100" src="/images/cicd/revert.jpg" >}}

那麼和 `reset` 這個相對直觀的命令相比 `revert` 有什麼好處呢 ?

> `revert` 可以撤銷中間任意一個 commit

比如下圖有一個 `Change` commit 後又有一個 `Change2` commit ， 假設這個 `Change` commit 的 id 是 `70a2..`:
{{< image classes="fancybox fig-100" src="/images/cicd/revert-2.jpg" >}}
若用 `git revert 70a2` 這樣最終得到的結果，就相當於只做了一個 `Change2` ，而這個效果是我們用 `reset` 沒有辦法輕鬆地達到的。

雖然用 Git 來做版本管理時，偶爾會需要**撤銷**某些操作，但對於已經推上遠端 Github / Gitlab repository 的程式碼，並且該分支已經有多人協作的時候，**撤銷**操作就不能隨便做了！對於這些「已經 push 到了遠端的 commit」，其 **commit 是只能往前走，不可以刪除和更改的**，故不可以進行 `git reset` 只能使用 `git revert`

{{< alert danger >}}
如果我們修改的分支是除了自己之外沒有任何其他人用，就可以用 `git reset` 把某些歷史 commit 直接砍掉，但注意這個時候想同步到遠端的話必須使用 `git push -f` 強制推送更改，因為遠端的 歷史 commit 可能和自己本地端不一樣。

對於「公有分支」絕對不應該使用 `git push -f` ; 但是如果這是「個人分支」的話其實是可以使用 `git push -f`。

{{< /alert >}}

# 「直接 Merge」 VS 「Rebase 後才 Merge」 的差異

假設情境（合併前）

```
      D---E  master
     /
A---B---C---F  origin/master

```

### 直接使用 merge

```
      D--------E
     /          \
A---B---C---F----G   master, origin/master

# G 為 merge commit，記錄了 D–E 和 F 的合併狀態。

```

### 使用 rebase 後才 merge

```
A---B---C---F---D^---E^   master, origin/master

# D^ 和 E^ 內容其實和 D 和 E 是一樣的，只是 commit SHA 不同。
```

為什麼 commit SHA 會改變呢 ？ 因為 git commit 生成的 SHA 會依賴前一個 commit ，由於 rebase 導致依賴的前一個 commit 和原本的不一樣，所以 commit SHA 才會改變。

{{< image classes="fancybox fig-100" src="/images/cicd/rebase-merge.jpg" >}}

### 表格整理

| 項目             | `merge`                                 | `rebase 後才 merge`                               |
| ---------------- | --------------------------------------- | -------------------------------------- |
| **歷史紀錄**     | 保留完整分支脈絡，會多一個 merge commit | 重寫歷史，變成線性結構                 |
| **Commit SHA**   | 不變                                    | 會改變（如 D → D'）                    |
| **衝突處理次數** | 只需處理一次                            | 每個 commit 都可能衝突，要多次處理     |
| **操作難度**     | 相對簡單                                | 需要每步都手動確認，再來 `git rebase --continue`             |
| **推薦使用時機** | 大範圍修改且預期有大 conflict           | 想保持線性紀錄                         |

{{< alert warning >}}
特別注意，是在新開的 branch 上使用 rebase ，而不是在 main/master 上使用 rebase，因為 rebase 其實算是一個「危險」的操作，會改寫 commit 的歷史
{{< /alert >}}

# origin 是什麼

在 Git 中，`origin` 是預設的遠端名稱，代表 clone 時的來源，有些 cli 會加上 origin 是為了明確指定要從哪個 remote 抓取資料，因為可能有多個遠端如 origin、upstream 等等 :

```bash
# 用來查看目前專案所設定的 remote repository
git remote -v

# Terminal:
# origin  git@github.com:Aryido/aryido.github.io.git (fetch)
# origin  git@github.com:Aryido/aryido.github.io.git (push)

# 目前只有一個 origin 遠端
```

那什麼情況下會有多個遠端倉庫呢？

- ##### **同步多個平台的倉庫，例如 GitHub 和 GitLab**
- ##### **Fork 開源專案：origin 與 upstream**

當從 GitHub 上 fork 一個開源專案時，預設的遠端名稱為 origin 會指向自己的 fork。但為了同步原始專案的更新，可以新增一個名為 upstream 的遠端，指向原始專案的倉庫。這樣可以方便地從原始專案拉取更新，然後是把更改推送到自己的 fork。

# fetch & pull 差異

- ### fetch

```bash
# 從預設 remote（origin）抓取所有分支更新
git fetch
git fetch origin

# 從指定的 remote（如 origin）抓下特定 <branch> 更新
git fetch origin <branch>

# 對於所有有設定的 remote ，抓取所有分支的更新，例如同時抓 origin 和 upstream 的所有 branch 更新
git fetch --all
```

**特別注意 `git fetch` 只是下載所需的資料，不會合併和更新任何的檔案**

- ### pull

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

# origin master 還是 origin/master ? 何時使用斜線 /

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

# reflog
reflog 是縮寫，全稱是 reference logs
- `git log`: commits 的紀錄
- `git reflog`: 使用者在 git 上的操作紀錄

使用場景： 例如用了 git reset 語法，將版本還原到太前面的版本，用 git log 上看不到那些比較新的 commit ，此時想回復到原來狀態該怎麼辦？此時可以用 `git reflog` 指令，它會詳細顯示你每個指令的 SHA-1 值。


---

### 參考資料

- [十分钟学会常用 git 撤销操作，全面掌握 git 的时光机](https://www.youtube.com/watch?v=ol7CMoJuAvI)

- [git 撤销修改](https://www.cnblogs.com/tomorrow0/p/13876637.html)

- [Git 課程學習筆記-ep4](https://medium.com/jordanttcdesign/git-%E8%AA%B2%E7%A8%8B%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-ep4-790f010a7fa3)

- [git-revert 你真的会用吗？](https://www.youtube.com/watch?v=KHkTF3MlGG0)

- [Git reset三种常用模式区别和用法](https://www.youtube.com/watch?v=zjTd1Wzu5lc)
