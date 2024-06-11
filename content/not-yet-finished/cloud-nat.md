---
title: Cloud NAT 概述

author: Aryido

date: 2024-05-30T18:52:24+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud
- gcp

tags:

comment: false

reward: false
---
<!--BODY-->
>  

<!--more-->

---





# Router
提供 VPC 網路的動態且可擴展路由功能
允許建立 VPN 或專用網路來連接 GCP VPC 網路和地端網路環境。

#  Cloud NAT
將內部私有網路的 IP 位址轉譯為外部公開網路的 IP 位址，以實現內部資源與外部網路通訊的功能。

NAT 技術在 GCP 中允許私有網路中的資源（通常是 VM 或 container）使用特定的 NAT gateway 將其內部私有 IP 位址轉譯為一個公開 IP 位址，以便與外部網路進行通訊。

Dynamic NAT 將內部資源的 IP 位址 mapping 到單個公開 IP 位址，而 Static NAT 則可將特定的內部 IP 位址轉換為指定公開 IP 位址。

多個內部資源能共享同一個公開 IP 位址。


# Firewall

流量控制：網路防火牆允許用戶根據特定的規則和條件來控制進出 VPC 的網路流量。用戶可設定規則來允許或拒絕特定 IP 位址、協議、port 或特定類型的流量。

安全策略：防火牆允許用戶根據特定的安全策略設定訪問控制，例如根據需求設置白名單或黑名單，從而限制特定 IP 位址或 IP 位址範圍的訪問。

應用層防火牆，能夠檢測和阻止特定 applications 或協議的流量，提高安全性。



VPC 網路規則：防火牆設置基於 VPC 網路規則，這些規則允許用戶對特定的 VPC 子網路設定安全規則，從而控制和管理安全性。



日誌和監控：GCP 的防火牆提供了監控和日誌記錄功能，用戶能查看和分析網路流量，檢測可能的安全風險或攻擊，並及時採取相應的措施。



---

### 參考資料


- [資訊科普系列(14) — Google Cloud Platform Networking（一）](https://medium.com/moda-it/google-cloud-platform-networking-%E7%B0%A1%E4%BB%8B-b0b2ec2ff7be)



- [[GCP 教學] 043 2小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)