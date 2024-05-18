---
title: GCP Compute Engine 概述

author: Aryido

date: 2024-05-13T23:26:00+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp

comment: false

reward: false
---
<!--BODY-->
>  Compute Engine 是託管在 Google 雲端上基礎架構即服務 (IaaS) 產品，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) 中的 Amazon **EC2**
> - Microsoft Azure 中的 **Virtual Machine**
>
> Compute Engine 的其他的稱呼還有 **compute engine instance** 、 **virtual machine instance** 、 **VM instance**。 啟動前可以選擇自己需要的 Machine Type ，例如 CPU、memory、disk 等等；再來 Boot disk OS 也可自行選擇 Linux 、 Windows 等等操作系統；針對容器虛擬化，可能使用專門優化來運行容器的 Container-Optimized OS (COS) image 在虛擬機上啟動容器服務。最後關於**備份資料**，GCP 也有提供相應的服務來面對災難發生時的處理。

<!--more-->

---

# Regions and zones
Compute Engine 資源託管在全球多個位置，分成 Region 和 Zone。資料中心是 Region ，機房是 Zone，不同的 Zone 是真的實體隔離的機房。一個 Region 內通常會有多個 Zone。例如 :
- Region: `us-west1`
  - Zone: `us-west1-a`
  - Zone: `us-west1-b`
  - Zone: `us-west1-c`

{{< image classes="fancybox fig-100" src="/images/google-cloud/region-zone.jpg" >}}


在全世界各地都有資料中心，這樣做的目的有：
- 高可用(High Availability)，避免因為資料中心掛掉造成 VM 服務不可用
- 網路低延遲(Low latency)，盡量靠近 user 減少延遲時間
- FedRAMP法規政策，各國都希望自己國民政府資料能留在自己國家內

# 創建 GCP VM

### GCP VM 的 Machine Types
可以使用 predefined machine types 如下 :
- ***General Purpose (E2、N2、N2D、N1)*** : 一般性的使用。
- ***Memory Optimized (M2、M1)*** : 這通常是給需要大量記憶體的 DB 或分析工具使用。
- ***Compute Optimized (C2)*** : 需要高速運算的Application 例如 game server。


另外還也可以自己 custom machine types，會有一個 **Bar** 可以去訂製例如 3.75 GB 這種特殊的規格，**其它雲端似乎沒這麼彈性**。
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm-machine-custom.jpg" >}}


GCP 也提供先佔虛擬機器（preemptible VMs）。與一般 VM 相比，preemptible VM 享有價格優勢，但可用性會隨使用情況而變化，適用於具有容錯性且擁有大規模批次運算需求的工作負載。

###  Disk Type
每個 Compute Engine 實例都有一個 **Boot disk**，包含作業系統；當需要更大的儲存空間時，可以添加額外 **Additional disks**。無論是 **Boot disk** 還是 **Additional disks**，都有各式 Type 可以選擇 :
- ***Standard Persistent Disk***

  傳統硬碟，hard disk drives (HDD)，速度較慢但成本便宜。
- ***Balanced Disk***

  介於 HDD 跟 SSD 中間的 Disk ，沒有像 SSD 那麼貴，可是它又比傳統的 Disk 還快。
  {{< alert warning >}}
如果用 Windows 系統的話，建議選擇 Balanced Disk 以上，因為 Windows 檔案非常的多而且 從開機開始它就一直有大量的讀寫，所以最好不要選傳統硬碟，不然的話會太慢，最好選擇 Balanced Disk 以上的。
{{< /alert >}}

- ***Extreme Disk***

  專給高端資料庫工作負載來用的，但也帶來比較高的成本，效能差異其實就是主要 IOPS 讀取跟寫入速度上的差異，並可以預配目標 IOPS。


### OS 系統
使用 operating system (OS) images 為 VM 建立啟動磁碟，在 GCP 裡把 vm 啟動起來前的系統規格定義，簡單稱為 Image，有以下幾種 :
- ***public OS images***

  由 Google、開源社群、第三方供應商提供和維護。預設情況下，所有 Google Cloud 專案都可以存取這些作業系統映像來啟動 Linux 或 Windows 操作系統；甚至授權版本 Redhat。
- ***自己客製化的 private custom OS images***

  自訂映像常用在把特定 lib 或應用預先下載好，使用這個 image 建立 VM 後就可以直接使用所需工具了。以下列表都可以作為 source 來製作自己的 custom OS images :
   - 現存的 Disk
   - 現存的 映像
   - snapshot
   - 儲存在 Cloud Storage 中的映像建立
   - Virtual disk (VMDK、VHD)
  {{< image classes="fancybox fig-100" src="/images/google-cloud/vm-os-image-custom.jpg" >}}

- Container-Optimized OS (COS)

  還有一個容器優化作業系統，針對容器進行了最佳化，由 Google 基於開源 Chromium OS 專案維護。可以部署 Docker 容器在 VM 上，設定 VM 啟動 Docker 容器。

# Back up and Restore
萬一你的 VM 主機崩潰救不回來了，若有**備份**，就可以復原 VM 主機。 GCP VM 也提供 **back up 備份 disk** 和 **replicate 複製  VM** 的方式去保護你的資料 :

| Scenarios                | Machine image | Persistent disk snapshot |
|--------------------------|---------------|---------------------------|
| Single disk backup       | Yes           | Yes                       |
| Multiple disk backup     | Yes           | No                        |
| Differential backup      | Yes           | Yes                       |
| Instance cloning         | Yes           | No                        |
| Base image for replication | No            | No                        |


### Snapshot
Snapshot 是基於 Persistent disk ，反映在具體的時間點上 disk 的內容。 2019/02 GCP 進一步推出了 Snapshot Cchedule 的功能，可以自動排程建立 Snapshot ，會依照**差異**來備份的，不會佔用大量儲存空間；也可以設定排程自動刪除舊的 Snapshot。
- 適用於備份和災難恢復
- 成本低於圖像，比圖像小，因為不包含操作系統等

### Machine Image
Machine Image 是基於整台 VM ，是一個新的 Compute Engine 資源，包含了該台 VM 的所有Disk(Boot Disk與其他Data Disk)、Configuration、Metadata、Permissions ，故適用於
- 多個磁碟備份
- clone VM instance

{{< alert warning >}}
Machine Image 包含了該台 VM 的所有Disk、Configuration、Metadata、Permissions。但 OS Image 只有 Boot Disk。
{{< /alert >}}


---

### 參考資料
- [Virtual machine instances](https://cloud.google.com/compute/docs/instances#instances_and_networks)

- [Google Cloud雲端平台介紹](https://jason-kao-blog.medium.com/google-cloud%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%B4%B9-fc3212c8359b)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [disk vs snapshot](https://stackoverflow.com/questions/27290731/google-compute-engine-what-is-the-difference-between-disk-snapshot-and-disk-ima)

- [Machine image VS Image](https://jason-kao-blog.medium.com/google-cloud-%E8%B3%87%E6%96%99%E5%84%B2%E5%AD%98%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-55dad31811e0)
