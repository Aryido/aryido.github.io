---
title: Terragrunt - provider 精簡

author: Aryido

date: 2022-09-29T22:40:07+08:00

thumbnailImage: "/images/terraform/terragrunt.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->

> Provide 是整個 terraform 最重要的元件，是決定要對哪一個平台操作 (e.g. AWS, Azure, gcp)，負責和雲端 API 的接口交互，可以在不了解 API 細節的情況下，通過 terraform 來編排資源。
<!--more-->

---
### Scenario
```
├── dev
│   ├── provider.tf
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│
├── stg
│   ├── provider.tf
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│
└── prod
│   ├── provider.tf
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│
```
如上在多環境中，幾乎每個環境的都有 provider.tf ，而且可能還都長一樣。

---
# 解決：使用 Terragrunt 動態建立 provider.tf

可在不同環境的資料夾外層，配置一個 terragrunt.hcl ，並把不同環境的資料夾內的 provider.tf 刪掉，然後每個環境的資料夾內，加入 terragrunt.hcl。

```
├── terragrunt.hcl
├── dev
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── terragrunt.hcl
│
├── stg
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── terragrunt.hcl
│
└── prod
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── terragrunt.hcl
│
```

最外層的 terragrunt.hcl 內容有:

```json
# gcp/terragrunt.hcl

generate "provider" {
  path = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents = <<EOF
    terraform {
      required_providers {
        google = {
          version = "~> 4"
        }
      }
    }

    provider "google" {
      project = var.project_id
      region  = var.region
    }
  EOF
}
```
不同環境內 terragrunt.hcl ，使用 include 來依賴外面 terragrunt.hcl
```json
include "root" {
  path = find_in_parent_folders()
}
```
以上我們把重複的 provider.tf 設定，放到上層 terragrunt.hcl，然後再用 include。則在 terragrunt 執行 command 時 (terraform command 前）會動態 generate 出 provider.tf 在各個環境資料夾下。

#### 小提醒

{{< alert info >}}
這邊使用 generate {} code block 來產生 provider.tf。會在 source 的目錄（也就是執行 terraform 的目錄）下產生 provider.tf
{{< /alert  >}}

{{< alert info >}}

如果要調整 provider.tf 的參數，這裡也支援使用 terragrunt 的 function 與變數。

{{< /alert  >}}

---
