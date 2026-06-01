---
title: "Git Config - 多帳號管理"

author: Aryido

date: 2025-05-23T09:04:47+08:00

thumbnailImage: "/images/cicd/git-logo.jpg"

categories:
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->

> 安裝完 Git 之後，可以使用 `git config` CLI 取得 Git 設定組態參數，git config 是一個記錄了 git 操作的所有基本檔案資料。
> 比方說 :
>
> - git init 創建時預設的 branch 名稱
> - 寫 git commit 的顯示模板
> - 當 push github 時的使用者資料
>
> 等等設定都可以在 git config 中調整修改。而在不同專案中，可能需要使用不同的 Git username 和 email，
> 例如個人專案中想使用「私人 Email」 ; 而在公司專案中則需要使用「公司 Email」，若每次切換專案時都要手動更改 Git 配置會十分麻煩。
> 這時 **Git Include** 可以根據資料夾自動套用對應的用戶設定，而無需每次手動調整。

<!--more-->

---

# Config CLI

`git config` CLI 取得 Git 組態參數被存放在：

- `/etc/gitconfig`：整台電腦的所有用戶專案設定。 層級參數是 `--system`
- `~/.gitconfig`：**目前電腦用戶的所有專案共用設定**。 層級參數是 `--global` (常用)
- 任何 repository 中的 `.git/config`：**只有當前專案**的設定 (常用)

{{< alert success >}}
當 config 設定衝突時，會以範圍小的優先( Local > Global > System )。
所以 `.git/config` 的設定優先權高於在 `~./gitconfig` 裡的設定。
{{< /alert >}}

```bash
# 查詢目前所有設定值
git config --list
# 查詢特定項目的設定值
git config user.name

# 設定某值
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"

# 刪除某項設定
git config --global --unset user.name
# 打開 vim 編輯
git config --global --edit
```

在「一台電腦使用多個 Git 帳號」的情況時， Git Config 就會需要調整，本篇文章紀錄一下：

# Git 多帳號管理

