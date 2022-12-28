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
> 雲端資源在使用的時候，可以特別注意收費的部分。 例如 GCP Filestore 和 AWS EFS 都是有關於 file share 的功能，但計費方式卻很不一樣。養成沒事看看雲端 billing 可以幫助止血...，踩完坑之後就來看看付費公式吧 !

<!--more-->

---

現在各大雲平台供應商，都有提供價錢試算，例如:
- [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator#id=5e256ac4-6a8a-4ffd-ae33-3dd379fe3cef)
- [AWS Pricing Calculator](https://calculator.aws/#/)

GCP console 在建立資源，旁邊也會有個畫面，貼心的給你看大概價錢花費，這部分比 AWS 還要方便點。

---

#  Amazon Elastic File System (Amazon EFS)
EFS 是一種 serverless檔案系統。**沒有最低預設費用、也沒有基礎設定費用。**

EFS 提供四種類別：
- 標準儲存類別
    - Amazon EFS Standard
    - Amazon EFS Standard-Infrequent Access(EFS Standard-IA)
- 單區域儲存類別
    - Amazon EFS One Zone
    - Amazon EFS One Zone-Infrequent Access(EFS One Zone-IA).

明顯可知存在單一區域一定比較便宜，但多 zone 的話，資料保存更加安全，但費用就會需要高一些。

| 區域	| 有效儲存架構 (每月每GB USD) – one-Zone  | 有效儲存架構 (每月每GB USD) – 標準
|-------------|:------------:|:------------:|
| 美國東部 (維吉尼亞北部)	| 0.043 USD  |	0.08 USD

---
# 費用計算
1. 有效儲存架構 (每月每GB USD) – one-Zone *0.043 USD*
2. 有效儲存架構 (每月每GB USD) – 標準 0.08 USD

是怎麼算出來的呢 ? 現在來算一次給你看 ! 首先參考以下表格:

{{< image classes="fancybox fig-100" src="/images/aws/efs-price.jpg" >}}

加上 AWS 有研究，平均而言，經常使用 20％ 的檔案，80％ 的檔案則不常存取。故以下根據US East (N. Virginia) 的 One Zone pricing)。

{{< alert info >}}
AWS 計算費用：

- Total Monthly Storage Required (GB). Ex. 1024 GB

- Percentage of files that are frequently accessed. Ex. 20%

{{< /alert >}}

Effective storage cost = (Storage cost for frequently accessed data + Storage cost for infrequently accessed data)/Total storage

```
[(1024 * 20%) * ($0.16)] + [(1024 * 80%)] * ($0.0133) = $43.663

$43.663 / 1024 = 0.04263964843 ~= $0.043/GB
```
這就是**one-Zone**價錢的算法。

若是**標準儲存**，則同樣使用80/20 法則:
```
[(1024 * 20%) * ($0.3)] + [(1024 * 80%)] * ($0.025) = $81.92

$81.92 / 1024 = $0.08/GB
```
{{< alert warning >}}
要注意若沒有 80/20 法則，都沒用不常存取的概念的話:

標準儲存 : **1024 * $0.3 = $307.2**

one-Zone儲存 : **1024 * $0.16 = $163.84**

{{< /alert >}}

---

#  GCP Filestore
Filestore 是一檔案儲存服務。為需要高吞吐量、低延遲和高 IOPS 的應用程式提供非常高的效能。 GCP 沒有 20% 常用 80% 不常用的概念，是 100% 常用再算費用的。

**GCP 會在建立時就開始產生 Filestore 費用**，這個和 AWS EFS 放在那邊，有用才算費用的計費方式很不一樣。 Filestore 依據容量 (以 GiB 為單位)收取費用，此費用是以秒為單位累計，刪除時就會停止。 Filestore 的費用 (無條件進位至最接近的秒數)。

{{< alert info >}}
GCP 計算費用：

- Service tier：
    - Basic HDD (Standard)
    - Basic SSD (Premium)
    - Enterprise, or High Scale SSD.

- Region: VM的建立位置

- Instance capacity:
    **必須為分配到的儲存空間容量支付費用，即便未使用也一樣。** ~~神坑~~

舉例來說，如果建立了 1 TiB 的執行個體，並在該ＶＭ中儲存 100 GiB 的資料，系統還是會向收取 1 TiB 儲存空間的費用。

{{< /alert >}}

另外如果流量輸入輸出 Filestore 與 VM 位於**相同** region ，則不會產生費用。若 region 不同，從 Filestore 輸出的流量就會產生費用。

us-west1-b 的 1 TiB 基本硬碟級費用:

- The unit cost for a Basic HDD tier instance in the Oregon region is **$0.000274 GiB / hr**

- The hourly cost for the instance is  **1024 * $0.000274 * 24 * 31 = $208.32**

---
# 結論

簡單看起來若都是 1TB 的話:
- AWS 標準儲存 : **$307.2**
- AWS one-Zone儲存 : **$163.84**
- GCP : **$208.32**

{{< alert danger >}}
**AWS 是使用多少算多少。**

**GCP 是規劃一塊空間給你，然後每秒算錢。**

所以在測試上，AWS會比較安全和方便，GCP容易忘記關資源而導致多付費。用多少算多少比較符合我們對雲端資源的想像，這邊感覺 GCP 做得不好。
{{< /alert >}}

{{< alert warning >}}
GCP 有點強迫，最小硬碟都要選 **1T**
{{< /alert >}}

{{< alert warning >}}
AWS 有 80/20 法則，規劃起來比 GCP 靈活很多。 但如果業務上都是經常取用且會跨region的話，GCP會便宜一些。
{{< /alert >}}

---

