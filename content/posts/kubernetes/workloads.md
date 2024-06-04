---
title: "Kubernetes - Workloads & Workload Resources"

author: Aryido

date: 2023-10-04T00:00:30+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- containerization
- kubernetes

tags:
- pod

comment: false

reward: false
---
<!--BODY-->
> **Workload 是指在 Kubernetes Pod 內運行的應用程式**。但是 Pod 並不能保證總是可用的，所以需要管理它們。但若直接管理 Pod 的話，工作量將會非常大且繁瑣，為了減輕負擔，Kubernetes 提供 Workload Resources 來管理一組 Pods。即 **Workload Resource 是 Kubernetes 中，定義和管理 Workload 的特定 API 物件**，例如 Deployment、StatefulSet 等等都是屬於 Workload Resource。
>

<!--more-->

---

在 Kubernetes 中，Pod 是最小的管理單元。Pod 整個運行周期，可以分成幾個階段 :
{{< image classes="fancybox fig-100" src="/images/kubernetes/pod-lifecycle.jpg" >}}

當因爲某些原因 Pod 被刪除，我們希望能夠自動回復成原樣，故需新建一個同樣的 Pod。那要如何做到呢 ?


> **Kubernetes 用 Workload Resource 來配置期望狀態，然後透過其 Controller 來確保正運作的 Pod 個數、狀態等等，和所配置的期望狀態相符**。

---

# Workload
準確來說，當談論 Workload 時，是指**實際運行的應用程序或服務**，Workload 可以是一到多個 Pod 所組成，故要達成並維持一個 Workload ，會需要注意 Pod 可能的生命周期，並實時監控 Pod 的狀態。

實際上維運時，我們並不直接與 Pod 做互動，而是透過一些比如說 Deployment、 ReplicaSet 等等的東西來管理，這些被稱為 Workload Resource。


# Workload Resources

Kubernetes 提供一系列 Workload Resources ，它是比 Pod 更高層級的抽象概念。**Kubernetes Controller 會根據定義的 Workload Resource，自動管理 Pod ，使 Pod 的期望狀態，和實際狀態保持一致**。

{{< alert info >}}
當在 Kubernetes 中創建 Workload 時，它會具有所需要的狀態。
Kuberenets 會監控所有 Workload 狀態，並將其與所需狀態進行比較，並根據 Workload Resource 的配置，如果當前狀態與所需狀態不匹配，Controller 會使它合乎要求。
{{< /alert >}}


常見的 Workload Resources 有：
{{< image classes="fancybox fig-100" src="/images/kubernetes/workload-resources-overview.jpg" >}}


- {{< hl-text blue >}}
Deployment （也間接包括 ReplicaSet）
{{< /hl-text >}}

    最常見的管理運行應用的方式。適合管理**無狀態** Workload 。 Deployment 中的所有 Pod 都是相互等價的，隨時可以被替換。

- {{< hl-text blue >}}
StatefulSet
{{< /hl-text >}}

    為**有狀態** Workload，能夠建立 Pod 與持久化儲存之間的關聯。例如，運行一個將每個 Pod 關聯到 PersistentVolume 的 StatefulSet。

    {{< alert success >}}
StatefulSet 中各個 Pod 內運行的應用，可以將資料複製到同一 StatefulSet 中的其它 Pod 中，提高整體的服務可靠性。
{{< /alert >}}


- {{< hl-text blue >}}
DaemonSet
{{< /hl-text >}}

    當必須在適當的節點上執行服務時，可以使用 DaemonSet。 DaemonSet 中的每個 Pod 執行類似於經典 Unix / POSIX 伺服器上的系統守護程式的角色。
- {{< hl-text blue >}}
Job / CronJob
{{< /hl-text >}}

    Job 表示一次性任務，而每個 CronJob 可以根據排程表重複執行。

- {{< hl-text blue >}}
其他第三方CRD
{{< /hl-text >}}

    在 Kubernetes 生態系統中，也可以找到一些提供額外操作的第三方  Workload Resources。 透過使用自訂資源定義（CRD），完成原本不是 Kubernetes 核心功能的工作。

---
### 參考資料

- [Kubernetes 工作負載管理](https://www.readfog.com/a/1678228938165424128)
- [Workloads](https://kubernetes.io/docs/concepts/workloads/)
- [IT 鐵人賽 k8s 入門30天 -- day15 k8s Workload 簡介](https://ithelp.ithome.com.tw/m/articles/10274936)