---
title: GCP - Instance Template 概述

author: Aryido

date: 2024-05-24T19:26:00+08:00

thumbnailImage: "/images/google-cloud/vm/vm-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-virtual-machine

comment: false

reward: false
---

<!--BODY-->

> Instance Template 是一個用於定義 GCP VM 配置的模板，其中包括如 machine type、bootdisk、startup script 等等 VM 的屬性，經常和 [Managed Instance Groups](https://aryido.github.io/posts/google-cloud/managed-instance-groups/) 結合使用來自動創建 VM 。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Launch Template**
> - Microsoft Azure : **沒有直接類似的，在 Virtual Machine Scale Sets 內直接設定 VM 所需參數**
>
> 概念上就是做好 VM 的模板，讓 VM 啟動完成後直接就可以達到我們想要的狀態，從這個方向出發的話 Instance Template 其實和 Startup Script、Custom Image 都蠻類似的，等等都會介紹和比較一下。

<!--more-->

---

# Startup Script

Startup Script 是 VM 在**啟動過程中**執行任務的腳本，可以用來自動安裝一些應用程式，這樣就不用自己再手動去做。其腳本是會放到 Metadata Server 上，其對應的 Metadata key 有兩種：

- `startup-script` : 注意不能超過 256 KB
- `startup-script-url` : 可以使用 cloud storage URI 如 `gs://BUCKET/FILE`

{{< alert success >}}
每個 VM 都是把其 metadata 存儲在 Metadata Server 上， VM 可以訪問 Metadata Server API ，不需要任何額外授權認證的，以 key-value pairs 方式儲存其屬性。
{{< /alert >}}

Startup Script 通常是 VM 在啟動後，才自動安裝我們想要的 Application ; 而 Custom Image 則是一開始把這些預先做好。雖然兩個作用有點相似，但在**啟動的時間點是兩個極端**。 對於 VM 來說，無論是 Startup Script 還是 Custom Image 都是 launch 一台 VM 的一種設定參數。

{{< alert danger >}}
特別注意，GCP Startup Script 會在**每次重新啟動 VM 時被觸發**。
{{< /alert >}}

# Instance Template

Boot-Disk Image 會放到 Instance Template 裡面 ; 而 Startup Script 會看需求才決定是否設定。Instance Template 經常會用來建立一個單獨的 VM，或者配合使用 Managed Instance Group(MIG) 來建立 VM ，這樣就不必每次都手動配置，只需根據模板創建新實例 VM，但有一些需要注意的事項 :

> - Instance Template 的目的是建立具有**相同配置**的 VM，因此 Instance Template 是**無法更新的**，如果真的需要變更配置，只能建立新的 Instance Template。

> - 還是由於 Instance Template 的目的是建立具有**相同配置**的 VM，所以建議若有使用 startup script 安裝應用，一定要記得**指定 version**。

{{< alert info >}}
Instance Template 不會被 GCP 收取任何費用
{{< /alert >}}


不難發現 Instance Template 能夠設定的東西基本上和 VM 是完全一樣的，故也不難想像為什麼 Azure 並沒有特地把機器設定的 Template 從 Virtual Machine Scale Sets 中拉出來，大概就是覺得有點多餘吧...



---

# 比較 Machine Image 、 Snapshot 、 Custom Image 、 Instance Template 之間差別

承之前 Compute Engine 的介紹，在加上本篇描述，會發現其實有非常多種方式把 VM 啟動起來。
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/backup.jpg" >}}

### Machine Image

是基於整台 VM 的，從 UI 設計也可以看得出來，我們可以針對 VM Instance 來看可以執行的動作，會發現有 `create new machine image` 的選項。一個 VM 可能會包含多個 disks，而 Machine Image **也是唯一可以一次性備份多個 disk 的方案**。
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/machine-image.jpg" >}}
{{< alert success >}}
目前自己看起來，使用 Machine Image 來作為 migration 遷移也是比較好的選擇，因為本質上最接近 VM 的 clone ，連 startup script 和 network tag 都會一起保存。
{{< /alert >}}

### Snapshot

是基於單一 disk 的，從頁面上要到 disk page ，再來針對某一個指定 disk 進行操作，概念上就是針對災難或意外情況的備份，重要的功能是**可以進行排程**，設定每隔多久要進行 backup。
{{< image classes="fancybox fig-100" src="/images/google-cloud/vm/snapshot.jpg" >}}

### Custom Image

也是基於單一 disk 的。目前自己看起來， Custom Image 現在只當作第一次啟動 VM 需要的作業系統定義檔會比較好。雖然也可以，但感覺已經不太適合用作備份概念了，因為
- 快速輕量的 backup 比不上 snapshot
- 方便完整性也沒有 Machine Image 好

{{< alert success >}}
目前自己看起來，感覺 Machine Image 就是來取代 Custom Image 在備份方面的不方便的種種方面。因為 Machine Image 包含了該台 VM 的所有 Disk、Configuration、Metadata、Permissions ; 但 Custom Image 只有 Boot Disk 部分。

或許使用 Machine Image 來做為大版本升級前的 backup 會是比較優的選擇。
{{< /alert >}}

### Instance Template

如果從備份角度來說的，Instance Template **完全不屬於備份用途**。偏向會和 Instance Group 緊密結合來做 auto-scale，目標是建立全新的 VM。

---

# Practice

> 希望一次為超過 10 個 VMs 設定：「 在維護 maintenance 期間有 high availability 」且這些 VMs 要「 在 crash 後嘗試 automatically restart 自動重啟」，則應該怎麼做？

這個關係到 VM maintenance policy 部分，可參考 [Set VM host maintenance policy](https://cloud.google.com/compute/docs/instances/setting-vm-host-options#available_host_maintenance_properties)。因為要一次啟動很多個 VMs 且都要有同一個 maintenance policy ，故選擇創建一個 instance template 來當共通範本。再來由於需要 high availability ，故範本內容會需要設定 :

- `Automatic Restart` 要 on 開啟
- `On Host Maintenance` 要設定成 `Migrate VM instance`

{{< alert warning >}}
特別注意 MIG 的 Autohealing 功能，它主要是針對**APP 應用層面**的故障，無法應對 host maintenance 系統維護或 instance crash 實例崩潰的情況
{{< /alert >}}

---

### 參考資料

- [Using startup scripts on Linux VMs](https://cloud.google.com/compute/docs/instances/startup-scripts/linux)

- [instance-templates office doc](https://cloud.google.com/compute/docs/instance-templates)

- [Google Cloud 雲端平台介紹](https://jason-kao-blog.medium.com/google-cloud%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%B4%B9-fc3212c8359b)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [disk vs snapshot](https://stackoverflow.com/questions/27290731/google-compute-engine-what-is-the-difference-between-disk-snapshot-and-disk-ima)
