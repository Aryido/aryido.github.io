---
title:  GCP - Network Tag 概述

author: Aryido

date: 2024-06-02T22:59:14+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud
- gcp

tags:
- vpc

comment: false

reward: false
---
<!--BODY-->
> 
<!--more-->

---

當我們在開 VM 時，會看到一些選項如下圖，可以選擇打勾或不打勾 HTTP 、 HTTPS，這個就是關於 Firewall-Rules 的設定，**是使用 tag 的方式來關聯 VM**：
{{< image classes="fancybox fig-100" src="/images/google-cloud/network/firewallrule-networktag.jpg" >}}
以上圖範例，當我們打勾後創建完成 VM ，接下來可以去 VPC Firewall 頁面看一下，會發現有創建一些新的 Firewall-Rules，其名稱會以 VM 所在的 VPC 名字為 prefix 命名 Firewall-Rule ，如下範例`{VPC_NAME}-allow-http` 、 `{VPC_NAME}-allow-https` 。

再來會發現在 Firewall-Rule 內的 target tags 內有 `http-server`、`https-server` ，這會對應 VM 內的 network tags。如果該 VM 的 network tags 有一樣的標籤值如 `http-server`、`https-server`，就會套用對應到的防火牆設定。


GCP 採用的是「標記機制」來對各個 VM 套用防火牆規則

# Network Tag

網路標記（Network Tags）

GCP 防火牆的管理可以用標籤來做管理

比如說我有十台機器好了
有三台是 AP 有三台是 DB 然後有四台是 WEB
我們可以用標籤來去管理很多台機器 你不用每一台機器都
單獨去設防火牆規則 你也不用一個一個 IP 去設 你可以用標籤去做整個群組化的管理

{{< alert info >}}
在 GCP 中，Tag 和 Label 是不一樣的東西。通常 GCP 提到 tag 都是指網路相關標記如 network tag ;而 Label 它是 key-value，拿來作預算及資源管理使用的。
{{< /alert >}}





流量控制：網路防火牆允許用戶根據特定的規則和條件來控制進出 VPC 的網路流量。用戶可設定規則來允許或拒絕特定 IP 位址、協議、port 或特定類型的流量。

安全策略：防火牆允許用戶根據特定的安全策略設定訪問控制，例如根據需求設置白名單或黑名單，從而限制特定 IP 位址或 IP 位址範圍的訪問。




---

### 參考資料

- [【Explanation】GCP 新手村 — Firewall](https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-firewall-39cd71353b1)

- [Add network tags](https://cloud.google.com/vpc/docs/add-remove-network-tags)

- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)


- [資訊科普系列(14) — Google Cloud Platform Networking（一）](https://medium.com/moda-it/google-cloud-platform-networking-%E7%B0%A1%E4%BB%8B-b0b2ec2ff7be)






https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-vpc-48aeb7198e02


https://jason-kao-blog.medium.com/google-cloud-%E7%B6%B2%E8%B7%AF%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-d6b74c178714