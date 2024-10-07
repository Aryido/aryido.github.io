---
title: "gcloud CLI 概述 - Auth Credentials "

author: Aryido

#date: 2022-10-22T16:13:45+08:00
date: 2024-05-22T22:26:59+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud
- gcp

tags:
  - cloud-sdk


comment: false

reward: false
---
<!--BODY-->

> 曾經在使用 gcloud CLI 時，有遇到自己錯誤理解的部分，是關於 gcloud 管理的兩組 credentials :
> - `gcloud auth application-default login`: 是用於應用程式的 credential ，此 command 會管理 GCP Client Libraries 等套件會用到的 credential 稱為 Application Default Credentials (簡稱ADC)
> - `gcloud auth login`: 此 command **只會**授權 gcloud CLI 工具權限，讓它可以使用 user credential 訪問 GCP 雲端資源，同時把當前的 account activate
>
> 除了以上講解，這兩個還有什麼不同呢 ? 接下來再多做一些探討。 

<!--more-->

---

# 緣起
有遇到一個問題，是一台 linux 主機上建了 jenkins ，想用它來啟動 terraform 實現自動化部屬，並且已經有一個 GCP 帳號是專門給 jenkins 使用的，並且已經在 linux 主機上執行了 `gcloud auth login` 且登錄成功，但最後在跑 job 執行 terraform 時，卻跑出 403 錯誤。

一開始不知道原因，直到執行```gcloud auth application-default login```之後，terraform 就順利啟動了。

原本我的想法是，我使用 ```gcloud auth login``` 登入成功之後，其他應用程式如 terraform ，它就應該會取得 gcloud credentials 才對，但卻還是發生了錯誤 ! 從這邊發覺 :
> - ```gcloud auth login``` 產生的 **gcloud credentials** 
> - ```gcloud auth application-default login``` 產生的 **application default credentials** 

以上兩個應該是**不一樣**的東西，以下詳細分析...

---

# Personal gcloud credentials
在使用 gcloud CLI 之前，必須透過 ```gcloud auth login``` 登錄，來授權認證 gcloud CLI 該工具，這樣才能讓 gcloud CLI 成功操作雲端資源。

在 login 認證完之後，自動就會在本機建立一個 folder : 例如我的 MAC 筆電， credential 會是儲存在 ```~/.config/gcloud``` 下，如圖可看到有建立一些 credential 相關的檔案:

{{< image classes="fancybox fig-100" src="/images/google-cloud/sdk/gcp-creds-folder.jpg" >}}
- **credentials.db** : 是 SQLite database ，它就是 gcloud CLI 儲存和取得 OAuth tokens 的方式。另外這個 database 毫無疑問應該是要 private ，除了 gcloud CLI 以外，其它程式都不應該讀取或修改它。
- **legacy_credentials** : 也是一個 folder ，內主要有一個 adc.json ，這就是就是我們使用```gcloud auth login```登錄，得到的一個 user credential。

一旦使用 ```gcloud auth login``` 登錄， gcloud 就會使用儲存的認證，來進行所有後續操作。想移除的話，可以執行 ```gcloud auth revoke```。 

> **```gcloud auth login``` 產生的 credential 只給 gcloud cli 用，並沒有給其他應用程式使用。**

---

# Application default credentials (簡稱 ADC)

gcloud CLI 也可管理 ADC，想要讓 user credential 關聯到 ADC 可以執行: 
```
gcloud auth application-default login
```

如同 ADC 這個名稱一樣，它完全是給**應用程式**使用的 credential，是其他應用程式的 library 和 Google API 溝通的一種策略， 例如 terraform 、 java SDk 等等均是使用 ADC 來授權的，和 gcloud CLI credential 是不一樣的。 


而其他應用程式怎麼取得這個 credential 呢 ? 當 client 端的應用程式嘗試和 GCP 雲端資源溝通時，會去查找 credential，其搜尋順序如下:

> - environment variable: ```GOOGLE_APPLICATION_CREDENTIALS``` 有沒有被設置

> - 檢查預設目錄下有沒有```application_default_credentials.json```

> - 如果上面都沒有，API 會檢查它是否在 GCP 雲中運行，並嘗試獲取 service account。

{{< alert warning >}}
由於可能在 environment variable 中進行了 ADC 配置，想取消該設置可以使用：
```bash
# 刪除環境變數
unset GOOGLE_APPLICATION_CREDENTIALS

# 重新認證 ADC
gcloud config unset auth/impersonate_service_account
gcloud auth application-default login
```
{{< /alert >}}


更進一步可以去查看 ```adc.json``` 和 ```application_default_credentials.json```內容，會發現它們兩個格式一樣，但內容都是不一樣的。 其實 **application_default_credentials.json 比較接近 service account 的概念，但是不用自己設定和下載。**

> **```gcloud auth application-default login``` 產生的 ADC ，gcloud CLI 不會使用**

---

# 結論

眼尖的話，其實也會發現認證同意的畫面也不一樣。
- 左邊:  ```gcloud auth login```
- 右邊:  ```gcloud auth application-default login```
{{< image classes="fancybox fig-100" src="/images/google-cloud/sdk/gcp-approve.jpg" >}}

application default credentials 只是來方便其他應用程式取用而已，因為在應用程式的概念上，其實應該是要設置 service account 給應用程式，但每個不同應用程式都設置的話，會有點繁瑣，但好處是可以精細的定義 IAM 許可權。

簡單來說，當一天要開始操作 GCP 雲端各種服務時，每日任務建議都要執行 :
```
gcloud auth login

gcloud auth application-default login

```
以上兩步來產生 credential 在本地端。

---

### 參考資料

- [Authorize the gcloud CLI](https://cloud.google.com/sdk/docs/authorizing)

- [How Application Default Credentials works](https://cloud.google.com/docs/authentication/application-default-credentials)

- [Application default credentials vs your personal gcloud credentials](https://jpassing.com/2020/01/14/google-application-default-credentials-vs-your-personal-gcloud-credentials/)

- [Difference between "gcloud auth application-default login" and "gcloud auth login"](https://stackoverflow.com/questions/53306131/difference-between-gcloud-auth-application-default-login-and-gcloud-auth-logi)

- [Google passively discourages the use of gcloud auth application-default.](https://stackoverflow.com/questions/72745805/warnings-because-of-user-credentials-without-quota-project)

- [Local/Remote Authentication with Google Cloud Platform](https://medium.com/google-cloud/local-remote-authentication-with-google-cloud-platform-afe3aa017b95)
