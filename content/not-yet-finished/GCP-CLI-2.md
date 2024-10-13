---
title: "gcloud CLI 概述 II - Authorize the gcloud CLI"

author: Aryido

#date: 2022-09-17T00:23:55+08:00
date: 2024-09-28T21:25:48+08:00

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
> 我們經常會透過 gcloud CLI 和  GCP 的雲端資源互動，為了能夠 Access 到 GCP 的雲端資源，就必須 authorize gcloud CLI，提供了兩個方式來 grant authorization 到 gcloud CLI :
> - **User account**: 代表「 End-user 」，是最常見的使用案例也是 best practice 
> - **Service account**: 是關聯到 Project 代表「 Application 」而不是「人」的使用者
> 
> 無論是 User account 和 Service account ，這兩個都屬於 Google Account 類型並且可以加入到 GCP IAM 成為一個 Principal，授權流程是使用 OAuth2 進行，授權成功之後就會有對應 Access 權限，可用來操作雲端資源了。
> 
<!--more-->

---


# Authorize with a 「 User Account 」

以下兩個 commands 都會從 GCP 上取得 account credential，並將它們存儲在本地系統上 : 

| Command           | Description                                                    |
|-------------------|----------------------------------------------------------------|
| `gcloud init`      | Authorizes access 之外還可以設置其他設定       |
| `gcloud auth login`| Authorizes access only.                                        |

 gcloud CLI 使用存儲的 account credential 存取 GCP 的資源，可以有很多個不同 account credential，但只使用一個處於 active 狀態的 account。


---

那我們現在就來試試看一些指令, 看是不是可以運作
從本機建立VM
好, 比如說我現在來試著建立一台機器看看
gcloud compute intances create instance-4 --zone asia-east1-b --
--machine-type f1-micro
好, 試試看能不能成功


那我就進去這個
Console 重新整理一下畫面
有真的有進來喔

instance-4, 對了那我想要連進來
從本機gcloud ssh到VM

要設定這個網路的標記讓它可以
套用防火牆規則


gcloud compute ssh instance-4 看看
順利的連線進來了喔





https://cloud.google.com/sdk/docs/cheatsheet






---

## 取得 access token
在Cloud Shell中，由於內建所安裝的 Cloud SDK 已經載入了使用者的權限，因此可以方便的呼叫相關的SDK與取得Token


```terminal
gcloud auth print-access-token

gcloud auth application-default print-access-token
```
我們可以選定一個所要呼叫的 Google API 進行呼叫，需要在 Header 處加上 Authorization 的 Bearer token ，即是上面取得的 access token
```terminal
# 例如使用Big Query
curl -H "Authorization:Bearer xxxxxxxxxxxx" https://www.googleapis.com/bigquery/v2/projects/.....
```
以上，如果要測試 Google 的 API ，透過這個方式還滿方便。

{{< alert warning >}}
gcloud auth print-access-token 和 gcloud auth application-default print-access-token 都是用來列印出使用者的授權 Token，但是兩者的用途略有不同。

簡單來說:
- ```gcloud auth print-access-token``` 是用來執行 gcloud CLI 相關的操作
-  ```gcloud auth application-default print-access-token``` 則是用來調用 Google Cloud APIs。

{{< /alert >}}



{{< image classes="fancybox fig-100" src="/images/google-cloud/sdk/authentication-methods.jpg" >}}

---

### 參考資料

- [Authorize the gcloud CLI](https://cloud.google.com/sdk/docs/authorizing)