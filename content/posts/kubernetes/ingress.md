---
title: "Kubernetes - Ingress"

author: Aryido

date: 2023-05-12T23:11:25+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
>  Kubernetes 若要把應用暴露於  **Cluster 外部** ，已知可以使用 **NodePort** 和 **LoadBlancer** 類型的 Service，但當 Service 越來越多時，我們就需要管理更多的 Port Number，也會使得維運上更加複雜。這時就可以考慮使用 Kubernetes Ingress ，它可用來代理不同 Kubernetes Service，能使 Node 對外開放**統一的 port** ，也可將外部的請求轉發到 **Cluster 內不同的 Service** 上，來實現負載均衡。 Ingress 專注於 Cluster 對外的暴露、負載均衡、L7轉發、 Virtual Hosting 等功能。
<!--more-->

---

## 緣起

Ingress 是一個可讓**請求從外部訪問 Kubernetes cluster** 的一個入口資源對象，其實就相當類似於 Nginx 等等這類代理服務器。

{{< image classes="fancybox fig-100" src="/images/kubernetes/ingress-2.jpg" >}}

那既然都有了 Nginx 等等這類負載均衡代理服務器，那為什麼還需要 Ingress 呢 ? 因為在 Kubernetes 使用中還會遇到一些問題，例如舉個實際例子，若我們使用 Nginx 來做反向代理 :
- 每次有新 Pod 時，都會需要更改 Nginx config ，把新的 Pod 資訊寫到 Nginx config 內
- 改完配置還要**重新啟動**或**重新部屬** Nginx 服務

若不想自己還要去手動更改 Nginx 設定，會需要自己來實做滾動更新、自動註冊等等...有些麻煩，但在這邊 Kubernetes Ingress 已經幫我們把上述問題統整起來，並**實現了上述所有的操作需求了** !

{{< alert info >}}
有了 Ingress 這個抽象，各家廠商可以依照這個 interface 去做各種不同的實現，比如說就 Kubernetes 這個 open source，目前[官方支持和維護 AWS、 GCE 和 Nginx Ingress Controller](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/)；我們可以根據自己的需求來自由選擇 Ingress Controller。Ingress 帶來的靈活度和自由度，對於使用容器時代來說，其實是非常有意義的 !
{{< /alert >}}

Ingress 會做的事情有:
- 檢查網址是否有在規則中，並進行分配
- 可設定是否有需要進行 SSL、TLS 驗證

很大的好處是**只需要對外開放一個 Port** ，再來才控制送來的請求應該被導向哪個 Service 服務

{{< alert warning >}}
使用 SSL、TLS 配置等 HTTP 相關的功能時，都必須通過 k8s Ingress 來進行，k8s service 無法做到。
Kubernetes 的 Service 只有四層代理，只支持 IP:Port 格式訪問。
而 Ingress **支持實現七層代理**。
{{< /alert >}}

---

##  Ingress、Ingress Rule、Ingress Controller 綜合講解
這三個名詞常常被提及，最混淆的是單單說 Ingress 這個詞的時候，但 Ingress Rule 和 Ingress Controller 這兩個還蠻具體可以解釋的，以下是它們的基本區別介紹：
### Ingress Rule
當我們在寫 Kubernetes Ingress Yaml 的時候，其中 ```spec.rules``` 這個部分，我們就稱呼 Ingress rule，非常具體的表示是指路由規則，簡單介紹如下
- ```host```：

  是 **optional**，沒有特別指定 host 屬性的話，指定 IP 的所有入站 HTTP 通信都會被 ingress 代理。

- ```http.paths``` :

  定義訪問路徑的 list ，每個路徑都有一個```backend.service``` 定義

- ```http.paths.backend``` :

  定義後端的 Service 服務，此外一般情況下在 Ingress 控制器中會配置一個 ```defaultBackend``` 默認後端，**但不是定義在 IngressRule 中。**

{{< alert success >}}
defaultBackend 設定是在 ```.spec.defaultBackend```，和 Ingress Rule 同個層級。
{{< /alert >}}
###  Ingress Controller
對於 Ingress Rule 的概念，就只是一組以 yaml 形式爲載體的聲明式的策略而已，它並不負責具體功能的執行。要真正完成對應的功能的具體實現，我們稱為 Ingress Controller。

