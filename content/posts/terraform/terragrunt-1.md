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

> Terragrunt 是 gruntwork 推出的一個 Terraform thin wrapper，在執行 Terraform 前可以先**調整** root module 內的 .tf 檔案，保持程式碼的精簡，並提供許多額外的工具和框架幫助開發...
<!--more-->

---
Don’t Repeat Youself 是軟體工程開發的基本原則之一，雖違反 DRY 不一定代表不好，在某些情形工程師可能會選擇更好的可讀性，而犧牲 DRY。在這裡使用 Terragrunt 可以讓 IaC code 更貼近 DRY 原則。

---

隨著使用的 module 越多，開始會發現許多重複的部分例如：

## - provider.tf
每個環境的 root module 都存在，而且內容幾乎一樣

## - backend 管理 state
可能只有不同環境下 state 的資料夾路徑不太一樣

## - variables.tf
例如 gcp 許多 module 都需要傳入重複的參數像是 project-id 、 location 等等

---
{{< alert info >}}
***Keep your terraform configuration DRY***

這個是 Terragrunt 一開始的核心理念
{{< /alert  >}}





---
