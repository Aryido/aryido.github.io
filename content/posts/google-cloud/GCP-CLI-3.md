---
title: "Google Cloud CLI : Auth Credentials "

author: Aryido

date: 2022-10-22T16:13:45+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud
- gcp

tags:


comment: false

reward: false
---
<!--BODY-->

> 最近使用 gcloud CLI 時，有遇到自己錯誤理解的部分，是關於 gcloud 管理的兩組 credentials :
> - gcloud auth application-default login
> - gcloud auth login
>
> 這兩個到底有甚麼不同呢 ? 來記錄一下吧 ~

<!--more-->

---

# 緣起
有遇到一個問題，是在一台 linux 主機上建了 jenkins ，想用它來啟動 terraform 實現自動化部屬，並且已經有一個 gcp 帳號是專門給 jenkins 的。然後已經在 linux 主機上執行 ```gcloud auth login``` 登錄成功，但最後在跑 job 執行 terraform 時，卻跑出 403 錯誤。

一開始不知道原因，直到執行```gcloud auth application-default login```之後，terraform 就順利啟動了。

原本我的想法是，我使用 ```gcloud auth login``` 之後，其他應用程式如 terraform ，它應該會取得 gcloud credentials (可能是從 credentials.db 或 adc.json 取得)。但卻發生了錯誤 ! 從這邊發覺```gcloud auth login```產生的 **gcloud credentials** 和```gcloud auth application-default login```產生的**application default credentials** 應該是**不一樣**的東西。

以下詳細分析...

---

## Personal gcloud credentials
在使用 gcloud cli 之前，必須透過 ```gcloud auth login``` 登錄，來授權認證 gcloud 該工具，這樣才能讓 gcloud cli 工作。

在 login 認證完之後，自動就會在本機建立一個 folder : 例如我的 MAC 筆電，在 ```~/.config/gcloud``` 下有建立一些 credential 相關的檔案，如圖:

{{< image classes="fancybox fig-100" src="/images/google-cloud/gcp-creds-folder.jpg" >}}
- credentials.db : 是 SQLite database ，它就是 gcloud 儲存和取得 OAuth tokens 的方式。另外這個 database 毫無疑問應該是要 private ，除了 gcloud 以外，其它程式都不應該讀取或修改它。
- legacy_credentials : 也是一個 folder ，內主要有一個 adc.json ，這就是就是我們使用```gcloud auth login```登錄，得到的一個 cached user credential。

一旦使用 ```gcloud auth login``` 登錄， gcloud 就會使用儲存的認證，來進行所有後續操作。想移除的話，可以執行 ```gcloud auth revoke```。 **```gcloud auth login``` 產生的 credential 只給 gcloud cli 用，並沒有給其他應用程式使用。**

{{< alert warning >}}
window 系統， folder 路徑是```~/AppData/XXX/gcloud/application_default_credentials.json```
{{< /alert >}}

---

## Application default credentials

如同 application default credentials 這個名稱一樣，它完全是給其他應用程式權限用的 credential ，其他應用程式怎麼取得這個 credential 呢 ? 有下列幾種方式:

- environment variable: ```GOOGLE_APPLICATION_CREDENTIALS``` 有沒有被設置

-  檢查預設目錄下有沒有```application_default_credentials.json```

-  如果上面都沒有，API 會檢查它是否在 GCP 雲中運行，並嘗試獲取 service account。


如果查看 ```adc.json``` 和 ```application_default_credentials.json```，會發現它格式一樣，但內容都是不一樣的。 **application_default_credentials.json 比較接近 service account，但是不用自己設定和下載。**

---

# 結論

眼尖的話，其實也會發現認證同意的畫面其實不一樣。
- 左邊:  ```gcloud auth login```
- 右邊:  ```gcloud auth application-default login```
{{< image classes="fancybox fig-100" src="/images/google-cloud/gcp-approve.jpg" >}}

application_default_credential 是其他應用程式的 library 和 Google API 溝通的一種策略， client 端的應用程式例如 terraform 、 java SDk 都可以使用 application_default_credential ，它和個人 gcloud credential 無關。當 client 端的應用程式嘗試載入 credential 時，會去查找，便於開發。

application default credentials 只是來方便其他應用程式取用而已，因為在應用程式的概念上，其實應該是要設置 service account 給應用程式，但每個不同應用程式都設置的話，會有點繁瑣，但好處是可以精細的定義 IAM 許可權。

---

# Reference
- [How Application Default Credentials works](https://cloud.google.com/docs/authentication/application-default-credentials)


- [Application default credentials vs your personal gcloud credentials](https://jpassing.com/2020/01/14/google-application-default-credentials-vs-your-personal-gcloud-credentials/)

- [Difference between "gcloud auth application-default login" and "gcloud auth login"](https://stackoverflow.com/questions/53306131/difference-between-gcloud-auth-application-default-login-and-gcloud-auth-logi)

- [Google passively discourages the use of gcloud auth application-default.](https://stackoverflow.com/questions/72745805/warnings-because-of-user-credentials-without-quota-project)

- [Local/Remote Authentication with Google Cloud Platform](https://medium.com/google-cloud/local-remote-authentication-with-google-cloud-platform-afe3aa017b95)
