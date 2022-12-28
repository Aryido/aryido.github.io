---
title: AWS load-balancer 基礎介紹

author: Aryido

date: 2022-10-17T22:00:06+08:00

thumbnailImage: "/images/aws/logo.jpg"

categories:
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
> 對於 Classic Load Balancer ，除非還有 ec2 運行在 ec2-classic 網路的場景，要不然已經**不建議**使用了，建議使用 Application Load Balancer 、 Network Load Balancer 取代。
>
<!--more-->

---

# [功能比較](https://aws.amazon.com/tw/elasticloadbalancing/features/)

|     功能     |    ALB      |    NLB     |
| :----------: |:----------:|:----------:|
|     協議     | HTTP、HTTPS、gRPC |   TCP、UDP  |
|靜態IP、彈性IP |     X      |       V     |
|重新導向       |    V      |      X    |
|固定回應       |    V      |      X    |
|HTTP 路由       |    V      |      X    |

目前看起來，如果發現應用程式需要**靜態 IP**，建議使用 Network Load Balancer ，他是唯一有提供的。至於在 Layer 7 應用層的功能，如**重新導向**、**固定回應**、依照不同的 Rule ，基於**路徑**、**主機**分配到不同的 target Group，則只能使用 Application Load Balancer。

---

# Load Balancer 架構

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
用來確認 ec2 還活著，這樣 Load Balancer 才知道可不可以送請求過去。
{{< /alert >}}

---

- Target group: 統一管理註冊的 instance

{{< alert info >}}
Target group 會使用 Listener 指定的 rule ，將請求路由至一個或多個已註冊的 instance。
{{< /alert >}}

{{< alert info >}}
每個 Target group 內設定 Health Check ，會對 Target group 內的所有目標 instances 檢查運作狀態。
{{< /alert >}}

---

- **Auto Scaling Group**

{{< alert info >}}
當完成 Launch template 設定後，就可以確定生出來的機器的型號，和裡面的一些軟體。再來設定擴展的 policy ，和設定擴展時使用的 AZ 與綁定的 ELB 與 Healthy check。
{{< /alert >}}
{{< alert info >}}
Auto Scaling Group 會被關連到 Target group，使得 Auto Scaling 時，可以在 target group 自動擴展和管理 instances。
{{< /alert >}}


---