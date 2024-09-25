---
title: GCP - Cloud NAT 概述

author: Aryido

date: 2024-07-02T20:18:34+08:00

thumbnailImage: "/images/google-cloud/nat/nat-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-network

comment: false

reward: false
---

<!--BODY-->

> Cloud NAT 全稱是 Cloud Network Address Translation，是 Google Cloud 代管的 IP 轉譯服務，可在不公開 IP 位址的情況下，讓 GCP VM 或 GKE 內的 Pod 可以高效的**連接上**「外部網路 Internet」，而外部資源無法直接存取 Cloud NAT gateway 後方的資源，維持獨立性與安全性。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **NAT gateways**
> - Microsoft Azure : **Azure NAT Gateway**
>
> Cloud NAT 是以 Software-defined Networking 服務，中間不存在 proxy instance ，故性能方面比傳統 NAT 好上不少。除了 VM、GKE 之外，也可使用在 Cloud Run、Cloud Functions、App Engine 等服務。

<!--more-->

---

雖然是在雲端運行 application，但不代表會想要讓公開 IP 位置對外，其中一個原因是怕遭到惡意攻擊。前面有提到「GCP Load-Balancer」，我們經常使用它來直接建立對外部網路的連線窗口，如果對外都用 Load-Balancer 的話，後端的資源的確可以不需要有 Public IP 也能被訪問。但沒有外部 IP 的話會有個問題: **雖然外網一般無法直接連線資源可以保證一些安全性，可是該資源自己也不能上網**！為解決此問題，可以使用 Cloud NAT 來幫忙。

# NAT (Network Address Translation) 架構

「傳統 NAT 」和「Cloud NAT」架構上有一些不同。傳統 NAT 會有一個 NAT proxy instance 處理所有後端資源與外部的連線，然而此種方式由於**單點**原因，可能會造成效能與單位時間吞吐量的降低。
{{< image classes="fancybox fig-100" src="/images/google-cloud/nat/nat-architecture.jpg" >}}

而 GCP Cloud NAT 並**不基於**「代理 VM」或「設備」，它會為**每個資源分配一個 NAT IP 和相對應的端口範圍**，這些分配的 IP 和 Ports 都是透過 GCP 進行**自動化管理**的，背後都是透過 [Andromeda](https://01.me/2014/03/networking-at-google/)(/ænˈdrɑː.mə.də/) 這個 Google 發展很久的網路虛擬化技術來實現，提供高效能 IP 轉譯服務。
{{< alert success >}}
IP 和 Ports 都是透過 GCP 進行**自動化管理**是很方便的，例如說這個資源可能只是偶爾會上網一下，那如果 Port 就直接被這個資源占用的話會有點浪費，而 Cloud NAT 是能把這些閒置的 Port 再收回來的，之後該資源又有上網需求的時候，會再把 Port 分配給它。
{{< /alert >}}

# Cloud NAT 設定

{{< image classes="fancybox fig-100" src="/images/google-cloud/nat/nat-settings.jpg" >}}

### Cloud NAT 類型

Cloud NAT 需要建立 NAT Gateway 和 NAT Router ，讓 VPC 內部的資源可以連接到 VPC 外部網路，有分成 Public NAT 和 Private NAT 兩種，是**可以同時存在**的。

- **Public**

  算最常使用的類型， 設定一組（可以一個或多個）Public IPs ，當資源要連線到外網時， NAT 會分配一個 Public IP 和 source port 給資源。例如說： 有一個 `VM-1` 沒有 external IP，但他需要去網路上 download 一些更新包，就可以選擇建立 **Public NAT Gateway**，再來自己手動設定一個 IP address 或者讓 GCP 自動分配 IP 。

- **Private**

  用在 on-premises networks 或者 other cloud provider networks 和 Cloud VPN 的連線，這種也稱為 Hybrid NAT。

  另外是 Inter-VPC NAT ，因為在 GCP 中 若有重疊 subnet 網段就不允許 VPC Peering，這時就會改用 `type=PRIVATE` 的 NAT 配置。

### Cloud Router

Cloud Router 使用 Border Gateway Protocol (BGP) 來 advertise IP prefixes 。 Cloud Router 不是實體設備，而是由充當 BGP speakers 和 responders 的軟體組成，作為 Cloud NAT 的 control plane ，將 Cloud NAT gatway 設定分組在一起。

{{< alert info >}}
除了和 Cloud NAT 搭配使用之外， Cloud Router 還可和以下產品配合使用：

- **Cloud Interconnect**
- **Cloud VPN**, specifically HA VPN
- **Router appliance** (part of Network Connectivity Center)
  {{< /alert >}}

### Cloud NAT Mapping

Cloud NAT 的 IP 分配有兩種模式: 「Automic」 和 「Manual」。另外還能夠設定 Static IP。

- **Automatic NAT IP address allocation**:

  值得注意的是，自動添加的 NAT IP 位址可以在 static external IP addresses 列表中看到，但並不列入 per-project quotas 的計算。自動添加的 NAT IP 不再使用時，也會自動刪除

- **Manual NAT IP address assignment**:

  自動分配 IP 的話是無法預測產生的 IP 是什麼，例如是要創建白名單，這時就應該使用手動分配 NAT IP，另外手動分配 IP 設定也可以盡量減少 IP 的數量。

# Cloud NAT 實現和運作說明

私有 Private IP 轉換為公有 Public IP 時，使用的是 「IP masquerading」 技術，簡而言之，就是將 Source IP 更改為路由器上的 Public IP，這樣外部網路才能識別，為了實現 IP masquerading ，會使用「Cloud Router」來當 VPC 的路由器，主要負責維護兩個網路間的路由或 Nat 連接等傳輸規則。Cloud NAT 使用 NAT Router 來管理連線，NAT Router 僅限特定 VPC 和其 region ，如果在多個地區中皆有資源需要上網，就必須在每個地區建立 NAT Router。

{{< alert success >}}
總結簡單來說，Cloud NAT 就可以讓你沒有 IP 的主機都可以上網。
{{< /alert >}}

---

### 參考資料

- [Cloud NAT overview](https://cloud.google.com/nat/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [利用 Cloud NAT 維持雲端的獨立性與安全性](https://medium.com/peerone-technology-%E7%9A%AE%E5%81%B6%E7%8E%A9%E4%BA%92%E5%8B%95%E7%A7%91%E6%8A%80/%E5%88%A9%E7%94%A8-cloud-nat-%E7%B6%AD%E6%8C%81%E9%9B%B2%E7%AB%AF%E7%9A%84%E7%8D%A8%E7%AB%8B%E6%80%A7%E8%88%87%E5%AE%89%E5%85%A8%E6%80%A7-dd696faad686)

- [Google Cloud NAT：在 Google Cloud 上實現私有 IP 地址的網路連接](https://kingcloud.tech/944.html)

- [How-to Guides】GCP 網路 — 使用 NAT 簡化存取網際網路權控管](https://medium.com/@kellenjohn175/how-to-guides-gcp-%E7%B6%B2%E8%B7%AF-%E4%BD%BF%E7%94%A8-nat-%E7%B0%A1%E5%8C%96%E5%AD%98%E5%8F%96%E7%B6%B2%E9%9A%9B%E7%B6%B2%E8%B7%AF%E6%AC%8A%E6%8E%A7%E7%AE%A1-a5aa0e936ded)
