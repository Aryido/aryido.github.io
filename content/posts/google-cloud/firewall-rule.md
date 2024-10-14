---
title: GCP - Firewall Rules 概述

author: Aryido

date: 2024-06-03T22:59:14+08:00

thumbnailImage: "/images/google-cloud/network/firewall-rules-logo.jpg"

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

> 通常在我們在 GCP 提到的防火牆，都是指最常用的 **Firewall-Rules 防火牆規則** ，只能應用在給定的 Project 和指定的 VPC 上，其可以讓資源管理者保護其 VPC 內服務的資料，不會收到未經授權的訪問或者意外流出資料，從而提高安全性和隱私性。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Security Groups**
> - Microsoft Azure : **Network security groups**
>
> GCP 防火牆其實是一個蠻大的類別，產品全稱是 Cloud Next Generation Firewall 簡稱 Cloud NGFW，而 Firewall-Rules 隸屬於其中的 Cloud NGFW Essentials 。如果想要把 Firewall-Rules 應用到 Organization 下的其他 Project 或者其他 VPC，則要使用 Firewall-Policies，本篇重點介紹 Firewall-Rules。

<!--more-->

---

Cloud NGFW，其中分成：

- Cloud NGFW Essentials
- Cloud NGFW Enterprise
- Cloud NGFW Standard

一開始 project 啟用，通常都有預設 VPC 叫 default，同時也有預設的 Firewall-Rules ，實際從 GCP Web UI 看到的畫面如下：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/default-firewall-rule.jpg" >}}

接下來可以點進去看這幾個 Firewall-Rules 設定來學習一下基本觀念 :

{{< alert warning >}}
**建議不要直接使用預設的 Firewall-Rules**，應該可以明顯發現有 IP Range 是 `0.0.0.0/0` 的設定，這會允許任何連線，十分不安全。若是 VM 有使用這個防火牆，可以到 VM 內的 /var/log/auth.log 查詢登入，應該可以發現很多不明的 IP 嘗試連線...
{{< /alert >}}

{{< alert info >}}
default-allow-icmp：用於 ping

default-allow-rdp：用於 Windows 遠端桌面協議的流量
{{< /alert >}}

# Firewall Rules

{{< image classes="fancybox fig-100" src="/images/google-cloud/network/firewall-rule-setting.jpg" >}}

Firewall-Rule 是定義在 VPC 上的，這時 VPC 就有了**在網路級別定義的分散式防火牆**功能，可對環境中進行流量輸入與輸出的管控。對於每個 Rule 還可以**啟用 Log** ，監控每一條有符合的封包，以後要做 Troubleshooting 的時候可以協助做故障排除。除此之外，也能查看和分析網路流量，檢測可能的安全風險或攻擊，並及時採取相應的措施。
{{< alert warning >}}
本身 Firewall Rules 是不計費的，但打開 Firewall Rules Logging 會產生額外花費。
{{< /alert >}}

Firewall-Rule 有幾個比較重要的觀念在這裡提出來：

### Actions & Directions

Actions 就分成

> `Allow`、`Deny` ，代表允許或拒絕這兩種動作

而 Directions 分成

> `傳入（Ingress）`或`傳出（Engress）`，代表流量方向

對於 Ingress 和 Egress 的定義，簡言之：

- **進到 VM 的流量都叫做 Ingress**
- **而離開 VM 的流量叫做 Egress**

{{< alert success >}}
Ingress 和 Egress 的主體都是 VM。

- 不管是從外網 Internet 傳進來 VM 的流量，或者從內網其他資源傳進 VM 的流量，對於該 VM 都叫 Ingress。

- 若是說從這台 VM 傳送流量出去外網 Internet ，或者是傳送流量到內網的其他主機，對於該 VM 都叫 Egress。
  {{< /alert >}}

### Targets & Filters

Targets 可以想成是代表**要把 firewall 用在哪個服務上**，每個 Firewall-Rule 都必須要設定 Target，其類型有分成：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/targets.jpg" >}}

如果選擇 `All instances in the network`，就不用在指定寫 Target-Tag 的名稱了，因為是直接幫我們把 Firewall-Rule 生效在該 vpc 內所有的 VM 上。

接下來是關於 filters 的設定，這裏會設置更細部的 IP-Range、協議、port 或 service-account。前面指定的不同的 Directions 就會對應不同的 filter :

- Ingress 是對應 **Source filters** ，它可以是單個 filter 或有多個 filters，但是多個的話只能是：

  - `IP-Range + source tags`
  - `IP-Range + source service-account`

  {{< alert danger >}}
  特別注意 Source filters **不能同時使用 tag 和 service-account**
  {{< /alert >}}

