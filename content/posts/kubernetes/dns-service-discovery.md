---
title: Kubernetes - DNS Service Discovery

author: Aryido

date: 2023-05-31T22:58:42+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 在 Kubernetes 中，Service Discovery 一般常用的 DNS 提供商是使用 kube-dns 或 CoreDNS ，而 CoreDNS 從 v1.13 開始成為 Kubernetes 默認 DNS 服務。Service Discovery 是一種機制，通過該機制，服務可以動態地發現彼此，而無需 hardcode 硬寫 IP 或 endpoint 配置，可以讓我們不需要知道 Service 的 Cluster IP ，只透過 Service 的名稱，就能找到相對應 Pod ，而非使用 IP 地址訪問。
>

<!--more-->

---

透過 Kubernetes DNS 可以讓**同一個 Cluster 中的所有 Pods ，都能透過 Service 的名稱找到彼此**。k8s 提供了兩種方式進行 Service Discovery：

### 環境變量 (不推薦使用)
當 Pod 創建時， kubelet 會在該 Pod 中**注入有標籤關聯的 Service 相關環境變量**。

{{< alert danger >}}
需要注意的是，要想一個 Pod 中注入某個 Service 的環境變量，則必須 Service 要先比該 Pod 創建。這一點，使得這種方式進行 Service Discovery，不太好用...
{{< /alert >}}

比如一個 ServiceName 爲```redis-master``` 的 Service，對應的 ```ClusterIP:Port``` 爲 ```10.0.0.11:6379```，則對應的環境變量爲：
{{< image classes="fancybox fig-100" src="/images/kubernetes/env-discovery.jpg" >}}

### DNS (最常使用的方式)
Cluster 可以使用預設 CoreDNS 或 kube-dns ，另外也可 add-on 使用其他 DNS 提供商來，來對 Cluster 內的 Pod 進行 Service Discovery。

---

## Kubernetes DNS
Kubernetes 會在 kube-system 命名空間中用 Pod 的形式運行一個 DNS 服務，一旦 Kubernetes 被建立後，便會自動運行。每個 Kubernetes service 都會自動註冊到 DNS 服務之中，註冊過程大致如下：

- 向 API Server 用 POST 方式提交一個新的 Service ，這個請求需要經過認證。

- Service 得到一個 ClusterIP，並保存到 etcd

- DNS 服務會以某種方式得知有 Service 的創建，據此創建必要的 **DNS A 記錄**。

例如 CoreDNS 會對 API Server 進行監聽，一旦發現有新建的 Service 對象，就創建一個從 Service 名稱映射到 ClusterIP 的域名記錄。

{{< alert info >}}
CoreDNS 控制器會關注新創建的 Service ，Service 就不必自行向 DNS 進行註冊。
{{< /alert >}}

{{< alert success >}}
補充一下，在 GCP 提供的 GKE :
- Autopilot mode 只能使用 kube-dns，無法修改 DNS 提供商
-  Standard mode 提供 :
   -  kube-dns (Default)
   -  Cloud DNS
{{< /alert >}}

若要 Pod 使用 Service Discovery 功能，會需要知道 DNS 服務器的位置才能使用它。因此每個 Pod 中的每個容器的 ```/etc/resolv.conf``` 都會被配置 DNS 服務的 domain name 與相對應的 IP 位址。

k8s Domain Name 解析規則，會因為 Pod 所在的 namespace 而返回不同的結果。若沒有指定 namespace 的 DNS 查詢，會被限制在 Pod 所在的**當前 namespace 內**。
- k8s 中有 namespace 的概念，由於不同的 namespace 中可以有同樣名稱的 service ，因此 DNS 解析的部份就需要考慮 namespace

- k8s cluster domain name，若是未設定，預設就會是 ```cluster.local```

---

## DNS 查詢範例
假定有:
- namespace 為 ```np1```
  - service 為 ```svc1```
  - 內有一個 ```Pod1```
- namespace 為 ```np2```
  - service ，為 ```svc2```


若在```Pod1``` 查詢 ```svc2```
會找不到，因為會是在是  ```Pod1``` 的 namespace  ```np1``` 找```svc2```，當然會找不到。

若在```Pod1``` 查詢 ```svc2.np2``` 或者 ```svc2.np2.svc.cluster.local``` 時，則會返回預期的結果，因為查詢中指定了 namespace。
{{< alert warning >}}
特別注意```my-svc.np2.svc.cluster.local```，中間有個 ```svc```
{{< /alert >}}


{{< alert info >}}
```np1``` namespace 中的 Pod 可以成功地解析
- ```my-svc.np2```
- ```my-svc.np2.svc.cluster.local```
{{< /alert >}}

---
