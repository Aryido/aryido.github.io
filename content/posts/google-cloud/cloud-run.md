---
title: GCP - Cloud Run 概述

author: Aryido

date: 2024-07-13T23:14:20+08:00

thumbnailImage: "/images/google-cloud/run/cloud-run-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - serverless
  - cloud-run
  - gcp-compute-service

comment: false

reward: false
---

<!--BODY-->

> Cloud Run 是一套基於 Knative 的全代管無伺服器(serverless)容器平台，也屬於 Google Cloud 中的 FaaS 服務，功能是可在 GCP 託管的環境中運行 **Container** 且已經具有**高擴展性**基礎架構。若從「無基礎建設的容器化平台（Containers without infrastructure）」的角度來說，對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **AWS App Runner**、**Fargate**
> - Microsoft Azure : **Azure Container Apps**、**Azure Container Instances**
>
> Cloud Run 一個蠻大的好處是 : 如果已經把程式打包成 Container Image 鏡象檔，那就可以使用**任何程式語言**來部署，但其實 **Container Image 化是可選的**，如果使用的是 Go、Node.js、Python、Java 等等常用的語言，也可以直接使用 Source Code 的方式來部署，讓我們可以使用 FaaS 的概念去執行如 Web Server 比較大型一點的程式。

<!--more-->

---

Cloud Run 讓使用者僅需透過簡單的指令或 Console 介面即可直接在 Google Cloud 上開發及快速部署具備高擴充性的「 容器化應用程式 」且管理其服務，並無需管理任何基礎架構。使用分成兩種類型：

- ##### Cloud Run Service: 用於運行回應 Web-Request 或 Events
- ##### Cloud Run Job: 用於完成任務後就可退出關閉的 Task

因為支援任何標準的 Container images 格式，故很經常會搭配 Cloud Build、Artifact Registry、Docker，此外也可直接與 Cloud Monitoring、Cloud Logging 整合。

---

# Cloud Run Services

Cloud Run Services 有提供一個可靠的 HTTPS 端點，我們只需確保應用會監聽 Port 並處理 HTTP 請求，故非常適合處理 Web-Request 或 Event 。
{{< image classes="fancybox fig-100" src="/images/google-cloud/run/service.jpg" >}}
故常用的情景如下：

- Web applications: Web 應用並且有簡單訪問 SQL ，並呈現動態 HTML 頁面的功能

- REST API、GraphQL API 或透過 HTTP(s) 或 gRPC 進行通信的 Private Microservices

- Cloud Run 服務可以訂閱 Pub/Sub ，接收從 Pub/Sub 「推送」的消息，也可以從比較廣義的 Eventarc 來接收事件。
  {{< alert success >}}
  Pub/Sub 把 Message 直接「 Push 」到 Cloud Run 應用程序指定的 URL（Cloud Run 的端點），無需應用程序手動拉取，這是 Google 推薦的 Best-Practice 。 原因是 Cloud Run 可以偏向如 Cloud function 那種「按需才啟動使用」的用法，使用 Pull 比較浪費資源
  {{< /alert >}}

重點特色列出如下 ：

- ### Unique HTTPS endpoint for every service

  每個 Cloud Run 服務都提供唯一的 `*.run.app` domain HTTPS 端點，並管理 TLS，也支援 WebSockets、gRPC。

- ### Request-based auto scaling

  由於 Request 的增加導致 CPU utilization 快速地上升時，Cloud Run 會 scale out ; 相反地如果 Request 減少，Cloud Run 也會幫忙移除閒置容器，如果要避免所有 active instance 被刪除，需要在 Cloud Run 配置 `minimum number of instances`。
  {{< alert info >}}
  一直沒有對服務請求時，會把最後一個剩餘的實例也刪除，這種情形稱為 scale to zero
  {{< /alert >}}
  當完全沒有 active instance 時，在收到請求后會按需創建新的 instance，這會由於 **Cold Start** 而對請求的回應時間產生負面影響 ; 另一方面由於 Cloud Run 支援 Concurrency ，故如果擔心成本或後面的系統過載，也可以設定 `Maximum concurrent requests per instance`來限制最多的 instances 數量。

  {{< alert warning >}}
  承上敘述，不難發現 Cloud Run 會自己決定何時停止向實例，這個不是使用者能確實掌握的。故若有保存檔案的需求，也有提供 Volumes 功能，如：Cloud Storage、network filesystem (NFS) 等等持久化存儲
  {{< /alert >}}

