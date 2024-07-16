---
title: GCP - Compute Engine 概述

author: Aryido

date: 2024-05-23T23:26:00+08:00

thumbnailImage: "/images/google-cloud/vm/vm-logo.jpg"

categories:
- cloud
- gcp

tags:
- virtual-machine
- gcp-virtual-machine

comment: false

reward: false
---
<!--BODY-->
>  Compute Engine 是託管在 Google 雲端上基礎架構即服務 (IaaS) 產品，其他的稱呼還有 **compute engine instance** 、 **virtual machine instance** 、 **VM instance**。 對應其他的雲端服務是 :
> - Amazon Web Services (AWS) : **EC2**
> - Microsoft Azure : **Virtual Machine**
>
> 啟動前可以訂製自己需要的 Machine Type ，例如 CPU、memory、disk 等等；再來 Boot disk OS 也可自行選擇 Linux 、 Windows 等等操作系統；針對容器虛擬化，可能使用專門優化來運行容器的 Container-Optimized OS (COS) image 在虛擬機上啟動容器服務。最後關於**備份資料**，GCP 也有提供相應的服務來面對災難發生時的處理。

<!--more-->

---

# Regions and zones
Compute Engine 資源託管在全球多個位置，分成 Region 和 Zone。資料中心是 Region ，機房是 Zone，不同的 Zone 是真的實體隔離的機房。一個 Region 內通常會有多個 Zone。例如 :
- Region: `us-west1`
  - Zone: `us-west1-a`
  - Zone: `us-west1-b`
  - Zone: `us-west1-c`

