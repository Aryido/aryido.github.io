---
title: "gcloud CLI 概述 - Initialization GCP Project"

author: Aryido

#date: 2022-09-15T23:13:26+08:00
date: 2024-05-22T20:14:37+08:00

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

> gcloud CLI 全稱是 Google Cloud Command Line Interface ，是創建和管理 GCP 雲端資源的「命令行」
工具，還捆綁了專用的子工具例如 BigQuery(bq CLI)、 Kubernetes 集群(kubectl CLI) 可配合使用，是隸屬在 Cloud SDK 內的。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **AWS CLI**
> - Microsoft Azure : **Azure CLI**
>
> 雖然可以使用 gcloud 來撰寫 script 自動化執行許多常見的任務，但實務上更多是會用例如 **Terraform** 等 IaC 工具來部屬和管理雲端資源， gcloud 現在基本上用最多是在初始化帳戶如 :**管理身份驗證(manage authentication)**，或是**自定義本地配置(customize local configuration)**，這些如權限管理、Project 設定、Billing 有關的功能會是 gcloud CLI 主要使用的地方。

<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/google-cloud/sdk/sdk-architecture.jpg" >}}

# GCP Project 初始化

GCP 的設計是把所有雲端資源建立在一個 GCP Project 上，故 GCP Project 概念上對應成其他雲端服務的部分類似為:
- **azure resource group**
- **aws account**

由於是一切雲端資源創立的起點，故使用 gcloud CLI 時最先要做的，是初始化設定雲端資源要操作哪個 GCP Project 上

```bash
# Default configuration ，可以直接從這步驟把 GCP Project 初始化完成
gcloud init

# 防止自動打開 web browser，常用在 remote ssh
gcloud init --console-only
```

{{< alert success >}}
gcloud init 時在 terminal 會有教學，幫助把 Account、Project、Region/Zone 都設定完成，設定完成之後 `default` 會是預設的 configuration 名稱
{{< /alert >}}

# 管理 gcloud CLI 配置

實務上蠻有可能會需要配置多個 gcloud CLI configuration 的，原因是可能「有多個 GCP-Accounts 會使用到」或者是「多個 GCP Projects 要切換」，故可以設定 :

```bash
# 創建/刪除 configuration
gcloud config configurations create [NAME]
gcloud config configurations delete [NAME]

---
# 列出 configurations
gcloud config configurations list

# 範例如下， True 為目前 active 的 configuration
# NAME         IS_ACTIVE     ACCOUNT            PROJECT               DEFAULT_ZONE  DEFAULT_REGION
# default      False         user@gmail.com     example-project-1     us-east1-b    us-east1
# dev          False         dev1@gmail.com     example-project-2     us-east1-c    us-east1
# prod         True          prod@gmail.com     example-project-3     us-east1-b    us-east1

# 進階查看 configuration 屬性
gcloud config configurations describe [NAME]

---
# 啟動 configuration
gcloud config configurations activate [NAME]

```

{{< alert success >}}
使用 `gcloud config configurations create [NAME]` 創建多個配置，並通過 `gcloud config configurations activate [NAME]` 來切換不同的帳戶和配置，是管理多個 GCP 帳戶的 Best Practice
{{< /alert >}}

一般來說 configuration 的儲存位置會在 :
- MacOS/Linux : `~/.config/gcloud`
- Windows : `%APPDATA%\gcloud`


可以用 gcloud CLI 確認 configuration 設定檔案儲存在本機哪裡 :
```bash
gcloud info --format='value(config.paths.global_config_dir)'

# 另外也可用
which gcloud 
```

{{< alert info >}}
which 是一個命令，主要用來在類 Unix 系統（如 Linux 或 macOS）中查找**指定命令的執行檔所在的路徑**。 which 命令會在系統的 PATH 環境變數指定的目錄中，逐一查找可執行文件，如果 which 沒有返回任何結果，可能是指定套件尚未安裝，或它不在 PATH 路徑中
{{< /alert >}}

### Project 操作

```bash
gcloud config set project [PROJECT_ID]

gcloud projects describe [PROJECT_ID]

gcloud projects list --format="json"

gcloud projects list --sort-by=[PROJECT_ID] --limit=5

gcloud config unset [PROJECT_ID]

```

### [其他常用命令](https://cloud.google.com/sdk/docs/cheatsheet)

```bash
# Display version and installed components.
gcloud version

# Display current gcloud CLI environment details.
gcloud info

gcloud auth list

# Grant gcloud CLI 的 credentials
gcloud auth login

gcloud auth login --no-browser
gcloud auth login --no-launch-browser

# Revoke 對 gcloud CLI 的 credentials
gcloud auth revoke <ACCOUNT_ID>

gcloud auth activate-service-account <SERVICE_ACCOUNT_ID> --key-file=<KEY_FILE_PATH> --project=<PROJECT_ID>

```

# Alpha & Beta Version

默認情況下， gcloud CLI 會安裝 `alpha`、`beta` 這種還未正式 release 的 gcloud CLI 版本，可以嘗試的使用它，這兩個的差別介紹如下附表 : 
| **Label** | **Description** |
|-----------|-----------------|
| `beta` | Commands 在功能上是完整的，但可能還未完成， Commands 還是可能發生 Breaking changes 且**可能**不會另行通知 |
| `alpha` | Commands 處於 early release 早期版本中，如有更改，恕不另行通知! |

使用上就是很簡單的在 gcloud 後面加上 Label 就可以使用了例如：

```
# 與 Compute Engine 相關的 Beta 版命令
gcloud beta compute

# Alpha 版本中管理 App Engine 命令
gcloud alpha app
```

---

### 參考資料

- [gcloud CLI overview ](https://cloud.google.com/sdk/gcloud)

- [Terraform - gcloud ](https://id.cloud-ace.com/redesigning-the-cloud-sdk-cli-for-easier-development/)

- [[GCP 教學] 024 從本機輕鬆管理雲端資源 - Cloud SDK 2分鐘快速安裝！說明欄有指令 [有字幕]](https://www.youtube.com/watch?v=REJmbdxQKp4&list=PLvYsUzEf9kaP39bS21SKn4Onp3KQRbFsA&index=26)
