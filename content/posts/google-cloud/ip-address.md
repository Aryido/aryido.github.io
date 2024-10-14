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
  - gcp-virtual-machine
  - gcp-network

comment: false

reward: false
---

<!--BODY-->

> GCP IP Address 經常是分配給 GCP VM Instance、GCP Load-Balancer、Cloud NAT 使用，讓他們可以和 GCP 其他雲端資源，或者是外部公共網路上的系統通訊。IP Address 種類劃分也蠻多的，會使用以下種類來描述不同的類型：
>
> - Internal IP Address <=> External IP Address
> - Private IP Address <=> Public IP Address
> - Ephemeral IP Address <=> Static IP Address
> - Regional IP Address <=> Global IP Address
>
> **在使用 IP Address 也常發生一些使用上的疏忽，例如一直 reserve IP 卻沒有使用它**，因為 IP 算是稀有資源，如果有保留固定 IP ，就算沒有使用還是會持續計費的，而且會更貴！

<!--more-->

---

可以參考下圖，了解 GCP IP Address 的基本分類關係：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/ip-address.jpg" >}}

# Internal IP Address

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/internal.jpg" >}}

Internal IP Address 是無法訪問外部 Internet 的，為 VPC 的內部 IP，承上圖也可知道 Internal IP Address 在設定時會需要指定 VPC 和 Subnet。若要自己寫指定 IP ，範圍當然要在 Subnet 範圍內，只要 Static-Internal-IP 之後，該 Address 便會從 dynamic allocation pool 中剔除，不會再被隨機分配使用直到重新釋放。

當使用 VPC Peering 之後，或者是 On-Premises 地端網路加上 Cloud VPN 、 Cloud Interconnect 等等後連上 VPC 後，因為都視為一個**相同的網路**，故都可以用 Internal IP Address 來相互通訊。

Internal IP Address 包含 Private Address，另外還多加了 Privately Used Public Address 的功能，詳細的 IP 範圍和解釋可以參考 [Valid Ranges](https://cloud.google.com/vpc/docs/subnets#valid-ranges) ; 而所有 Private Address 都是 Internal IP Address。

{{< alert warning >}}
承上敘述，可以知道基本上 Private IP Address 是 Internal IP Address 的 Subset ，因為 Internal IP Address 包含的範圍更大，這也是經常搞混的差異。
{{< /alert >}}

# External IP address

External IP Address 是公開的(advertised)，所以可以被外網 Internet 訪問。因為會分配給需要被 VPC 外部網路訪問的雲端資源，故其 Address 不能被 Private Networks 所保留，即**不能**是`RFC 1918`保留地址：

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

External IP address 可由 GCP 提供，也可以使用 [bring your own IP (BYOIP)](https://cloud.google.com/vpc/docs/bring-your-own-ip)的方式，將自己的 on-premises 地端網路 IP 位址帶到 Google。

{{< alert success >}}
GCP 的文檔描述看完後，發現其實 Public IP Address 和 External IP Address 基本是兩個一樣的敘述，可以先當成一樣的東西。
{{< /alert >}}

### Regional and global IP addresses

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/external.jpg" >}}

當我們把 GCP Project 中的 IP Addresses 列出來時， GCP 會把 Addresses 標上 Global 或者 Regional 來表示這個 IP 是如何被使用的，例如說當 IP 是被使用在一個 Regional 的 VM 時， IP 也會被標記成 Regional 像是 `us-east4` 、`europe-west2`。承上圖，也可以發現只有在 External 類型才會需要設定 Regional/Global。

# Ephemeral and Static IP Addresses

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/static-ephemeral.jpg" >}}

一般來說建立 VM 後，Internal IP 預設是會從 Subnet 中隨機賦予一個 IP 給 VM ， 這個就是 Internal IP 。 VM 在 Stop 或 Restart 都還是會 Reserve 保留著 IP，把機器刪除後才會 Release 釋出 IP。

如果 VM Instance 被刪除後再重新創建，基本上是會被分配一個全新的 Internal IP 的，如果希望 Internal IP 固定的話，可以:

- 建立並 Reserve 一個 static internal IP，然後分配給 VM

- 把一個正在使用的**臨時 internal IP**( ephemeral internal IP)，升級成 static internal IP

同理預設 VM 的 External IP 也為暫時的(Ephemeral)，同樣如果希望 External IP 固定的話，也可以把它 Static 化。
{{< alert warning >}}
刪除 vm 不會自動釋放使用者自己 Static-IP 。必須手動刪除
{{< /alert >}}

以下為 `2024/06` 查詢到的計費價格：

|           功能            |     花費     |
| :-----------------------: | :----------: |
|   外部 IP – Standard VM   | $0.005/小時  |
| Unused External Static IP | $0.0131/小時 |

{{< alert danger >}}
注意，若有保留 Static-External-IP ，卻沒有使用的話，價格還會更高喔!
{{< /alert >}}

GCP 每個 VM 創建時，都會分配一個 Internal IP，並且也可以選擇是否附加 External IP。 不管是 Internal IP 或者是 External IP 其實都可以 static 化，把它固定下來使之不會變動。

---

# Practice

> 有一個服務需要 open to everyone，若不希望當服務崩潰刪除重啟後 IP 會改變，因為這樣還需要在去 DNS server 上更改 IP 地址，而且還進一步要求不要有 downtime，那應該怎麼做才是 minimal cost and setup 呢？
>
> - A. Create a script that updates the IP address for the domain when the server crashes or is replaced.
> - B. Reserve a static internal IP address, and assign it using Cloud DNS.
> - C. Reserve a static external IP address, and assign it using Cloud DNS. **(O)**

A. 要編寫腳本來監控和更新 DNS 記錄。雖然可以自動化更新，但因為 DNS 更新不是即時生效的(TTL)，這不符合要避免 downtime 的需求。

B. static internal IP 只能在 VPC 內部使用，不能公開訪問。

C. 而 static external IP 與 Cloud DNS 結合使用，就不需要更改 DNS 記錄，這種方法提供了一個簡單、穩定的解決方案，並且避免了停機時間。

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
