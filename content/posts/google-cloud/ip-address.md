---
title: GCP - IP addresses 概述

author: Aryido

#first-date: 2022-11-26T23:20:50+08:00
date: 2024-06-02T14:51:14+08:00

thumbnailImage: "/images/google-cloud/network/vpc-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - virtual-machine
  - vpc

comment: false

reward: false
---

<!--BODY-->

> GCP IP Address 經常是分配給 GCP VM Instance 和 GCP Load-Balancer 使用，讓他們可以和 GCP 其他雲端資源，或者是外部公共網路上的系統通訊。IP Address 劃分也蠻多的，會使用以下種類來描述不同的類型：
>
> - Internal IP Address <=> External IP Address
> - Private IP Address <=> Public IP Address
> - Ephemeral IP Address <=> Static IP Address
> - Regional IP Address <=> Global IP Address
>
> 此篇會全部做簡單的介紹。而在使用 IP Address 也常發生一些使用上的疏忽，例如一直 reserve IP 卻沒有使用它，因為 IP 算是稀有資源，如果有保留固定 IP ，就算沒有使用還是會持續計費的，而且會更貴！

<!--more-->

---

可以參考下圖，了解 GCP IP Address 的基本分類關係：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/ip-address.jpg" >}}

# Internal IP Address

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/internal.jpg" >}}

Internal IP Address 是無法訪問外部 Internet 的，為 VPC 的內部 IP，承上圖也可知道 Internal IP Address 在設定時會需要指定 VPC 和 Subnet。弱勢自己寫指定 IP ，範圍當然要在 subnet 範圍內，只要 static internal IP 之後，該 address 便會從 dynamic allocation pool 中剔除，不會再被隨機分配使用直到重新釋放。

當使用 VPC Peering 之後；或者是 on-premises 地端網路加上 Cloud VPN 、 Cloud Interconnect 等等後連上 VPC 後，因為都視為一個**相同的網路**，故都可以用 Internal IP Address 來相互通訊。

Internal IP Address 包含 private Address，另外還多加了 privately used public Address 的功能，詳細的 IP 範圍和解釋可以參考 [Valid Ranges](https://cloud.google.com/vpc/docs/subnets#valid-ranges) ; 而所有 private Address 都是 Internal IP Address。

{{< alert success >}}
承上敘述，可以知道基本上 Private IP Address 是 Internal IP Address 的 Subset ，因為 Internal IP Address 包含的範圍更大，這也是經常搞混的差異。
{{< /alert >}}

# External IP address

External IP address 是 advertised 公開的，所以可以被外網 internet 訪問。因為會分配給需要被 VPC 外部網路訪問的雲端資源，故其 address 不能被 private networks 所保留，即不能是`RFC 1918`保留地址：

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

External IP address 可由 GCP 提供，也可以使用 [bring your own IP (BYOIP)](https://cloud.google.com/vpc/docs/bring-your-own-ip)的方式，將自己的 on-premises 地端網路 IP 位址帶到 Google。

{{< alert success >}}
GCP 的文檔描述看完後，發現其實 Public IP Address 和 External IP Address 基本是兩個一樣的敘述，可以先當成一樣的東西。
{{< /alert >}}

### Regional and global IP addresses

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/external.jpg" >}}

當我們把 GCP Project 中的 IP addresses 列出來時， GCP 會把 addresses 標上 global 或者 regional 來表示這個 IP 是如何被使用的，例如說當 IP 是被使用在一個 regional 的 VM 時， IP 也會被標記成 regional 像是 `us-east4` 、`europe-west2`。承上圖，也可以發現只有在 External 類型才會需要設定 Regional and global。

# Ephemeral and Static IP Addresses

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/static-ephemeral.jpg" >}}

一般來說建立 VM 後，Internal IP 預設是會從 Subnet 中隨機賦予一個 IP 給 VM ， 這個就是 Internal IP 。它在 Stop 或 Restart 都還會 Reserve 保留著，把機器刪除後就會 Release 釋出。

如果 vm instance 被刪除後再重新創建，基本上是會被分配一個新的 Internal IP 的，如果希望 Internal IP 固定的話，可以:

- 建立一個並 Reserve 一個 static internal IP，然後分配給 VM

- 把一個正在使用的**臨時 internal IP**( ephemeral internal IP)，升級成 static internal IP

同理預設 VM instance 的 External IP 也為 Ephemeral 暫時的，同樣如果希望 External IP 固定的話，也可以把它 static 化。
{{< alert warning >}}
刪除 vm 不會自動釋放 staticIP 位址。必須手動刪除
{{< /alert >}}

以下為 `2024/06` 查詢到的計費價格：

|           功能            |     花費     |
| :-----------------------: | :----------: |
|   外部 IP – Standard VM   | $0.005/小時  |
| Unused External Static IP | $0.0131/小時 |

{{< alert danger >}}
注意，若有保留 static external IP ，卻沒有使用的話，價格還會更高喔!
{{< /alert >}}

GCP 每個 vm instance 創建時，都會分配一個 internal IP，並且也可以選擇是否附加 external IP。 不管是 internal IP 或者是 external IP 其實都可以 static 化，把它固定下來使之不會變動。

---

# Vocabulary

{{< alert info >}}
**ephemeral**[ɪˋfɛmərəl] :

adj.短暫的

ephemeral IP 是臨時 ip 的意思，和 ethereal [ɪˋθɪrɪəl] 意思相近。

{{< /alert >}}

{{< alert info >}}
**on-premises**[ˋprɛmɪsɪz] :

n. 地端

相反是雲端 cloud，不要唸成 promise [ˋprɑmɪs] 承諾

{{< /alert >}}

---

### 參考資料

- [vm IP addresses office doc](https://cloud.google.com/compute/docs/ip-addresses)

- [vpc IP addresses office doc](https://cloud.google.com/vpc/docs/ip-addresses)

- [What are the two types of IP addresses in the context of Google Cloud?](https://eitca.org/cloud-computing/eitc-cl-gcp-google-cloud-platform/gcp-networking/ip-addresses/examination-review-ip-addresses/what-are-the-two-types-of-ip-addresses-in-the-context-of-google-cloud/)

- [A Comprehensive Guide to IP Addressing and Firewall Rules on Google Cloud Platform](https://medium.com/@srivastavayushmaan1347/a-comprehensive-guide-to-ip-addressing-and-firewall-rules-on-google-cloud-platform-202f903fc397)

- [How-to Guides GCP FinOps — 閒置資源清理任務 Part1](https://medium.com/@kellenjohn175/how-to-guides-gcp-finops-%E9%96%92%E7%BD%AE%E8%B3%87%E6%BA%90%E6%B8%85%E7%90%86%E4%BB%BB%E5%8B%99-part1-166905733811)
