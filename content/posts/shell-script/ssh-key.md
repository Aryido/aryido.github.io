---
title: "SSH Key"

author: Aryido

date: 2026-05-15T19:37:26+08:00

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

  再來是詢問是否設定密碼來保護 key，若有設定密碼的話，之後使用每次使用時，這把金鑰時就要輸入密碼

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
接下來就是把上面產生的產生的 public-key 放到遠端服務器，[例如這邊就是放到 GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。再來可選擇設定金鑰代理，如果**有設定金鑰密碼**，但又不想每次使用金鑰就要輸入一次，可以考慮設定金鑰代理，現在比較新的做法是使用 `~/.ssh/config` 檔案，讓密鑰能自動載入 ssh-agent 中，以下範例：
```bash
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/<private-key>
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

### 參考資料

- [什麼是 SSH？ | 安全殼層 (SSH) 通訊協定](https://www.cloudflare.com/zh-tw/learning/access-management/what-is-ssh/)

- [Day21：【技術篇】SSH 的基本運作原理](https://ithelp.ithome.com.tw/articles/10277498)

- [使用 SSH 金鑰與 GitHub 連線](https://cynthiachuang.github.io/Generating-a-Ssh-Key-and-Adding-It-to-the-Github/)




