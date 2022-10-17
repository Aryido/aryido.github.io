---
title: AWS load-balancer 基礎介紹

author: Aryido

date: 2022-10-17T22:00:06+08:00

thumbnailImage: "/images/aws/logo.jpg"

categories:
- cloud

tags:
- aws

comment: false

reward: false
---
<!--BODY-->
> AWS 目前有多種  Load Balancing
> - Application Load Balancer
> - Network Load Balancer
> - Classic Load Balancer
>
> 對於 Classic Load Balancer ，除非還有 ec2 運行在 ec2-classic 網路的場景，要不然已經不建議在使用了，建議使用 Application Load Balancer 、 Network Load Balancer 取代。
>
<!--more-->

---

需要彈性的應用程式管理，建議使用 Application Load Balancer。如果應用程式需要絕佳的效能和靜態 IP，建議使用 Network Load Balancer。

# [產品比較](https://aws.amazon.com/tw/elasticloadbalancing/features/)

|     功能     |    ALB      |    NLB     |
| :----------: |:----------:|:----------:|
|     協議     | HTTP、HTTPS |   TCP、UDP  |
|靜態IP、彈性IP |     X      |       V     |

---

# Application Load Balancer 架構
ALB 可以依照不同的 Rule 分配到不同內部的 EC2 Group，提供了更有彈性的機制。支持基於**路徑**、**主機**的路由。
{{< image classes="fancybox fig-100" src="/images/aws/alb.jpg" >}}

## Component

- Listener：通常用來定義 forwards 請求的規則。

{{< alert info >}}
定義 Load Balancer 要監聽的 Protocol、port。
{{< /alert >}}

{{< alert info >}}
Listener 內定義 Routing rules，負責要怎麼對應到 target Group。
{{< /alert >}}

---

- Health Check：

{{< alert info >}}
用來確認 EC2 還活著Load Balancer 才知道可以送請求過去。
{{< /alert >}}

---

- Target group: 統一管理註冊的 instance

{{< alert info >}}
Target group 會使用 Listener 指定的 rule ，將請求路由至一個或多個已註冊的目標。
{{< /alert >}}

{{< alert info >}}
每個 Target group 內設定 Health Check ，會對 Target group 內的所有目標檢查運作狀態。
{{< /alert >}}

---

- **Auto Scaling Group**

{{< alert info >}}
當完成 Launch Configuration 設定後，設定擴展的 policy 即設定擴展時使用的 AZ 與綁定的 ELB 與 Healthy check。
{{< /alert >}}
{{< alert info >}}
Auto Scaling Group 會被關連到 Target group，使 Auto Scaling 可以在 target group 自動擴展和管理 instances。
{{< /alert >}}


---