初次使用 GitHub 時，大家都會先以 SSH 方式連接到 GitHub，並綁定唯一的公鑰，前面也有文章說明了 [SSH](https://aryido.github.io/posts/shell-script/ssh-key/#ssh-config)。
另外可能會遇到需要管理更多的 GitHub 帳號的情況，這時就會遇到一個問題：「**要如何讓一臺電腦同時管理多個 GitHub 帳號呢？**」

### includeIf

目前推薦的解決方案是使用 「**includeIf**」，它的邏輯是：「**用資料夾分流**」，這是 Git 官方後來支援的自動路由法。步驟如下：

##### 創建專用資料夾

在電腦裡建立專用資料夾，例如:

- `~/projects/personal/` （裡面的專案打算都用自己 github 帳戶
- `~/projects/work/` （裡面的專案要用公司的 github 帳戶

例如說 `~/projects/work/` 要用特定 Git 用戶資訊，就在此資料夾中新建一個 `.gitconfig` 檔案。
在裡面加入 Git 用戶資訊，也就是在這個 folder 下想要用的 email 與 name

```ini
[user]
    name = company-name
    email = company@company.com
```

接下來要把資訊和 config 做關聯。

##### 使用 includeIf

打開全域設定檔 `~/.gitconfig`，改成：

```ini
# 全域預設用主要帳號（例如個人帳號）
[user]
    name = personal-name
    email = personal@email.com

# 如果專案路徑在 ~/projects/work/ 底下，自動載入另一份設定
[includeIf "gitdir:~/projects/work/"]
    path = ~/projects/work/.gitconfig

```

{{< alert info >}}
需要注意 `.gitconfig` 文件有後面配置覆蓋前面配置的特性。
所以要確保專用資料夾的設定生效，並且不被覆蓋，請將 includeIf 放在文件最後
{{< /alert >}}

---

上面是比較新的方法，以下也附上舊的做法，稱 「SSH Host 别名法」：

##### Step1. 先清除電腦裡「 Git 全域（Global）設定」裡面的 Git 使用者名稱與信箱

```
git config --global --unset user.name
git config --global --unset user.email
```

{{< alert warning >}}
當 commit 時，Git 會自動附加上 name 和 email 訊息。

故若不把 「全域設定的使用者名稱與信箱」刪除，且在其他帳號要用的專案又忘記設定 local 身份時，Git 就會自動用這個全域的身份訊息去提交，導致 Commit 紀錄裡出現混亂的作者身份。
所以是建議要清除
{{< /alert >}}

##### Step2. 生成 SSH key，然後去 github 綁定並設定 `~/.ssh/config` 內容


[SSH Config](https://aryido.github.io/posts/shell-script/ssh-key/#ssh-config) 是 OpenSSH 用來集中管理連線設定的檔案，例如 MAC 通常會放在 `~/.ssh/config` 路徑下，設定完之後只要 `ssh <別名>` 就能連線。舉例如下：

```ini
Host github.com
  # HostName github.com
  # User Aryido
  IdentityFile ~/.ssh/<private-key-file>
  IdentitiesOnly yes
```

{{< alert success >}}
OpenSSH 不要求 Host 區塊內參數順序，但建議固定排版，方便維護
{{< /alert >}}

- ##### 「以 Host 區塊」來管理「主機別名」，本範例使用 `github.com`
  - 例如說上面範例 `HostName` 並沒有使用，所以預設會解析的主機名為 `github.com`
      {{< alert warning >}}
  **主機別名可以是自己另外取的名字**，但如果 「主機別名」 用其他名稱，那就要寫上 `HostName`，要不然DNS 會嘗試解析「自己另外取的別名」
  {{< /alert >}}
  - HostName 也可以是 IPV4 地址
  - 可用「通配符」例如 `Host *.dev` 來作用在所有匹配的主機上
  - 後面 Host 規則會覆蓋前面規則


- ##### User 沒寫，會用目前本機電腦使用名稱
  - SSH 連上 GitHub 時通常建議明寫「User」，但如果 git remote URL，使用者已在 URL 裡定義了，那 User 可省略（可以使用 `git remote -v` 來查看）

- ##### IdentityFile 和 IdentitiesOnly
  - 「IdentityFile」 指定「要拿哪把私鑰」
    - 可寫多行，代表依序嘗試多把 key
    - 若不寫，SSH client 會嘗試預設檔名（如 id_rsa、id_ed25519）以及 agent 裡的 key

  - 「IdentitiesOnly」 限制 SSH「只用明確指定的 key」
    - 設定 yes 之後，只送設定的 key，不亂試 agent 裡其他 key
    - 可避免因為送太多錯誤 key 而回 Too many authentication failures
    - 可避免把不相關的 key 暴露給遠端主機（隱私）


所以在這邊，要做的事情是在 `~/.ssh` 內生成 SSH key ，命名管理好並配置上 Github 並編輯 `~/.ssh/config`:

```ini
# default
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/<key-1>
  IdentitiesOnly yes

# 第二個帳戶，要建立 github 別名
Host two.github.com
  HostName github.com
  User user1
  IdentityFile ~/.ssh/<key-2>
  IdentitiesOnly yes

# 第三個帳戶，要建立 github 別名
Host three.github.com
  HostName github.com
  User user2
  IdentityFile ~/.ssh/<key-3>
  IdentitiesOnly yes
```

如果 SSH Key 都有配上 github 了，可以測試 ssh 連結，例如：

```ini
ssh -T git@two.github.com
# Hi two! You've successfully authenticated, but GitHub does not provide shell access.
# 出現這個表示連接成功
```

##### Step3. 克隆專案要記得用別名

將專案 clone 到本地要特別注意，因為上一步配置了 `~/.ssh/config` 檔，所以我們 clone 的時候要用別名，例如：

```bash
# 錯誤：（在網頁上看常規的 SSH Clone 名稱會需要改一下）
git clone git@github.com:XXXX #（不能使用這個）
# 正確：（用別名地址下載）
git clone git@two.github.com:XXXX
```

##### Step4. 綁項目對應 git 帳號

由於前面有把 global 身份取消了，現在要針對各個專案來設定身份訊息，所以去到新下載的專案內，設定 config：

```bash
git config user.name XXXX
git config user.email YYYY
```

---

# includeIf 和 SSH Host 比較

includeIf 目前比較推薦，這樣做的話，只要專案是下載到**指定的資料夾**，那就：

- 不需要去修改 `.ssh/config` 裡的 Host 網址
- git clone 時可以完全照抄 GitHub 官方網址，直接下載
- Git 在 Commit 時就會會自動指定身份訊息；不需要再針對專案做特別預設

---

### 參考資料

- [開始 - 初次設定 Git](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-%E5%88%9D%E6%AC%A1%E8%A8%AD%E5%AE%9A-Git)

- [如何一臺電腦管理多個 github 帳戶](https://juejin.cn/post/7014421400261754911)

- [如何在一台電腦使用多個Git帳號](https://medium.com/@hyWang/%E5%A6%82%E4%BD%95%E5%9C%A8%E4%B8%80%E5%8F%B0%E9%9B%BB%E8%85%A6%E4%BD%BF%E7%94%A8%E5%A4%9A%E5%80%8Bgit%E5%B8%B3%E8%99%9F-907c8eadbabf)

- [在電腦中設定 Git 帳號及 ssh key；在一台電腦中使用多個 Git 帳號；如何更改 Git 遠端網址。](https://molly1024.medium.com/%E5%9C%A8%E9%9B%BB%E8%85%A6%E4%B8%AD%E8%A8%AD%E5%AE%9A-git-%E5%B8%B3%E8%99%9F%E5%8F%8A-ssh-key-%E5%A4%9A%E5%80%8B-git-%E5%B8%B3%E8%99%9F%E5%8F%88%E8%A9%B2%E5%A6%82%E4%BD%95%E8%A8%AD%E5%AE%9A-71cabc421b17)

- [用資料夾區分不同的 Git 用戶設定](https://lynkishere.com/Others/git-multiple-users-config/)

- [includeIf 用法](https://www.cnblogs.com/librarookie/p/15697181.html)

- [【Git教學】 超輕鬆 git config 設定指南](https://www.maxlist.xyz/2022/05/26/git-config-setting/)
