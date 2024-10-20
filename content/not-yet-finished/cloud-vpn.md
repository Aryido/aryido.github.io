---
title: GCP - Cloud VPN & Cloud Interconnect 概述

author: Aryido

date: 2024-10-10T18:18:34+08:00

thumbnailImage: "/images/google-cloud/vpn-interconnect/vpn-logo.jpg"

categories:
- cloud
- gcp

tags:


comment: false

reward: false
---
<!--BODY-->
> Cloud VPN 全稱是 Cloud Virtual Private Network，能夠將 On-premise Network 和 GCP VPC Network 連接在一起，流量透過 IPSec 協議進行加密。而 Cloud Interconnect 可以說是加強版的 Cloud VPN，它是真的從你 On-premise 的設備做**一個實體的線路**，一直到 Google 的機房裡面。兩個分別對應其他的雲端服務是 :
> - Cloud VPN
>   - Amazon Web Services (AWS) : **AWS VPN**
>   - Microsoft Azure : **Azure VPN** 
> - Cloud Interconnect
>   - Amazon Web Services (AWS) : **AWS Direct Connect**
>   - Microsoft Azure : **Azure ExpressRoute**
>


<!--more-->

---

# Cloud VPN 類型

Cloud VPN 把 On-premise 和 Cloud 兩個不同 Network 之間傳輸的流量，由一個 VPN Gateway 透過 IPSec 協議進行加密，再由另一個 VPN Gateway 解密，此操作可以更加保護資料在網路上傳輸的安全。

{{< alert info >}}
此外 Cloud VPN 也支持把兩個 VPC 透過「兩個 Cloud VPN 實例」連接在一起
{{< /alert >}}

### HA VPN

{{< image classes="fancybox fig-100" src="/images/google-cloud/vpn-interconnect/ha-vpn.jpg" >}}

創建 HA VPN Gateway 時， GCP 會自動選擇兩個 external IP 以支持高可用性 ; HA VPN Gateway 接口都支持多個隧道，也可以創建多
個 HA VPN Gateway。

刪除 HA VPN Gateway 時，GCP 會 release IP 


可以將高可用性VPN 網關配置為只有一個活動接口和一
個外部IP 地址；但是，此配置不會提供服務可用性達到99.99% 的SLA。

單個地區中通過 IPsec VPN 連接將本地網絡連接到 VPC 。HA VPN 提供服務可用性達 99.99% 的服務等級協議(SLA)。

### Classic VPN
Classic VPN Gateway 只有單個接口、單個外部 IP 地址，使用靜態路由（基於政策或基於路由）的隧道。

可以為 Classic VPN 配置動態路由(BGP)，但僅適用於連接到 Google Cloud VM instance 上運行的第三方 VPN Gateway 軟體的隧道。

傳統VPN 網關提供服務可用性達99.9% 的服務等級協議(SLA)。


---

# 使用 VPN 前需知道的一些資訊



在建立 Cloud VPN 前，我們需要建立本地端的 VPN Gateway 、 雲端的 VPN Gateway 以及兩個 VPN Tunnel， Cloud VPN 的 Gateway 屬於一個 Regional 的資源，因此需要使用一個 external IP 。

本地端的 VPN Gateway 可以是硬體的主機，也可以是軟體的 VPN 服務，安裝在本地或其他雲端伺服器中。


# Cloud Router
Cloud VPN 支援靜態以及動態路由，如果需要使用動態路由，則需要開啟 Cloud Routers 的功能。 Cloud Routers 支援透過 BGP (Border Gateway Protocol) 管理 Cloud VPN Tunnel 的路由。

要設定 BGP ，必須為兩端的 VPN 分配一個額外的 IP Address，這兩個 IP Address 必須是 link-local IP，於 IP Range 169.254.0.0/16 之中。這個 IP 位置不屬於兩邊網路的 IP，而是專門提供給 BGP 使用。



