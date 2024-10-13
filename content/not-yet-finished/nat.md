---
title: NAT 概述

author: Aryido

date: 2024-07-02T20:18:34+08:00

thumbnailImage: ""

categories:


tags:


comment: false

reward: false
---
<!--BODY-->
> Cloud NAT 全稱是 Cloud Network Address Translation，是 Google Cloud 代管的 IP 轉譯服務，可在不公開 IP 位址的情況下，讓「某些 GCP 資源」可以高效的**連接上**「外部網路 Internet」，而外部資源無法直接存取 Cloud NAT gateway 後方的資源，維持獨立性與安全性。對應其他的雲端服務是 :
> - Amazon Web Services (AWS) : **NAT gateways**
> - Microsoft Azure : **Azure NAT Gateway**
> 
> Cloud NAT 是以 Software-defined Networking 服務，中間不存在 proxy instance ，故性能方面比傳統 NAT 好，目前最經常是用在 VM、GKE 上，除此之外也提供 Cloud Run、Cloud Functions、App Engine 等服務，能夠透過**共用**一組 IP 位址對外通訊，建立 NAT gateways 連線到 VPC 外部網路，由於**共用**一組 IP，也解決了 IP 短缺問題。

<!--more-->

---


對於安全層面來說，私有IP地址對於外部網路是不可見的，因此可以更好地保護內部資源的安全性。 但是，如果想與外部建立連接，又必須使用public IP地址。 相反，如果使用公有IP地址，則會面臨IP地址大小限制的問題，因此，Google Cloud NAT可以很好地協助解決這些問題。

# 前言
雖然是在雲端運行 application，但不代表會想要讓公開 IP 位置對外，因為怕遭到惡意攻擊。前面有提到「Load-Balancer」，我們經常使用它來直接面對 User， 如果對外都用 Load-Balancer 的話，後端的資源的確可以不需要有 Public IP 也能被訪問。但沒有外部 IP 的話會有個問題，雖然外網一般無法直接連線資源可以保證一些安全性，可是該資源自己也**不能上網**！ 但我們很多時候，又需要有對外 API 連線，或者有固定公開網路 IP 的需求，這時解決方法有以下幾種（資源都先以 VM 舉例）：

- ### VPN Gateway

    透過 VPN gateway 只授權某些連線可以進行且會加密流量，外部對內、內部對外皆可通，使用 VPN 就會 VPN 伺服器自己的 IP 位址路由，互動的 application 只能看到 VPN 的 IP 位址，而非真實的 IP 位址。但此種方法 VM 仍然需開啟 Public IP ，因此仍有被攻擊的風險。

- ### Bastion Host (堡壘主機)

    Bastion Host 有時也稱 Jump Server 跳版主機，作為進入內部網絡的一個檢查點，會處理所有外部的請求，且並不會將跳版主機後面的 VM IP 位置暴露。但此種方式只解決了外部的請求連線，在內部主動對外發出請求的狀況下，仍然會需要 Public IP 位置。

# NAT (Network Address Translation)
以上方法都是一些簡單的解決方法，這邊開始進入本章主角：NAT，而「傳統 NAT 」和「Cloud NAT」又有一些不同。傳統 NAT 會有一個 NAT proxy instance 處理所有後端資源與外部的連線，然而此種方式可能會造成效能與單位時間吞吐量的降低。
{{< image classes="fancybox fig-100" src="/images/google-cloud/nat/nat-architecture.jpg" >}}

不同於傳統的 NAT 代理解決方案， GCP Cloud NAT **不基於**「代理 VM」或「設備」，它會為**每個資源分配一個 NAT IP 和相對應的端口範圍**，這些分配的 IP 和端口範圍都是透過 GCP 進行**自動化管理**的，背後都是透過 [Andromeda](https://01.me/2014/03/networking-at-google/)(/ænˈdrɑː.mə.də/) 這個 Google 發展很久的網路虛擬化技術來實現，提供高效能網路位址轉譯服務。 
{{< alert success >}}
其實簡單來說，Cloud NAT 就可以讓你沒有 IP 的主機都可以上網。
{{< /alert >}}

# Cloud NAT 實現說明
private IP 轉換為 public IP 時，使用的是 IP masquerading 技術，簡而言之，就是將 source Ip 更改為路由器上的 public IP，這樣外部網路才能識別，為了實現 IP masquerading ，會使用「Cloud Router」充當 VPC 的路由器，主要負責維護兩個網路間的路由或 Nat 連接等傳輸規則。

IP 和端口範圍都是透過 GCP 進行**自動化管理**是很方便的，例如說這個資源可能只是偶爾會上網一下，那 Port就直接被這個資源占用會有點浪費，那 Cloud NAT 它能夠把這些閒置的 Port 再收回來，以後又要上網的時候，會再把這個 Port 分配它 ; Cloud NAT 也能夠支持設定 Static IP、手動指定的 NAT IP，還蠻強大的。

# Cloud NAT 類型
Cloud NAT 會建立 NAT gateway，讓  private subnet 中的資源可以連接到 VPC 網路外部，分成 Public NAT 和 Private NAT，但是是**可以同時存在**的。

### Public NAT
最常使用此類型， 會使用一組可以連線到外網的 Public IPs。當資源要連線到外網時， Public NAT 會分配一個 external IP 和 source port。例如說： 有一個 VM-1 在 subnet-1 且沒有 external IP，但他需要去網路上 download 一些更新包，則就可以在 subnet-1 上建立 Public NAT gateway 並設定一個 IP address 給它，

### Private NAT
會用在 on-premises networks 或者  other cloud provider networks 和 Cloud VPN 的連線，這種也稱為 Hybrid NAT。另外是 Inter-VPC NAT ，因為在 GCP 中 若有重疊 subnet 網段就不允許 VPC Peering，這時就會改用 `type=PRIVATE` 的 NAT 配置。








NAT所需注意事項
每個 VM 執行個體的最低通訊埠數設定
NAT IP有65536port會先扣掉了1024需被使用數，剩下的64512可被使用，已預設來說最低通訊埠數為64，所以單一個NAT IP可供給1008台VM(64512/64)
以上面的設定來說若遇到NAT連線數吃滿不夠可以做三件事
提高通訊埠數數量
開啟更多台的VM
又或是增加NAT IP數量
總結
以上設定在GCP上並不難，在實務面上比較需要知道NAT上面會有什麼限制(連線數)以及甚麼東西會吃到NAT設定(區域與網域網段)。


---

### 參考資料

- [Cloud NAT overview](https://cloud.google.com/nat/docs/overview)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [利用 Cloud NAT 維持雲端的獨立性與安全性](https://medium.com/peerone-technology-%E7%9A%AE%E5%81%B6%E7%8E%A9%E4%BA%92%E5%8B%95%E7%A7%91%E6%8A%80/%E5%88%A9%E7%94%A8-cloud-nat-%E7%B6%AD%E6%8C%81%E9%9B%B2%E7%AB%AF%E7%9A%84%E7%8D%A8%E7%AB%8B%E6%80%A7%E8%88%87%E5%AE%89%E5%85%A8%E6%80%A7-dd696faad686)

- [Google Cloud NAT：在Google Cloud上實現私有IP地址的網路連接](https://kingcloud.tech/944.html)


