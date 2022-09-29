---
title: Terragrunt - Backend / State 設定自動化

author: Aryido

date: 2022-09-27T22:55:22+08:00

thumbnailImage: "/images/terraform/terragrunt.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->

> Terraform Backend 可將 Terraform State 存儲在雲端位置，例如 S3 bucket, azure blob storage, gcp cloud storage，並提供 lock 以防止 race conditions 。 Terragrunt 還進一步讓流程更簡便...
<!--more-->

---
### 問題 Scenario
原本要使用 Terraform Backend，我們原本要配置：
```json
terraform {
  backend "gcs" {
    bucket = "terraform-backend-d950cd"
    prefix = "dev/state"
  }
}
```
想想在不同環境下， terraform backend 基本上都長得一樣，大概只 prefix 會有變化，我們很自然的會想用變數去控制，但是

{{< alert danger >}}
***terraform backend不支持任何類型的變量表達式***

```json
terraform {
  backend "gcs" {
    # Using variables does NOT work here!!!
    bucket = var.terraform_state_bucket
    prefix = var.terraform_prefix
  }
}
```
{{< /alert  >}}

這代表對於不同環境，只能 copy/paste 到每個 Terraform module中。而且還必須非常小心地以免兩個 module 覆蓋彼此的狀態文件！

---
# 解決：使用 Terragrunt 自動建立 GCS bucket

可在不同環境的資料夾**外層**，配置一個 `terragrunt.hcl`，內容有:

```json
remote_state {
  backend = "gcs"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
  config = {
    project  = "${local.project_id}"
    location = "${local.region}"
    # bucket   = "terraform-backend-${uuid()}"
    bucket   = "terraform-backend-3c4658a3d73f"
    prefix = path_relative_to_include()
  }
}
```

{{< alert info >}}

使用 remote_state {} code block 設定 remote backend state

{{< /alert  >}}

{{< alert info >}}

使用 generate，會在 terraform 的內產生 backend.tf

{{< /alert  >}}

然後 不同環境的資料夾內，也建一個 terragrunt.hcl ，並依賴外面的 terragrunt.hcl，語法為:


```json
include "root" {
  path = find_in_parent_folders()
}
```

{{< alert info >}}
  include code block 可以想成是 terragrunt 繼承其他的 terragrunt.hcl 的設定語法
{{< /alert  >}}

---
