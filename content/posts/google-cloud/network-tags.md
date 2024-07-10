---
title:  GCP - Network Tag 概述

author: Aryido

date: 2024-06-03T22:59:14+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

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
> Network Tag 在 GCP 中，只是一個簡單的字符串標示並不會建立出雲端資源，**會簡稱為 Tag** 並可選擇附加到如 VM 或 Instance template 上，其設計想法上是可以由這個標示，更有效地控制和管理 VM 的網路防火牆安全設定。 Network Tag 算是 GCP 比較特別的設計，其他雲端似乎沒有比較類似的對應，由於不是一個獨立的 cloud resource ，所以是無法單獨建立 Tag 的，但對於其關聯的 GCP Firewall Rules ，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) :  **Security Groups**
> - Microsoft Azure : **Network security groups**
> 
> 特別要注意的事情是在 GCP 中，Tag 和 Label 是不一樣的東西。通常 GCP 提到 tag 都是指 network tag 這個網路安全相關防火牆設定 ; 而 Label 是拿來作預算及資源管理使用。
<!--more-->

---

當我們在開 VM 時，會看到一些選項如下圖，可以選擇打勾或不打勾 HTTP 、 HTTPS，這個就是關於 Firewall-Rules 的設定，而 Firewall-Rules 和 VM 怎麼關聯的，**就是使用 Tag 的方式來關聯 VM**：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/firewallrule-networktag.jpg" >}}

# Network Tag 範例
以上圖為範例，當我們打勾後創建完成 VM ，接下來可以去 VPC Firewall 頁面看一下，會發現有創建一些新的 Firewall-Rules，其名稱會以 VM 所在的 VPC 名字為 prefix 命名 Firewall-Rule ，如下範例`{VPC_NAME}-allow-http` 、 `{VPC_NAME}-allow-https` 。再來會發現在 Firewall-Rule 內的 target tags 內有 `http-server`、`https-server` ，如果該 VM 的 Network Tags 有一樣的標籤值就會套用對應的防火牆設定。

{{< alert success >}}
GCP 採用的是「標記機制」來對各個 VM 套用防火牆規則。比如說有十台機器，有三台是 frontend、四台是 backend、三台是 DB ，就可以用 Network Tag 去做整個群組化的管理，，可以針對虛擬機們進行分組，分別客製化 VM群體 對應的 firewall rules，不用每一台機器都單獨去設防火牆規則。
{{< /alert >}}

# Specifications 規格

- ##### Network Tag 的名稱長度限制在 63 characters 以內，必須以**小寫字母開頭**，可以包含數字和 `-`，且必須以**小寫字母或數字結尾**，不能是大寫字母

- ##### 每台 VM 最多標上 64 個 Network Tags

- ##### Permissions 方面，同時需要注意兩部分：
    - 針對**當建立 vm 時，把 Tag 付上或者刪除**，需要 `Instance Admin` 權限
    - 針對 **firewall rules 的 CRUD 操作**，需要 `Security Admin` 權限


# Considerations 注意事項

### 1. Network Tag 的作用範圍
假設有 VPC A 和 VPC B 是 Peering 並且有：

> - VPC A 中有一個 VM instance-1，添加了 Network Tag `web-server`
> - VPC A 的 firewall rule 的 Target tags 也有 `web-server`，且簡單設定為接受所有 port 80 HTTP 流量進來

即使 VPC A 和 VPC B 是 Peering，這條防火牆規則也只在 VPC A 有效，VPC B 中的實例不會受到這條規則的影響。VPC Peering 後各個網路依然會保持各自的獨立性，**Network Tag 不會跨越 VPC Peering 生效**。

### 2. 一些 Network Tag 的更動會發生 Propagation Delay
關於 Propagation Delay **只會發生在更動 source tag 上**，但大部分都只要等個幾秒鐘就可以了，少部分可能會要等幾分鐘，對於一般 IP 防火牆的設定 Network Tag，基本上是立即生效的。

{{< alert info >}}
source tag 是什麼呢？

我們在建立 VM 時，可以替 VM 設定一個 Network Tag，這個如果沒有和任何已經存在的 firewall rule 的 target tag 對應到也沒關係。 再來我們可以再建立一個 new firewall rule，這時選擇 ingress 流量選擇指定 source tag ，若這個 source tag 和 VM 中的 Network Tag 對應上的話，就能讓該 VM 能夠發送流量到目標 VM。當防火牆規則使用 source tag 時，只有帶有這些 Network Tag 的 VM 的流量會被考慮目標 VM 考慮。
{{< /alert >}}

---

### 參考資料

- [Add network tags](https://cloud.google.com/vpc/docs/add-remove-network-tags)

- [【Explanation】GCP 新手村 — Firewall](https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-firewall-39cd71353b1)


- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)