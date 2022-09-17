---
title: Google Cloud CLI - 2

author: Aryido

date: 2022-09-17T00:23:55+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- google-cloud

tags:
- gcp

comment: false

reward: false
---
<!--BODY-->
> 使用 gcloud CLI 時，第一步都是要登入雲端才能獲得授權。 gcloud CLI 提供了兩個選項：
> - User account authorization
> - Service account authorization

<!--more-->

這邊 User account authorization 使用:
```
gcloud auth application-default

# Credentials saved to file: [~/.config/gcloud/application_default_credentials.json]
```
來產生 credential 在本地端

{{< alert info >}}
[gcloud auth application-default login" V.S "gcloud auth login](https://stackoverflow.com/questions/53306131/difference-between-gcloud-auth-application-default-login-and-gcloud-auth-logi)
{{< /alert >}}

## 取得 access token
在Cloud Shell中，由於內建所安裝的Cloud SDK已經載入了使用者的權限，因此可以方便的呼叫相關的SDK與取得Token


```terminal
gcloud auth application-default print-access-token
```
我們可以選定一個所要呼叫的Google API進行呼叫 ….，需要在Header處加上Authorization的Bearer token，即是上面取得的access token...
```terminal
# 例如使用Big Query
curl -H "Authorization:Bearer xxxxxxxxxxxx https://www.googleapis.com/bigquery/v2/projects/.....
```
以上，如果要測試Google的API，透過這個方式還滿方便。

---