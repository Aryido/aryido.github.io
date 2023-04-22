---
title: "Terraform-provider-archive"

author: Aryido

date: 2023-04-21T22:48:00+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->
> terraform archive 是一個 HashiCorp 公司提供的一個 Provider，可以讓使用者在 Terraform 中，將指定的目錄或檔案打包(例如 zip )。
>
> 雲端上運行的無伺服器計算服務非常熱門，著名的如 AWS Lambda 、 Azure Functions、GCP Cloud Functions，也常使用 terraform 工具來進行部屬。這時再使用 terraform archive ，方便地把例如 python code 打包成 zip，簡便地部署到服務上。


<!--more-->

[terraform archive](https://registry.terraform.io/providers/hashicorp/archive/latest/docs/data-sources/file) 使用起來十分簡單，提供了一個 Terraform Data Source ，可以把:
- 內容(content)
- 單個 file 、或多個 files
- directory

以上列出的範例，都可以方便地做成一個 archive 檔案。下面給一個簡單的範例，把 archive 傳到 google storage bucket 上 :

```json

resource "random_id" "code" {
  byte_length = 8
}

resource "google_storage_bucket" "test" {
  project       = var.project_id
  name          = "test-${random_id.code.hex}"
  location      = var.bucket_location
}

resource "google_storage_default_object_acl" "policy" {
  bucket = google_storage_bucket.test.name
  role_entity = [
    "READER:allUsers",
  ]
}

data "archive_file" "test" {
  type        = "zip"
  source_dir  = "${path.module}/source/test"
  output_path = "${path.module}/source/test.zip"
}

resource "google_storage_bucket_object" "source" {
  name   ="test.zip"
  source = data.archive_file.test.output_path
  bucket = google_storage_bucket.test.name
}

```

---


