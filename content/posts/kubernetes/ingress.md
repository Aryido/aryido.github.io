---
title: Kubernetes - Ingress

author: Aryido

date: 2023-05-12T23:11:25+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
>  Kubernetes **cluster 內部**使用 kube-dns 實現服務發現的功能；至於要暴露應用於 Kubernetes **cluster 外部** ，已知可以使用 **NodePort** 和 **LoadBlancer** 類型的 Service，除此之外，還可以使用 Kubernetes Ingress 服務。Ingress 可用來代理不同 Kubernetes Service ，將外部的請求轉發到 **cluster 內不同的 Service** 上，來實現負載均衡。
<!--more-->

## 緣起

Ingress 資源對象，是一個可讓**請求從外部訪問 cluster 的一個入口資源對象**，其實就相當類似於 nginx 等等這類負載均衡代理服務器。

{{< image classes="fancybox fig-100" src="/images/kubernetes/ingress-2.jpg" >}}

那既然都有了 nginx 等等這類負載均衡代理服務器，那為什麼還需要 Ingress 資源對象呢 ? 因為在 Kubernetes 使用中還會遇到一些問題，例如舉個實際例子，若我們使用 nginx 來做反向代理 :
- 每次有新服務 pod 加入時，都會需要**改 Nginx 配置**
- 改完配置還要**重新啟動**或**重新部屬** Nginx 服務

若不想自己還要去手動更改 Nginx 設定，或者還要滾動更新 Nginx Pod ，則可能會需要再加上**服務發現 server** 比如 Zookeeper、Eureka 之類的來自動註冊... 這個想法是沒有錯的 ! 但自己去設定這些 infra 功能有些麻煩，所以 **Ingress 幫我們實現了上述所有的操作需求了** !

{{< alert success >}}
服務發現的功能 Ingress 自己已經實現了，不需要使用第三方的服務了；再加上域名規則定義，路由資訊的刷新依靠 Ingress Controller
{{< /alert >}}

{{< alert info >}}
有了 Ingress 這個抽象，各家廠商可以依照這個 interface 去做各種不同的實現，比如說就有 Nginx Ingress Controller；我們可以根據自己的需求來自由選擇 Ingress Controller。

Ingress 帶來的靈活度和自由度，對於使用容器時代來說，其實是非常有意義的 !
{{< /alert >}}

---

## Ingress

下面給一個簡單的範例 yaml ，IngressRule 的定義依賴於 path ，每一個 path 都對應一個後端 Service :

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
在 Kubernetes 1.19 及更高版本中，Ingress API 版本已升級為正式版 ```networking.k8s.io/v1```，並且 Ingress/v1beta1 標記為**已棄用**。更進一步在 Kubernetes 1.22 中，```networking.k8s.io/v1``` **已被移除**。

這其中 yaml 有重大更新，故不要用到舊的版本:
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



{{< alert info >}}
可以將 Ingress 狹義的類比爲 Nginx 中的配置文件 nginx.conf。
{{< /alert >}}

{{< alert success >}}
Ingress 會用來管理 Service 。故可以說  Ingress ，就是 Service 的 Service ~
{{< /alert >}}

###  Ingress Controller

Ingress Controller 通過不斷地監聽 kube-apiserver，當得知到 Service、Pod 有變化後，Ingress Controller 會結合 Ingress 的配置，更新反向代理負載均衡器。  常用的各種反向代理項目，比如 Nginx 等，都已經爲 Kubernetes 專門維護了對應的 Ingress Controller。例如 ingress-nginx，它以 pod 形式運行。

###  IngressRule
以 yaml 聲明 ingress-controller 會按照策略生成配置文件

- host：

  **可選字段**，沒有特別指定 host 屬性的話，通過指定 IP 地址的所有入站 HTTP 通信都會被 ingress 代理。

- http.paths :

  定義訪問路徑的 list ，每個路徑都有一個backend.service 定義

- http.backend :

  定義後端的 Service 服務，此外一般情況下在 Ingress 控制器中會配置一個 defaultBackend 默認後端，**但不是定義在 IngressRule 中。**


另外當我們想要在 Kubernetes 內，爲應用進行 TLS 配置等 HTTP 相關的操作時，都必須通過Ingress 來進行。

{{< alert warning >}}
Kubernetes 的 Service 只有四層代理，只支持 IP:Port 格式訪問。
而 Ingress api 支持實現七層代理。
{{< /alert >}}





