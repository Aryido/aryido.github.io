---
title: GCP - Cloud Functions 概述

author: Aryido

date: 2024-07-12T23:26:00+08:00

thumbnailImage: "/images/google-cloud/functions/functions-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - serverless

comment: false

reward: false
---

<!--BODY-->

> Cloud Functions 是一個**無伺服器的雲端執行環境** (serverless execution environment)，寫出來的 code 完全託管（fully managed）給 GCP 並且無需配置任何 Infra 也不用管理任何 Servers ，只需要專注在自己的程式碼邏輯即可，為標準的 **FaaS** 類型服務。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **AWS Lamda**
> - Microsoft Azure : **Azure Functions**
>
> Cloud Functions 可以用 Java、Python、Node.js、Go 等 coding language 來撰寫，常用於部署單一化用途的程式，或用於連接和擴展其他的 GCP 雲端服務，以 Events and triggers 為其使用的設計核心思想。

<!--more-->

---

使用了 Cloud Functions 之後，因為有不需要另外建置相關的 Server 的便利性，加上又能應對突然的流量高峰有不錯的擴展性，對於 Use-Cases 在官方文件也給出了蠻好的整理，例如適用於：

- 輕量級 ETL、Data Processing
- 觸發應用程式 Build

其具備 「**fine-grained**」 和「**依用量自動配置資源 (On-demand)**」
的特點，因此很常用做輕量級 API 和 Webhook 的一些非同步工作。也因為只需將 Cloud Functions 綁定在指定的 Event ，再加上 Cloud Functions 可以直接訪問 Google Service Account credential ，因此可以和大多數 GCP 服務無縫連結，非常適合做為各種服務之間的溝通橋樑。

{{< alert success >}}
Cloud Functions 常用來當作雲端服務之間的「粘合劑」。
{{< /alert >}}

# Cloud Functions Version

Cloud Function 為在雲端環境執行程式的簡單方法之一，是 Serverless Micro Service 且強調**事件驅動**。目前有提供兩個版本：

- **Cloud Functions (1st gen)**，簡稱 V1
- **Cloud Functions (2nd gen)**，簡稱 V2 (推薦使用)

Cloud Functions (2nd gen) 是基於 「 Cloud Run 」 和 「Eventarc」 構建的，**會和 Cloud Run 共享一些 resource quotas** ，這個要注意一下。
{{< image classes="fancybox fig-100" src="/images/google-cloud/functions/functions-v1v2.jpg" >}}

Cloud Functions (2nd gen) 有增強不少功能 :

- Request Timeout 對於 HTTP-Triggered 增加到 **60 mins**
- 因為需要支援來自 Cloud Storage 或 BigQuery 的 large streams ，也能應對 compute-intensive 的情境， 加上 parallel workloads 平行處理等等，故對於 **Memory 和 CPU 都提供了更強且更好的定制性**
- Eventarc 的**支援數量增加**非常多

- 流量分割和 Concurrency 都比 1st gen 增強很多

1st Gen 是典型的 Serverless Functions 架構，故每個 function instance 在需要時才會被建立，因此容易出現「 **Cold Start** 」，解決方法是設定「最小 instance 數量」，但相對這會增加成本。
2nd Gen 對於每個 function 有更高的並發，故提供更高的吞吐量和延遲。

{{< alert info >}}
Cloud Functions 「 Cold Start」冷啟動，是系統需要時間啟動新的容器執行個體以服務新請求時所會遇到的延遲。通常還會發生在：

- 從零到一的執行個體縮放事件
- 配置服務單一**並行**請求
- 流量縮放期間

{{< /alert >}}

# 設計 Cloud Function

Cloud Function 是一個事件驅動（Event-driven）的計算平台，由於是以 Function 為單位，設計會面臨到一些系統設計的選擇和取捨，例如:

- Function 應該負責的事情是甚麼
- 透過甚麼事件去驅動 Function

以下列出 Cloud Function 有些需要注意的事項及限制：

- 每個 Cloud Function 有 「 **time out 限制** 」，超過時間限制後會拋出錯誤狀態
  {{< alert warning >}}
  雖然 V2 版本 time out 時間對於 http-trigger 有增長時間，但還是簡單記憶一下，若處理會耗費近 10 分鐘左右的話，這會是一個需要注意的時間。
  {{< /alert >}}

- 雖然大部分常用語言都有，但 Cloud Function Runtime 還是會面臨一些語言是**沒有支援**的

- 多個 Cloud Function 間並**不共享** Memory、Global Variable、File System
  {{< alert warning >}}
  如果真的有資料共享的需求，可以考慮使用 GCP 上的 storage bucket。但其實並不推薦讓多個 Cloud Function 共享資料的情形，最好以無狀態的方式來使用各個 cloud function。
  {{< /alert >}}

- Cloud Function 要想好怎麼管理，尤其 project 內可能散落好幾個不同功能的 Cloud Function。可以利用以下方法好好非類 function：

  - 善用自訂的命名原則
  - **多利用 tag (推薦)**

- 在多數情況下其實都是「非密集存取」且「不會耗用大量網路資源」的情境，這時 Cloud Function 有機會比 VM 便宜很多。
  {{< alert info >}}
  [GCP calculator](https://cloud.google.com/products/calculator/?utm_source=google&utm_medium=cpc&utm_campaign=japac-SG-all-en-dr-SKWS-all-all-trial-DSA-dr-1605216&utm_content=text-ad-none-none-DEV_c-CRE_655856180813-ADGP_Hybrid%20%7C%20SKWS%20-%20BRO%20%7C%20DSA%20-All%20Webpages-KWID_39700076131768134-dsa-1456167871416&userloc_9198554-network_g&utm_term=KW_&gad_source=1&gclid=EAIaIQobChMIjNTywra4hwMV-V0PAh0rcQ5iEAAYASAAEgKKtvD_BwE&gclsrc=aw.ds&hl=en&dl=CiRkODQ4ZDY4Ny0wN2NkLTRiNDAtYjIyOC0yYjdhMmFlZmM2YzMQExokNkI4MzE5NTUtOTkxRC00NEFFLTk4QTItRTFBQzBCRTE0RTY0) 簡單估算，一個月存取約 50000 次，平均時間 3 秒，只需要 1.54 美金，如果好好運用可以非常便宜。
  {{< /alert >}}

---

### 參考資料

- [Cloud Functions Doc](https://cloud.google.com/functions/docs/console-quickstart)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [淺談 Serverless Solution — 以 GCP Cloud Function 為例](https://medium.com/%E5%AE%85%E7%94%B7%E9%9B%9C%E5%AD%B8%E7%AD%86%E8%A8%98/%E6%B7%BA%E8%AB%87serverless-solution-%E4%BB%A5gcp-cloud-function%E7%82%BA%E4%BE%8B-6374bf74df98)

- [使用 Cloud Functions 和 Cloud Scheduler 定期清理未使用的 IP 資源](https://vocus.cc/article/65757016fd897800011f2853)

- [GCP – Cloud Functions – Develop it the right way](https://medium.com/google-cloud/gcp-cloud-functions-develop-it-the-right-way-82e633b07756)
