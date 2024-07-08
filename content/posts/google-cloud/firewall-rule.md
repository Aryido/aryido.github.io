---
title:  GCP - VPC Firewall Rules 概述

author: Aryido

date: 2024-06-03T22:59:14+08:00

thumbnailImage: "/images/google-cloud/network/firewall-rules-logo.jpg"

categories:
- cloud
- gcp

tags:
- network
- vpc

comment: false

reward: false
---
<!--BODY-->
> GCP Firewall 提供精細的安全控制機制的雲端資源，可以讓資源管理者保護其 VPC 內資料，不會收到未經授權的訪問或者意外流出資料，從而提高安全性和隱私性。 GCP 防火牆其實是一個蠻大的類別，產品全稱是 Cloud Next Generation Firewall 簡稱 Cloud NGFW，其中可分成：「 Cloud NGFW Essentials 」、「 Cloud NGFW Enterprise 」、「 Cloud NGFW Standard 」。但通常在我們在 GCP 提到的防火牆，其實都是指最常用的 **Firewall-Rules 防火牆規則** ，隸屬於 Cloud NGFW Essentials，只能應用在給定的 project 和指定的 VPC，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) :  **Security Groups**
> - Microsoft Azure : **Network security groups**
>
> 如果想要把 Firewall-Rules 應用到 organization 下的其他 project 或者其他 VPC，則要使用 Firewall-Policies，本篇重點介紹 Firewall-Rules。
<!--more-->

---

一開始 project 啟用，通常都有預設 VPC 叫 default，同時也有預設的 Firewall-Rules ，實際從 GCP Web UI 看到的畫面如下：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/default-firewall-rule.jpg" >}}
接下來就點進去看這幾個 Firewall-Rules 設定來學習一下基本觀念。
{{< alert warning >}}
**建議不要直接使用預設的 Firewall-Rules**，應該可以明顯發現有 IP Range 是 `0.0.0.0/0` 的設定，這會允許任何連線，十分不安全。若是 VM 有使用這個防火牆，可以到 VM 內的 /var/log/auth.log 查詢登入，應該可以發現很多不明的 IP 嘗試連線...
{{< /alert >}}

{{< alert info >}}
default-allow-icmp：用於 ping

default-allow-rdp：用於 Windows 遠端桌面協議的流量
{{< /alert >}}

# Firewall Rules
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/firewall-rule-setting.jpg" >}}
VPC firewall rules 是定義在 VPC 上的，這時 VPC 其實也是作為**在網路級別定義的分散式防火牆**，對環境中進行流量輸入與輸出的管控。對於每個 Rule 還可以**啟用 Log** ，監控每一條有符合的封包，以後要做 Troubleshooting 的時候可以協助做故障排除。除此之外，也能查看和分析網路流量，檢測可能的安全風險或攻擊，並及時採取相應的措施。
{{< alert warning >}}
本身 Firewall Rules 是不計費的，但打開 Firewall Rules Logging 會產生額外花費。
{{< /alert >}}
Firewall Rules 有幾個比較重要的觀念在這裡提出來：

### Actions & Directions 
Actions 就只分成 `Allow`、`Deny` ，允許或拒絕這兩種。而 Direction 分成 `傳入（Ingress）`或`傳出（Engress）`流量，個別都只能二選一，不能同時選擇兩者。對於 Ingress 和 Egress 的定義，簡言之：
- **進到 VM 的流量都叫做 Ingress**
- **而離開 VM 的流量叫做 Egress** 

{{< alert success >}}
Ingress 和 Egress 的主體都是 VM。

- 不管是從外網 Internet 傳進來 VM 的流量，或者從內網其他資源傳進 VM 的流量，對於該 VM 都叫 Ingress。

- 若是說從這台 VM 傳送流量出去外網 Internet ，或者是傳送流量到內網的其他主機，對於該 VM 都叫 Egress。
{{< /alert >}}


### Target
設定  firewall rules 時，在前面的部分已經確定了**Actions & Directions**，接下來是決定  Target & filters。 Targets 可以想成是代表**要把 firewall 用在哪個對象上**，每個 Firewall-Rule 都必須要設定 Target，分成： 
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/targets.jpg" >}}
選擇 `All instances in the network`，就不用寫 target tag 名稱了，因為是直接幫我們把 firewall rule 生效在該 vpc 內所有的 VM 上。 

### filters
接下來是關於 filters 的設定，這裏會設定更細部的 IP-Range、協議、port 或 service-account。前面指定的不同的 Directions 就會對應不同的 filter :

- Ingress 是對應 **Source filters** ，它可以是單個 filter 或有多個 filters，但是多個的話只能是：
    -  `IP-Range + source tags`
    -  `IP-Range + source service-account`

        {{< alert danger >}}
特別注意 Source filters **不能同時使用 tag 和 service-account**。 
{{< /alert >}}


- Egress 是對應 **Destination filter**，從 Web UI 上只有看到 `IP-Range` 能使用。

### Priority
Priority 的值設定範圍是`0–65535`，數字越小優先度越高，**若 priority 一樣， Deny rules take precedence over Allow Rule**，已禁止為優先。 每個 VPC 網路都有兩個**雖然我們看不到但是為預設的規則**的 IPv4 防火牆規則，如果 IPv6 啟用，該 VPC 還會再多有兩個隱含的 IPv6 防火牆規則，隱含規則為:
- Allow all egress
- Deny all ingress

默認情況下阻止所有流量進入，允許所有流量出去，隱含的規則無法刪除。

{{< alert warning >}}
通常 GCP 防火牆有使用的話，主機本身就不要再設定自己的防火牆，萬一網路不通有問題的時候，會比較難 Troubleshooting。
{{< /alert >}}

# Best Practice
- 實施 Least-Privilege Principles 原則
- 考慮先使用  hierarchical firewall policy rules 來禁止流量
- 考慮使用 service account 來允許進入 VM 
- 根據 IP-Range 創建的 rules，盡量不要太多個
- 考慮使用只允許來自 LB 的流量進入 VM
- 從 source IP 範圍中刪除 `0.0.0.0/0`
- **新增 `130.211.0.0/22` 和 `35.191.0.0/16`，為 health-check 網路。**

---

### 參考資料

- [VPC firewall rules](https://cloud.google.com/firewall/docs/firewalls)

- [Google Cloud — 網路服務簡介](https://jason-kao-blog.medium.com/google-cloud-%E7%B6%B2%E8%B7%AF%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-d6b74c178714)

- [GCP 新手村 — Firewall](https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-firewall-39cd71353b1)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)