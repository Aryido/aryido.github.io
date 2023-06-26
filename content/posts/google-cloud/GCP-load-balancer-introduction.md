---
title: GCP load-balancer 架構簡介

author: Aryido

date: 2023-06-26T23:47:00+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp

comment: false

reward: false
---
<!--BODY-->
> GCP 負載平衡的設置主要分為 Frontend（前端）與 Backends（後端），目前也有多種 Load Balancer，console上是從**流量類型**大概分成了三個 :
> - HTTP(S) load balancing
> - TCP load balancing
> - UPD load balancing
>
> 但實際上依照細部功能，還有分 Global 或 Regional ，總體設定蠻細緻的，可參考下圖 :
> {{< image classes="fancybox fig-100" src="/images/google-cloud/load-balancer-options.jpg" >}}
<!--more-->

---

## GCP load-balancer 架構
雲端服務最大的優勢 Load Balancer 與 Autoscaling 就是雲端彈性展現的代表。 GCP 負載平衡的設置主要分為
- Frontend
- Backend Service
- Backends

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb.jpg" >}}

---

###  Frontend
訪問 GCP 資源的流量，都會先傳入 Frontend ，再依據**連線方式**與**轉發規則**被送往不同的 backend service 。主要有四個組件:

{{< image classes="fancybox fig-100" src="/images/google-cloud/frontend.jpg" >}}

- Forwarding rule：透過設置 IP 讓傳入流量進入負載平衡器，並運用對應的 Protocol 與 Port 將流量轉至 Proxy

- Target proxy：作為 Client 與 Server 間的橋梁，可設置 SSL 憑證安全連線

- SSL certificate：設定加密協議，這個憑證可以是 Google 幫我們管理，或由我們自行管理

- URL map：依照使用者訪問的 URL ，將請求導向對應的 backend service

---

### Backend service & Backends
GCP 在負載平衡中我們需要了解的重要概念是 Backend service 與 Backends 的不同

#### Backend service
Backend service 主要是定義負載平衡器如何分配流量到我們設置的 Backends 資源，會藉由 health check 指定的頻率向指定的 port 探測並取得回應，確保資源健康之後， Backend service 才將流量導向 Backends；如果回應的速度低於 health check 設立的門檻，那麼該 instance 就會被判定為不健康， request 不會導向已被判定為不健康的 instance
{{< alert warning >}}
這邊提醒，要對後端資源進行健康檢查就必須設置Firewall rule ，允許來自 Health check 的流量

- 130.211.0.0/22
- 35.191.0.0/16
{{< /alert >}}

#### Backends
Backends 的內容關聯可以分成以下幾個 :
- Instance group：多個 vm 放在一個群組以集中管理，可以有 Auto Scaling 功能，分為
  - Google 代管的 managed instance group
  - 我們自行管理的 unmanaged instance group

- Cloud Storage：針對 html、css、js、圖片和影片等靜態內容，直接存在 Cloud Storage 可有效節省資源

- Network Endpoint Group（NEG）: 對於 cloud run 、 k8s Pods 等等 container 群組都是屬於此類

---

## External HTTP(S) Load Balancer
這算是我們最常使用的負載平衡器，是給 HTTP / HTTPS 使用的，那如何分流的步驟如下 :
- Forwarding rule 指定一個外部 IP 和 Port，讓 User 知道，並讓流量透過單一 IP 進入負載平衡器，接下來會把 request 轉發到 Target proxy
- Target proxy 會依 URL Map ，將流量導向對應的 Backend service 。若使用 HTTPS 負載平衡器，則 SSL 憑證也是由 Target proxy 關聯的
-  Backend service 是會對後端資源進行 health check ，確保流量送至健康的Backends，

---

### 參考資料
- [什麼是負載平衡？原理、6大 GCP Load Balancer 完整介紹](https://blog.cloud-ace.tw/networking-website/load-balance/gcp-load-balancer-introduction/)