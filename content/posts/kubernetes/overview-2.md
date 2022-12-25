---
title: Kubernetes 基礎介紹 - 2

author: Aryido

date: 2022-12-24T12:30:06+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 在上一篇文章中，簡單介紹了 Kubernetes 的架構，接下來簡介 Kubernetes 在部屬 app 時的單位 Pod 。 Pod 對多容器的支持是 K8 最基礎的設計理念，但 Pod 應該怎麼被管理呢 ? 怎麼和外網連線呢 ? 這些部分由如下元件提供功能解決 :
> - Deployment
> - Service
> - Ingress
>
> 像在實現進階的操作如:負載均衡、滾動更新、安全與監控等概念，都會跟這些元件有關係。

<!--more-->

---
# Kubernetes 進階三元件

## Deployment
如果我們要透過指令建立並監控一個 Pod 讓它可以 scaling、rollback 是很花時間的。Kubernetes 提供我們 Deployment 元件，可以幫我們達成
- 部署 application
- applications 版本升級
- 無停機服務(zero downtime deployment)
- Rollback

## Service

{{< image classes="fancybox fig-100" src="/images/kubernetes/service.jpg" >}}

建立外部服務與 Pods 的溝通管道就是  Service ，在 Kubernetes 中用來定義「 Pod 要如何被連線及存取」，具體舉例 :

- 使用 type =  ClusterIp，則 **Kubernetes Cluster中**的其他服務，可以透過這個 ClusterIp 訪問到正在運行中的 Pods。

{{< alert warning >}}
type =  ClusterIp，則服務只能夠在集群內部訪問。為 type 默認值。
{{< /alert >}}


- 使用 type = NodePort，則在 **Kubernetes Cluster外，但在同一個 Node 上的其他服務**，可以透過這個 NodePort 訪問到 Kubernetes Cluster 內正在運行中的 Pods。

- 如果 Kubernetes Cluster 是架在第三方雲端服務(cloud provider)，例如 EKS、GKE 等等，則可以透過這些 cloud provider 提供的 LoadBalancer ，幫我們分配流量到每個 Node (指定 type =   LoadBalancer)。

{{< alert danger >}}
甚麼是 cluster 外，但同一個 node 上呢 ? 舉例來說，要實現 type =  LoadBalancer 的服務，Kubernetes 首先需進行 type =  NodePort ，再來  cloud-controller-manager 組件才能配置**外部**負載均衡器以將流量轉發到已分配的節點端口。
{{< /alert >}}

## Ingress

{{< image classes="fancybox fig-100" src="/images/kubernetes/ingress.jpg" >}}

在 Service 中，我們是將每個 Service 元件對外的 port number 跟 Node 上的 port number 做 mapping，這樣在我們的 Service 變多時，port number 以及分流規則的管理變得相當困難。

而 Ingress 可以透過 HTTP/HTTPS，在我們眾多的 Service 前搭建一個 reverse-proxy。這樣 Ingress 可以幫助我們統一一個對外的 port number，並且根據 hostname 或是 pathname 決定封包要轉發到哪個 Service 上。 **透過 Ingress 不但能使 Node 對外開放的 port 統一，結合 Ingress Controller 更能在 Kubernetes Cluster 中實現負載平衡 的功能**。

---

{{< image classes="fancybox fig-100" src="/images/kubernetes/k8s-scheduling.jpg" >}}
