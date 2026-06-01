---
title: "SSH Key"

author: Aryido

date: 2025-04-30T19:37:26+08:00

thumbnailImage: "/images/linux/ssh/ssh.jpg"

categories:
  - language
  - shell-script
  - ci-cd

tags:
  - git

comment: false

reward: false
---

<!--BODY-->

> SSH 是 Secure Shell 的縮寫，為一種通訊協定，會對裝置之間的連線進行**驗證**和**加密**，安全地傳送 command。通常用於遠端控制伺服器、管理基礎架構和傳輸檔案。SSH 是在應用層和傳輸層上執行的，透過稱為公開金鑰加密，「**public key 可供任何人使用**」 ; 另一個金鑰為「**private key 由其擁有者保密**」，兩個 key 相互對應。此外傳輸的數據是經過壓縮的，所以可以加快傳輸的速度。並且許多作業系統，包含 macOS、Linux 都天然支援。
> {{< image classes="fancybox fig-100" src="/images/linux/ssh/public-private-key.jpg" >}}

<!--more-->

---

軟體工程師最常使用到 SSH key 的時機，大概就是設定 GitHub、 Gitlab 等等倉儲時候的了。還記得在 2021 年時，GitHub 有來個設定大改版通知，要[廢除輸入帳密的方式，全面改用使用 SSH 金鑰](https://github.blog/security/application-security/token-authentication-requirements-for-git-operations/)。

# 配置 GitHub 金鑰

透過 SSH (Secure Shell) 協定與 GitHub 建立安全連線，設定好好以後，在 git push 或 pull 時就不需要每次輸入帳號密碼了。使用 SSH Key-based 登入時，主要就是做兩件事情：

- 產生 key-pair
- 將產生的 **Pub key** 放到遠端服務器，在這裡也就是我們的 GitHub

### [Generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

通常本地的 ssh-key 都是放在 `~/.ssh` 這個目錄下，然後使用 `ssh-keygen` 這個指令產生金鑰

```shell
# 若 .ssh目錄不存在，就自己建一個，並設定正確的權限
mkdir ~/.ssh
chmod 700 ~/.ssh

# 選擇 ED25519 作為金鑰的加密演算法，並使用 email 作為標籤，創建一個新的 SSH 金鑰。
ssh-keygen -t ed25519 -C "your_email@example.com"

```

{{< alert info >}}
常見的 SSH 金鑰類型包括 RSA、ECDSA 和 EdDSA，它們提供不同層級的安全性和效能。Ed25519 是使用橢圓曲線的 EdDSA 簽章方案的一個版本，現在非常推薦。github 官方教學範例也是使用 `ed25519`
{{< /alert >}}

在 ssh-keygen 中常用參數如下：

- `-t`：指定金鑰的加密演算法，預設使用 rsa。現在都是推薦 ed25519 ，但可能會發生舊系統無法生成的狀況。若系統不支援，可以改回用 rsa
- `-f`：指定金鑰的檔名。預設檔名會隨演算法而變動，例如使用 rsa 加密時，其檔名預設為 id_rsa（私鑰id_rsa，公鑰id_rsa.pub
- `-b`：指定金鑰長度（bits）。在使用預設的加密演算法 rsa 時，建議把這項調到 4096
- `-C`：通常使用 email 標籤

在產生金鑰的過程中，還會詢問一些事項(可以全部使用預設值，按 Enter略過)：

- `Enter file in which to save the key (/home/username/.ssh/id_ed25519)`:

  確認金鑰儲存的位置與檔名。但預設檔名無法表明金鑰的用途，所以建議都要改名，例如我是用 **<電腦主機> + <服務>** 來命名 : macbook-air-m3_github

- `Enter passphrase (empty for no passphrase)` ; `Enter same passphrase again`

  再來是詢問是否設定密碼來保護 key，若有設定密碼的話，之後每次使用這把金鑰時，就要輸入密碼

完成上述設定後，會看到 fingerprint 與 randomart ，就代表產生成功了:

```bash
Your identification has been saved in /home/username/.ssh/github_key.
Your public key has been saved in /home/username/.ssh/github_key.pub.
The key fingerprint is:
......  your_email@example.com
The key’s randomart image is:
+---[ED25519 256]----+
|                 |
|                 |
...
|                 |
|                 |
+----[SHA256]-----+
```

這時候再去 `~/.ssh` 目錄下看，應該會發現有多出 public-key 和 private-key 。

### Adding your SSH key to the ssh-agent (Optional)

接下來就是把上面產生的產生的 public-key 放到遠端服務器，[例如這邊就是放到 GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。再來可選擇不要不設定「**金鑰代理**」，如果**有設定金鑰密碼**，但又不想每次使用金鑰就要輸入一次，可以考慮使用。

現在比較新的做法是使用 `~/.ssh/config` 檔案，讓密鑰能自動載入 ssh-agent 中，以下範例：

```bash
Host github.com
  IdentityFile ~/.ssh/<private-key-file>
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes
```

> Note: 沒有設定密碼，或是不想起一個 agent 在背景跑，可以跳過這個步驟

完成金鑰上傳後，驗證下這組金鑰是不是正常工作:

```bash
ssh -T git@github.com
```

{{< alert warning >}}
GitHub 完成金鑰的產生與配置後，[還要更改 git remote](https://docs.github.com/en/get-started/git-basics/managing-remote-repositories#switching-remote-urls-from-https-to-ssh)，否則即便配置 SSH，還是會走 https 的傳輸協定。

```bash
git remote -v
#origin  https://github.com/<user_name>/<repo>.git (fetch)
#origin  https://github.com/<user_name>/<repo>.git (push)

# 改成 `git@github.com` 開頭，讓 Github 對於走不同的伺服器倉庫網址
git remote set-url origin git@github.com:<user_name>/<repo>.git

```

{{< /alert >}}

---

# [SSH Config](#ssh-config)

補充一些進階用法

##### AddKeysToAgent 和 UseKeychain 經常一起使用
```config
Host github.com
  # HostName github.com
  # User Aryido
  IdentityFile ~/.ssh/<private-key-file>
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes
```

- 「AddKeysToAgent」可以在使用私鑰連線時，自動把 key 加到 ssh-agent，減少反覆輸入
- 「UseKeychain」 **是 macOS 專用的參數**，開放可存取 Keychain


##### ProxyCommand
例如已經有自己建立 gitlab 服務器：
```config
Host gitlab.aryido.com
  HostName gitlab-ssh.aryido.com
  ProxyCommand /opt/homebrew/bin/cloudflared access ssh --hostname %h
  IdentityFile ~/.ssh/XXXXX
  IdentitiesOnly yes
```

ProxyCommand 是 SSH 的「自訂中繼連線命令」參數，SSH 不直接連目標主機而是先執行指定的外部命令建立一條通道，再把 SSH 流量送進去
- 常見場景是「要由跳板機（bastion）連內網主機」、「需要 SOCKS Proxy」等等
{{< alert warning >}}
ProxyCommand 會依賴本機工具（如 cloudflared）是否存在，要特別注意依賴
{{< /alert >}}
- 有一些常見的參數可以使用如：
  - `%h`：目標主機(HostName)
  - `%p`：目標埠號(Port)

##### ProxyJump
ProxyJump 較新故優先建議使用，需要 OpenSSH 7.3+。單純跳板機情境通常可改寫成： `ProxyJump <bastion>`
```ini
Host company-inner-env
  HostName XXX.XX.XXX.X
  User Aryido
  ForwardAgent yes
  IdentityFile <private_key_path>
  IdentitiesOnly yes
Match originalhost env-1 !exec "route -n get default | grep -q 'XXX.XX.XXX.X'"
  ProxyJump company-bastion
  AddKeysToAgent yes

Host company-bastion
  HostName YYY.Y.YY.Y
  IdentityFile <private_key_path>
  IdentitiesOnly yes

```
另外補充 `Match` 語法，它是 ssh_config 裡的「條件區塊」，成立就套用該區塊內的參數，不成立就略過，在這裡：

> `Match originalhost env-1 !exec "route -n get default | grep -q 'XXX.XX.XXX.X'"`

可拆成 :
  - `originalhost env-1`: 命令列輸入的原始主機名必須是 env-1
  -  `!exec "..."`: exec 會在本機執行 shell 命令，用「退出碼」判斷是否成立
  
所以這段意思是：當命令列輸入的原始主機名必須是 env-1 且 `route ... | grep ...` 找不到指定 IP 時，那就會使用跳板機 bastion。


{{< alert danger >}}
ForwardAgent 代表：
把本機的 ssh-agent 轉送到遠端主機，在遠端也能「借用本機金鑰能力」再去 SSH 到其他主機

雖然它不會把私鑰檔案直接複製到遠端，但遠端若被入侵，可能濫用 agent 做後續連線，所以只在必要且可信主機上開啟
{{< /alert >}}


---

### 參考資料

- [什麼是 SSH？ | 安全殼層 (SSH) 通訊協定](https://www.cloudflare.com/zh-tw/learning/access-management/what-is-ssh/)

- [Day21：【技術篇】SSH 的基本運作原理](https://ithelp.ithome.com.tw/articles/10277498)

- [使用 SSH 金鑰與 GitHub 連線](https://cynthiachuang.github.io/Generating-a-Ssh-Key-and-Adding-It-to-the-Github/)

- [SSH 跳板機原理與配置：實現無縫跳板連接，一步直達目標主機](https://zhuanlan.zhihu.com/p/1895906139855627003)