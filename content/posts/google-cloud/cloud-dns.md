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

> Cloud DNS 是 Google 提供的代管式的全球 Domain Name System(網域名稱系統服務)，為一個**分布式的分層資料庫(hierarchical distributed database)** 用於存儲 IP addresses 和 Domain Name 的對應關係，還可以建立 DNS Zone 並在其下管理和創建 Record 。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Amazon Route 53**
> - Microsoft Azure : **Azure DNS**
>
> Cloud DNS 是**提供代管功能而不是註冊**，而代管的好處是有一個「共同管理維護」的介面 ; 還能「基於地理位置」將流量轉到最接近的服務器從而提高性能與速度 ; 結合「 GCP 雲端安全服務」保護應用程式免於如 DDoS 攻擊。 最後比較特別的是 Google 的 Cloud DNS 服務號稱是 [100% SLA](https://cloud.google.com/dns/sla) 服務保證絕對不會中斷， Google 對其 DNS 服務設計非常的有信心。

<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/google-cloud/dns/dns-sla.jpg" >}}

開頭也特別提到一點， 「**Cloud DNS 是代管而不是註冊**」， 那「註冊」跟「代管」有什麼不一樣呢？

> - 註冊其實就是**買域名**或者是**租用域名**，原本 Google 有一個 「Google Domain」提供了 DNS 註冊，但是於 `2023/09/07` 給了 Squarespace

> - 雖然說我們只能找別的域名供應商買域名或者是租域名，但我們可以去設定 Cloud DNS 接管服務，這就是代管

一般來說，在哪裡買或租了域名，則默認的域名解析服務就是該服務商提供的，而 GCP 提供域名代管，例如在 GoDaddy 買了一個域名後，可以去 GCP console 設定 DNS 的資料而不用在 GoDaddy 做管理。

{{< alert danger >}}
不確定是 Google Domain 被 Squarespace 收購，還是 Google 把 Google Domain 賣給 Squarespace。 但似乎從 `2024/05` 開始陸續傳出許多 [domain missing 的災情](https://forum.squarespace.com/topic/295590-missing-domain/)...
{{< /alert >}}

# Cloud DNS

域名註冊完後需要設定 DNS Server 才能使用，故接下來要先創建管理介面，一般稱為 DNS 的 Namespace ，**在 GCP 頁面上是對應到 DNS Zone**。

{{< alert warning >}}
我個人是覺得取名為 DNS Zone 蠻容易誤會的。因為在 GCP 上 Zone 通常是指地理區域的實際機房位置，例如說台灣是 `asia-east1` Region ，而其 Region 內有 Zone `asia-east1-a` 代表機房位置的代號，但 DNS Zone 和地理區域其實沒有任何關係，它是管理 DNS Record 的一個介面名稱罷了，故我覺得用到 Zone 這個名詞蠻不好的。
{{< /alert >}}

而 DNS Zone 分成兩種類型 Zone Type 或稱為 Visibility :

- **Private**: 需要設置對應 VPC，其 DNS Records 僅供 GCP VPC 內部可見
- **Public**: 在一般外網可見，故外部可以訪問，**一般來說最常使用的是這個**。

{{< image classes="fancybox fig-100" src="/images/google-cloud/dns/public-private-dns.jpg" >}}
接下來做一些 Console 上的名詞說明：

- ##### Zone name

  DNS 分為許多不同的 Zone ，其代表的是 DNS Namespace ，而 Zone name 就是代表 Namespace 的名稱。 CLoud DNS 的管理以 Zone 來劃分，一個 Zone 裡面會保存有相同 DNS Name Suffix。

- ##### DNS name

  在域名商那邊申請到的 DNS 名稱，例如是申請 `helloworld.com.` 就是貼到上圖中 GCP Console 的 **DNS name** 欄位中。

{{< alert danger >}}
注意一些特別的細微事情，以上述範例假如是申請 `helloworld.com`

1. 若在 GCP Console 在上建立 Zone 並指定 DNS name 輸入是 `helloworld.com`，在完成後看 DNS name 會發現多**加了個點在最後面**:
   `helloworld.com.`

2. DNS name 輸入是 `helloworld.com.` 有在最後面加點，在完成後看 DNS name 會保持 `helloworld.com.`

3. 若用 terraform 建立 Zone ，那在 DNS name 那邊設定一定要寫成如 `helloworld.com.` ，最後面一定要加點，要不然會直接回報錯誤
   {{< /alert >}}

- ##### DNSSEC (DNS Security Extensions)
  是一套針對 DNS 的網際網路工程任務群組 (IETF) 的擴展，用於驗證對網域名稱尋找的回應。 DNSSEC 不會為這些查找提供隱私保護，但會阻止攻擊者操控對 DNS 請求的回應或對該回應進行 DDOS 攻擊。

{{< alert warning >}}
常見錯誤是把 DNS zone 、 DNS name 、 DNS Server 當成同一個東西，可參考 [Cloudflare](https://www.cloudflare.com/zh-tw/learning/dns/glossary/dns-zone/) 講解。
{{< /alert >}}

# DNS Record

設定完 DNS Zone 之後，接下來就會在該 Zone 內添加 Record 紀錄。 **Record 是 DNS resource 和 domain name 之間的 Mapping**，每個 DNS Record 都有 :

> - type
> - TTL(time to live)

在新增 Record 時， Cloud DNS 除了 standard 標準解析之外，還有 Routing Policy 路由政策可以進階設定。 Routing Policy 是在 DNS 查詢時的演算法，用以確定應如何解析特定的 DNS Record，有提供三種可以選擇： 「Standard」、「Weighted Round Robin」、「Geolocation」，除非有設定 Failover Routing Policy，要不然上面三個路由政策只能選一個無法組合。

{{< alert warning >}}
TTL 作用是設定每一筆紀錄在 DNS **快取伺服器**所保留的時間，所以當變更這筆 DNS 紀錄的時候，要等到你設置的 TTL 時間後才會全球生效。
{{< /alert >}}

接下來簡要介紹並列出有用過的 Cloud DNS 支援的 DNS Record 類型:

- ### A / AAAA Record

  A Record 將域名映射到對應的 **IPv4** 。常用在想直接將個人網域指向一個固定 IPv4 ，其操作是將 DNS Name 例如 helloworld.com， 指向如 VM 的固定 IP 例如：`123.45.67.89`。而 AAAA Record 將域名映射到對應的 **IPv6**

- ### CNAME Record（Canonical Name Record）

  將一個域名（或子域名）映射到另一個域名。如果想將個人網域指向另一個名稱，而不是直接指向 IP，則會新增一個 CNAME 記錄。

  例如將 **DNS Name 例如 helloworld.example.com** 指向另一個目標 **DNS Name 例如： helloworld.main.com**，這樣當有人輸入 helloworld.example.com 時，DNS 解析會自動將它轉發到 helloworld.main.com。

- ### SOA (start of authority)
  SOA 這種 Record 用於指向 DNS Zone 的 Authoritative Name Server 權威名稱伺服器，指定該 Zone 的 administrative contact 資訊。

---

### 參考資料

- [Cloud DNS overview ](https://cloud.google.com/dns/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [DNS 是什麼？Google DNS 代管服務 7 大功能與設定法](https://blog.cloud-ace.tw/networking-website/dns/dns-and-cloud-dns-intro/)

- [GCP-設定 DNS 網域](https://snoopy30485.github.io/2018/06/20/GCP-%E8%A8%AD%E5%AE%9ADNS%E7%B6%B2%E5%9F%9F/)

- [GCP VM 綁定自定網址](https://medium.com/%E5%B7%A5%E7%A8%8B%E9%9A%A8%E5%AF%AB%E7%AD%86%E8%A8%98/gcp-vm-%E7%B6%81%E5%AE%9A%E8%87%AA%E5%AE%9A%E7%B6%B2%E5%9D%80-76378ebcacd7)