- Egress 是對應 **Destination filter**，從 Web UI 上只有看到 `IP-Range` 能使用

### Priority

Priority 的值設定範圍是 `0–65535` ，數字越小優先度越高，也就是`0`是最高優先級，然後`65535`是最小最後的。**若 priority 一樣， Deny rules take precedence over Allow Rule**，已禁止為優先。每個 VPC 網路都有兩個**雖然我們看不到但是為預設的規則**的 IPv4 防火牆規則，如果 IPv6 啟用，該 VPC 還會再多有兩個隱含的 IPv6 防火牆規則，隱含規則為:

> - Allow all egress
> - Deny all ingress

默認情況下阻止所有流量進入，允許所有流量出去，隱含的規則無法刪除。

{{< alert warning >}}
通常 GCP 防火牆有使用的話，主機本身就不要再設定自己的防火牆，萬一網路不通有問題的時候，會比較難 Troubleshooting。
{{< /alert >}}

# Best Practice

- 實施 Least-Privilege Principles 原則
- 考慮先使用 hierarchical firewall policy rules 來禁止流量
- 考慮使用 service account 來允許進入 VM
- 根據 IP-Range 創建的 rules，盡量不要太多個
- 考慮使用只允許來自 LB 的流量進入 VM
- 從 source IP 範圍中刪除 `0.0.0.0/0`
- **新增 `130.211.0.0/22` 和 `35.191.0.0/16`，為 health-check 網路。**

通常如果發生了 Firewall 阻擋了請求，發生的情況會是**根本無法收到任何 response**，會導致 timeout 。所以如果有收到例如說 `403 Forbidden` 的 response ，這可能就和 Firewall 阻擋沒有關係了。

---

# Practice

> Compute Engine 上創建了一個 **SQL Server 2017 instance** 來測試新版本的功能，如何連接到此 instance ?

會需要在 GCP Console 中設置 :

- Windows 的 username 和 password，這是連接 Windows VM 必要的步驟
- 然後要**檢查** Firewall-Rule 有沒有開啟 `3389` port，因為 RDP(遠程桌面協議)使用的是 `3389` port，而不是 `22` port， SSH 才是使用 `22` port。

{{< alert info >}}
default VPC 是有預設 Firewall-Rule 開啟 port 3389 的
{{< /alert >}}

# Practice

> Project 內有 5 個 VPC，每個都有自己的 CIDR 範圍和防火牆規則。如果要列出 `network-3` 的 firewall rules ，以下哪個正確？
>
> - A. `gcloud compute firewall-rules list –filter network=network-3` **(O)**
> - B. `gcloud vpc network=network-3 –list firewall-rules`
> - C. `gcloud compute network=network-3 –list firewall-rules`
> - D. `gcloud vpc firewall-rules list –filter network=network-3`

此題非常陷阱，也看得出來 Google 在這邊的分類還蠻混亂的。從文件上來看， firewall-rules 屬於 Cloud NGFW(Next Generation Firewall)看起來是個新的類別，但順著看下去是文件名稱是 [VPC firewall rules](https://cloud.google.com/firewall/docs/firewalls#best_practices_for_firewall_rules)，乍看之下是屬於 VPC 類別，而且從 console UI 上我們也是從 VPC Network 類別下找到 Firewall 的。但是從 `gcloud cli` 正確寫法是：[`gcloud compute firewall-rules`](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules)，這種只能特別記憶一下了...

# Practice

> 已經設置了 firewall rule ，且允許入站 permit inbound 連接到名為 `my-vm` 的 instance。若希望**只有在沒有其他規則拒絕該流量時才應用此規則**，即： apply this rule only if there is not another rule that would deny that traffic ，那應該給這個 rule 設置什麼優先級？
>
> - A. 1000
> - B. 1
> - C. 65535 **(O)**
> - D. 0

C. 首先要知道 Priority 的值設定範圍是`0–65535`且數字越小優先度越高。故使用`65535`代表最低的優先級，即是這個 rule 最後才會應用，只有在沒有其他更高優先級的規則時才會使用這個規則。

---

### 參考資料

- [VPC firewall rules](https://cloud.google.com/firewall/docs/firewalls)

- [Google Cloud — 網路服務簡介](https://jason-kao-blog.medium.com/google-cloud-%E7%B6%B2%E8%B7%AF%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-d6b74c178714)

- [GCP 新手村 — Firewall](https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-firewall-39cd71353b1)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)
