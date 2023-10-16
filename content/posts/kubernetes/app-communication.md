---
title: "Kubernetes - Service : 應用之間是如何溝通"

author: Aryido

first draft: 2023-06-11T00:29:32+08:00

date: 2023-10-12T20:09:23+08:00

thumbnailImage: "/images/kubernetes/service-logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
>  kubernetes 可以創建多個 Pods，Pod 內有一個或多個 Container，那麼 Container 之間是怎麼溝通的的呢 ? 這裡歸類出一些 case :
>
> - 不同網路下，不同 pod 間的 container 的通訊
> - 同一網路下，不同 pod 間的 container 的通訊
> - 同一個 pod 中，不同的 container 的通訊
>
> 以下來對這些 case 進行說明。

<!--more-->

---

## Scenario
比如一個 web 應用，通常會有 frontend 和 backend ，分別創建在**不同的 pod 裡**。在正常情況下，我們**只期望外部訪問 frontend ，而不希望 backend 直接被外部訪問到的**。假設我們已經寫完  backend 的 k8s yaml，其中 ```containerPort: 8080``` 並啟動了 backend Pod 。接下來想要開始寫 frontend k8s yaml，一個重點問題是:

- ### frontend 要如何才能訪問到 backend Pod 呢?

先使用 ```kubectl get pods -o wide``` ，來查看到到後端 Pod 的資訊。這邊假設看到 backend Pod IP 為 ```10.1.1.69```，那麼前端能不能通過 ```10.1.1.69:8080``` 這個 Pod IP ，來訪問後端 Pod 呢？

本機測試一下吧 ~

首先先創建一個 k8s frontend service yaml ，讓本機端可以透過 k8s frontend service 連接到**前端 Pod** ，再來**前端 Pod** 才用 ```10.1.1.69:8080``` 這個 Pod IP ，來訪問**後端 Pod**。 frontend service yaml 要填幾個重點 :
- ```spec.selector```，並給一個標籤值
- ```spec.type: NodePort```
- ```spec.ports.nodePort: :30002```
{{< alert warning >}}
nodePort ```30000-32767```，注意範圍，不要超過或小於了
{{< /alert >}}

再來創建 k8s frontend deployment yaml ，並重點要加上 :
- ```spec.selector.matchLabels``` 且要和 k8s backend service yaml 的 ```spec.selector``` 標籤值一模一樣
- 環境變數傳入 ```10.1.1.69:8080``` 使得前端可以訪問後端

YAML 部屬成功後，本機端應該就可以經由 ```http://localhost:30002/``` 正常訪問網站資源了 ! 但要來考慮一些需要面對的問題 :

- {{< alert danger >}}
Deployment 會根據 replicas 創建對應數量的 Pods，此時如果某個 Pod 掛了， k8s 會發現 Pods 數量和 replicas 定義的不一樣，變會重新再起一個 Pod ，此時 **Pod 的 IP 可能就會變了**。
上面針對 Pod IP 把它交給應用是不對的 !
{{< /alert >}}

- {{< alert danger >}}
另外我們設置了 replicas 的數量主要是為了做**負載均衡**，所以如果在應用裡將 IP 寫死指定某個 Pod 的話，就起不到負載均衡了。
{{< /alert >}}

上面的寫法並不好 ~
**不要直接寫死後端 Pod IP 直接連接，應該在後端 Pod 前面加一個 backend service 來管理 Pod !**

故新寫一個 k8s backend service yaml ，把後端暴露出來。寫完 YAML 部屬後，可通過 ```kubectl get service -o wide``` 查看 Service 詳情。這邊假設 k8s backend Service yaml 的 type 為 ClusterIP 且 Service IP 是 ```10.99.100.101``` 且使用了 ```8080``` 作為對外 Port。

**雖然我們本機端是不能直接通過這個 ```10.99.100.101:8080``` 訪問到 backend Service 的；但是如果在 k8s 裡的 Pod ，可以通過 ```10.99.100.101:8080``` 來訪問到的**。故再回到之前 k8s frontend deployment yaml 更新它，把環境變數改傳入  ```10.99.100.101:8080``` 這個  Service ClusterIP ，這樣前端也是可以訪問後端了 ! 但繼續考慮一下問題 :
- {{< alert danger >}}
使用 service 的 ClusterIP 雖然可以解決了由於 pod 的重啟更換 IP 的問題，但是如果 service 重啟，那麼 service的 IP 也會改變，這肯定是不行的...
{{< /alert >}}

**上面解法全部都不是好解法呢**...那怎麼樣才是好的解法呢 ? 以下說明一下各個場景 :

