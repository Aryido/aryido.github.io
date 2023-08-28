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
> Pod 的生命週期是動態的，因為 cluster 會根據需求，動態地創建或銷毀 Pod ，這自然也伴隨着 Pod IP 地址的更動。 Kubernetes Service 在 pod 的前方提供了一個抽象層，創建一個穩定的網路端點，不只可以**對內部建立 Pod 之間的通信，另外也可以建立外部服務與 Pod 的溝通管道**，為 Pod 提供統一的代理接口。同時 Service 讓服務間可以用 domain name 的方式存取 pod，並爲這些 Pod 進行負載分配。

<!--more-->

---

Kubernetes 集群內部 Pod 之間的溝通，預設是通過 Service。Kubernetes 系統在每個節點上都會運行一個 kube-proxy，會監控 Service 和 Pod ，Pod 會因爲伸縮、更新、故障等情況發生變化，而 iptables 進行相應修改。

{{< image classes="fancybox fig-100" src="/images/kubernetes/service-2.jpg" >}}
前往 Service (ClusterIP:port) 的網路流量，會由 iptables 重新導向到 Service 所代理的其中一個 Pod，進行負載平衡。
- **Service 的 IP 也稱為 ClusterIP**

  是 Sevice 創建時，會被分配一個唯一的 IP 地址，這個 IP 地址與 Service 的生命週期是綁定在一起，故 Service 重新佈署時可能會導致 ClusterIP 改變。

- **LabelSelector**

  **Service 透過 LabelSelector 來關聯 Pod** ，每個 Node 上的 kube-proxy 會透過 API Server watch 隨時監控 Service 或 LabelSelector 匹配的 Pod 對象是否有變動。

{{< alert warning >}}
LabelSelector 代表著**鬆耦合**，Service 和 Pod 建立順序並沒有強制誰先誰後， Service 也可以比 Pod 早建立。
{{< /alert >}}


---

## [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

假定已經有一組 Pod，每個 Pod 都在偵聽 TCP port 9376 ，同時還被打上 ```app=my-app``` 標籤。接下來定義一個 Service :

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  # type 一共有四種(ClusterIP, NodePort, LoadBalancer, ExternalName)
  # 預設是 ClusterIP
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    # 此為 Pod 對外開放的 port number
    targetPort: 9376
```

透過標籤選擇器 LabelSelector，關聯到有標籤為 ```my-app``` 的 Pod，該 Service 會將所有具有標籤 ```my-app``` 的 Pod 的 TCP 9376 端口，暴露到 Service 80 端口上
- **targetPort**：

  容器接收流量的端口，例如我們在 Pod 中運行一個 port 9376 的 web container，所以我們指定 my-service 的 targetPort 為 9376
- **port**：

  創建的 Service 的 Cluster IP，是哪個 port 去對應到 targetPort

{{< alert info >}}
Service 能夠將一個接收 ```port``` 映射到 ```targetPort```。若是 targetPort 不設定，默認情況 targetPort 會為與 port 相同。
{{< /alert >}}

---

## Service 類型

### ClusterIP
Service 預設是 ClusterIP ，透過內部 IP 地址暴露服務，此 IP 只有內部可以使用，無法被 cluster 外部的 client 訪問。

{{< alert info >}}
為 Private IP ，是 Service 在 cluster 內的專屬地址，僅可在 cluster 內使用
{{< /alert >}}

### NodePort
在工作節點的 IP 地址上，選擇一個 port 來將外部請求，轉發到目標 Service 的 clusterIP 和 Port ，所以這個類型的 Service 可以收到內部也可以收到外部 Client 的請求。

{{< alert info >}}
根據不同狀況可為 Public 或者 Private IP
{{< /alert >}}


{{< alert warning >}}
K8s部署時，預留的 NodePort 端口範圍是  ```30000~32767```
{{< /alert >}}

### LoadBalancer
LoadBalancer 類型的 Service ，會指向 k8s cluster 對應一個實際存在的負載均衡設置。通常會結合雲端平台如 GCP、AWS、AZURE 等等，我們可以透過這些 cloud provider 提供的 LoadBalancer ，幫我們分配流量到每個 Node 。

{{< alert info >}}
為 Public IP ，雲端商會給該服務的對外 IP
{{< /alert >}}

{{< alert warning >}}
Kubernetes 提供兩種內建的雲端負載均衡機制 :
- TCP負載均衡器 : Service
- HTTP(S)負載均衡器 : Ingress。
{{< /alert >}}

---

### 參考資料

- [K8s network之五：Kubernetes集群Pod和Service之間通信的實現原理](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-five.html)

- [小信豬: [Kubernetes] Service Overview](https://godleon.github.io/blog/Kubernetes/k8s-Service-Overview/)

- [[Day 9] 建立外部服務與Pods的溝通管道 - Services](https://ithelp.ithome.com.tw/articles/10194344)

