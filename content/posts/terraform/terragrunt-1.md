---
title: Terragrunt - Introduce

author: Aryido

date: 2022-09-27T20:45:15+08:00

thumbnailImage: "/images/terraform/terragrunt.jpg"

categories:
- terraform

tags:
- terragrunt

comment: false

reward: false
---
<!--BODY-->

> Terragrunt 是 gruntwork 推出的一個 Terraform thin wrapper，在執行 Terraform 前可以先**調整** root module 內的 .tf 檔案，保持程式碼的精簡，並提供許多額外的工具和框架幫助開發，藉此可以讓你的 IaC code 更貼近 DRY 原則。
<!--more-->

---
# Terragrunt 管理基礎設施

Terraform 本質上只是一個**配置文件管理工具**，沒有函數復用的功能，導致了不得不用重複的方式去解決擴展性問題。

{{< image classes="fancybox fig-100" src="/images/terraform/terraform-wrapper.jpg" >}}

1. 管理 remote state 設定
2. 管理 backend Storage Bucket
3. 管理 module 之間的相依性
4. 保持良好的開發原則

{{< alert info >}}
例如 Don’t Repeat Youself 是軟體工程良好的開發原則之一，雖違反 DRY 不一定代表不好，在某些情形工程師可能會選擇更好的可讀性，而犧牲 DRY。
{{< /alert >}}

---

隨著使用的 module 越多，開始會發現許多重複的部分例如：

## - provider.tf
每個環境的 root module 都存在，而且內容幾乎一樣

## - backend 管理 state
可能只有不同環境下 state 的資料夾路徑不太一樣

## - variables.tf
例如 gcp 許多 module 都需要傳入重複的參數像是 project-id 、 location 等等

接下來幾篇，會開始實作 code 重複部分，如何用 Terragrunt 優化。

---
{{< alert success >}}
***Keep your terraform configuration DRY***

這個是 Terragrunt 一開始的核心理念
{{< /alert  >}}

---
