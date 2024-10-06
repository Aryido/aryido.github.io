---
title: GCP - Cloud Load Balancing 架構概述

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
> Cloud Load Balancing 是 GCP 透過平均分發流量到多個 server ，以防止單一伺服器的過載從而減少系統故障的風險的產品，對應其他的雲端服務是 :
> - Amazon Web Services (AWS) : **Elastic Load Balancing**
> - Microsoft Azure : **Azure Load Balancer**
> 
> 因為只需透過配置**單個**對外 IP 地址和憑證，就可讓負載平衡器的內部的管理的所有 VM 被外網存取，故從購買 External IP 的角度來說 Load Balancing 也可算是一種降低維運成本的方式。 
> 
> 目前從 GCP console 上由**流量類型**劃分成了「 HTTP(S) 」和 「 TCP/UDP 」兩大類 Load Balancing ，然後在依照細部應用場景還有分成 「Global/Regional」 和 「Internal/External」 等等各種組合，總體設定蠻精細的，可以對應不同的場合需求。
>
> {{< image classes="fancybox fig-100" src="/images/google-cloud/lb/lb-types.jpg" >}}
<!--more-->

---

# GCP Load-Balancer 架構
雲端服務最大的優勢是有「 按需收費 」與 「 按需擴展 」的特性，其中 Load-Balancer 大概就是最展現 Autoscaling 屬性的代表服務吧。 GCP Load-Balancer 的 Component 主要分為:

- **Frontend**
- **Backend Service**
- **Backends**

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/lb-architecture.jpg" >}}

這邊特別注意，在 GCP load-balancer 架構中，提到  Frontend、Backend 這些關鍵字，**這些跟我們常聽到「網頁前端」、「網頁後端」是不一樣的東西 !** 這裡是代表 Load-Balancer 的架構組成組件。

---

#  Frontend
Load-Balancer 訪問的流量，都會先傳入 Frontend Component，再依據**連線方式**與**轉發規則**被送往不同的 backend service 。主要有四個組件如下圖:

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/frontend.jpg" >}}

- ### Forwarding rule：
  
  Global Load-Balancer with **single anycast IP**，透過設置 IP 讓傳入流量進入負載平衡器，並運用對應的 Protocol 與 Port 將流量轉至 Proxy。

- ### Target proxy：
  
  設定 HTTP request / response 常見的 header，作為 Client 與 Server 間的橋梁，可設置 SSL 憑證安全連線。
  {{< alert info >}}
  還可以設定一個參數叫`X-Cloud-Trace-Context`，可以透過監控紀錄追蹤 HTTP Request，這在 Microservice 架構找問題是很重要的追蹤參考。
  {{< /alert >}}

- ### SSL certificate (Optional)：
  
  設定加密協議，這個憑證可以是 Google 幫我們管理，或由我們自行管理。

- ### URL map：
  
  定義一些規則來將不同需求或類型的流量導到不同的 backned service，例如:
    - By Path - `如不同的網址路徑` 如 `aryido.com/a` 或 `aryido.com/b`
    - By Host - `不同的站台` 如 `asite.aryido.com` 與 `bsite.aryido.com`
    - By HTTP - `headers (Authorization header)` 或 `methods (POST、GET...)`

  {{< alert info >}}
  有點像是 AWS ALB 的 Target Group
  {{< /alert >}}


# Backend Service
Backend Service 主要是定義 Load-Balancer 如何分配流量到設置的 Backends，**可以理解成 Backend Service 管理 Backends**，會有 Health-Check 組件和連接一到多個 backends 。

### Health-Check
Backend Service 會藉由 Health-Check 設定的頻率，向指定的 Port 探測並取得回應，確保資源健康之後， Backend service 才將流量導向 Backends。

這邊提醒，要對後端資源進行健康檢查就必須設置 Firewall-Rule 允許來自 Health-Check request 流量的 source ip : 

- `130.211.0.0/22`
- `35.191.0.0/16`

{{< alert danger >}}
如果沒有允許均會導致 Health-Check 檢查失敗，但不是機器真的壞掉，而是沒有開防火牆導致收不到回應。這邊初次使用 GCP Load-Balancer 時超級容易忽略的。
{{< /alert >}}

另外可以在 Backend Service 這裡設定一些功能如 :
- [Cloud CDN 提供 **CDN 服務**](https://aryido.github.io/posts/google-cloud/cloud-cdn/)

- 指定「會話親和性(Session affinity)」，會將來自同一個 Client 端的請求，都送到到相同 backend，可確保使用者 session 在一段時間內保持一致，從而提高應用程式性能或用戶體驗

- 服務超時設定，定義負載均衡器在將後端實例視為失敗之前，其需等待回應的最長時間

- 流量分佈演算法，讓負載均衡器用於在後端實例或服務之間分配傳入請求的方式

{{< alert warning >}}
這邊注意一下，Load-balancer 的 Backend Service 或者是 Instance Group 本身，都可以設定 health check，但檢查完成後的行為不太一樣：
- Backend Service : 確保資源健康之後， Load-balancer  才會將流量導向後面的 App
- Instance Group :  檢查若資源不健康，會把不健康的資源砍掉，並重新建立新的資源

Load-balancer 的健康檢查，只是用來確定流量分配的實例是否健康，並**不能**自動重新創建實例，使之復原成健康的樣子
{{< /alert >}}


### Backends
為從 Load Balance Frontend 接收流量的 endpoint group ，可以分成以下幾個 :

{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/backend.jpg" >}}

- ##### [Instance group](https://aryido.github.io/posts/google-cloud/managed-instance-groups/)
  多個 vm 放在一個群組以集中管理，可以有 Auto Scaling 功能，分為 : 
    - Google 代管的 Managed Instance Group(MIG)
    - 我們自行管理的 Unmanaged Instance Group(UMG)

  {{< alert info >}}
  相當於 AWS Auto Scaling Group
  {{< /alert >}}

- ##### [Cloud Storage](http://localhost:1313/posts/google-cloud/cloud-storage/)
  
  針對 html、css、js、圖片和影片等靜態內容，直接存在 Cloud Storage 可有效節省資源，常和 CDN 搭配使用。


- ##### Network Endpoint Group (NEG)
  
  對於 cloud run 、 k8s Pods 等等虛擬化容器類型的群組，都是屬於 NEG。

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