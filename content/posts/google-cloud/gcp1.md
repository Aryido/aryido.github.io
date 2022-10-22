---
title: Google Cloud CLI - 1

author: Aryido

date: 2022-09-15T23:13:26+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud

tags:
- gcp

comment: false

reward: false
---
<!--BODY-->
>  gcloud CLI 可以用於創建和管理 gcp 資源的工具。雖然可以使用 CLI 從命令行或腳本，自動化執行許多常見的任務。但更多是會用例如  **Terraform** 來部屬資源。故現在基本上用最多是在初始化
> - **管理身份驗證(manage authentication)**
> - **自定義本地配置(customize local configuration)**
>
> 這些和權限管理、project設定有關的功能。

<!--more-->

---

# gcloud config
GCP 的設計，是把所有資源件在一個 project 上，可以想成類似
- **azure resource group**
- **aws account**。

進行 gcloud init 可以設定 project
```
# Initialize, authorize, and configure the gcloud CLI
gcloud init

```
{{< alert warning >}}
Project ID 限制:

- Must be 6 to 30 characters in length.
- Can only contain lowercase letters, numbers, and hyphens.
Must start with a letter.
- Cannot end with a hyphen.
- Cannot be in use or previously used; this includes deleted projects.
- Cannot contain restricted strings, such as google and ssl.
{{< /alert >}}

---

## 列出目前 active account

建立完成之後，查詢 gcloud cli 目前作用的帳戶方式為
```
gcloud config configurations list

# 以下輸出範例
# true 為 active 的 account
NAME     IS_ACTIVE  ACCOUNT              PROJECT                DEFAULT_ZONE  DEFAULT_REGION
default  True       xxxxx@gmail.com      test-project-235152    asia-east1-a  asia-east1

```

## 切換 active account 的方式
```
# activate 後面接 configurations 的 name 就可以切換帳號了
gcloud  config  configurations activate <project name>
```

---

## 列出projects
```
gcloud projects list --format="json"

gcloud projects list --sort-by=projectId --limit=5
```

## 帳號切換 project
```
# change project in active account.
gcloud config set project
```

---

## 其他常用命令
```
# Display version and installed components.
gcloud version

# Display current gcloud CLI environment details.
gcloud info
```

---