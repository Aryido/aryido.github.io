---
title: Terragrunt - 整理

author: Aryido

date: 2022-10-06T22:31:20+08:00

thumbnailImage: "/images/terraform/terragrunt.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->

> 基本上 terragrunt 的使用和 terraform 都一樣，所以才說 terragrunt 是一層 wrapper。
> ```bash
> terragrunt init
> terragrunt plan
> terragrunt apply
> ```
> 和 terraform 都一樣對吧 !
<!--more-->

---

# Pros & Cons
各大雲端供應商目前都有在 Github 上發布了許多常用的 modules ，可以直接使用，省下開發維護 Terraform code 的成本。但強迫使用 terragrunt 並實作 Terraform module 也增加了開發與維護的時間成本。

比起聲明一個特定的 resource，當你要實作一個可重複使用的 Terraform module，需要考慮的 input/output 以及 resource 的參數處理上勢必會複雜許多。算就使用官方提供的 module 其實看懂也是要花一些時間的。

## Pros

- DRY程式碼
- runtime 注入變數
- 在 terraform 的 lifecycle 之前與之後，可以 hook ，執行額外的程式

## Cons
- 程式碼會變得有點複雜
- 需要額外注意 terragrunt 與 terraform 之間的 lifecycle

---

#### 小提醒

{{< alert warning >}}
本地開發時 module，發生錯誤，可以先把本地 cache 清除再重新 init

```bash=
rm -rf .terragrunt-cache

terragrunt init
```

{{< /alert  >}}

---
