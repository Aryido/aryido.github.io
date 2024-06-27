---
title: GCP - Load Balancer 架構概述

author: Aryido

#first-date: 2023-06-26T23:47:00+08:00
date: 2024-06-17T16:54:22+08:00

thumbnailImage: "/images/google-cloud/lb/lb-logo.jpg"

categories:
- cloud
- gcp

tags:
- load-balancer

comment: false

reward: false
---
<!--BODY-->
> Cloud Load Balancing 是 GCP 透過平均分發流量到多個 server ，可以防止單一伺服器的過載，從而減少系統故障的風險的產品，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) : Elastic Load Balancing
> - Microsoft Azure : Azure Load Balancer
> 
> 因為只需透過配置單個負載平衡器的對外 IP 地址和憑證，故可以達到降低維運成本的目的，目前 若從 GCP console 上，由**流量類型**大概分成了兩類 : HTTP(S) load balancing、TCP/UPD load balancing，但實際上依照細部功能，還有分 Global/Regional 、Internal/External 等等，總體設定蠻細緻的 :
>
> {{< image classes="fancybox fig-100" src="/images/google-cloud/lb/lb-types.jpg" >}}
<!--more-->

---

# GCP load-balancer 架構
雲端服務最大的優勢 Load Balancer 與 Autoscaling ，就是雲端彈性展現的代表。 GCP 負載平衡的設置主要分為:
- **Frontend**
- **Backend Service**
- **Backends**

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/lb-architecture.jpg" >}}

這邊特別注意，在 GCP load-balancer 架構中，我們常提到  Frontend、Backend 這些關鍵字，**記住這些跟我們常聽到網頁前端、後端是不一樣的東西 !** 是代表 load-balancer 的組成組件。


#  Frontend
load-balancer 訪問的流量，都會先傳入 Frontend ，再依據**連線方式**與**轉發規則**被送往不同的 backend service 。主要有四個組件:

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/frontend.jpg" >}}

### Forwarding rule：
Global load balancing with **single anycast IP**，透過設置 IP 讓傳入流量進入負載平衡器，並運用對應的 Protocol 與 Port 將流量轉至 Proxy

### Target proxy：
設定 HTTP request / response 常見的 header，作為 Client 與 Server 間的橋梁，可設置 SSL 憑證安全連線
{{< alert info >}}
還可以設定一個參數叫 `X-Cloud-Trace-Context` ，可以透過 Stackdriver 紀錄追蹤 HTTP Request。這在 Microservice 架構找問題很重要的追蹤參考。
{{< /alert >}}

### SSL certificate：
設定加密協議，這個憑證可以是 Google 幫我們管理，或由我們自行管理

### URL map：
定義一些規則來將不同需求或類型的流量導到不同的 backned service，例如:
- By Path — `如不同的網址路徑 - aryido.com/a aryido.com/b`
- By Host — `不同的站台 - asite.aryido.com 與 bsite.aryido.com`
- By HTTP - `headers (Authorization header) or methods (POST, GET,等)`

{{< alert info >}}
有點像是 AWS ALB 的 Target Group
{{< /alert >}}


# Backend service
Backend service 主要是定義負載平衡器如何分配流量到我們設置的 Backends 資源，由 health checks 和一到多個 backends 組成，可以先理解成 Backend service 管理 Backends。

Backend service 會藉由 health check 設定的頻率，向指定的 port 探測並取得回應，確保資源健康之後， Backend service 才將流量導向 Backends。
{{< alert danger >}}
這邊提醒，要對後端資源進行健康檢查就必須設置 Firewall rule ，允許來自 Health check 的流量

- 130.211.0.0/22
- 35.191.0.0/16

這邊初次使用，超級容易忽略的。
{{< /alert >}}

也可以在 Backend service 這裡設定 :
- Cloud CDN 提供 **CDN 服務**
- 指定「會話親和性(Session affinity)」，會將來自同一用戶端的後續請求，都送到到同一 backend，這可確保使用者 session 在一段時間內保持一致，從而提高應用程式性能和用戶體驗

- 服務超時設定，定義負載均衡器在將後端實例視為失敗之前等待回應的最長時間

- 流量分佈演算法，使負載均衡器用於在後端實例或服務之間分配傳入請求的方式


# Backends
為從 Load Balance Frontend 接收流量的 endpoint group ，可以分成以下幾個 :
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/backend.jpg" >}}

### Instance group：
多個 vm 放在一個群組以集中管理，可以有 Auto Scaling 功能，分為
  - Google 代管的 Managed Instance Group(MIG)
  - 我們自行管理的 Unmanaged Instance Group(UMG)

{{< alert info >}}
相當於 AWS Auto Scaling Group
{{< /alert >}}

### Cloud Storage：
針對 html、css、js、圖片和影片等靜態內容，直接存在 Cloud Storage 可有效節省資源，常和 CDN 搭配使用


### Network Endpoint Group（NEG）:
對於 cloud run 、 k8s Pods 等等虛擬化容器類型的群組，都是屬於 NEG

---

# External HTTP(S) Load Balancer
這算是我們最常使用的負載平衡器，是給 HTTP / HTTPS 使用的，分流的步驟如下 :
- Forwarding rule 指定一個外部 IP 和 Port，讓 User 知道，並讓流量透過單一 IP 進入負載平衡器，接下來會把 request 轉發到 Target proxy
- Target proxy 會依 URL Map ，將流量導向對應的 Backend service 。若使用 HTTPS 負載平衡器，則 SSL 憑證也是由 Target proxy 關聯的
-  Backend service 是會對後端資源進行 health check ，確保流量送至健康的Backends，

{{< alert danger >}}
HTTP(S) Load Balancer 為全球性的 GCP 資源，新建立完 Load Balancer 之後，更新時間約為 5 分鐘，在啟動之前完成使用 curl 開啟該 Load Balancer 的 IP 會出現 502 的錯誤訊息
{{< /alert >}}

---

### 參考資料
- [什麼是負載平衡？原理、6大 GCP Load Balancer 完整介紹](https://blog.cloud-ace.tw/networking-website/load-balance/gcp-load-balancer-introduction/)

- [Cloud Load Balancing 概覽](https://cloud.google.com/load-balancing/docs/load-balancing-overview?hl=zh-cn)

- [IT邦-twmjavacoe-Load Balancer](https://ithelp.ithome.com.tw/m/articles/10296010)

- [Compare GCP Load Balancing with AWS](https://rickhw.github.io/2017/11/30/GCP/Compare-GCP-Load-Balancing-with-AWS/)

- [Google Cloud雲端平台介紹](https://jason-kao-blog.medium.com/google-cloud%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%B4%B9-fc3212c8359b)