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
  - docker

comment: false

reward: false
---

<!--BODY-->

> Virtual Private Cloud 虛擬私有雲網路，簡寫為 VPC、網路、VPC Network、Network 等等都可以，是 Google 使用 [Andromeda](https://01.me/2014/03/networking-at-google/)(/ænˈdrɑː.mə.də/) 網路虛擬化技術實現的一個雲端資源，提供如 GCP-VM、GKE、Serverless Workloads 或 App Engine 等等雲端服務的「網路功能」，能讓 User 高自由度的建立管理和優化網路架構。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Amazon VPC**
> - Microsoft Azure : **Azure Virtual Network**
>
> GCP-VPC 和 AWS-VPC 架構上蠻不一樣的，GCP-VPC 是全球性的，只要在同一個 GCP-VPC 內，就算不同 Region 也能使用 Internal IP ; 但如果是不同的 GCP-VPC 就算在同一個 Region 下也不能互相通訊。而 AWS-VPC 是針對 Region 來設計的，故 AWS-VPC 只要跨 Region 就不是內網無法直接溝通，需再多做其他設定才能連線到彼此。

<!--more-->

---

前面有介紹了 GCP Compute Engine，而這些 VM 就是放在 Google 全世界分佈的實體資料中心，而在使用 VM 時，通常會需要**網路設定**，而統籌管理這些網路功能的平台就是 VPC，其基本功能有：

- 可以建立許多 Subnet，並且 Subnet 可以位於不同的 Region
- 不同 Region 內的資源，只要它們在同個 VPC 內，就可以用 Internal IP 互相連線
- VPC 各自擁有自己的 Firewall 和 Route 來限制和保護資源之間的通訊
- 可自動或手動分配給雲端主機 IP
- 可與其他 VPC Peering

{{< alert warning >}}
GCP 專案內預設只先可以開 5 個 VPC 而已。若預設的 Quota 不足，還要跟 Google Cloud 申請，才能創建更多。
{{< /alert >}}

# VPC 架構簡介

簡單說 VPC 就是串接雲端服務的「網路」，每個 GCP-VPC 都是網路隔離的（network isolation），但在同一個 GCP-VPC 內視為**同一個內網環境**，可以透過 Internal IP 來相互通訊。

### Subnet

官方文件中有表示 subnet 和 subnetwork 為同一個意思。
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/vpc.jpg" >}}

以上圖來配合說明，每個 VPC 上都可以切出多個 Subnets 並定義一個 IPv4 位址範圍網段; 每一個 Subnet 會對應到一個 Region，兩個不同的 Subnet 對應到同樣的 Region 是沒問題的，**但要確保切的網段不能有重疊部分**，再來還要注意[有些 Subnet 範圍是不可以使用的](https://cloud.google.com/vpc/docs/subnets#unusable-ip-addresses-in-every-subnet)例如說 :

> - `0.0.0.0/8` 代表 local network
> - `127.0.0.0/8` 代表 Local host
> - `199.36.153.4/30` 和 `199.36.153.8/30` 代表 Private Google Access IP addresses

如果真的要開始自行切分 Subnet ，**務必要參考有效範圍和禁止範圍**。

{{< alert success >}}
可以發現，其實 VPC 並沒有 Mapping 到任何 Region，真正 Mapping 的是在 Subnet。
{{< /alert >}}

Subnet 配置在 Region 上的，故 Region 裡面的資源如 VM，若有需要 IP 地址，則會領取到該 Region 上某個 Subnet 網段內的 IP ，也[注意每個 Subnet 網段內有四個不可用的 IP](https://cloud.google.com/vpc/docs/subnets#unusable-ip-addresses-in-every-subnet)，主要是 :

> - IPv4 range 的 **First IP**，代表 Network address
> - IPv4 range 的 **Second IP**，代表 Default gateway address
> - IPv4 range 的 **Second-to-last IP**，為 GCP 保留的，以備將來使用
> - IPv4 range 的 **Last IP**，代表 Broadcast address

簡單來說就是 subnet 的**最前兩個和最後兩個**，總共這 4 個 IP 不能使用。

- ##### 為什麼同個 Region 也會要切分 Subnet ?

  同個 Region 也要切分 Subnet 是因為我們在雲端上，會建立不同類型的資源例如資料庫、 VM 等，每種類型的資源都有**自己的存取要求**，例如我們可以為 Public 資源 和 Private 資源建立不同的 Subnet ，而 User 可將不同存取權限的資源，分配到不同的 Subnet 中管理並隔離 :

  - 可以從 Internet 存取 Public subnet 中的資源
  - 但無法從 Internet 存取 Private subnet 中的資源
  - 但是 Public subnet 中的資源可以和 Private subnet 中的資源通訊

- ##### Subnet 網段的擴張

  隨著服務越來越多，Subnet 的 IP address 數量可能不夠，這個時候就需要來進行 IP range 的擴展。在 Google Cloud 上進行 VPC 網段擴張並不會影響到資源的運作，但在進行擴展時有幾個注意事項：

  - 網段之間不能重複衝突的部分
  - 網段只能擴張不能縮減，因此建議一開始配置不要太大夠用就好，後續隨著需求再增加

### 預設 VPC

一般來說 GCP Project 初始化開起來後，就會有一個預設 VPC 名字叫做 `default` ，它是在全世界每個 Region 都建了切好 Subnet 網段給我們。雖然已經有預設的 VPC ，但其實**不建議**在 Prod 環境中使用，應當建立了自己的 VPC ， GCP 官方給出的 Best Practices 也是這樣說明的，原因是：

- ##### **不需要每個 Region 中都有一個 Subnet**

  應該是按照自己的需求來切分。例如可能只需要用台灣的 Region 部署服務而已，若使用 `default` VPC 就會有很多網段都給其他國家用掉了，可以自己分配的網段就會少很多，十分浪費，所以建議自訂 VPC。

- ##### Peering 的限制

  未來有計劃要使用 VPC Network Peering 或 Cloud VPN 等服務來連接不同的 VPC 網絡，就會一定會建議要自行規劃網段，因為在 Auto 模式下生成的 Subnet IP Range 都是固定的，故兩個 VPC 都用 Auto 模式切分 Subnet IP Rang ，一定會產生衝突。

---

# Practice

> 如果要創建一個 custom VPC with a single subnet ，且該 subnet 的範圍必須盡可能大，應該使用哪個範圍？
> - A. `0.0.0.0/0`
> - B. `10.0.0.0/8` **(O)**
> - C. `172.16.0.0/12`
> - D. `192.168.0.0/16`

首先 `0.0.0.0/0` 還算蠻特例的，通常用於代指**所有 IP**，無法用來作為 VPC 子網的範圍。而接下來 [`10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16`](https://cloud.google.com/vpc/docs/subnets#valid-ranges) 都是 「RFC 1918」承認的 valid IPv4 ranges，都可以使用來作為 subnet 範圍，故若要選最大的就用 `10.0.0.0/8` 吧。

{{< alert danger >}}
比較特別的是可以注意一下 [`172.17.0.0/16`](https://cloud.google.com/vpc/docs/subnets#additional-ipv4-considerations)，這個範圍是 default Docker bridge network 的範圍，所以如果你產品有用到 docker ，務必小心 docker 網路和 vpc 網路的重疊問題
{{< /alert >}}

---

### 參考資料

- [Virtual Private Cloud (VPC) overview](https://cloud.google.com/vpc/docs/overview)

- [資訊科普系列(14) — Google Cloud Platform Networking（一）](https://medium.com/moda-it/google-cloud-platform-networking-%E7%B0%A1%E4%BB%8B-b0b2ec2ff7be)

- [GCP 圖解教學 - Network - VPC-Subnet & Region-Zone 關係介紹](https://www.youtube.com/watch?v=yygf4MOmI-E)

- [[GCP 教學] 010 整個世界都是我的區域網路！Virtual Private Cloud VPC 網路概念介紹](https://www.youtube.com/watch?v=dMLF89FevAA)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)