- ### 按使用量付費定價

  Cloud Run 計費單位為四捨五入 `100` 毫秒，計費時間會用 Cloud Monitoring 指標來公開，可參考 [container/billable_instance_time 指標](https://cloud.google.com/monitoring/api/metrics_gcp#gcp-run)。
  {{< image classes="fancybox fig-100" src="/images/google-cloud/run/billable-time.jpg" >}}
  有兩種定價模型，會對分配給實例的 CPU 和記憶體收費：

- ##### 基於請求的 CPU allocated only during request processing
    - 每次 request 會有費用
    - 正常處理 SIGTERM 訊號的關閉會有費用
    - 如果「流量零星、會有突發尖峰」時，就可以考慮使用這種模式。
- ##### 基於實例的 CPU always allocated
    - 需要為實例的整個生命週期內付費
    - 當傳入「流量穩定且變化緩慢」時，建議使用這種模式

{{< alert success >}}
由於使用情境的多樣化，可以根據不同的情形選擇計費模式。
{{< /alert >}}

- ### Built-in traffic management （內置預設的流量管理）

  每個 deployment 部署都會創建一個新的**不可變修訂版本**稱為 immutable revision ，也因為有這種設定，故 Cloud Run 可以[針對部屬有各種不同策略](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration)例如：

  - Rollbacks
  - gradual rollouts
  - traffic migration

  無論是因為部署的新版本產生問題需進行程式修復而把所有流量導回舊版本，以確保服務不會中斷 ; 還是測試實際運作的效能或表現，而嘗試用金絲雀部署，這些都可從 Cloud Run 內的 revision 來簡單的實現流量分配的功能。

---

# Cloud Run Jobs

用來執行一個 script 自動化 Shell 腳本，只要作業完成就停止服務就是一個很好的範例 ; Schedule 排程作業這種情景也非常適合使用 ; 另外也支援平行處理，因為可以並行啟動許多相同的獨立實例如 array job ，以上情形都可以用到 Cloud Run Jobs

{{< alert warning >}}
預設情況下，每個任務預設運行 10 分鐘，可以變更為更短或更長，而最長限制是 24 小時
{{< /alert >}}

---

# Practice

> 有一個 user-management service 它有 add、update、delete 和 list 等 API 操作，每個操作都由一個 Docker 容器微服務實現。 Processing Load 處理負載可以從低到非常高不等，希望將 user-management service 部署在 Google Cloud 上，以實現可擴展性並最小化管理，應該怎麼做？
>
> - A. Deploy your Docker containers into Cloud Run. **(O)**
> - B. Start each Docker container as a managed instance group.
> - C. Deploy your Docker containers into Google Kubernetes Engine.

A. Cloud Run 是一個專為運行容器化的應用而設計的 serverless 平台，會根據負載自動擴展，動態調整容器實例數量，並且無需手動管理基礎設施，適合處理處理波動大的應用場景。按需計費，節省成本，適合微服務架構，符合本題 scalability 和 minimal administration 的要求。

B.C. 都是偏複雜的解決方式，像若要使用 managed instance group ，還會需要管理設置 Load-Balancer ; 而 Google Kubernetes Engine 提供更強大的功能如節點管理、更新、擴展策略，對於最小管理需求的應用，這些都不是最佳選擇。

---

### 參考資料

- [Cloud Run Doc](https://cloud.google.com/run/docs/overview/what-is-cloud-run)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Cloud Run 是什麼？6 大特色介紹與實作教學](https://blog.cloud-ace.tw/application-modernization/serverless/cloud-run-overview-and-tutorial/)

- [如何設定 Cloud Scheduler 定期去觸發 Cloud Run: Setting up Cloud Scheduler to Trigger Cloud-Run](https://andy51002000.blogspot.com/2020/03/cloud-schedulercloud-run-setting-up.html)
