---
title: Google Cloud 常用指令

author: Aryido

date: 2022-09-15T23:13:26+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- google-cloud

tags:
- gcp

comment: false

reward: false
---
<!--BODY-->
> 最近開始碰google Cloud了，來記錄些常用到的簡單指令吧!(待修正排版)

<!--more-->
# Gcloud config
GCP 的設計是把所有資源件在一個 project 上(很不精確的描述，類似 azure resource group 與 aws account)
進行 gcloud init 可以設定 project
```
 gcloud init
```
{{< alert warning >}}
Project ID requirements:

- Must be 6 to 30 characters in length.
- Can only contain lowercase letters, numbers, and hyphens.
Must start with a letter.
- Cannot end with a hyphen.
- Cannot be in use or previously used; this includes deleted projects.
- Cannot contain restricted strings, such as google and ssl.
{{< /alert >}}

建立完成之後, 查詢的方式為
```
gcloud  config  configurations list

# 以下輸出範例
NAME     IS_ACTIVE  ACCOUNT              PROJECT                DEFAULT_ZONE  DEFAULT_REGION
default  True       xxxxx@gmail.com      test-project-235152    asia-east1-a  asia-east1

```
切換 configurations 的方式
```
# activate 後面接 configurations 的 name 就可以切換設定擋了

gcloud  config  configurations activate default
```

使用 gcloud CLI 時，第一步都是要登入雲端才能獲得授權。 gcloud CLI 提供了兩個選項：
- User account authorization
- Service account authorization

這邊 User account authorization 使用:
```
gcloud auth application-default

# Credentials saved to file: [~/.config/gcloud/application_default_credentials.json]
```
來產生 credential 在本地端

{{< alert info >}}
[Difference between "gcloud auth application-default login" and "gcloud auth login"](https://stackoverflow.com/questions/53306131/difference-between-gcloud-auth-application-default-login-and-gcloud-auth-logi)

https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login

https://cloud.google.com/sdk/gcloud/reference/auth/login
{{< /alert >}}


在Cloud Shell中，由於內建所安裝的Cloud SDK已經載入了使用者的權限，因此可以方便的呼叫相關的SDK與取得Token...
取得 access token…

```terminal
gcloud auth application-default print-access-token
```
我們可以選定一個所要呼叫的Google API進行呼叫 ….，需要在Header處加上Authorization的Bearer token，即是上面取得的access token...
```terminal
# 例如使用Big Query
curl -H "Authorization:Bearer xxxxxxxxxxxx https://www.googleapis.com/bigquery/v2/projects/.....
```
以上，如果要測試Google的API，透過這個方式還滿方便。

列出projects
```
gcloud projects list --format="json"

gcloud projects list --sort-by=projectId --limit=5
```

```
gcloud info
```
---