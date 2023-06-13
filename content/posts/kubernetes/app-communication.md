---
title: "Kubernetes -  應用之間是如何溝通"

author: Aryido

date: 2023-06-11T00:29:32+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
>  kubernetes 可以創建多個 Pods，Pod 內有一個或多個 Container，那麼 Container 之間是怎麼溝通的的呢 ? 這裡歸類出一些 case :
> - 同一個 pod 中，不同的 container 的通訊
> - 同一網路下，不同 pod 間的通訊
> - 不同網路下，不同 pod 間的通訊
>
> 以下來對這些 case 進行說明。

<!--more-->

---

## Scenario
比如一個 web 應用，通常會有 frontend 和 backend ，分別創建在不同的 pod 裡。在正常情況下，我們是只期望外部訪問 frontend ，而不希望 backend 直接被外部訪問到的。假設我們已經寫完  backend k8s yaml，其中 ```containerPort: 8080``` 並啟動了 backend Pod 。接下來想要開始寫 frontend k8s yaml，一個重點問題是  **frontend 要如何才能訪問到 backend Pod 呢**?

這裡可以使用 ```kubectl get pods -o wide``` ，可以查看到到後端 Pod 的資訊。這邊假設 backend Pod IP 為 ```10.1.1.69```，那麼前端能不能通過 ```10.1.1.69:8080``` 訪問後端 Pod 呢？

## 測試
本機測試的話，先創建一個 k8s frontend service yaml ，讓 local 可以透過 k8s frontend service 連接到前端 Pod，並且 frontend service yaml 要填幾個重點 :
- ```spec.selector```
- ```spec.type: NodePort```
- ```spec.ports.nodePort: :30002```
{{< alert warning >}}
nodePort ```30000-32767```，注意範圍，不要超過或小於了
{{< /alert >}}

接下來創建 k8s frontend deployment yaml ，並重點要加上 :
- ```spec.selector.matchLabels``` 且要和 k8s backend service yaml 的 ```spec.selector``` 一樣
- 環境變數傳入 ```10.1.1.69:8080``` 使得前端可以訪問後端

這樣本機端就可以經由 ```http://localhost:30002/``` 正常訪問前端了 ! 但要來考慮一些問題了 :

- {{< alert danger >}}
Deployment 會根據 replicas 創建對應數量的 Pods，此時如果某個 Pod 掛了， k8s 會發現 Pods 數量和 replicas 定義的不一樣，變會重新再起一個 Pod ，此時 Pod 的 IP 可能就會變了。
上面針對 Pod IP 把它交給應用是不對的 !
{{< /alert >}}

- {{< alert danger >}}
另外我們設置了 replicas 的數量主要是為了做**負載均衡**，所以如果在應用裡將 IP 寫死指定某個 Pod 的話，就起不到負載均衡了。
{{< /alert >}}

我們決定新寫一個 k8s backend service yaml ，把後端暴露出來。寫完起動後，可通過 ```kubectl get service -o wide``` 查看 Service 詳情。這邊假設 k8s backend Service yaml 的 type 為 ClusterIP 且 Service IP 是 ```10.99.100.101``` 且使用了 ```8080``` 作為對外 Port。

**我們本機端是不能通過這個 ```10.99.100.101:8080``` 訪問到 backend Service 的。但是如果在 k8s 裡的相同網路應用，可以通過 ```10.99.100.101:8080``` 來訪問到的**。故更新 k8s frontend deployment yaml 把環境變數傳入  ```10.99.100.101:8080``` ，這樣前端就可以訪問後端了 !

但繼續考慮一下問題 :
- {{< alert danger >}}
使用 service 的 ClusterIP 雖然可以解決了由於 pod 的重啟更換 IP 的問題，但是如果一個 service 重啟，那麼 service的 IP 也會改變，這肯定是不行的...
{{< /alert >}}

**上面解法全部都不是好解法呢**...那怎麼樣才是好的解法呢 ? 以下說明 :

---

## 同一個 pod 中，不同的 container 的通訊
Pod 內部， container 是共用一個網路 namespace 的，所以**可以通過 localhost 來互相通訊**。對 container 來說， **hostname 就是 Pod 的名稱**。因為 **Pod 中的所有 container 共用同一個 IP 和 port 空間**，因此需要為每個容器分配不同的 port 。

{{< alert info >}}
Pod 中的應用需要自己協調管理 port 的使用。
{{< /alert >}}

{{< alert info >}}
k8s Pod 在啟動其他 container 的時候會先啟動一個叫 pause container，新創建的 container 和 pause container 共用同一個網路 namespace，而不是和宿主機共用。
{{< /alert >}}

---

## 同一網路下，不同 pod 間的通訊
這種 case 可能是最常見的場景。首先承上範例，使用 ```kubectl exec -it <deployment-name> /bin/bash``` 命令進入到 Pod 內部，接著使用 ```env``` 命令查看系統的環境變數。會發現可以看到和 Service 有關的環境變數如下:
- ```< service-name 轉成全大寫且"中橫線- " 變成"下底線_ " >_HOST=10.99.100.101```
- ```< service-name 轉成全大寫且"中橫線- " 變成"下底線_ " >_PORT=8080```

###  - 使用環境變數來訪問 Service
可以發現 ```10.99.100.101``` 和 ```8080``` 剛好是前面提過的 backend Service IP 和對應的 Port 。有了這**環境變數**，我們就可以動態獲取 backend Service IP ， 藉由 Service IP 當然就可以連接到 deployment 的某個 Pod ，便可完成通信 !

{{< alert danger >}}
可以使用環境變數來解析 Service IP ，但有一個限制，Pod 和 Service 須在**同一個 namespace 中**。
{{< /alert >}}

---

## 不同網路下，不同 pod 間的通訊
承前範例， 如果 Deployment 和 Service 都沒有指定 namespace的話，預設是創建在了 **default** 的 namespace。 如果沒有在同一個 namespace 下，那就推薦使用 DNS 來溝通服務。

### - 使用 DNS 來解析服務位址
除了可以使用環境變數來解析 Service IP ，但用的更多的應該是使用 DNS。在創建 k8s-Service 時， k8s 會創建一個相應的 DNS 紀錄，形式邊來說是 ```<service-name>.<namespace>.svc.cluster.local```。

---