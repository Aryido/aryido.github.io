---
title: "AWS ECS"

author: Aryido

date: 2022-12-17T16:30:30+08:00

thumbnailImage: "/images/aws/ecs-logo.jpg"

categories:
  - cloud
  - aws

comment: false

reward: false
---

<!--BODY-->

> Amazon Elastic Container Service（ECS）標誌著 AWS 進入 CaaS 市場。在 Kubernetes 還沒有出現時，各家雲端大廠對於**容器化的管理工具**都有自己實作。對應在 AWS 上的容器編排平台，是在 2014 年宣佈的 ECS 服務。後續進一步改進，分成兩個模式：
>
> - EC2 啟動類型
> - Fargate 啟動類型 (**無需管理伺服器或集群**。)
>
> 以下對服務架構分別進行簡單介紹。

<!--more-->

---

# Cluster

可以選擇使用 AWS Fargate（Serverless 模式）或 EC2（自行管理實體主機）模式，比較重要的 Component 有：

- Services： 確保特定數量的 Task 持續運行，提供高可用性與穩定性
- Tasks： 根據 Task Definition 建立出來的 workload
- Metrics: 整合 AWS CloudWatch，能監控容器的 CPU、記憶體使用量、日誌與錯誤警報等
- Configuration: 設定 Auto Scaling policy 等等

```bash
# 列出目前這個帳號在預設 Region（區域）下所有的 Cluster所有的叢集
aws ecs list-clusters

# 列出叢集內所有的底層 EC2 機器
aws ecs list-container-instances --cluster <claster_name>
```

# Service

service 是一個 workload 調度介面，會依照 task definition 的定義，來執行任務以滿足所需的實例數，另外它還可將**任務實例**(也就是 container)，連接到負載均衡器，以便平衡流量和接收外部的流量。

{{< alert success >}}
要接入外部的流量，ECS 經常和 AWS ELB 搭配。
{{< /alert >}}

# Task definition & Task

為應用程式的 blueprint ，寫著容器鏡像要如何啟動的定義，也會聲明資源需求，如CPU、內存、網絡端口等等...，以 JSON 格式撰寫的文本檔案。 Task 就是真實運行的 workload，是透過 Task Definition 產生出來的。

```bash
# 撈出特定 Service 正在運行的 Tasks
aws ecs list-tasks \
--region <region> \
--cluster <claster_name> \
--service-name <service_name> \
--query 'taskArns[]' \
--output text

# 詳細 query 到指定 Task 的 AWS 內網 IPv4 位址
aws ecs describe-tasks \
--region <region> \
--cluster <claster_name> \
--tasks <task_id> \
--query 'tasks[].containers[0].networkInterfaces[0].privateIpv4Address'

```

{{< alert info >}}
Service 將 Task 部署到 Cluster 內，Task 是一層用來運行 Container 的 Wrapper
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/aws/ecs-service.jpg" >}}

---

# ECS with EC2

{{< image classes="fancybox fig-100" src="/images/aws/ecs-ec2.jpg" >}}

在啟動 task 之前，需要把 EC2 加到集群，且每個 EC2 實例都運行一個 agent 來與 ECS 溝通，ECS Agent 可互相傳遞例如：正在運行的 task、資源使用狀況 ; 啟動 & 停止 task 等等。所以若是要自建 EC2 並加入 到 ECS 管理容器的話，會需要 :

- 選擇使用 Amazon ECS-optimized 的 AMI， image 內自動有安裝 ECS agent

- 自己使用 user-data 或 ssh 進入 EC2 安裝 ECS Agent ，可參考連結[ Installing the Amazon ECS Container Agent](https://docs.aws.amazon.com/zh_tw/AmazonECS/latest/developerguide/ecs-agent-install.html)

以上二選一來安裝 ECS Agent，再來還需要設定

```shell=
#!/bin/bash
echo ECS_CLUSTER=your_cluster_name >> /etc/ecs/ecs.config
```

這樣的話，EC2 才知道是對應到哪個 ECS Cluster。且除了 ECS agent 之外，一般來說用戶還要處理許多底層基礎設施。比如 :

- EC2 實例類型、AMI、EBS大小
- VPC子網、安全組等。

---

# ECS with Fargate

承上我們知道集群配置的繁瑣，故在 2017 年，AWS 推出了 Fargate，它更封裝了 infra 底層的設定， Fargate 也成為在 AWS ECS 上啟動容器化工作負載推薦選項。

{{< image classes="fancybox fig-100" src="/images/aws/ecs-fargate.jpg" >}}

從上面架構圖中可知， 在 Fargate 模式下，用戶在自己的帳戶中看不到運行的 EC2，機器全部交由 AWS 管理，在一個專用的私有 VPC 中運行。

{{< alert info >}}
ECS 和 Kubernetes 相似的地方:

- **ECS task definition** 類似於 **k8s pod**

- **ECS service** 類似於 **k8s deployment**
  {{< /alert >}}

---

### 參考資料

- [什麼是 Amazon Elastic Container Service？](https://docs.aws.amazon.com/zh_tw/AmazonECS/latest/developerguide/Welcome.html)

- [30天鐵人賽介紹 AWS 雲端世界 - 28: AWS 上的容器服務 Elastic Container Service(ECS)](https://ithelp.ithome.com.tw/m/articles/10195572)

- [Day 15 - AWS ECS & Fargate](https://ithelp.ithome.com.tw/articles/10323140)

- [AWS CSA Associate 學習筆記 - Serverless](https://godleon.github.io/blog/AWS/AWS-CSA-associate-Serverless/)
