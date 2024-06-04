---
title: "Kubernetes - Service : DNS Discovery"

author: Aryido

first draft: 2023-05-31T22:58:42+08:00

date: 2023-10-12T20:08:23+08:00

thumbnailImage: "/images/kubernetes/service-logo.jpg"

categories:
- containerization
- kubernetes

tags:
- network
- kubernetes-service

comment: false

reward: false
---
<!--BODY-->
> Kubernetes 是可以支援 ```ClusterIP:Port``` 、```PodIP:Port``` 的形式，來完成相互溝通的，但是這樣會帶來些問題，因為 Kubernetes 內部 Pod 和 Service 都有機會重啟的，這會導致 Pod 和 Service 的 IP 發生變化；但 Service 名字等一些標識資訊是不會經常變動的，所以 **Kubernetes 更推薦通過 Service 的名字來訪問服務**，這就是服務發現。Service Discovery 是一種機制，通過該機制，服務可以動態發現彼此，而無需 hardcode 硬寫 IP 或 endpoint 配置。可以讓我們只透過 Service 的名稱，就能找到相對應 Pod ，而非使用 IP 地址訪問。
>

<!--more-->

---

在 Kubernetes 中要和 Service 溝通有兩種主要方式，分別是
- **Environment Variable**，已經是內建的
- **DNS**，必須要安裝 addon 才會有


透過 Kubernetes DNS 可以讓**同一個 Cluster 中的所有 Pods ，都能透過 Service 的名稱找到彼此**。

### 環境變數 (不推薦使用)
當 Pod 創建時， kubelet 會在該 Pod 中**注入同一個 namespace 中有標籤關聯的 Service 的相關環境變量**。

{{< alert danger >}}
使用環境變數來取得 IP有一個限制，**pods 須在一個 namespace 中**，同一個 namespace 中的 pod 才會共享環境變量，如果不在同一個 namespace 就無法使用這個方法。
{{< /alert >}}

比如一個 ServiceName 爲```redis-master``` 的 Service，對應的 ```ClusterIP:Port``` 爲 ```10.0.0.11:6379```，則對應的環境變量爲：
{{< image classes="fancybox fig-100" src="/images/kubernetes/env-discovery.jpg" >}}

{{< alert warning >}}
若 Pod 建立的時間比 Service 還要早， kubelet 在填入環境變數的資訊時，會沒有 Service 可以填入，這會發生一些錯誤。
{{< /alert >}}


### DNS (最常使用的方式)

Service-A 中 Pod 與 Service-B Pod 之間的溝通，可以在其容器的環境變數中使用 Service IP 或是 Service Name 來實現。但由於 Service IP 提前可能並不會知道，因此引入服務發現，它的作用**就是監聽 Service 變更並更新 DNS**。

DNS 則是 addon，不會預設安裝進 Kubernetes 中，但目前普遍 Kubernetes 安裝相關的專案都會協助安裝 DNS service。最常見的是 kube-dns 和 CoreDNS，另外也可 add-on 使用其他 DNS 提供商來對 Cluster 內的 Pod 進行 Service Discovery。

Kubernetes cluster 上的每個 Service 在創建時，都會被自動委派一個格式為
- #### ```<service>.<ns>.svc.<zone>```

的名稱，詳細 Record 就暫不討論。

---

## Kubernetes DNS

在 Kubernetes 中，Service Discovery 一般常用的 DNS 提供商是使用 kube-dns 或 CoreDNS ，而 CoreDNS 從 v1.13 開始成為 Kubernetes 默認 DNS 服務。

{{< alert success >}}
補充一下，在 GCP 提供的 GKE :
- Autopilot mode 只能使用 kube-dns，無法修改 DNS 提供商
-  Standard mode 提供 :
   -  kube-dns (Default)
   -  Cloud DNS
{{< /alert >}}

Kubernetes 會在 kube-system 命名空間中用 Pod 的形式運行一個 DNS 服務，一旦 Kubernetes 被建立後，便會自動運行。每個 Kubernetes service 都會自動註冊到 DNS 服務之中，註冊過程大致如下：

- 向 API Server 用 POST 方式提交一個新的 Service ，這個請求需要經過認證。

- Service 得到一個 ClusterIP，並保存到 etcd

- DNS 服務會以**某種方式**得知有 Service 的創建，據此創建必要的記錄。

某種方式是甚麼呢 ? 舉例 CoreDNS ，它會對 API Server 進行監聽，一旦發現有新建的 Service 對象，就創建一個從 Service 名稱(```metadata.name```)映射到 ClusterIP 的域名記錄，Service 不必自行向 DNS 進行註冊。 Service 物件註冊到 DNS 服務之後，就能夠被其它 Pod 發現，Service 物件有一個 **Label Selector 屬性**，同樣有這標籤的 Pod 就會被納入到負載均衡目標之中。

若要 Pod 使用 Service Discovery 功能，會需要知道 DNS 服務器的位置才能使用它。因此每個 Pod 中的每個容器的 ```/etc/resolv.conf``` 都會被配置 DNS 服務的 domain name 與相對應的 IP 位址，接下來就需要通過外掛程式來啟動集群內的 DNS 服務了。

{{< image classes="fancybox fig-100" src="/images/kubernetes/coredns.jpg" >}}


{{< alert info >}}
k8s Domain Name 解析規則，會因為 Pod 所在的 namespace 而返回不同的結果。若沒有指定 namespace 的 DNS 查詢，會被限制在 Pod 所在的**當前 namespace 內**。
- k8s 中有 namespace 的概念，由於不同的 namespace 中可以有同樣名稱的 service ，因此 DNS 解析的部份就需要考慮 namespace

- k8s cluster domain name，若是未設定，預設就會是 ```cluster.local```
{{< /alert >}}

---
### 參考資料

- [K8s network之二：Kubernetes的域名解析、服務發現和外部訪問](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-two.html)

- [淺談 Kubernetes 中的服務發現](https://blog.fleeto.us/post/demystifying-kubernetes-service-discovery/)

- [[Kubernetes] Connecting Applications with Services](https://godleon.github.io/blog/Kubernetes/k8s-Connecting-Apps-with-Services/)

- [CoreDNS系列1：Kubernetes内部域名解析原理、弊端及优化方式](https://hansedong.github.io/2018/11/20/9/)