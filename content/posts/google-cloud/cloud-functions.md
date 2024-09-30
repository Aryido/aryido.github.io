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
  - gcp-compute-service

comment: false

reward: false
---

<!--BODY-->

> Cloud Functions 是一個**無伺服器的雲端執行環境** (serverless execution environment)，會把寫出來的 code 完全託管給 GCP 並且無需配置任何 Infra 也不用管理任何 Servers 就可以執行了，對於程式設計師來說只需要專注在自己的程式邏輯即可，基本完全省去管理硬體的煩惱，為最標準的 **FaaS** 類型服務。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **AWS Lamda**
> - Microsoft Azure : **Azure Functions**
>
> Cloud Functions 可以用 Java、Python、Node.js、Go 等常用 coding language 來撰寫，適用於部署單一化用途的程式，或用於連接擴展其他的 GCP 雲端服務，以 Events and triggers 為其核心設計思想。

<!--more-->

---

使用了 Cloud Functions 之後，因為有不需要另外建置相關的 Server 的便利性，加上又能應對突然的流量高峰的擴展性，對於其 Use-Cases 部分，在官方文件也給出了蠻好的整理，例如適用於：

- 輕量級 ETL、Data Processing
- 觸發應用程式 Build

{{< image classes="fancybox fig-100" src="/images/google-cloud/functions/functions-glue.jpg" >}}

其具備 「**fine-grained**」 和「**依用量自動配置資源 (On-demand)**」
的特點，因此很常用做輕量級 API 和 Webhook 等非同步工作。也可將 Cloud Functions 綁定到指定的 Event ，再加上其可以直接訪問 Google Service Account credential 的特性，因此可以和大多數 GCP 服務無縫連結，如官網提供的圖所述，可 glue 各種服務。

{{< alert success >}}
即 Cloud Functions 常用來當作雲端服務之間的溝通橋樑。
{{< /alert >}}

# Cloud Functions Version

Cloud Function 為在雲端環境執行程式的簡單方法之一，是 Serverless Micro Service 且強調**事件驅動**。目前有提供兩個版本：

- **Cloud Functions (1st gen)**，簡稱 V1
- **Cloud Functions (2nd gen)**，簡稱 V2 (推薦使用)

Cloud Functions (2nd gen) 是基於 「 Cloud Run 」 和 「Eventarc」 構建的，**會和 Cloud Run 共享一些 resource quotas** ，這個要注意一下。
{{< image classes="fancybox fig-100" src="/images/google-cloud/functions/functions-v1v2.jpg" >}}

Cloud Functions (2nd gen) 有增強不少功能 :

> - Request Timeout 對於 HTTP-Triggered 增加到 **60 mins**
>
> - 因為需要支援來自 Cloud Storage 或 BigQuery 的 large streams ，也需要能夠應對 compute-intensive 的情境， 加上 parallel workloads 平行處理需求等等，故對於 **Memory 和 CPU 都提供了更強且更好的定制性**
>
> - Eventarc 的**支援數量增加**非常多
>
> - 流量分割和 Concurrency 都比 1st gen 增強很多

1st Gen 是典型的 Serverless Functions 架構，故每個 function instance 在需要時才會被建立，因此容易出現「 **Cold Start** 」，最簡單的解決方法是設定「最小 instance 數量」，但相對這會增加成本 ; 而 2nd Gen 對於每個 function 有更高的並發，故提供更高的吞吐量和延遲。

{{< alert info >}}
Cloud Functions 「 Cold Start 」冷啟動，是系統需要時間啟動新的容器執行個體以服務新請求時，所會遇到的延遲。
{{< /alert >}}

# 設計 Cloud Function

Cloud Function 是一個事件驅動（Event-driven）的計算平台，以下列出 Cloud Function 需要注意的事項及限制，以及提供官方的 best practices 設計思路，主要會是先圍繞在避免不必要的 cold starts 發生：

- ##### Cloud Function 有 Time Out 的限制

超過時間限制後 Cloud Function 會拋出錯誤狀態。如果發生 Time Out ，是會被收取整個超時時間的費用，且超時還可能導致不可預知的行為或後續調用的冷啟動，導致額外的延遲。

{{< alert warning >}}
雖然 V2 版本 time out 時間對於 http-trigger 有增長時間，但還是簡單記憶一下，若處理會耗費近 10 分鐘左右的話，這會是一個需要注意的時間。
{{< /alert >}}

- ##### 確保 HTTP 函數發送 HTTP 回應

**如果 Cloud Function 是 HTTP 觸發的，一定記得要發送 HTTP 回應**。如果沒有這樣做，可能會導致 Time Out 發生 。而每一種 coding language 判定為「有發送 HTTP 回應」的形式會有點不太一樣，可以參考 google 給出的[範例](https://cloud.google.com/functions/docs/bestpractices/tips#ensure_http_functions_send_an_http_response)。

- ##### Always delete temporary files

因為 Cloud Function 的臨時目錄儲存是 in-memory filesystem，故寫 tem-file 是會直接消耗記憶體的。如果都沒顯式刪除 tem-file 可能會導致記憶體不足錯誤而發生 cold starts。

- ##### 以 Idempotent Functions 方式設計

意思是即使 Cloud Function 被調用多次，也是產生相同的結果，在最佳實踐中會建議這樣設計，原因是 Function 有 retries 重試的功能。當要處理上一次失敗而重新調用的 Function，在**冪等**的設計下 retries 比較不會發生奇怪的問題。

{{< alert warning >}}
多個 Cloud Function 間也並**不共享** Memory、Global Variable。
如果真的有資料共享的需求，可以考慮使用 GCP 上的 storage bucket。但其實並不推薦有讓多個 Cloud Function 共享資料的情形，最好以無狀態的方式來使用各個 cloud function。
{{< /alert >}}

### 管理 Cloud Function

由於一個 GCP Project 內可能散落好幾個不同功能的 Cloud Function，故可以利用以下方法分類 function：

- 善用自訂的命名原則
- **多利用 tag (推薦)**

在多數情況下我們使用 Server 的場景其實都是「非密集存取」且「不會耗用大量網路資源」的情境，這時 Cloud Function 有機會比 VM 便宜很多。[GCP calculator](https://cloud.google.com/products/calculator?hl=en) 簡單估算，一個月存取約 50000 次，平均時間 3 秒，只需要 1.54 美金，如果好好運用可以非常便宜。

雖然大部分常用語言 Cloud Function Runtime 都有支援，但還是會面臨一些語言是**沒有支援**的，這時可能就是 Cloud Run 可以考慮使用的場景了。

---

### 參考資料

- [Cloud Functions Doc](https://cloud.google.com/functions/docs/concepts/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [淺談 Serverless Solution — 以 GCP Cloud Function 為例](https://medium.com/%E5%AE%85%E7%94%B7%E9%9B%9C%E5%AD%B8%E7%AD%86%E8%A8%98/%E6%B7%BA%E8%AB%87serverless-solution-%E4%BB%A5gcp-cloud-function%E7%82%BA%E4%BE%8B-6374bf74df98)

- [使用 Cloud Functions 和 Cloud Scheduler 定期清理未使用的 IP 資源](https://vocus.cc/article/65757016fd897800011f2853)

- [GCP – Cloud Functions – Develop it the right way](https://medium.com/google-cloud/gcp-cloud-functions-develop-it-the-right-way-82e633b07756)
