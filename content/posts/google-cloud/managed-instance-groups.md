---
title: GCP - Managed instance groups 概述

author: Aryido

date: 2024-06-16T22:54:22+08:00

thumbnailImage: "/images/google-cloud/lb/lb-logo.jpg"

categories:
- cloud
- gcp

tags:
- virtual-machine
- load-balancer

comment: false

reward: false
---
<!--BODY-->
> Cloud 是最能展現**自動伸縮擴展服務**功能的平台，而 GCP 的 Autoscaling Groups of Instances 代表產品是 Managed Instance Groups (簡稱 MIGs) ，雖然名稱有一點點不太直覺。 GCP 會根據自訂義 Autoscaling Policy 來自動添加或刪除 VM ，這些自動縮放而產生的 VM 會有一個**群組**來管體，就是 MIG。對應其他的雲端服務是 :
> - AWS : **Auto Scaling groups**
> - Azure : **Virtual Machine Scale Set** 
>
> MIGs 的 Autoscaling Policy 能夠基於 Application 的 CPU/Memory 使用率、網路流量等等設定，自動增加或減少資源，根據業務需求靈活調整資源數量從而保證**高性能**和**成本彈性**。
<!--more-->

---

在 GCP 中，Instance Groups 是一個用於管理 VM instance 的集合，分成 Managed（受管理）或 Unmanaged（非受管理）的，這兩種類型各有其用途：

# Managed Instance Groups

每次 Autoscale 的時候，都會去參考 Instance-Template，故這是必要的選項。再來依照 Instance-Template 裡面的 image 確定 disk OS ; machine type 確定硬體規格等等。
{{< alert success >}}
很自然地，若有 MIG 正在使用 instance-template，則 instance-template 就無法被刪除
{{< /alert >}}

大部份的設定在先前建立 Instance-Template 時都已經完成了，接下來 Managed Instance Group 基本只要設定 Name 、 Location 以及 Autoscaling :

### Location
有分成 Single zone 和 Multiple zones ，若設定成 Multiple zones，則不須擔心當其中一機房掛掉時，服務就掛掉了，以實現更高的可用性。


### Autoscaling 自動縮放
也稱作 「Autoscaling Policy」 ，可根據設定自動增加或減少 VM instance 的數量，還可以設置最小和最大 instance 數量以確保上下限。
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/autoscaling.jpg" >}}

Autoscaling Policy 至少要有一個 「Autoscaling signals (縮放信號)」。常用的 signals 指標有：
- CPU utilization  — Defult 的 `60%` CPU utilization
- HTTP load balancing utilization
- Cloud Monitoring metrics
- Cloud Pub/Sub queue 


還可以進一步設定 **Initialization Period**（舊稱呼是 cool-down period），可以設定等待多少時間後，才再根據指標決定 VM 是否繼續增加或銷毀。另外還有 **Scale-in Controls** 選項讓 VM 數量短時間不要下降太快，例如在 10 分鐘內降幅不要高於10%，或是 5 分鐘內最多不可以關超過 3 台 VM 。

### Auto Healing 自動修復 
主要是會需要 health-check 來偵測 VM 的狀態，而在 health-check 內可以設定等待多久之後才檢查，讓應用程式可以完成初始化。 如果 health-check 發現 VM instance 出現問題，MIG 會自動替換它們，以維持正常運行的 VM 數量，確保應用程序高可用性和其高性能。

{{< alert warning >}}
雖然名稱說是 Auto Healing 自動修復，但其實是把不健康的資源砍掉，然後再新建一個資源出來，和我們正常理解的修復不太一樣。
{{< /alert >}}

### 更新策略（Updates strategy）
到目標 Managed Instance Group 內，最上面有 Update VMs 選項，在裡面可以設定新的 Instance-Template，這樣就可以產生新的 VM ，同時可以簡單設定更新的策略。  MIG 提供 zero downtime 來發布新的 Application 版本，當 Application 有 update/upgrade 時，通過 rolling updates 逐漸將新實例添加到實例群組中。
{{< alert success >}}
除了 rolling updates，還有 Canary Deployment 也有支援。
{{< /alert >}}


# Unmanaged Instance Groups
Unmanaged Instance Groups（UIG）**不提供自動縮放、自動修復**功能。 Unmanaged 的意思其實是 **GCP 不控管這個群體的狀態**，僅僅只是幫忙劃分個群體類別，並附上和 VPC 和 Port mapping 等網路通訊功能，把狀態管理完全交由使用者。 故 UIG 可以自由地配置不同的 VM instance ，主要用於需要使用現有已經存在的一些 VM。

---

# Instance Groups 結合 Load-Balancer

上述說明了兩種類型的 VM instance groups：

- **Managed instance groups (MIGs)** : 使用 instance-template 建立的 VM，功能有 Auto scaling、 auto healing 等等，只有 MIG 有 Autoscaling 功能。
- **Unmanaged instance groups** : Unmanaged Instance Groups 沒有提供 Auto scaling 功能，僅只是把一些 VM 資源聚在一起方便管理，可能相關性不大，可能會有不同 machine type、不同 image。


雖然 Unmanaged instance groups 看起來有點沒作用，但是還是把這一些不能隨意 auto-scale 的 instances 聚在一起成一個 Instance Group ，其中一個很大的原因是可以與 Google Cloud 的 Load-Balancer 結合使用。 

Load-Balancer 控制分流到 instance 使用的抽象介面是 instance group，故有 UMIG 後就不是綁定在 MIGs 上，可以把一些特別的定義的 instances group 起來給 Load-Balancer。

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/mig.jpg" >}}

---

### 參考資料

- [Autoscaling groups of instances](https://cloud.google.com/compute/docs/autoscaler)

- [Google Cloud雲端平台介紹](https://jason-kao-blog.medium.com/google-cloud%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%B4%B9-fc3212c8359b)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Day 05：Instance Groups](https://ithelp.ithome.com.tw/m/articles/10315523)