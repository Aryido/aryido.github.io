---
title: GCP - Compute Service 總整理

author: Aryido

date: 2024-07-20T23:41:17+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> 前面介紹了 Google Cloud 中有常見的幾個 Compute Service 運算服務，分別是：
>
> - [Compute Engine](https://aryido.github.io/posts/google-cloud/compute-engine/)
> - [Cloud Functions](https://aryido.github.io/posts/google-cloud/cloud-functions/)
> - [Cloud Run](https://aryido.github.io/posts/google-cloud/cloud-run/)
> - [Google Kubernetes Engine](https://aryido.github.io/posts/google-cloud/gke/)
>
> 另外補充一個一個現在可能比較少再繼續更迭的 App Engine。以上這五種運算服務應該在什麼情況下選擇呢？ 選擇正確的基礎架構服務來運行 APP 是很重要的，故以下對其做一些廣義的整理和筆記：
> {{< image classes="fancybox fig-100" src="/images/google-cloud/compute-service-summary/compute-services.jpg" >}}

<!--more-->

---

# 服務簡介

{{< image classes="fancybox fig-100" src="/images/google-cloud/compute-service-summary/compute-services1.jpg" >}}

- ### Compute Engine => 抽象級別 : IaaS

  也稱 Virtual machines 虛擬機，需要自行配置 CPU、 memory、disk 或者 GPUs 等等，並決定要運行的作業系統和安裝其他軟體，還可以做其他設定來支持 Autoscale 功能。如果需求可能要對底層**硬體**基礎架構進行更多控制，那就建議使用 Compute Engine。

- ### Google Kubernetes Engine => 抽象級別 : IaaS 或 PaaS

  Kubernetes 本來就是一個 Open-Source Container Orchestration，用於自動化容器化應用程式的部署、擴展和管理 ; 而雲端託管的 Kubernetes Clusters 更無縫整合 GCP 其他服務。如果本身就對 K8S 有一定熟悉度且有使用很多 GCP 其他雲端服務想整合，那可以嘗試使用 GKE。

- ### Cloud Run => 抽象級別 : FaaS 或 PaaS

  一個完全託管的**容器化**無伺服器平臺，可運行單個容器，面對大流量也可自動擴展以回應 Web Request 或其他事件，由於只要能打包成容器即可運作，支援的語言情境更加廣泛。 關鍵字是**容器化**，如果不需要像 Kubernetes 那樣複雜的操作，且需求是比較起 Cloud Functions 更加複雜一點的容器化微服務，可考慮使用 Cloud Run。

- ### Cloud Functions => 抽象級別 : 標準 FaaS

  事件驅動 Event-driven serverless functions，上傳程式碼即可運作，概念上是強烈的使用才付費的，所以 Cloud Functions 不會一直運行著，當事件發生時 Cloud Functions 才會調用函數方法，請求結束之後，就 shutdown。 為完全微服務化設計，如果有強烈 Event-driven 的 輕量 ETL pipeline 流程設計，可以使用 Cloud Functions 。

{{< alert info >}}
承前知道在 Google Cloud 中，有兩種運算服務同為 FaaS 架構也同為 Serverless 服務，若更加細分的話，一個是

- Cloud Run 定義為 **Serverless platform**
- Cloud Functions 定義為 **Serveless logic**

{{< /alert >}}

- ### App Engine => 抽象級別 : PaaS 運算服務

  一個完全託管的無伺服器平臺，用於完整的 Web 應用程式，會負責處理網路、應用擴展和資料庫擴展，更新版本等操作。

---

# 其他整理比較

{{< image classes="fancybox fig-100" src="/images/google-cloud/compute-service-summary/compute-choose.jpg" >}}

如果只是一個小型開發團隊，並且希望專注在 code 上，那麼 Cloud Run 或 App Engine 等無伺服器選項是一個不錯的選擇 ; 如果擁有更大的團隊，有自己的工具和流程，可以使用 Compute Engine 或 GKE 定制自己的 infra。

- ### 計費模式

  - Compute Engine 和 GKE 計費模式偏向基於資源，需要為預置的實例付費，與使用方式無關
  - Cloud Run、App Engine 和 Cloud Functions 偏向按請求計費

- ### [「VM Image」 V.S 「Cloud Run」 V.S 「GKE」](https://www.youtube.com/watch?v=jh0fPT-AWwM)

  在 GCP 中想使用**運行容器**的解決方案，其中一種方法是使用 GKE。 GKE Cluster 是整合 GCP 雲端 IAM 安全及監控服務、可設定高可用，且對於節點 Node 可雲端自動擴充、自動修復功能 ; 借助 GKE for Anthos，還可以混合多雲和地端平台，自由移動 workload。

  如果不想像 GKE 那樣寫許多 YAML 檔並管理 Infra，只想專注於建立無狀態容器化應用，可以考慮 Cloud Run，同樣也可以借助 Cloud Run for Anthos ，混合自己的私人託管環境並部署容器。

  最後是使用 GCE 虛擬機器，就類似在本地電腦內啟動 docker，並在其上面運行各式容器。對於此情景有建議的虛擬機器作業系統 <**容器最佳化作業系統**> 稱 Container-Optimized OS，它針對運行容器進行了最佳化，並由 Google 維護。

- ### 「Cloud Run」V.S「App Engine 和 Cloud Functions」

  - 依靠 App Engine 上多年的經驗，現在 Cloud Run 是 GCP Serverless 最新的推薦工具，Cloud Run 改進了 App Engine 體驗，[主要是與 Google Cloud 其他雲端服務的集成](https://cloud.google.com/appengine/migration-center/run/compare-gae-with-run)。

  - Cloud Functions 代表了 Google Cloud 的**事件驅動**型無伺服器計算服務，關鍵是「**輕量級事件驅動方法**」; Cloud Run 偏向做為一個完全託管的計算平臺，提供**容器化 Web 應用程式**的自動擴展，關鍵字是「**容器化靈活性**」

  {{< image classes="fancybox fig-100" src="/images/google-cloud/compute-service-summary/functions-run.jpg" >}}

- ### 可移植性和開源需求

  基於「可移植性和開源支援」，可考慮 GKE 和 Cloud Run 因為都基於開源框架：

  - GKE 集群由 Kubernetes 開源集群管理系統提供支援
  - Cloud Run 遵照 Knative 開源專案提供支援

---

### 參考資料

- [Where should I run my stuff? Choosing a Google Cloud compute option](https://cloud.google.com/blog/topics/developers-practitioners/where-should-i-run-my-stuff-choosing-google-cloud-compute-option)

- [GCP 零基礎入門 (11) - 運算服務總結](https://ithelp.ithome.com.tw/m/articles/10325523)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Cloud Functions vs. Cloud Run: when to use one over the other](https://cloud.google.com/blog/products/serverless/cloud-run-vs-cloud-functions-for-serverless)
