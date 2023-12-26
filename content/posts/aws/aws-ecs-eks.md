---
title: "AWS Overview: ECS | EKS "

author: Aryido

date: 2022-12-13T21:28:20+08:00

thumbnailImage: "/images/aws/logo.jpg"

categories:
- aws

comment: false

reward: false
---
<!--BODY-->
> ECS (Elastic Container Service) 和 EKS（Elastic Kubernetes Service）都是 AWS 上提供的 Container Orchestration ( 容器管理工具 )，核心都是**中央控制管理**運行的容器化應用程式，以下來簡單介紹一下吧。
<!--more-->

---
# Container Orchestration
在 Docker 出世後，將應用程式打包成 image 單元，就成了蠻標準化的格式。當我們 run image 後的執行實例，稱為 Container ，它具有應用程式運行所需的一切，包括 libraries、runtime。它讓我們可以**快速部署**應用程序，而不須煩惱環境設定。

但隨著 Container 越來越多，我們需要的一個中央控制中心，來幫助我們管理這些  Containers。

{{< alert info >}}
Container Orchestration :
- Managing
- scaling
- deploying containers
{{< /alert >}}

以下是著名的 Container Orchestration
{{< image classes="fancybox fig-100" src="/images/aws/container-orchestration.jpg" >}}

---

# ECS
Amazon Elastic Container Service (ECS) 是 AWS 自己提供的**部署容器**以及**管理容器**的雲端服務。**使用 ECS 無須另外付費**，但要支付例如，EC2 instance 或 EBS 磁碟區等 AWS 資源的費用。

## ECS Architecture overview
{{< image classes="fancybox fig-100" src="/images/aws/ecs.jpg" >}}

在上面圖中，可特別注意下，**ECS agent** 這個東西。因為 EC2 是透過 ECS agent 去向 ESC Cluster 註冊的，這樣 ESC Cluster 才能中央控管那些跑在 EC2 內的 container 。

---

# EKS
Amazon Elastic Kubernetes Service (Amazon EKS) ，是在 AWS 上執行並託管  Kubernetes，無需自己安裝的 Kubernetes 控制平面或節點。

## ECS Architecture overview
{{< image classes="fancybox fig-100" src="/images/aws/eks.jpg" >}}

為什麼需要 EKS 呢 ? 他解決了什麼問題呢 ? 一般來說，自己維護的 k8s cluster 必需要很注意 master node 的穩定性，像是 :
- 升級 master node kubernetes version
- master node 的高可用性 high availability
- etcd 的升級，etcd 的備份和還原
- 更新 certificate

使用 EKS 還有一些好處 :
- 可以使用 EC2 IAM role ，解決了跟 AWS 提供的 service 整合問題
- Master node 和 worker node 之間用 private link 連結，走的是內部網路，網路效能可以提升
- auto scale 不須擔心機器問題。

另外從商業考量來說，其實 Container Orchestration 目前 k8s 還是最大宗，占掉很多市場，對於已經在使用 k8s 管理許多服務的大型公司，換成 ECS 的誘因並不大。
{{< image classes="fancybox fig-100" src="/images/aws/co-market-share.jpg" >}}

但另一方面，雲端平台目前 AWS 還是佔最大的市場的，雲端平台對於**高可用 HA** 和 **auto scale** 的功能，是可以解決 Kubernetes 遇到的一些痛點，所以 AWS 提供了一個 EKS 來搶占這塊市場，目前看起來蠻成功的。
{{< image classes="fancybox fig-100" src="/images/aws/eks-market-share.jpg" >}}

---