---

## 不同網路下，不同 pod 間的通訊

- ### 使用 DNS 來解析服務位址

在創建 k8s-Service 時， Kubernetes 會創建一個內部相應的 DNS 紀錄，這個名稱是根據 **k8s-Service 名稱**和 **Namespace** 生成的，可以使用一致的 DNS 名稱而非 IP 存取 Service。形式是 :

```<service-name>.<namespace>.svc.cluster.local```
{{< alert warning >}}
特別注意，中間有個 ```svc```
{{< /alert >}}
{{< alert success >}}
重要常用設定 ! DNS 查詢會因為  Pod 所在的 namespace ，而返回不同的結果。不指定 namespace 的 DNS 查詢會被限制在本身 Pod 所在的 namespace 內。 要訪問其他名字空間中的 Service，需要在 DNS 查詢中指定名字空間。
{{< /alert >}}

假定有:
- namespace 為 ```np1```；service 為 ```svc1```；內有一個 ```Pod1```
- namespace 為 ```np2```；service ，為 ```svc2```

在```Pod1``` 查詢 ```svc2```
會找不到，因為會是在是  ```Pod1``` 的 namespace  ```np1``` 找```svc2```，這當然會找不到。若在```Pod1``` 查詢 ```svc2.np2``` 或者 ```svc2.np2.svc.cluster.local``` 時，則會返回預期的結果，因為查詢中指定了 namespace。在本範例中```np1``` namespace 中的 Pod 可以成功地解析:
- ```my-svc.np2```
- ```my-svc.np2.svc.cluster.local```




## 同一網路下，不同 pod 間的通訊
這種情況有兩種方式可以使用: DNS 、 環境變數

#### 使用 DNS 來解析服務位址 (比較推薦)

> 承前範例， 如果 Deployment 和 Service 都沒有指定 namespace 的話，預設是創建在了 **default** 的 namespace。故在創建 k8s-Service 時， k8s 會創建一個相應的 DNS 紀錄，形式邊來說是 :
>
> ```<service-name>.default.svc.cluster.local```。

#### 使用環境變數來訪問 Service

> 承前範例，使用 ```kubectl exec -it <deployment-name> /bin/bash``` 命令進入到 Pod 內部，接著使用 ```env``` 命令查看系統的環境變數，會發現可以看到和 Service 有關的環境變數如下:
>
> ```< backend-service-name 轉成全大寫且"中橫線- " 變成"下底線_ " >_HOST=10.99.100.101```
>
> ```< backend-service-name 轉成全大寫且"中橫線- " 變成"下底線_ " >_PORT=8080```
>
> 可以發現 ```10.99.100.101``` 和 ```8080``` 剛好是前面提過的 backend Service IP 和對應的 Port 。有了這**環境變數**，我們就可以動態獲取 backend Service IP ， 藉由 Service IP 當然就可以連接到 deployment 的某個 Pod ，便可完成通信 !

{{< alert danger >}}
可以使用環境變數來解析 Service IP ，但有一個限制，Pod 和 Service 須在**同一個 namespace 中**。
{{< /alert >}}

---

## 同一個 pod 中，不同的 container 的通訊

- ### 解答: 用 localhost 來互相通訊

在 kubernetes 集群中，每個 Pod 會被分配一個 IP 位址。 同一個 Pod 內的多個容器會共用這個 IP 位址對外通信，同時，這些容器之間也可以直接通過 ```localhost：Port``` 的方式進行通信。因為 container 是共用一個網路 namespace 的，所以**可以通過 localhost 來互相通訊**，也因此流量並不會經過 eth0 介面。 localhost 不依賴 **data link layer** 和 **physical layer** 協議，一旦 transport layer 偵測到目的是 localhost ，資料封包離開 network layer 時，就會被傳回本機的連接埠應用。 這種模式傳輸效率較高，非常適合容器間進程的頻繁通訊。

對 container 來說， **hostname 就是 Pod 的名稱**。因為 **Pod 中的所有 container 共用同一個 IP 和 port 空間**，因此需要為每個容器分配不同的 port 。

{{< alert warning >}}
Pod 中的應用需要自己協調管理 port 的使用。
{{< /alert >}}

{{< alert info >}}
k8s Pod 在啟動其他 container 的時候會先啟動一個叫 pause container，新創建的 container 和 pause container 共用同一個網路 namespace，而不是和宿主機共用。
{{< /alert >}}

---
### 參考資料

- [K8s network之四：Kubernetes集群通信的實現原理](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-four.html)
- [窺探 kubernetes 中 pod 之間的通信](https://testerhome.com/articles/24797)