從地端去連到 Google 的雲端 基本上 是透過網際網路 Internet 去存取的
但是有的人會擔心說 這樣子傳輸的封包會不會被監聽
所以今天做 VPN 就是把 我們地端跟雲端之間的溝通管道
再做一層加密傳輸 這樣的話即使被監聽 對方也沒有辦法破解
Cloud VPN 要使用的話 我們地端需要準備硬體的 VPN 設備
支援的廠牌 比如說什麼 Fortinet、Cisco 一些很大的網路廠牌其實都有支援
大家可以再去查詢 Google 的文件 看到各種支援廠牌 它都有提供說明書可以給你看
然後來操作設定這樣子 所以 它做起來之後 雲端跟地端其實就會變成同一個內網
就是我今天去存取雲端 我不需要打雲端的外部 IP 我打內部 IP 我就到雲端了
雲端也是可以打地端的 IP 就直接到地端 也不用打外部 IP 除此之外
Cloud VPN 會建議你這個傳輸通道 要建兩條 為什麼要建兩條
主要的原因就是說 如果今天有一條 萬一有什麼狀況的話 那不就完全不能溝通了嗎
所以會建兩條同時跑 那萬一有一條有問題的話 它就會自動切到另一條
就可以確保你的傳輸線路是穩定的 除此之外 它會有一個 BGP 的通訊協定
簡單的說 就是因為雲端會有雲端的一大堆網段 地端也會有一大堆網段
兩邊的路由或者是網段有更新的話 你通常兩邊都要去做一些設定
才能確保兩邊一直都保持暢通 但是人工設定其實是很麻煩的
所以 Google 提供了 BGP 的協定 可以確保說不管是雲端的網段有異動
還是地端的網段有異動的話 它都可以自動的跟對方講 那之後我在做雲地傳輸的時候
就不會說找不到對方機器的這個問題 或者是 網路有問題還要重新去設定路由
這些都不用做 好，那這個 Cloud Interconnect 其實是加強版的 Cloud VPN






高速混合雲方案 Cloud Interconnect
也就是說剛剛講的 VPN 我們是在地端做一個設備
然後連到雲端 所以我們到雲端上 它其實只是在
網際網路上做一個加密傳輸通道而已 可是 Cloud Interconnect

就是說 Cloud Interconnect 完全不會碰到 Internet 你的封包完全不會有被監聽的可能

這個東西 因為它有實體的建置工作 在台灣目前有中華電信跟是方電訊
可以來你的公司去拉這個線 會先拉到這兩家廠商的機房
然後再插到 Google 的機器上 因為從頭到尾都是走線路 不會跟別人塞車
所以它的速度是非常非常快的 它的 Latency 非常的低
可能是 1/2/3 反正它的 Latency 非常的低 剛剛講的是毫秒
它也可以去做 HA 你可以做兩條實體線路
你也可以直接到兩個不同的廠商 然後再到 Google 的機房
目前許多大型的 上市公司或政府機關都有用
就像上面寫的這樣子給大家參考 這只是其中一個 Google 公開的例子

Extend your on-premises network to Google's network through a highly available, low-latency connection. You can use Dedicated Interconnect to connect directly to Google or use Partner Interconnect to connect to Google through a supported service provider.




---

### 參考資料

