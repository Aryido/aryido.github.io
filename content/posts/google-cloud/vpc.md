---
title: GCP - Virtual Private Cloud 概述

author: Aryido

date: 2024-05-30T18:52:24+08:00

thumbnailImage: "/images/google-cloud/network/vpc-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - network
  - gcp-network

comment: false

reward: false
---

<!--BODY-->

> Virtual Private Cloud 虛擬私有雲網路，可簡寫成 VPC、網路、VPC Network、Network 等等都可以，是 Google 使用 [Andromeda](https://01.me/2014/03/networking-at-google/)(/ænˈdrɑː.mə.də/) 網路虛擬化技術實現的一個雲端資源，提供如 GCP-VM、GKE、Serverless Workloads 和 App Engine 等等雲端服務的網路功能，能讓 User 可以高自由度地建立管理和優化網路架構。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Amazon VPC**
> - Microsoft Azure : **Azure Virtual Network**
>
> GCP-VPC 和 AWS-VPC 架構上蠻不一樣的，GCP-VPC 是全球性的，只要在同一個 GCP-VPC 內，就算不同 Region 也能使用 Internal IP ; 但如果是不同的 GCP-VPC 就算在同一個 Region 也不能互相通訊。而 AWS-VPC 是針對 Region 來設計的，故只要跨 Region 就不是內網無法直接溝通需再多做其他設定。

<!--more-->

---

前面有介紹了 GCP Compute Engine，而這些 VM 就是放在 Google 全世界分佈的實體資料中心，而在使用 VM 時，會需要**網路設定**，而統籌管理這些網路功能的平台就是 VPC，其基本功能有：

- 可以建立許多 Subnet，並且 Subnet 可以位於不同的 Region
- 不同 Region 內的資源，只要它們在同個 VPC 內，就可以用 Internal IP 互相連線
- VPC 各自擁有自己的 Firewall 和 Route 來限制和保護資源之間的通訊
- 可自動或手動分配給雲端主機 IP
- 可與其他 VPC Peering

{{< alert warning >}}
GCP 專案內預設只先可以開 5 個 VPC 而已。若預設的 Quota 不足，還要跟 Google Cloud 申請更多。
{{< /alert >}}

# VPC 架構簡介

簡單說 VPC 就是串接雲端服務的網路，每個 GCP-VPC 都是網路隔離的（network isolation），但在同一個 GCP-VPC 內視為**同一個內網環境**，可以透過 Internal IP 來相互通訊。

### Subnet

官方文件中，有表示 subnet 和 subnetwork 為同一個意思。
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/vpc.jpg" >}}

以上圖來配合說明，每個 VPC 上都可以切出多個 Subnets 並定義一個 IPv4 位址範圍網段; 每一個 Subnet 會對應到一個 Region，兩個不同的 Subnet 對應到同樣的 Region 是沒問題的，**但要確保切的網段不能有重疊部分**。

{{< alert success >}}
可以發現，其實 VPC 並沒有 Mapping 到任何 Region，真正 Mapping 的是在 Subnet。
{{< /alert >}}
Subnet 是配置在 Region 上的，故 Region 裡面的資源如 VM，若有需要 IP 地址，則會領取到該 Region 上某個 Subnet 網段內的 IP address，但注意每個 Subnet 網段內有四個不可用的 IP 位址，可參閱 [Unusable addresses in IPv4 subnet ranges](https://cloud.google.com/vpc/docs/subnets#unusable-ip-addresses-in-every-subnet)。

- ##### 為什麼同個 Region 也會要切分 Subnet ?

  同個 Region 也要切分 Subnet 是因為我們在雲端上，會建立不同類型的資源例如資料庫、 VM 等，每種類型的資源都有**自己的存取要求**，例如我們可以為 Public 資源 和 Private 資源建立不同的 Subnet ，而 User 可將不同存取權限的資源，分配到不同的 Subnet 中管理並隔離 :

  > - 可以從 Internet 存取 Public subnet 中的資源
  > - 但無法從 Internet 存取 Private subnet 中的資源
  > - 但是 Public subnet 中的資源可以和 Private subnet 中的資源通訊

- ##### Subnet 網段的擴張

  隨著服務越來越多，Subnet 的 IP address 數量可能不夠，這個時候就需要來進行 IP range 的擴展。在 Google Cloud 上進行 VPC 網段擴張並不會影響到資源的運作，但在進行擴展時有幾個注意事項：

  > - 網段之間不能重複衝突的部分
  > - 網段只能擴張不能縮減，因此建議一開始配置不要太大夠用就好，後續隨著需求再增加

### 預設 VPC

一般來說 GCP Project 初始化開起來後，就會有一個預設 VPC 名字叫做 `default` ，它是在全世界每個 Region 都建了切好 Subnet 網段給我們。雖然已經有預設的 VPC ，但其實**不建議**在 Prod 環境中使用，應當建立了自己的 VPC ， GCP 官方給出的 Best Practices 也是這樣說明的，原因是：

- ##### **不需要每個 Region 中都有一個 Subnet**

  應該是按照自己的需求來切分。例如可能只需要用台灣的 Region 部署服務而已，若使用 `default` VPC 就會有很多網段都給其他國家用掉了，可以自己分配的網段就會少很多，十分浪費，所以建議自訂 VPC。

- ##### Peering 的限制

  未來有計劃要使用 VPC Network Peering 或 Cloud VPN 等服務來連接不同的 VPC 網絡，就會一定會建議要自行規劃網段，因為在 Auto 模式下生成的 Subnet IP Range 會產生衝突。

---

### 參考資料

- [Virtual Private Cloud (VPC) overview](https://cloud.google.com/vpc/docs/overview)

- [資訊科普系列(14) — Google Cloud Platform Networking（一）](https://medium.com/moda-it/google-cloud-platform-networking-%E7%B0%A1%E4%BB%8B-b0b2ec2ff7be)

- [GCP 圖解教學 - Network - VPC-Subnet & Region-Zone 關係介紹](https://www.youtube.com/watch?v=yygf4MOmI-E)

- [[GCP 教學] 010 整個世界都是我的區域網路！Virtual Private Cloud VPC 網路概念介紹](https://www.youtube.com/watch?v=dMLF89FevAA)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)
