---
title: Kubernetes 基礎介紹 - 1

author: Aryido

date: 2022-12-19T23:38:06+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- containerization
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 從第一次聽到 Kubernetes 以來，已經有一年多了，永遠都記得 k8s 名稱的由來只是保留「開頭 K」及「結尾 S」，然後中間的英文字母數量剛好是 8 個英文字就這樣命名了...。全球三大雲服務商，AWS、Azure 和 GCP 都有提供託管 Kubernetes 集群服務( EKS、AKS、GKE )，可見其有名火熱程度。現在終於有機會在工作上碰到這項技術，就來寫些簡單筆記吧 !
<!--more-->

---
# k8s 簡介

- open source container orchestration tool
- origin developed by Google
- helps you manage containerized applications in different deployment environments

## K8s Features
- high availability
    {{< alert info >}}
基本上避免停機，no downtime, it's always accessible by the users
    {{< /alert >}}
- scalability
        {{< alert info >}}
代表高性能 high performance
    {{< /alert >}}
- disaster recovery
  {{< alert info >}}
面對資料毀所只能依靠 backup 和 restore
    {{< /alert >}}

---
# Kubernetes Architecture
{{< image classes="fancybox fig-100" src="/images/kubernetes/cloud-architecture.jpg" >}}
基本概念有 :
- Control Plane ( master node )
- some Worker nodes
  - each worker node has a **kubelet** process running on it

{{< alert success >}}
kubelet 是做甚麼呢 ?

communicate master and worker; can execute tasks
{{< /alert >}}

以下會概述說明一下
 - 安排容器調度的 Control Plane Components
 - 容器運行的機器 Worker Node Components。

---
# 控制平面組件（Control Plane Components）

Control Plane 功能是主控 cluster 狀態和調度容器，且通常 Control Plane 內，不會運行任何 container 應用，可參閱 [Creating Highly Available Clusters](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)。Control Plane 身為 Kubernetes 運作中心，負責管理所有其他 Node，有幾個重要組件：
- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager
- **cloud-controller-manager**(optional)

{{< alert info >}}
為了簡單起見，通常會在**同一個機器**上，安裝所有控制平面組件。因為都在同一個機器，所以舊稱是 master node，但其實控制平面組件，可以分散在不同機器上運行，所以為了避免誤會中央控管只能在一台機器上，使用 Control Plane 是比較好的描述。
{{< /alert >}}

### kube-apiserver
kubectl 就是把指令送到這裏
，負責 Node 之間的溝通橋樑。每個 Node 彼此不能直接溝通，必須要透過 apiserver 轉介。

### etcd

用來存放 Kubernetes Cluster 的資料作為備份，可以透過 etcd 幫我們還原 Kubernetes 的狀態。**類似 terraform state 的存放處**

### kube-controller-manager
controller 就是 Kubernetes 裡一個個負責監視 Cluster 狀態的 Process，不符和時會更新。

### kube-scheduler
 Kubernetes 的 Pods 調度員，scheduler 會監視新建立但還沒有被指定要跑在哪個 Node 上的 Pod，選一個最適合放置的 Node 把 Pod 放上去。

### cloud-controller manager (Optional)
Cloud Controller Manager 會把集群連接到雲提供商的 API 之上，若在自己私有的環境中運行 Kubernetes ，就有cloud-controller manager。有些 k8s 功能會需要依賴雲平台，例如 Service yaml 設定 load-balancer type

---

##  Node Components

Node 也被經常稱呼為 worker， Node Components 主要有:
- kubelet
- kube-proxy
- Container Runtime
這些會在每個 node 上運行，負責維護運行的 Pod 並提供 Kubernetes 運行環境。

### kubelet
kubelet 負責與 API Server 溝通，確保 containers 處於運行狀態且健康。 kubelet 它保證 containers 都運行在 Pod 中，且不會管理不是由 Kubernetes 創建的容器。

### kube-proxy
維護節點上的一些網絡規則，實現 Kubernetes Service 概念的一部分。透過 kube-proxy 允許從集群內部或外部的網路與 Pod 進行通信。

### Container Runtime
Kubernetes 支持許多容器運行環境，例如 containerd、 CRI-O 以及 Kubernetes CRI (容器運行環境接口) 的其他任何實現。

---
