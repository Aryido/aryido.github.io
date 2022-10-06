---
title: Terragrunt - variables 精簡

author: Aryido

date: 2022-10-05T23:27:20+08:00

thumbnailImage: "/images/terraform/terragrunt.jpg"

categories:
- terraform

tags:
- terragrunt

comment: false

reward: false
---
<!--BODY-->

> 變數管理也是一個讓 code 更加 DRY 的方式之一， terragrunt 有蠻多傳遞變數的方式，這邊舉例 inputs
<!--more-->

---
### Scenario
```
├── dev
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│
├── stg
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│
└── prod
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│
```
如上在多環境中，幾乎每個 root module 的 variables.tf 內，可能都有重複的參數要傳。 例如 gcp 中可能都需要 region，故我們可能是在每個環境資料夾內，放一個 env.tfvars，例如:
```
# env.tfvars
region=us-east1
# ... 其他省略
```
想說怎麼樣共同管理重複的變數呢?

---
# 解決：使用 Terragrunt inputs 從外部統一傳入變數

在不同環境的資料夾外層，配置一個 terragrunt.hcl ，並把不同環境的資料夾內的 provider.tf 刪掉，然後每個環境的資料夾內，加入 terragrunt.hcl。

```
├── terragrunt.hcl
├── dev
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│   ├── terragrunt.hcl
│
├── stg
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│   ├── terragrunt.hcl
│
└── prod
│   ├── main.tf
│   │── outputs.tf
│   ├── variables.tf
│   ├── env.tfvars
│   ├── terragrunt.hcl
│
```

最外層的 terragrunt.hcl 內容有:
```json
locals {
  region     = "us-east1"
}

inputs = {
   region     = "${local.region}"
}
```

內層 terragrunt.hcl 的依賴外面 terragrunt.hcl

```json
include "root" {
  path = find_in_parent_folders()
}
```
如此一來所有 env.tfvars 內的 region 變數都可以拿掉了，由外部統一傳入。

#### 小提醒

{{< alert warning >}}
變量按以下順序加載：

- 環境變量。

- terraform.tfvars文件（如果存在）。

- terraform.tfvars.json文件（如果存在）。

- 命令行上的任何-var/-var-file選項按提供的順序排列

**下面層級會覆蓋上面層級，Terraform 使用最後一個值來覆蓋以前的值。**

{{< /alert  >}}

{{< alert info >}}

運行 Terragrunt 命令時，Terragrunt 都會將 inputs 輸入設置為環境變量。類似:

```
$ terragrunt apply

# Roughly equivalent to:

TF_VAR_region="us-west1" \

terraform apply
```

{{< /alert  >}}

---