因為要有 Ingress Controller ，才能滿足 Ingress 的要求，提供對應的功能，故會需要一個啟動中的實現 :
- 可能需在 k8s 上先部署 Ingress 控制器，例如 ingress-nginx，它其實就是一個 pod 並有著負載均衡功能的服務
- 或者我們當用 Google Kubernetes Engine (GKE) ，它自動提供了一個名為 GKE Ingress 的內置 Ingress Controller，真正實做，就是我們用的 Google Cloud Load-balancer

{{< alert success >}}
Ingress Controller 會不斷地監聽 kube-API Server，當得知到 Ingress yaml、Service、Pod 有變化，會自動更新配置。服務發現的功能 Ingress 自己已經實現了，為 Ingress Controller 的基本功能。
{{< /alert >}}

### [Ingress YAML](https://kubernetes.io/docs/concepts/services-networking/ingress/)
這是最容易產生歧異的名詞，首先 Ingress 概念一定包含 Ingress Rule 。 從 Ingress 整個 Yaml 來說，還需要指定 ```apiVersion```、```kind```、 ```metadata``` 和 ```spec``` ，Ingress Rule 只是在 ```spec``` 下的詳細子設定罷了，但有蠻多文章就直接把 Ingress Rule 簡稱為 Ingress。

再來當我們說 Ingress 時，有時候是在介紹整個 Kubernetes Ingress 的功能和架構，因為 Ingress 以 yaml 聲明， Ingress Controller 按照其 yaml 產生功能，這就包含抽象的資源定義以及默認的實現，故
Ingress 也有可能是指 Ingress Controller + Ingress Rule。

### 總結

- Ingress rule 類比爲: Nginx 中的配置文件 nginx.conf
- Ingress Controller 類比爲: Nginx 這個軟體
- Ingress 類比爲: 依照 nginx.conf 設定執行對應功能的 Nginx 服務

---

## Ingress Example

下面給一個簡單的範例 yaml ，Ingress 的定義依賴於 path ，每一個 path 都對應一個後端 Service :

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  defaultBackend:
    service:
      name: default-backend
      port:
        number: 80
  rules:
    http:
      paths:
      - path: /svc1
        pathType: ImplementationSpecific
        backend:
          service:
            Name: svc-1
            Port:
              number: 80
      - path: /svc2
        pathType: ImplementationSpecific
        backend:
          service:
            Name: svc-2
            Port:
              number: 80
```

{{< alert danger >}}
在 Kubernetes 1.19 及更高版本中，Ingress API 版本已升級為正式版 ```networking.k8s.io/v1```，並且 Ingress/v1beta1 標記為**已棄用**。更進一步在 Kubernetes 1.22 中，```networking.k8s.io/v1``` **已被移除**。
{{< /alert >}}

Ingress 新版本在 yaml 有重大更新，故**小心不要用到舊的版本**:
- ```spec.backend``` 變成 ```spec.defaultBackend```
- ```serviceName``` 變成 ```service.name```
- ```servicePort``` 分成兩種:
  - ```service.port.number```
  - ```service.port.name```

- ```pathType``` 現在對於每個指定的路徑都是**必需的**。值可以是：
  - Prefix
  - Exact
  - ImplementationSpecific

上面這個 yaml Ingress 資源的定義，配置了 :
- 路徑爲 ```/svc1``` 和  ```/svc2``` 的路由
- 所有 ```/svc1``` 和  ```/svc2``` 的請求，都會被 Ingress **個別轉發**至名爲 ```svc-1``` 和  ```svc-2``` 的服務的 80 端口的 / 路徑下。



{{< alert success >}}
Ingress 會用來管理 Service 。故可以說  Ingress ，就是 Service 的 Service ~
{{< /alert >}}

---
---
### 參考資料

- [K8s network之二：Kubernetes的域名解析、服務發現和外部訪問](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-two.html)
- [[Day 19] 在 Kubernetes 中實現負載平衡 - Ingress Controller](https://ithelp.ithome.com.tw/articles/10196261)
- [Day24 了解 K8S 的 Ingress](https://ithelp.ithome.com.tw/articles/10224065)
