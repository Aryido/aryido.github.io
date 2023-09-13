---
title: "Kubernetes - Service"

author: Aryido

date: 2023-05-27T10:07:23+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> Pod 的生命週期是動態的，因為 cluster 會根據需求，動態地創建或銷毀 Pod ，重啟的 Pod 自然也伴隨着 IP 地址的更動。為了解決這問題，kubernetes 在 客戶端和 Pod 間，引入了一個名為 Service 的組件，它會在 pod 的前方提供了一個穩定的網路端點。**不只可以建立內部 Pod 之間的通信，讓 Pod 間可以用 domain name 的方式相互溝通；另外也可以建立外部與 Pod 的溝通管道**。 最後 Service 也有能力爲這些 Pod 進行負載分配，平均每個 Pod 的使用率。
<!--more-->

---

# 簡介
{{< image classes="fancybox fig-100" src="/images/kubernetes/service-2.jpg" >}}
當 kubernetes 創建一個 Service 時， 若 Service 有 labelSelector 關聯到 Pod，則 :
  - Control Plane 會使用 kube-controller-manager 中的 EndpointSlice controller 組件，**自動創建 endpointsSlice (舊版稱 endpoints)。**

  - endpointsSlice 包含該 Service 通過 labelSelector 關聯的所有 Pod IP 資訊。

  - EndpointSlice controller  會監聽 Pod 的狀態，若發現 Pod 有更動，會自動修正 endpointsSlice 中的 Pod IP 資訊。

{{< alert success >}}
以上流程其實這也說明了 Service 是依靠 EndpointSlice 才找到 Pod 的
{{< /alert >}}


**Service 的 IP 也稱為 ClusterIP**，是 Sevice 創建時，會被分配一個唯一的 IP 地址，這個 IP 地址與 Service 的生命週期是綁定在一起，故 Service 重新佈署時可能會導致 ClusterIP 改變。

再來 Kubernetes 在啟動後，會在每個 Node 上都會運行一個 kube-proxy，而 kube-proxy 也是一個 Pod ，更確切的說，就是以 daemonSet 形式啟動的 Pod。kube-proxy 會監聽 service 和 endpointsSlice 的變化，並自動對 iptables 進行相應修改。

而前往 Service (ClusterIP:port) 的網路流量，會由 iptables 重新導向到 Service 所代理的其中一個 Pod，進行負載平衡。以上解釋了為甚麼 Pod 無論 IP 怎麼變動，Service 都可以正確把流量倒到 Pod 得原因了。

```
                            +--------------------------+
                            | EndpointSlice-controller | ---------------+
                            +--------------------------+                |
                                         |                              |
                                      modifies                     monitored by
                                         v                              v
  +----------+   labelSelector   +---------------+   labelSelector   +------+
  | Service  | <---------------> | EndpointSlice | <---------------> | Pods |
  +----------+                   +---------------+                   +------+
       ^                                 ^
       |                                 |
       +----------monitored by-----------+
                       |
                       |
                  +------------+   modifies    +---------+
                  | kube-proxy | ------------> | iptables|
                  +------------+               +---------+

```


{{< alert success >}}
LabelSelector 代表著**鬆耦合**，Service 和 Pod 建立順序並沒有強制誰先誰後， Service 也可以比 Pod 早建立。
{{< /alert >}}


---

# [Service YAML](https://kubernetes.io/docs/concepts/services-networking/service/)

假定有一組 Pod，每個 Pod 都在偵聽 TCP port 9376 ，同時還被打上 ```app=my-app``` 標籤，簡單範例如下 :
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app # Deployment 的標籤
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app # Deployment 會管理有這個 Label 的 Pods
  template:
    metadata:
      labels:
        app: my-app  # Pod 的標籤
    spec:
      containers:
      - name: my-app
        image: my-app:1.0.0
        ports:
        - containerPort: 9376 # Pod 對外開放的 port number
```
特別注意，我們在```spec.template.metadata.labels```
打上標籤```app: my-app```，這代表設定由這個 Deployment 創建的每個 Pod 的 labels，即**當一個新的 Pod 被 Deployment 創建時，這個 labels 會被附加到新創建的 Pod 上。**
{{< alert warning >}}
通常 ```selector.matchLabels``` 的設置應該和 ```template.metadata.labels``` 一致。這樣 Deployment 才能管理它自己創建的 Pod。如果不一樣，會由於 label 不匹配， Deployment 不會視這些 Pod 為其所管理，就不會執行 Deployment 內設定的如 auto scale policy 等策略設定。
{{< /alert >}}



接下來定義一個 Service，為 Pod 提供穩定的通訊服務 :

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app  # 連結有同標籤名的 Pod
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376  # 對應 Pod 對外開放的 port number
```

透過標籤選擇器 LabelSelector，關聯到有標籤為 ```my-app``` 的 Pod，該 Service 會將所有具有標籤 ```my-app``` 的 Pod 的 TCP 9376 端口，暴露到 Service 80 端口上
- **targetPort**：

  container 接收流量的端口。例如我們在 Pod 內運行一個 port 9376 的 web container，所以我們要指定 Service 的 targetPort 為 9376，才能和 Pod 內的應用通信。
- **port**：

  創建的  Cluster IP 的 port ，這個 port 會去對應到 targetPort

{{< alert success >}}
Service 能夠將一個接收 ```port``` 映射到 ```targetPort```。若是 targetPort 沒有特別寫在 yaml 設定值的話，默認情況 targetPort 會為與 port **相同**。
{{< /alert >}}

---

# Kubernetes 的 Service 類型
一共有以下四種類型，若無特別標示，**預設是 ClusterIP**
###  ClusterIP

ClusterIP 為 Kubernetes Service 預設類型，此 IP 只有內部可以使用，無法被 cluster 外部的 client 訪問。為 Private IP ，僅可在 cluster 內使用。

### NodePort

在 Node 的 IP 地址上，選擇一個 port 來將外部請求，轉發到目標 Service 的 clusterIP:Port 。故這個類型的 Service ，根據不同狀況可為 Public IP 或者 Private IP。

{{< alert warning >}}
K8s部署時，預留的 NodePort 端口範圍是  ```30000~32767```
{{< /alert >}}

### LoadBalancer

LoadBalancer 類型的 Service ，會指向 k8s cluster 對應一個實際存在的負載均衡設置。通常會結合雲端平台如 GCP、AWS、AZURE 等等，我們可以透過這些 cloud 提供的 LoadBalancer ，來實現 Pod 對外的通訊及負載平衡。

{{< alert info >}}
是 Public IP ，雲端商會給該服務對外的 IP
{{< /alert >}}

{{< alert warning >}}
Kubernetes 提供兩種內建的負載均衡機制 :
- TCP 負載均衡器 : Service
- HTTP(S) 負載均衡器 : Ingress
{{< /alert >}}

---

# 參考資料

- [K8s network之五：Kubernetes集群Pod和Service之間通信的實現原理](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-five.html)

- [小信豬: [Kubernetes] Service Overview](https://godleon.github.io/blog/Kubernetes/k8s-Service-Overview/)

- [[Day 9] 建立外部服務與Pods的溝通管道 - Services](https://ithelp.ithome.com.tw/articles/10194344)

- [容器技术 K8s kube-proxy iptables 再谈](https://juejin.cn/post/7134143215380201479)

