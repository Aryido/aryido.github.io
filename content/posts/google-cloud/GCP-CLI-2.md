---
title: Google Cloud CLI - 2

author: Aryido

date: 2022-09-17T00:23:55+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp

comment: false

reward: false
---
<!--BODY-->
> 使用 GCP 時，都要登入雲端才能獲得授權。 gcloud CLI 提供了兩個方式：
> - User account authorization
> - Service account authorization

<!--more-->

---

這邊 User account authorization 使用:
```
gcloud auth application-default login

# Credentials saved to file: [~/.config/gcloud/application_default_credentials.json]
```
來產生 credential 在本地端，application-default 是一個專門給應用程式用的 credential，簡化使用 Service account 繁瑣的步驟。

官網上的簡介:

{{< alert info >}}
### gcloud auth application-default login

獲取新的 user credential，預設是用於應用程式的 credential。
{{< /alert >}}

{{< alert info >}}
### gcloud auth login :

認證 Cloud SDK ，只授權 gcloud 這個 CLI 工具權限，讓它可以使用 user credential 訪問 GCP。
{{< /alert >}}

---

## 取得 access token
在Cloud Shell中，由於內建所安裝的 Cloud SDK 已經載入了使用者的權限，因此可以方便的呼叫相關的SDK與取得Token


```terminal
gcloud auth application-default print-access-token
```
我們可以選定一個所要呼叫的Google API進行呼叫，需要在 Header 處加上 Authorization 的 Bearer token ，即是上面取得的 access token
```terminal
# 例如使用Big Query
curl -H "Authorization:Bearer xxxxxxxxxxxx https://www.googleapis.com/bigquery/v2/projects/.....
```
以上，如果要測試Google的API，透過這個方式還滿方便。

---