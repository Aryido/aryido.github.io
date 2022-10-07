---
title: GCP Filestore vs AWS EFS 收費標準

author: Aryido

date: 2022-10-07T22:03:14+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud

tags:
- gcp
- aws

comment: false

reward: false
---
<!--BODY-->
> 對於初學者，雲端資源在使用的時候，真的要注意收費的部分。 GCP Filestore 和 AWS EFS 的計費方式真的很不一樣，養成沒事看看雲端 billing 可以幫助止血...，採完坑之後就來研究付費算法吧 ! (未完成)

<!--more-->

---

現在各大雲平台供應商，都有提供價錢試算，例如 [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator#id=5e256ac4-6a8a-4ffd-ae33-3dd379fe3cef) 或者 [AWS Pricing Calculator
](https://calculator.aws/#/)，GCP 在 console 還有貼心顯示每個資源大概會花多少錢的Summary ~~(但還是改不了 GCP 會讓人採坑的事實...)~~

---

#  GCP Filestore
GCP 好像沒有 20%常用 80% 不常用的概念


---

#  AWS EFS
AWS算法是

- Total Monthly Storage Required (GB). Ex. 500 GB
- Percentage of files that are frequently accessed. Ex. 20%

Effective storage cost = (Storage cost for frequently accessed data + Storage cost for infrequently accessed data)/Total storage

Example: Effective storage cost = (500 x 20%) x ($0.16) + (500 x 80%) x ($0.0133) / 500 =
= ($16 + $5.32)/500
= $0.043/GB-Month

---

# 結論
{{< alert warning >}}

{{< /alert >}}

{{< alert warning >}}

{{< /alert >}}

---

---