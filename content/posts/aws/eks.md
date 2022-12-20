---
title: "AWS EKS"

author: Aryido

date: 2022-12-17T18:45:30+08:00

thumbnailImage: "/images/aws/eks-logo.jpg"

categories:
- cloud

tags:
- aws

comment: false

reward: false
---
<!--BODY-->
> ECS 很常拿來與 Kubernetes 比較，而  2017 aws 又進一步宣佈了 Amazon Elastic Container Service for Kubernetes(EKS)，使 aws 平台可以託管 k8s 服務。EKS 服務可以省去安裝以及操作自己的 Kubernetes 叢集的時間，輕鬆的在 AWS 上執行 Kubernetes；進一步地，可使用 Fargate 模式在 *EKS* 上，可連 node 機器都不用管理。
<!--more-->

---

 傳統 Kubernetes Cluster 主要有兩部分:
 - 安排容器調度的 Control Plane
 - 容器運行的機器稱 Worker Node。

Control Plane 裡面涵蓋有儲存狀態的 ETCD、controller manager 、Scheduler、APIServer。若是自己創建 的 Kubernetes Cluster ，需要自己安裝這些元件，後續仍需要對 Control Plane 進行相關管理、維護、升級工作。

# kubernetes in cloud
{{< image classes="fancybox fig-100" src="/images/kubernetes/architecture.jpg" >}}

aws 官網上面安裝 EKS 有兩種方式:
- [eksctl](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/eksctl.html)
- [cloudformation + aws cli](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/getting-started-console.html)

透過 eksctl 建立資源，是最快最方便的，但其實很容易忽略掉一些地方，例如權限管理的部份。故也推薦按照官網教程，用 cloudformation + aws cli 走一遍流程會更有感覺。

接下來就是 kubernetes 放到雲平台上真正好要和有威力的功能了。 kubernetes 基本上是以 pod 為單位做控制，然後 schedule 到 node 上面。但當 Pod 出現故障或需要 reschedule 到其他節點時，其原因可能是 node 機器不夠了，這時雲端提供的機器 Auto Scaling 就展現了威力，可以自動調整 nodes 數量。那 kubernetes 它是怎麼辦到的呢 ?

## [Cluster Autoscaler](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/autoscaling.html#cluster-autoscaler)
Cluster Autoscaler 是 aws 為 k8s autoscaling 提供的實作。當 Pod 出現故障或 reschedule ， Cluster Autoscaler 會自動調整 cluster 中的 node 數量。

## [AWS Load Balancer Controller](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/alb-ingress.html)
若 k8s Service yaml 的 type 設定為 LoadBalancer 會發現 EKS 很神奇地部屬了 Load Balancer。為什麼會這樣呢 ? 原因就是安裝了 aws-load-balancer-controller。

簡單介紹了以上功能，都是需要另外自行安裝的 (Add-ons)，EKS 服務在啟動時大致上都幫我們裝好了。

{{< alert info >}}
將 k8s Service type 設定為 LoadBalancer，會用 Network Load Balancer(NLB)，是 L4 of the OSI model，根據 IP + port 做負載均衡。

設定 k8s Ingress 的話，會用Application Load Balancer(ALB)，是 L7 of the OSI model，可根據域名、內文做負載均衡，會做 TCP 交握，比較耗時。
{{< /alert >}}

---
# EKS with Fargate
{{< image classes="fancybox fig-100" src="/images/aws/eks-comparation.jpg" >}}
上圖很直接的表示在 Amazon EKS 使用 AWS Fargate 模式與一般 EC2 模式的區別。

在 EKS 中提供了 AWS Fargate Profile。可配置一個設定檔，可讓 Amazonn EKS 中的 Pod 採用 Fargate 技術運行。

- 僅需設定 CPU 跟 memery
- 可設定 Scaling 範圍，會自動擴展或減少，擴展速度比 EC2 快
- 不需要管理 nodes