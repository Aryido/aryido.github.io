---
title: "AWS ECS"

author: Aryido

date: 2022-12-17T16:30:30+08:00

thumbnailImage: "/images/aws/ecs-logo.jpg"

categories:
- cloud

tags:
- aws

comment: false

reward: false
---
<!--BODY-->
> Amazon Elastic Container Service（ECS）標誌著 AWS 進入 CaaS 市場。在 Kubernetes 還沒有出現時，各家雲端大廠對於**容器化的管理工具**都有自己實作。對應在 AWS 上的容器編排平台，是在 2014 年宣佈的 ECS 服務。後續進一步改進，發布 ECS with Fargate，可讓我們運行 container，而**無需管理伺服器或集群**。 故 Amazon ECS 具有兩個常用模式：
> - EC2 啟動類型
> - Fargate 啟動類型
>
> 以下分別進行介紹。
<!--more-->

---

整體架構上來說，可以想成給 ECS 一個 image ， 然後就會生出 container，其中的過程被 ECS 封裝起來調用。以下舉例兩個比較重要的 ECS Component

# service
service 是 ECS 的一個 component ， 是一個調度程式。service 會依照 task definition 的定義，來執行任務以滿足所需的實例數，另外它還可將**任務實例**(也就是 container)，連接到負載均衡器，以便平衡流量和接收外部的流量。

# task definition
容器鏡像包裝在 ECS **task definition**中，該 task definition 會聲明資源需求，如CPU、內存、網絡端口等等...

{{< alert info >}}
 Service 將 Task 部署到你的 Cluster 裡，Task 是一層用來運行 Container 的 Wrapper
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/aws/ecs-service.jpg" >}}

---

# ECS with EC2
{{< image classes="fancybox fig-100" src="/images/aws/ecs-ec2.jpg" >}}

在啟動 task 和 service 之前，需要把 EC2 加到集群才能被管理。這需要每個 EC2 實例，都運行一個 agent 來與 ECS control plane 溝通。所以若是要自建 EC2 並加入 到 ECS 管理容器的話，會需要 :

- 選擇使用 Amazon ECS-optimized 的 AMI，會有 image 自動有安裝 ECS agent

- 自己使用 user-data 或 ssh 進入 EC2 安裝 ECS Agent ，可參考連結[ Installing the Amazon ECS Container Agent](https://docs.aws.amazon.com/zh_tw/AmazonECS/latest/developerguide/ecs-agent-install.html)

以上二選一來安裝 ECS Agent，再來還需要設定

```shell=
#!/bin/bash
echo ECS_CLUSTER=your_cluster_name >> /etc/ecs/ecs.config
```
這樣的話，EC2 才知道是對應到哪個 ECS Cluster。


且除了 ECS agent 之外，一般來說用戶還要處理許多底層基礎設施。比如 :
- EC2 實例類型、AMI、EBS大小
- VPC子網、安全組等。

如上述所說，在調度集群中的第一個 ECS 任務之前，有相當多的基礎設施需要完成。這主要是因為 ECS 處理 EC2 的方式 —— **EC2 只在 task 和 service 級別，而不是配置和擴展機群**。

---

# ECS with Fargate
因為集群配置的繁瑣，故在 2017 年，AWS 推出了 Fargate，它更封裝了 infra 底層的設定， Fargate 也成為在 AWS ECS 上啟動容器化工作負載推薦選項。

{{< image classes="fancybox fig-100" src="/images/aws/ecs-fargate.jpg" >}}

從上面架構圖中可知， Fargate 其實還是使用 EC2 實例來運行 container，但用戶在自己的帳戶中看不到運行的 EC2。Fargate 是由 AWS 官方提供 EC2 實例，在一個專用的私有 VPC 中運行 Fargate 任務。


---
{{< alert info >}}
ECS 和 Kubernetes 相似的地方:
- **ECS task definition** 類似於 **k8s pod**

- **ECS service** 類似於 **k8s deployment**
{{< /alert >}}

{{< alert warning >}}
ECS 和 Kubernetes 不同的的地方是集群的概念。
- ECS 中，集群是任務和服務的獨立邏輯邊界
- Kubernetes 集群，是主節點和工作節點的集合。
{{< /alert >}}