- [Cloud VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Cloud VPN](https://ithelp.ithome.com.tw/m/articles/10277295)

- [Cloud VPN（虛擬私有網路）](file:///var/folders/65/88bp1cm11_q82cggtjxfk5nw0000gn/T/MicrosoftEdgeDownloads/7273a90a-a0fb-4544-ba7c-9a29b3bd93b1/0201000050-dm1%20(1).pdf)



https://ikala.cloud/google-cloud-vpn/

https://medium.com/@kellenjohn175/how-to-guides-gcp-%E7%B6%B2%E8%B7%AF-%E5%BE%9E%E9%9B%B6%E9%96%8B%E5%A7%8B%E6%90%AD%E5%BB%BA-vpn-virtul-private-network-dc53c14bdbee


You are hosting an application on bare-metal servers in your own data center. The application needs access to Cloud Storage. However, security policies prevent the servers hosting the application from having public IP addresses or access to the internet. You want to follow Google-recommended practices to provide the application with access to Cloud Storage. What should you do?

A. 1. Use nslookup to get the IP address for storage.googleapis.com. 2. Negotiate with the security team to be able to give a public IP address to the servers. 3. Only allow egress traffic from those servers to the IP addresses for storage.googleapis.com.
B. 1. Using Cloud VPN, create a VPN tunnel to a Virtual Private Cloud (VPC) in Google Cloud. 2. In this VPC, create a Compute Engine instance and install the Squid proxy server on this instance. 3. Configure your servers to use that instance as a proxy to access Cloud Storage.
C. 1. Use Migrate for Compute Engine (formerly known as Velostrata) to migrate those servers to Compute Engine. 2. Create an internal load balancer (ILB) that uses storage.googleapis.com as backend. 3. Configure your new instances to use this ILB as proxy.
D. 1. Using Cloud VPN or Interconnect, create a tunnel to a VPC in Google Cloud. 2. Use Cloud Router to create a custom route advertisement for 199.36.153.4/30. Announce that network to your on-premises network through the VPN tunnel. 3. In your on-premises network, configure your DNS server to resolve *.googleapis.com as a CNAME to restricted.googleapis.com.

A. 手動解析 IP 並向安全團隊要求公共 IP 地址
這個選項建議使用 nslookup 手動獲取 storage.googleapis.com 的 IP 地址，並與安全團隊協商以給予服務器公共IP地址。然而，這種方式並不符合 Google 的最佳安全實踐，因為裸機服務器的安全策略禁止公共IP地址，而且 IP 可能會動態變化，這會導致訪問的不穩定。這種方式沒有考慮到 Google Cloud 的最佳實踐和推薦方法。

不推薦：此解決方案未遵循 Google 的推薦做法，並且管理靜態 IP 是不可行的。

B. 使用VPN和Squid代理
這個選項建議通過 Cloud VPN 連接到 Google Cloud，並在 Compute Engine 上運行 Squid 代理服務器來轉發流量。雖然這是一種可行的方式，但它增加了管理和維護代理服務器的複雜性，並且需要額外的代理設置，而 Google 提供了更簡單的方式來達到相同的目的。

不推薦：這種方法過於復雜，需要額外的代理管理，不是最佳實踐。

C. 遷移到 Compute Engine 並使用內部負載均衡器
這個選項建議使用 Migrate for Compute Engine 將裸機服務器遷移到 Google Cloud，並使用內部負載均衡器來代理流量。這個解決方案過於複雜且不必要，因為問題的關鍵是如何讓本地的裸機服務器訪問 Cloud Storage，而不是將服務器遷移到雲端。

不推薦：不需要將本地服務器遷移到雲端以解決這個問題。

D. 使用 Cloud VPN 或 Interconnect，並通過 Cloud Router 設置私有 Google 端點
這個選項是 Google 的最佳實踐之一。通過使用 Cloud VPN 或 Interconnect 創建一個私有連接，並使用 Cloud Router 宣告 199.36.153.4/30 網段（Google 的私有端點），可以確保裸機服務器無需公共 IP 也能訪問 Cloud Storage。同時，通過將 *.googleapis.com 解析為 restricted.googleapis.com，你可以確保所有流量都經由私有 Google 端點，符合安全政策要求。

推薦：這是 Google 推薦的最佳實踐，允許私有網絡訪問 Google API 和服務，且不需要公共 IP 地址。




Your team needs to directly connect your on-premises resources to several virtual machines inside a virtual private cloud (VPC). You want to provide your team with fast and secure access to the VMs with minimal maintenance and cost. What should you do?
A. Set up Cloud Interconnect.
 
B. Use Cloud VPN to create a bridge between the VPC and your network.
C. Assign a public IP address to each VM, and assign a strong password to each one.
D. Start a Compute Engine VM, install a software router, and create a direct tunnel to each VM.

A 和 B 解法都不錯，但 A 比較貴 Ｂ 是一般推薦解法



Your company has received a new project where it needs to migrate on-premise servers and data to Google Cloud gradually but until then you need to setup a VPN tunnel between on-premise and Google Cloud. Which service will you use in conjunction with Cloud VPN for a smooth setup?

A. Cloud CDN
B. Cloud NAT
C. Cloud Run
D. Cloud Router

Correct Answer: D
Option D is correct: Google Cloud Router enables you to dynamically exchange routes between your Virtual Private Cloud (VPC) and on-premises networks by using Border Gateway Protocol (BGP). The Cloud Router automatically learns new subnets in your VPC network and announces them to your on-premises network.
Option A is incorrect: Cloud CDN leverages Google’s globally distributed edge points of presence to accelerate content delivery for websites and applications served out of Google Compute Engine and Google Cloud Storage. Cloud CDN lowers network latency, offloads origins, and reduces serving costs.
Option B is incorrect: Cloud NAT enables you to provision your application instances without public IP addresses while also allowing them to access the internet for updates, patching, config management, and more in a controlled and efficient manner.
Option C is incorrect: Cloud Run is a managed compute platform that automatically scales your stateless containers.