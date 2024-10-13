---
title: "GCP -  Cloud Build 概述"

author: Aryido

date: 2024-10-13T23:50:08+08:00

thumbnailImage: "/images/google-cloud//-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> 對應到其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : \*\*\*\*
> - Microsoft Azure : \*\*\*\*

<!--more-->

---


# Cloud Build

## 簡介
Cloud Build 的宗旨是無伺服器 CI 平台。以前我們需要在本地端或者準備專門的 VM，去處理部署的部分，現在可透過 Cloud Build 完成線上 build 和  簡單 deploy 的服務。

## CloudBuild YAML簡介
簡單範例如下
```yaml
steps:
    - name: "hashicorp/terraform"
      env:
        - PROJECT_ID=$PROJECT_ID
      script: |
        terraform init
        terraform apply -var "project_id=$PROJECT_ID" -auto-approve
```

- name 
最關鍵的屬性，每一個 name 代表一個 docker image 且可以是不同得 docker image 。代表 Cloud Build 每 step 都會設置一個新 container 環境，每個 step 環境互相獨立。

官方有提供一些內建的 [cloud-builders](https://github.com/GoogleCloudPlatform/cloud-builders)，值得一提的是 gcr.io/google.com/cloudsdktool/cloud-sdk 這個 image，


## substitutions

以下三種寫法都一樣
```yaml
    - name: "gcr.io/google.com/cloudsdktool/cloud-sdk"
      entrypoint: "bash"
      args:
          [
            "-c",
            "bash script.sh $LOCATION ${_ZONE_CODE1}",
          ]

    - name: "gcr.io/google.com/cloudsdktool/cloud-sdk"
      entrypoint: "bash"
      args:
          [
            "script.sh",
            "$LOCATION", 
            "${_ZONE_CODE1}"
          ]
          
    - name: "gcr.io/google.com/cloudsdktool/cloud-sdk"
      env:
        - LOCATION=$LOCATION
        - ZONE_CODE1=${_ZONE_CODE1}
      script: |
          #!/usr/bin/env bash
          bash script.sh $LOCATION $ZONE_CODE1

```
使用 args，可以直接讀取到 substitutions 和預設的變數

## Secret value
要用secret ，只能用args
```yaml
- name: "gcr.io/cloud-builders/gke-deploy"
    env:
      - PROJECT_ID=$PROJECT_ID
      - LOCATION=$LOCATION
    secretEnv: ["ACCESS_KEY", "SECRET_KEY"] # refer to secrets ONLY in the args field of a build step.
    entrypoint: "bash"
    args: ["script.sh"]
```

{{< image classes="fancybox fig-100" src="/images/google-cloud//.jpg" >}}

{{< alert warning >}}

{{< /alert >}}

自動建構 Image 的Cloud Build
首先就是 Cloud Build 就像剛剛有稍微講過 就是我今天程式碼開發完成
你可以上傳到 GCP 它會經過 Cloud Build 幫你做成 Image
幫你把它 Build 起來 它就是可以做到 DevOps 或者是 CI/CD 裡面的自動整合
自動部署等等的一些作業 它包含了兩部分 一個是 Trigger
就是透過一些事件去驅動它 讓它去跑一些建構的 Job
就是誰觸發它了 然後就開始去跑它的 建立 Docker Image 的動作
它主要的一些細部設定 我們也是用 YAML 檔的方式 它就是一個很詳細的設定檔
比如說發生什麼事情了 然後接下來我要執行哪些作業 我會把這些
工作詳細的寫在 YAML 檔裡面 然後讓它開始跑這樣子 那 Artifact Registry


---

### 參考資料

- [Overview of Cloud Build](https://cloud.google.com/build/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)
