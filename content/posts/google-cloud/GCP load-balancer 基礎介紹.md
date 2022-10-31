---
title: GCP load-balancer 基礎介紹

author: Aryido

date: 2022-10-23T15:10:00+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud

tags:
- aws

comment: false

reward: false
---
<!--BODY-->
> GCP 目前也有多種 Load Balancer，console上是大概分成了三個 :
> - HTTP(S) load balancing
> - TCP load balancing
> - UPD load balancing
>
> 但實際上依照細部功能，其實有蠻細緻的設定的
> {{< image classes="fancybox fig-100" src="/images/google-cloud/load-balancer-options.jpg" >}}
<!--more-->

---

## GCP load-balancer 架構
GCP 負載平衡的設置主要分為
- Frontend
- Backend Service
- Backends

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb.jpg" >}}

## Frontend
訪問 GCP 資源的流量，都會先傳入 Frontend ，再依據**連線方式**與**轉發規則**被送往不同的 backend service 。主要有四個組件:

{{< image classes="fancybox fig-100" src="/images/google-cloud/frontend.jpg" >}}

- Forwarding rule：透過設置 IP 讓傳入流量進入負載平衡器，並運用對應的 Protocol 與 Port 將流量轉至 Proxy

- Target proxy：作為 Client 與 Server 間的橋梁，可設置 SSL 憑證安全連線

- SSL certificate：設定加密協議，這個憑證可以是 Google 幫我們管理，或由我們自行管理

- URL map：依照使用者訪問的 URL ，將請求導向對應的 backend service

---

## Backend service & Backends
GCP 在負載平衡中我們需要了解的重要概念是 Backend service 與 Backends 的不同

### Backend service
Backend service 主要是定義負載平衡器如何分配流量到我們設置的 Backends 資源，會藉由 health check 確保資源健康後，才將流量導向 Backends
{{< alert warning >}}
這邊提醒，要對後端資源進行健康檢查就必須設置Firewall rule ，允許來自 Health check 的流量

- 130.211.0.0/22
- 35.191.0.0/16
{{< /alert >}}

### Backends

- Instance group：多個 vm 放在一個群組以集中管理。可以分為
  - Google 代管的managed instance group
  - 我們自行管理的 unmanaged instance group

- Cloud Storage：針對 html、css、js、圖片和影片等靜態內容，直接存在 Cloud Storage 可有效節省資源

---