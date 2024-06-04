---
title: "Kubernetes - Deployment"

author: Aryido

first draft: 2023-03-13T23:52:05+08:00

date: 2023-10-01T18:28:05+08:00

thumbnailImage: "/images/kubernetes/deployment-logo.jpg"

categories:
- containerization
- kubernetes

tags:
- pod

comment: false

reward: false

---
<!--BODY-->
> Deployment 是 Kubernetes 中，最常使用的的一種工作負載(Workloads)，它以 YAML 格式描述 Pod ，提供聲明式(declarative)的設定。除了定義 Pod 的狀態，更進一步可以管理 :
> - Pod 的 replica 數量
> - 升級回滾的策略
>
> Deployment 是用來**編排無狀態 pod 的一種控制器資源**，官方也建議應該透過 Deployment 來佈署 Pod & Replicaset，而非直接對  Pod & Replicaset 進行管理。
>
<!--more-->

---
# 緣起

在說 Deployment 之前，先來了解一下發展的歷史。

在 Kubernetes 初期，是使用 [ReplicationController](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicationcontroller/#responsibilities-of-the-replicationcontroller) 來控制 Pod，但後面淘汰了 它。因為剛開始定義的 ReplicationController  職責太過狹隘，僅確保【所需的 Pod 數量】和 【定義 label selector】，但並沒有負責 LivenessProbe、StartupProbe、Auto-Scale 等等功能，都由外部控制器負責。為了建立更高層級的 API ，故產生了 **ReplicaSet** 。

那 Deployment 和 ReplicaSet 是甚麼關係呢?

{{< image classes="fancybox fig-100" src="/images/kubernetes/deployment.jpg" >}}

Deployment 是比 ReplicaSet 更高層的抽象，通常建議用 Deployment 來管理 Pod。而現在實際中，不建議也很少去直接使用 ReplicaSet 來部屬 Pod。

{{< alert success >}}
Deployment 可以產生多個 ReplicaSet；一個 ReplicaSet 可以產生多個 Pod。
{{< /alert >}}

---
# Deployment
Deployment 會生成 ReplicaSet，而接下來 ReplicaSet 會建立 Pod 。其分別的名稱形式會是 :
- **ReplicaSet**: ```[DEPLOYMENT-NAME]-[POD-TEMPLATE-HASH-VALUE]```
- **Pod**: ```[DEPLOYMENT-NAME]-[POD-TEMPLATE-HASH-VALUE]-[UNIQUE-ID] ```

同時 Deployment 會把 pod-template-hash 標籤，加入至每個建立並管理的 ReplicaSet ；而 ReplicaSet 也同時和會把 pod-template-hash 標籤，加入到每個 Pods 中。更具體的說， pod-template-hash 標籤的值，是**透過 PodTemplate 進行雜湊得到的**，且會被加到的標籤位置，如下範例 :
```
# kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            0           15s

# kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-67594d6bf6   3         3         3         9m

# kubectl describe rs/nginx-deployment-67594d6bf6

....(略)
Pod Template:
  Labels: app=nginx
          pod-template-hash=2315082692
....(略)

# kubectl get pod --show-labels
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-67594d6bf6-22nrx   1/1       Running   0          12s       app=nginx,pod-template-hash=2315082692
nginx-deployment-67594d6bf6-ccx87   1/1       Running   0          12s       app=nginx,pod-template-hash=2315082692
nginx-deployment-67594d6bf6-rkztl   1/1       Running   0          12s       app=nginx,pod-template-hash=2315082692
```
承上說明，我們可以看到標籤位置:
- ReplicaSet 是在 **label selector**
- Pod 是在 **template label**

{{< alert warning >}}
注意，```POD-TEMPLATE-HASH-VALUE```  的值，和 ```pod-template-hash``` 的值，是不一樣喔 ! 而不一樣的原因，應該是因為不同的 hash 方法所導致，因為透過相同的 YAML 重新建立 deployment 也還是會得到相同的結果。

例如以上範例 :
- ```POD-TEMPLATE-HASH-VALUE = 67594d6bf6```
- ```pod-template-hash = 2315082692```
{{< /alert >}}

該標籤有一些作用 :

- 可確保 Deployment 管理的子 ReplicaSets 不重複。

- 利用 Pod 的 label 標上的 pod-template-hash 值，就可以確認哪些 Pod 是屬於同一組的


ReplicaSet 的主要功能是會確保 Pod 數量符合 YAML 指定的數字。然後會依據一些**策略**，新 ReplicaSet 會逐步更新成新的 Pod，而舊 ReplicaSet 會逐步減少 Pod ，直到 Pod 數量穩定。注意，這時候**並不會刪除老的 ReplicaSet**，系統會將其保存下來，以備**回滾**使用。

{{< image classes="fancybox fig-100" src="/images/kubernetes/deployment-scale-replicas.jpg" >}}

{{< alert info >}}
一個 Deployment 擁有多個 ReplicaSet 主要就是爲了支持**回滾操作**
{{< /alert >}}

{{< alert success >}}
Deployment 實際控制的是 ReplicaSet 的數目以及每個 ReplicaSet 的屬性。而這個應有的 Pod 數量，是通過 ReplicaSet 自己來管理。
{{< /alert >}}

## 高階功能及用途

- **滾動更新** : 如果在更新過程中，新版本 Pod 有問題，那麼滾動更新就會停止。
  {{< alert warning >}}
由於應用本身還有舊版本的 Pod 在線，所以滾動更新停止並不會對服務造成太大的影響，這時候就可以查看 log 找 Pod 錯誤原因了。
  {{< /alert >}}

- **回滾操作**：升版過程中發現問題時，可以回滾到之前的版本，以保證 app 的可用性。
  {{< alert warning >}}
Deployment 會有 ReplicaSet 的版本紀錄，故可以使用舊的 ReplicaSet 把 app 的版本退回舊的穩定狀態
  {{< /alert >}}
- Deployment 可以設定在任何時間區間內，只有指定比例的 Pod 處於離線狀態
- Deployment 可以設定在任何時間區間內，只有指定比例的 Pod 被創建，這個比例默認是 DESIRED 的 25%

---

# [Deployment YAML](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment # deployment name
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels: # 必須要與下面的 pod label 有相符合
      app: nginx  # replicaset 設定會套用在有 app=nginx 標籤的 pod 上
  template: # 以下為 pod 的定義
    metadata:
      labels:
        app: nginx # 設定給 pod 的 label 資訊
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
{{< alert warning >}}
```selector.matchLabels``` 的設置應該和 ```template.metadata.labels``` 一致。這樣，Deployment 才能正確地識別和管理它自己創建的 Pod。replicaSet 才能把 app 的版本退回舊的穩定狀態
{{< /alert >}}

{{< alert danger >}}
```selector.matchLabels``` 的設置應該和 ```template.metadata.labels``` 一致，**但不要與其他控制器（例如 Deployment 和 StatefulSet）重疊**。雖然Kubernetes 不會阻止這樣做，但可能會發生衝突執行及難以預料的行為。
{{< /alert >}}
{{< alert danger >}}
API version ```apps/v1``` 之後，```.spec.selector``` 被設定之後就無法再修改了!
{{< /alert >}}

---
### 參考資料

- [[Kubernetes] Deployment Overview](https://godleon.github.io/blog/Kubernetes/k8s-Deployment-Overview/)
- [Kubernetes 工作負載管理](https://www.readfog.com/a/1678228938165424128)
- [详解 Kubernetes Deployment 的实现原理](https://draveness.me/kubernetes-deployment/)
