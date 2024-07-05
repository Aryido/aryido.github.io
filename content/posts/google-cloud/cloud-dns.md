---
title: GCP - Cloud DNS 概述

author: Aryido

date: 2024-07-01T23:26:00+08:00

thumbnailImage: "/images/google-cloud/dns/dns-logo.jpg"

categories:
- cloud
- gcp

tags:
- dns

comment: false

reward: false
---
<!--BODY-->
> Cloud DNS 是 Google 提供的高性能、代管式的全球網域名稱系統服務，是一個**分布式的分層資料庫**，可以創建 DNS Zone 和 record 而無需自己管理 DNS Service ，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) : **Amazon Route 53**
> - Microsoft Azure : **Azure DNS**
>
> Cloud DNS 是**提供代管功能而不是註冊**，而代管的好處是有一個「共同管理維護」的介面 ; 還能「基於地理位置」將流量轉到最接近的服務器從而提高性能與速度 ; 結合「 GCP 雲端安全服務」保護應用程式免於如 DDoS 攻擊。 最後比較特別的是 Google 的 Cloud DNS 服務號稱是 [100% SLA](https://cloud.google.com/dns/sla) ，服務絕對不會中斷，一旦使用上未能達到此標準，客戶都可以申請相關補償。
<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/google-cloud/dns/dns-sla.jpg" >}}

開頭也特別提到一點， 「**Cloud DNS 是代管而不是註冊**」， 那「註冊」跟「代管」有什麼不一樣呢？

- 註冊其實就是買域名或者是租用域名，原本 Google 有一個 「Google Domain」有這種功能，但是已經被 Squarespace 於 `2023/09/07` 收購了。 

- 雖然說我們只能找別的域名供應商買域名或者是租域名，但我們可以去設定 Cloud DNS 接管服務，這就是代管。

一般來說，在哪裡買或租了域名，則默認的域名解析服務就是該服務商提供的，而 GCP 提供域名代管，例如在 GoDaddy 買了一個域名後，可以去 GCP console 設定 DNS 的資料而不要在 GoDaddy 做管理。

{{< alert warning >}}
網路域名、網域、域名都是指同一個東西，本文章都以域名來稱呼。
{{< /alert >}}

# Cloud DNS
域名註冊完後，需要設定 DNS 才能使用，接下來設定 DNS 的 Namespace ，就是 DNS Zone 。而 Cloud DNS Zone 分成兩種類型 :

- **Private zone**: 需要設置對應 VPC，其 DNS records 僅供 GCP VPC 內部可見
- **Public zone**: 在一般外網可見，故外部可以訪問，**一般來說最常使用的是這個**。

{{< image classes="fancybox fig-100" src="/images/google-cloud/dns/public-private-dns.jpg" >}}
接下來做一些簡單的名詞說明：

- ##### Zone name

  DNS 分為許多不同的 zone ，而 zone name 用來區分在「 DNS 命名空間」中以不同方式管理。一個 DNS zone 裡面會保存有相同 DNS Name Suffix。

- ##### DNS name

  在域名商那邊申請到的 「 DNS 名稱，例如：helloworld.com 」就是貼到上圖中的 **DNS name** 欄位中。

{{< alert warning >}}
常見錯誤是把 DNS zone 、 DNS Name 、 DNS Server 當成同一個東西，可參考 [Cloudflare](https://www.cloudflare.com/zh-tw/learning/dns/glossary/dns-zone/) 講解。
{{< /alert >}}

- ##### DNSSEC (DNS Security Extensions)
  是一套針對 DNS 的網際網路工程任務群組 (IETF) 的擴展，用於驗證對網域名稱尋找的回應。 DNSSEC 不會為這些查找提供隱私保護，但會阻止攻擊者操控對 DNS 請求的回應或對該回應進行 DDOS 攻擊。


# DNS Record
設定完 DNS Zone 之後，接下來就會在該 Zone 內添加 Record 紀錄。 **Record 是 DNS resource 和 domain name 之間的 Mapping**，每個 DNS Record 都有 type、 TTL(time to live)，在新增 Record 時， Cloud DNS 除了 standard 標準解析之外，還有 Routing Policy 路由政策可以進階設定。 Routing Policy 是在 DNS 查詢時的演算法，用以確定應如何解析特定的 DNS Record，有提供三種可以選擇： 「Standard」、「Weighted Round Robin」、「Geolocation」，除非有設定 Failover Routing Policy，要不然上面三個路由政策只能選一個無法組合。

{{< alert warning >}}
TTL 作用是設定每一筆紀錄在 DNS **快取伺服器**所保留的時間，所以當變更這筆 DNS 紀錄的時候，要等到你設置的 TTL 時間後才會全球生效。
{{< /alert >}}

接下來簡要介紹並列出有用過的 Cloud DNS 支援的 DNS Record 類型:

- ### A / AAAA Record
A Record 將域名映射到對應的 **IPv4** 。常用在想直接將個人網域指向一個固定 IPv4 ，其操作是將 DNS Name 例如 helloworld.com， 指向如 VM 的固定 IP 例如：123.45.67.89。而 AAAA Record 將域名映射到對應的 **IPv6**

- ### CNAME Record（Canonical Name Record）

將一個域名（或子域名）映射到另一個域名。如果想將個人網域指向另一個名稱，而不是直接指向 IP，則會新增一個 CNAME 記錄。其操作將 DNS Name 例如 helloworld.example.com 指向另一個目標 DNS Name 例如： helloworld.main.com。這樣當有人輸入 helloworld.example.com 時，DNS 解析會自動將它轉發到 helloworld.main.com。

- ### SOA (start of authority)
SOA 這種Record 用於指向 DNS Zone 的 Authoritative Name Server 權威名稱伺服器，指定該 Zone 的 administrative contact 資訊。


---

### 參考資料

- [Cloud DNS overview ](https://cloud.google.com/dns/docs/overview)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [DNS 是什麼？Google DNS 代管服務7大功能與設定法](https://blog.cloud-ace.tw/networking-website/dns/dns-and-cloud-dns-intro/)

- [GCP-設定DNS網域](https://snoopy30485.github.io/2018/06/20/GCP-%E8%A8%AD%E5%AE%9ADNS%E7%B6%B2%E5%9F%9F/)

- [GCP VM 綁定自定網址](https://medium.com/%E5%B7%A5%E7%A8%8B%E9%9A%A8%E5%AF%AB%E7%AD%86%E8%A8%98/gcp-vm-%E7%B6%81%E5%AE%9A%E8%87%AA%E5%AE%9A%E7%B6%B2%E5%9D%80-76378ebcacd7)
