---
title: GCP - Filestore 概述

author: Aryido

date: 2024-07-24T19:05:55+08:00

thumbnailImage: "/images/google-cloud/filestore/filestore-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-storage-service

comment: false

reward: false
---

<!--BODY-->

> Filestore 是 Google Cloud 上完全託管的 Network FileSystem(簡稱 NFS)，目的讓不同的機器甚至是不同的作業系統都可以彼此分享檔案。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Elastic File System (EFS)**
> - Microsoft Azure : **Azure Files**
>
> Filestore 讀寫速度超級快，作為 file storage 並**支持併發同時訪問同一個檔案**，還提供其他不同種類的存儲類型一些可選替代方案，例如把 Filestore 作為一種「 Persistent Disk 的 block storage 存儲種類 」或者「 類似 Cloud Storage FUSE 的 object storage 存儲種類」等等，因此它適用連接整合到多種 GCP Client 端服務如：VM、GKE、Cloud Run 等等。

<!--more-->

---

一般來說 VM 的 Disk 要多寫多讀是做不到的。為解決此問題，當時在 Linux 上有 Network FileSystem 的產生，它可以將網路遠端的 NFS 伺服器分享的目錄，掛載到不同機器上。對於不同機器上自己看起來，那個遠端 NFS 主機的目錄就好像是自己的一個磁碟分割槽一樣。 **Filestore 用法跟 NFS 用法完全一樣**，只是託管給 GCP ， Google 還進一步將 NFS 和 Multi-Writer 功能統整，允許多個 User 在 Filestore 對共用檔案進行併發讀寫訪問，並支援 `NFSv3 protocol` 和新增最新的 `NFSv4.1 protocol`。
{{< alert info >}}
`NFSv4.1 protocol` 至 2024/7 還是 preview 版本
{{< /alert >}}
接下來看其中的設置：

# Service tiers

Filestore 對於 service tiers 也有蠻大的變動，在過去的 Legacy service tiers 有： Standard、Premium、High scale SSD、Enterprise，但現在 Console 上精簡成： Basic、Zonal、Regional。每個 tier 都提供不同的容量選項和性能，以下價錢均為 `2024-07-24` 當天擷取的畫面：

### Basic

Basic-tier 有分成 Basic-HDD 和 Basic-SSD ：

- Basic-HDD 取代舊的 **Standard**
  {{< image classes="fancybox fig-100" src="/images/google-cloud/filestore/basic-hdd.jpg" >}}
- Basic-SSD 取代舊的 **Premium**
  {{< image classes="fancybox fig-100" src="/images/google-cloud/filestore/basic-ssd.jpg" >}}

SSD 在 Read/Write throughput 及 IOPS 上比 HDD 表現好很多，兩個均適用於一般文件共享。Basic tier 在容量上比較有限制，只有 `1 ~ 63.9` TiB，若對於容量有更大的要求的話就只能考慮 Regional 、 Zonal。

{{< alert warning >}}
Basic-SSD 起跳是`2.5 ~ 63.9` TiB
{{< /alert >}}

### Zonal 、 Regional

Zonal/Regional tier 容量範圍選擇比 Basic-tier 寬廣不少，從 `1 ~ 100` TiB。

- Zonal (`10 ~ 100 TiB`) 取代舊的 **High scale SSD**
  {{< image classes="fancybox fig-100" src="/images/google-cloud/filestore/zonal.jpg" >}}
- Regional (`1 ~ 9.75 TiB`) 取代舊的 **Enterprise**
  {{< image classes="fancybox fig-100" src="/images/google-cloud/filestore/regional.jpg" >}}

Zonal 相比 Basic-SSD 和 Regional 有更便宜的價格，性能上也比 Basic-HDD 還要好上一些 ; Regional 基本上要注意的點和 Zonal 差不多，但提供比 Zonal 還要更好的平均性能，價錢上也比 Basic-SSD 還便宜一些。

{{< alert info >}}
Zonal 、 Regional 的**性能和容量有存在一些線性關係**，可藉由自行調整容量對應至需求的 Read/Write throughput 及 IOPS
{{< /alert >}}

# 省錢 Tip

所有使用過 Filestore 的人應該都有被那恐怖的費用驚嚇到，最讓人詬病的缺點就是容量低消至少要 `1 TiB` ，再加上是依照**創建時劃分的容量收費而非實際使用的容量**，所以換算下來最便宜 Basic-HDD 的方案一個月要破 5000 台幣，絕對是第一次使用 GCP 會踩到的坑...，以下針對一些使用情境來優化價格花費：

- ##### 使用 Basic-SSD 之前，一定先看看 Zonal 、 Regional 能不能先行滿足需求

- ##### Basic-HDD 要容量超過 10 TiB 才會有一個性能的跳躍，容量再往上升性能也不會增加，故可詳細比較 Zonal 、 Regional 的線性增量來選擇價格合理的方案

- ##### SSD 的性能是固定的，不會因為容量變大而增強，這和 Basic-HDD 不一樣，不要誤用

- ##### 對於檔案的 share，Google 也有推出 GCS FUSE，若不需要支持高併發讀寫，這個不錯的選擇，價格也比 Filestore 便宜。

{{< alert success >}}
特別注意 Basic-HDD 和 Basic-SSD 的價格差非常多。換到 Basic-SSD 前可以先看看 Regional、Zonal 能不能先符合需求
{{< /alert >}}

# 雲端儲存服務 Network FileSystem 的發展

相對於 Block Storage 與 Object Storage 類型的 cloud 儲存服務，File Storage 類型的 cloud 儲存服務相當晚才推出:

- 2006 AWS S3 - **Object Storage**
- 2008 AWS Elastic Block Store(EBS) - **Block Storage**
- 2015 Azure File Storage - **File Storage**
- 2016 AWS Elastic File System(EFS) - **File Storage**
- 2018 GCP Filestore - **File Storage**

雖然雲端的 File Storage 推出比較晚，但其實對於傳統的本地 IT 環境，基於 NFS/SMB 共享儲存應用如 NAS 與檔案伺服器設備的使用，都已經有超過 10 幾年以上的歷史了，例如大量的 NAS 都是被應用於這個目的。當年在公有雲 File Storage 服務尚未推出之前，因為雲端缺乏基於 NFS/SMB 傳輸架構的儲存服務這一點，也成了妨礙企業用戶將 IT 環境移植到公有雲的障礙之一，因此後續個家雲服務商陸續推出支援 NFS 的儲存服務，算解決了自建 File Storage 服務在可用性、穩定性、擴展性、管理與成本等方面的問題，提供完全托管、按需收費等特性。

{{< alert danger >}}
再次提醒，至少 GCP Filestore 使用體感，並不算是**按需收費**就是了...
{{< /alert >}}

---

### 參考資料

- [Filestore overview ](https://cloud.google.com/filestore/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [第十三章、檔案伺服器之一：NFS 伺服器](https://linux.vbird.org/linux_server/centos6/0330nfs.php)

- [【GCP 教學】在 GCP 挑選正確的儲存服務](https://mile.cloud/zh/resources/blog/google-cloud-platform-block%20storage_710)

- [【新興的公有雲平臺 NAS 檔案儲存服務】公有雲檔案儲存服務的型態與類型](https://www.ithome.com.tw/tech/141356)
