---
title: Kubernetes - Deployment

author: Aryido

date: 2023-03-13T23:52:05+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 在 Kubernetes 中，Pod 是最小的管理單元，但是 Pod 並不能保證總是可用的，因此 Kubernetes 實現了一系列控制器來管理 Pod，我們稱為**工作負載 workloads**。故 workloads 基本可以解釋成管理一個或多個 **Pod** 的方式，主要簡單舉例以下幾種：
>- Deployment
>- DaemonSet
>- Job & CronJob
>- StatefulSet
>
> 這些稱為控制器，會使 Pod 的期望狀態和設定狀態盡量保持一致。這裏先介紹  Deployment 控制器。
<!--more-->

---

Deployment 是一種高層次的抽象，用於:
- 管理 Pod 的副本數量
- 管理升級回滾的策略

在說 Deployment 之前，先來了解一下發展的歷史。在 Kubernetes 初期，是使用 Replication Controller 來控制 Pod，**但後面淘汰了 Replication Controller，轉而使用 ReplicaSet** 來控制 Pod。 那 **Deployment 和 ReplicaSet 是甚麼關係呢?**

{{< alert success >}}
Deployment >> ReplicaSet >> pod
{{< /alert >}}

Deployment 是比 ReplicaSet 更高層的 controller，通常是比較常用 Deployment 來管理 Pod。 ，現在實際中也很少去直接使用 ReplicaSet 來部屬 Pod。。

{{< alert info >}}
Deployment 可以擁有多個 ReplicaSet，一個 ReplicaSet 可以擁有多個 Pod。
{{< /alert >}}

## Deployment
每當操作 Deployment 的時候，就會生成一個新的 ReplicaSet，該 ReplicaSet 會確保應用程序的 Pod 數量符合指定的副本數量。然後逐步更新成新的 Pod，而舊的 ReplicaSet 會逐步減少 Pod 直到新的 ReplicaSet 全部接管。注意，這時候**並不會刪除老的 ReplicaSet**，系統會將其保存下來，以備**回滾**使用。

{{< alert info >}}
一個 Deployment 擁有多個 ReplicaSet 主要就是爲了支持**回滾操作**
{{< /alert >}}

{{< alert info >}}
Deployment 實際控制的是 ReplicaSet 的數目以及每個 ReplicaSet 的屬性。而一個應用版本，對應的就是一個 ReplicaSet，而這個版本應有的 Pod 數量，是通過 ReplicaSet 自己來管理。
{{< /alert >}}

---

## 用途

- **滾動更新** : 如果在更新過程中，新版本 Pod 有問題，那麼滾動更新就會停止。
sdf
  {{< alert warning >}}
由於應用本身還有舊版本的 Pod 在線，所以滾動更新停止並不會對服務造成太大的影響，這時候就可以查看 log 找 Pod 錯誤原因了。
  {{< /alert >}}

- **回滾操作**：升版過程中發現問題時，可以回滾到之前的版本，以保證 app 的可用性。
  {{< alert warning >}}
Deployment 會有 ReplicaSet 的版本紀錄，故可以使用舊的 ReplicaSet 把 app 的版本退回舊的穩定狀態
  {{< /alert >}}
- Deployment Controller 會確保任何時間窗口內，只有指定比例的 Pod 處於離線狀態
- Deployment Controller 同時也會確保在任何時間窗口內，只有指定比例的 Pod 被創建，這個比例默認是 DESIRED 的 25%。