{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/region-zone.jpg" >}}


在全世界各地都有資料中心，提供了：
- 高可用(High Availability)，避免因為資料中心掛掉造成 VM 服務不可用
- 網路低延遲(Low latency)，盡量靠近 user 減少延遲時間
- FedRAMP法規政策，各國都希望自己國民政府資料能留在自己國家內

# 創建 GCP VM

雲端資源的一大特性就是，用多少算多少，但還是要注意就算暫時關機 VM，**還是會依照 Disk 跟 static ip 等等佔用而需要收取費用**。GCP 還提供 Spot VM，與一般 VM 相比， Spot VM 有更低的價格`(25.46 >> 9.44)`，可以設定幾小時之後停止或刪除 VM，可根據需求選擇此功能。

### GCP VM 的 Machine Types
可以使用 preset machine types 如下 :
- ***General Purpose (E2、N2、N2D、N1)*** : 一般性的使用。
- ***Compute Optimized (C2)*** : 需要高速運算的Application 例如 game server。
- ***Memory Optimized (M2、M1)*** : 這通常是給需要大量記憶體的 DB 或分析工具使用。

另外還可以自己 custom machine types，會有一個 **Bar** 可以去訂製例如 22GB Memory 這種特殊的規格，算是 GCP 特有的便利功能，**其它雲端似乎沒這麼彈性**。
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/vm-machine-custom.jpg" >}}

###  Disk Type
每個 Compute Engine 實例都有一個 **Boot-Disk**，此 disk 主要會包含作業系統，當需要更大的儲存空間時，可以添加額外 Disk 簡稱 Data-Disk。無論是 **Boot-Disk** 還是 **Data-Disk**，都有各式 Type 可以選擇 :
- ***Standard Persistent Disk***

  傳統硬碟，hard disk drives (HDD)，速度較慢但成本便宜。
- ***Balanced Disk***

  介於 HDD 跟 SSD 中間的 Disk ，沒有像 SSD 那麼貴，可是它又比傳統的 Disk 還快。
  {{< alert warning >}}
如果用 Windows 系統的話，建議選擇 Balanced Disk 以上，因為 Windows 檔案非常的多而且從開機開始它就一直有大量的讀寫，所以最好不要選傳統硬碟，不然的話會太慢，最好選擇 Balanced Disk 以上的。
{{< /alert >}}

- ***Extreme Disk***

  專給高端資料庫工作負載來用的，但也帶來比較高的成本，效能差異其實就是主要 IOPS 讀取跟寫入速度上的差異，並可以預配目標 IOPS。


Boot-Disk 和 Data-DiskDisk 最主要的差別，可以點進去看其內容資訊有沒有 **Source** 這個參數，Boot-Disk 會有 **Source** 參數代表使用什麼作業系統。另外使用 Disk 上還有一些事情可以注意一下 : 
- Disk **可以增大，但不能縮小**
- 可以設立快照排程備份 Disk，預設保留 14 天 （後續會再簡單介紹）
- 還可以設定 Deletion rule，會在刪除 VM 時決定要不要順便刪除 Disk 。

{{< alert info >}}
影響 IOPS 的要素有： 機器 core 數量、vcpu 變化、Disk 大小
{{< /alert >}}


### OS 系統
承前面介紹的 Disk ，其內部設定會需要選擇 Image ，這個就是代表著要選擇安裝什麼 OS 系統，在 GCP 裡把 VM 啟動起來前的作業系統規格定義，稱為 operating system images(OS Images) 或者 Source Image，**更簡單會略稱為 Image**。 選擇一個 Image 來建立 VM Boot-Disk，有以下幾種類型的 Images 可以選擇 :

- ***Public OS images***

  由 Google、開源社群、第三方供應商提供和維護。預設情況下，所有 Google Cloud 專案都可以存取這些作業系統映像來啟動 Linux 或 Windows 操作系統；甚至授權版本 Redhat。

- ***Private Custom OS images***

  自己客製化 private custom OS images 的原因，是還可以把自己業務所需特定 lib 或 application 預先下載好，因此使用這個 image 建立 VM 後可以直接使用已載好的應用。以下列表都可以作為 source 來製作自己的 custom OS images :
  - - 現存的 Disk
  - - 現存的另一個 image
  - - snapshot
  - - 儲存在 Cloud Storage 中的映像建立
  - - Virtual disk (VMDK、VHD)

{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/vm-os-image-custom.jpg" >}}


- ***Container-Optimized OS (COS)***

  此 OS 是特別為了 Container 優化作業系統，由 Google 基於開源 Chromium OS 專案維護。可以部署 Docker 容器在 VM 上 並設定 VM 啟動 Docker 容器。

{{< alert warning >}}
OS Image 概念上是一開始就把需要的 Application 與設定預先做好 ; 相反地 Startup Script 通常是 VM 在啟動後，才自動安裝或設定我們想要應用。
{{< /alert >}}

---

# Back up and Restore
承上說明 disk 時，有提到 Boot-Disk 和另外開的一個 Data-Disk，此概念就和備份息息相關，因為如果資料和啟動系統放在一起， VM 出問題開不起來會比較麻煩 ; 但如果一開始就有把資料分開的話，VM 出問題後可以直接開一台全新的，再把 Data-Disk 掛到新的 VM 就可以了。 VM 主機崩潰救不回來，經常會依靠**備份**來復原資料，有 **replicate 複製 VM** 的 Machine Image 方式，和 **back up 備份 disk** 的 snapshot 功能 :

| Scenarios                | Machine Image | Snapshot |
|--------------------------|---------------|---------------------------|
| Single disk backup       | Yes           | Yes                       |
| Multiple disk backup     | Yes           | No                        |
| Differential backup      | Yes           | Yes                       |
| Instance cloning         | Yes           | No                        |
| Base image for replication | No            | No                        |

### Machine Image
Machine Image 是**基於整台 VM** ，為一個新的雲端資源，包含了該台 VM 的所有 Disks (包含 Boot Disk 與其他 Data Disks)、Configuration、Metadata、Permissions ，故適用於
- 多個磁碟備份
- **Clone** VM instance

### Snapshot
Snapshot 是基於 Persistent disk ，反映在具體的時間點上 disk 的內容。 2019/02 GCP 進一步推出了 Snapshot Cchedule 的功能，可以自動排程建立 Snapshot ，會依照**差異**來備份的，不會佔用大量儲存空間；也可以設定排程自動刪除舊的 Snapshot。
- **適用於備份和災難恢復**
- 成本低於圖像，比圖像小，因為不包含操作系統等

{{< alert warning >}}
Machine Image 包含了該台 VM 的所有Disk、Configuration、Metadata、Permissions。但 snapshot 只有備份 Boot Disk。
{{< /alert >}}

---

# VM lifecycle
建立好 VM 之後，運行時會有各種不同狀態：
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/vm-lifecycle.jpg" >}}

---

### 參考資料
- [Virtual machine instances](https://cloud.google.com/compute/docs/instances#instances_and_networks)

- [Google Cloud雲端平台介紹](https://jason-kao-blog.medium.com/google-cloud%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%B4%B9-fc3212c8359b)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Google Cloud — 資料的儲存與處理服務](https://jason-kao-blog.medium.com/google-cloud-%E8%B3%87%E6%96%99%E5%84%B2%E5%AD%98%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-55dad31811e0)