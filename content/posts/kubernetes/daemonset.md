---
title: Kubernetes - DaemonSet

author: Aryido

date: 2023-03-14T22:35:58+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> k8s Cluster 並不直接與 Pod 做互動，而是透過一些管理元件來處理 Pod ，這些管理元件總體被稱為 Workload，這裏介紹 DaemonSet 控制器。DaemonSet 用於提供 Node 基本設施的 Pod，會確保在**所有(或是特定)節點**上，一定運行著指定的一個 Pod。若想只運行在特定節點運行 DaemonSet Pod，可藉由給定的**標籤**，讓 Pod 可以只在特定節點上運行。
>

<!--more-->
---

DaemonSet 基本上是確保在 Kubernetes 中，每一個 Node 上，都會有一個指定的 Pod 運行。當有新的 Node 加入到 Kubernetes Cluster 後，會自動在那個 Node 上長出相同的 DaemonSet Pod，當有 Node 從 Kubernetes Cluster 移除後， DaemonSet Pod 就會自動被清除掉。

但若想**只運行在特定 node** 的話 ，則需要配合 (目前不詳細討論) :
- Affinity and Anti-Affinity
- Taints and Tolerations

{{< alert success >}}
DaemonSet 通常用於運行在每個 Node 上的守護進程(Daemon)
{{< /alert >}}

值得注意的是，當你直接使用```kubectl get pods``` 是看不到 DaemonSet Pod 的，這是因為 get pods 會去撈名為 Default 的 Namespace 內的資源，但是 DaemonSet Pod  不在 Namespace 內，而是在 kube-system 內 !

---

## DaemonSet

DaemonSet 常見的場景用法：

- 在每個節點上日誌收集
  {{< alert info >}}
logstash、fluentd
  {{< /alert >}}
- 在每個節點上運行監控
  {{< alert info >}}
Prometheus Node Exporter、 Datadog
  {{< /alert >}}

---

## 比較 Deployment 與 DaemonSet
- DaemonSet 除了 kind 為 DaemonSet 之外，其實其他的 yaml 寫法部分與 Deployment 基本沒有什麼不同。
- DaemonSet 是確保 Node 一定會有一個 Pod 在運行著，因此 yaml 有設定 Pod template 部分。

- **DaemonSet 沒有 Replica 相關設定!**


{{< alert warning >}}
當 DaemonSet 建立後，就不建議修改.spec.selector，一但修改了可能會造成 DaemonSet 無法認得 pod。
{{< /alert >}}
