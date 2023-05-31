---
title: Kubernetes - Service

author: Aryido

date: 2023-05-27T10:07:23+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> Pod 的生命週期是動態的，因為 Deployment 可以動態地創建和銷毀 Pod，自然也伴隨着 IP 地址的更動。 Kubernetes Service 可創建一個穩定的網路端點，設定訪問 Pod 的策略，並爲這些 Pod 進行負載分配。 Service 不只可以**建立外部服務與 Pod 的溝通管道，對內部也可以建立 Pod 之間的通信**，為 Pod 提供統一的代理接口，確保每次存取應用程式服務時，都能連結到正在運行的 Pod。

<!--more-->

---
Kubernetes 中運行著 Pod 時，可以通過 ssh 登錄到 cluster 中的**任何一個節點**上，並使用 ```kubectl get pods``` 拿出 Pod 的 IP 地址，並使用諸如 curl 之類的工具，向這 IP 地址發出查詢請求，都是可以通的 !

{{< image classes="fancybox fig-100" src="/images/kubernetes/service-2.jpg" >}}

**Service 的 IP 也稱為 ClusterIP** ，是 Sevice 創建時，會被分配一個唯一的 IP 地址，這個 IP 地址與 Service 的生命週期是綁定在一起。

- Port Proxy

  Service Port 用於接收 client 請求，再轉發至 Pod 上面對應的端口，它運作在TCP/IP protocol 的**四層傳輸層**。

- LabelSelector

  Service 透過 **LabelSelector** 來關聯 Pod ，每個 Node 上的 kube-proxy 會透過 API Server watch 隨時監控 Service 或 LabelSelector 匹配的 Pod 對象是否有變動。

{{< alert warning >}}
LabelSelector 代表著**鬆耦合**，Service 和 Pod 建立順序並沒有強制誰先誰後，Service 也可以比 Pod 早建立。
{{< /alert >}}

Pod 會因爲伸縮、更新、故障等情況發生變化，而 Service 會對這些變化進行跟蹤。同時 Service 的名字、IP 和端口都不會發生變化。

---

## [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

假定已經有一組 Pod，每個 Pod 都在偵聽 TCP port 80，同時還被打上 ```app=my-app``` 標籤。接下來定義一個 Service 來發布 TCP 偵聽器。

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

透過標籤選擇器，關聯到標籤為 ```my-app``` 的 Pod，control-plane 會自動為設置了 ```LabelSelector``` 的 Kubernetes Service 創建 EndpointSlice。

該 Service 會將所有具有標籤 ```my-app``` 的 Pod 的 TCP 80 端口，暴露到 Service 端口上
- targetPort：

  容器接收流量的端口，例如我們在 Pod 中運行一個 port 80 的 web container，所以我們指定 my-service 的 targetPort 為 80
- port：

  創建的 Service 的 Cluster IP，是哪個 port 去對應到 targetPort

{{< alert warning >}}
舊版本的 Kubernetes 中，服務的端點資訊是由 Endpoints 資源類型來管理的。現在新版本稱為 EndpointSlices 。
{{< /alert >}}

{{< alert info >}}
Service 能夠將一個接收 ```port``` 映射到 ```targetPort```。默認情況 targetPort 會為與 port 相同。
{{< /alert >}}

---

## Service 類型

### ClusterIP
透過內部 IP 地址暴露服務，此 IP 只有內部可以使用，無法被 cluster 外部的 client 訪問。

{{< alert info >}}
為 Private IP ，是 Service 在 cluster 內的專屬地址，僅可在 cluster 內使用
{{< /alert >}}

### NodePort
在工作節點的 IP 地址上，選擇一個 port 來將外部請求，轉發到目標 Service 的 clusterIP 和 Port ，所以這個類型的 Service 可以向收到內部也可以收到外部 Client 的請求。

{{< alert info >}}
根據不同狀況可為 Public or Private IP
{{< /alert >}}


{{< alert warning >}}
K8s部署時，預留的 NodePort 端口範圍是  ```30000~32767```
{{< /alert >}}

### LoadBalancer
LoadBalancer 類型的 Service 會指向 **k8s cluster外部**的一個實際存在的負載均衡設置，通常結合雲端平台如 GCP、AWS、AZURE 等等，我們可以透過這些 cloud provider 提供的 LoadBalancer ，幫我們分配流量到每個 Node 。

{{< alert info >}}
為 Public IP ，雲端商會給該服務的對外 IP
{{< /alert >}}

---


{{< alert warning >}}
Kubernetes 提供兩種內建的雲端負載均衡機制 :
- TCP負載均衡器 : Service
- HTTP(S)負載均衡器 : Ingress。
{{< /alert >}}





