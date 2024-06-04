---
title: AWS 與 GCP reliability 不同的地方比較 - 1

author: Aryido

date: 2022-09-24T19:31:37+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud
- common

tags:
- virtual machine 


comment: false

reward: false
---
<!--BODY-->
> 近期有機會來比較一下 AWS 和  GCP 的一些差別，也看了一些文章(~~練英文QQ~~)。 GCP 和 AWS 都有 auto scaling 的功能，當我們在某些時候，需要比較多的資源處理事情時，可以自動增加機器來維持高 reliability。 那這部分 GCP 和 AWS 有甚麼區別呢 ?

<!--more-->
# 簡介
Cloud compute 通常被視為一種 ethereal resource ，即可以自由啟動並關閉，然後按秒計費。 概念上資源看起來是無限的，這也是與 on-prem 相比的優勢和賣點之一。 **可以根據 loading 進行 scale，且不需要的時候可以移除。**

但實際上資源並不是無限的。 Cloud 還是一個在真實世界的機房，故機房內的 CPU 和 GPU 還是有其上限數量。因此 **當我們需要spin up VM 時，還是可能會有 resource availability issues**。還是可能在需要時，機器不夠用...

# Benchmarking Harness
在 gcp 和 aws 上配置需要的 GPU 資源，然後在一天中的隨機時間啟動 GPU，更進一步設計一天中有不同時段 loading 來模擬真實世界狀況。

在兩週的時間裡，大約啟動了了 3,000 個 T4 GPU。
- y 軸正值: 表示成功啟動 VM 所需的持續時間
- y 軸負值: 表超過 200 秒還沒啟動的就丟到這裡

{{< image classes="fancybox fig-100" src="/images/google-cloud/aws-vs-gcp-reliability-1.jpg" >}}

## 測試結果
**The results are pretty staggering !!**  AWS 很平均的約 11.4 秒生成一個新的 GPU。 但 GCP 平均卻要 42.6 秒...

#### AWS 在啟動時間上比 GCP 快 3.73 倍 !

進一步考慮一下兩家 cloud vendors 的差異:
- GCP 可根據需要，配置 CPU 的數量，且可附加 GPU 到任意 VM 作為hardware accelerator 來加速。
- AWS 只能預設已附加 GPU 的 VM

先不考慮那些超時200秒的部分(下次再討論)，看起來 GCP 無法快速的創建 VM。故:

---

# 結論
{{< alert warning >}}
####  - 如果要求是，在需要時，要快速啟用 VM 的情境的話，AWS 似乎是比較好的選擇。
####  - 如果願意等待比較久的時間，那 Google 因為可客製 hardware accelerator 服務，彈性比較高，是可以考慮的。
{{< /alert >}}

---

# Vocabulary
{{< alert info >}}
**ethereal**[ɪˋθɪrɪəl] : 注意念法

adj. 如空氣般輕的；飄逸的；精緻的；縹緲的；天上的；非人間的

{{< /alert >}}

{{< alert info >}}
**launch**[lɔ:ntʃ] : 注意念法，不要念成午餐

**spin up**

軟體工程師的範圍，以上常當作 *開機*

{{< /alert >}}

{{< alert info >}}
**on-prem**: 注意念法
**premises**[ˋprɛmɪsɪz] :

軟體工程師的範圍，以上常當作 cloud 的反義字，概念上是 local
{{< /alert >}}

{{< alert info >}}
**excess**[ɛkˋsɛs]: 注意念法，不要念成 exist

n.超越

adj.過量的
{{< /alert >}}

{{< alert info >}}
**benchmarking harness**:

n. 基準測試框架
{{< /alert >}}

{{< alert info >}}
**staggering**[ˋstægərɪŋ]:

adj.搖晃欲倒的；驚人的
{{< /alert >}}

# Reference
https://freeman.vc/notes/aws-vs-gcp-reliability-is-wildly-different
---