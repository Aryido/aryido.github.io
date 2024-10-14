---
title: GCP - Pub/Sub 概述 I - 架構

author: Aryido

date: 2024-08-17T23:18:06+08:00

thumbnailImage: "/images/google-cloud/pubsub/pubsub-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - data-exchange

comment: false

reward: false
---

<!--BODY-->

> Pub/Sub 是 Google 推出的 Message Service ，作為中介層(Middleware) 可讓系統解耦，解耦的兩系統透過 Publish-Subscribe 的模式來 「異步 asynchronous」 收發消息，實現高可靠(highly reliable) 和高可擴展(scalable)的服務。 簡單以 Event-Driven 消息傳遞設計為主體概念的話，對應到其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **SQS + SNS**
> - Microsoft Azure : **Azure Service Bus Messaging**
>
> Pub/Sub 也簡化了許多 Message Service Infra 的管理如 Broker、Exchange、Queue 等這些底層架構組件並不會直接被使用者接觸，而是 GCP 完全代管且提供 [Pub/Sub service level agreement (SLA)](https://cloud.google.com/pubsub/sla?hl=en)，developer 僅需要瞭解 Message、Topic、Publisher、Subscription、Subscriber 這些接近應用程式端的 Components，算是降低門檻達到快速使用的目的。

<!--more-->

---

# Pub/Sub Components

Pub/Sub 需瞭解 Publisher、Topic、Subscription、Subscriber 等幾個關鍵名詞就可以開始設計 Message Service，其運作流程和關係簡單用下圖來進行介紹：

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/pubsub-components.jpg" >}}

例如有兩個不同的 Applications 會個別產生 Message 資料的主體，在這裏 Applications 個別稱為

- `Publisher 1`
- `Publisher 2`

而 Publisher 也可稱為 「Producer 產生者」，會把 Message 它發送至指定 Topic。上圖中 Publishers 個別產生的 `Message A`、`Message B` 是決定送到同一個指定的 Topic，也代表是兩個 Message 是同一種類型且格式都一樣。

再來可能由於想分給**兩個不同系統**去處理同一個 Message，故讓 Topic 附加到

- `Subscription 1`
- `Subscription 2`

這樣架構下不論是訂閱 `Subscription 1` 還是 `Subscription 2` 其中的哪一個，都可以收到 `Message A`、`Message B`，這就是所謂的 **Fan-out** 架構。

最後上圖中發現 `Subscription 1` 有連接到兩個 Subscriber 分別是

- `Subscriber 1`
- `Subscriber 2`

這兩個 Subscriber 會並行一起幫忙處理 Message 增快處理速度，像圖中表示是 `Subscriber 1` 處理 `Message B` 而 `Subscriber 2` 處理 `Message A`，其中一個正常處理完 Message 之後，其他的 Subscriber 就不會處理該消息了。

比較簡單的是 `Subscription 2`，僅連接到單個訂閱者 `Subscriber 3` 來處理資料。

# Pub/Sub Resources Naming Guideline

Pub/Sub 的許多資源如 Topic、Subscription、Schema 等名稱會符合以下格式：

> `projects/{PROJECT-ID}/{COLLECTION}/{ID}`

其中 `COLLECTION` 會是 `topics` 、`subscriptions`、`schemas`、`snapshots` 其中一個，然後最後的 `ID` 也有命名限制如下 :

- 不能以 `goog` 為開頭
- 必須以字母開頭
- 長度限制 `3 ~ 255` 個 characters
- 只能包含：`[A-Za-z][0-9]-_.~+%`

### Topic & Schema

Publisher 會發送 Message 至 Topic 這個目的地，然後加上要創建 Subscription 必須訂閱某個指定 Topic 才行，故要使用 Pub/Sub 的**首先任務是需先創建 Topic** 。另外 Topic 可以設定 Schema ，用於強制 Pub/Sub Message 中資料欄位的格式，有 **Apache Avro** 和 **Protobuf** 可以選擇， **是 Optional 功能**。

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/topic.jpg" >}}

Pub/Sub 有兩種類型的 Topic :

- `standard topic`
- `import topic`

`import topic` 比較特別，它可以讓外部資料源(External DataSource)，把**外部資料 Ingest 到 GCP 的 Topic**，也就是上圖中的 `Enable ingestion` 的功能。例如說現在可以把 AWS Kinesis 的 streaming data 導入 GCP 的 Topic 。

{{< alert success >}}
官網上建議 `import topic` 使用在 streaming data 上。如果是考慮是 **Batch 批量** ingest 資料而不是 streaming data ingestion，則 :

- 到 BigQuery 中可考慮使用 **BigQuery Data Transfer Service (BQ DTS)**
- 到 Cloud Storage 中考慮使用 **Storage Transfer Service (STS)**

{{< /alert >}}

### Subscription & Subscriber

Subscription 會連接指定的 Topic ; 而 Subscriber 也稱為 「Consumer 消費者」會連接 Subscription 並處理 Message。 創建 Subscription 時要選擇 delivery Message 的方式：

> - **Pull subscription**: Subscriber 會去請求 Pub/Sub 來拉取資料
> - **Push subscription**: Pub/Sub server 會推送資料至目標 EndPoint
> - **Export subscription**: 可直接把 Pub/Sub 的資料寫到 BigQuery 或 Cloud Storage

默認 Subscription 對於 Message 處理的設定，是 **at-least-once delivery with no ordering guarantees**，保證至少會傳送一次且沒有排序。若以上並不滿足有其他進階需求，Pub/Sub 也有提供 [exactly-once delivery](https://cloud.google.com/pubsub/docs/exactly-once-delivery) 和 [ message ordering](https://cloud.google.com/pubsub/docs/ordering) 可以設定，屬進階內容暫且略過。

{{< alert warning >}}
目前簡單看起來要讓消息排序，只支持在**同一 region 下**有序
{{< /alert >}}

Pub/Sub Subscription 對於「**過期時間**」有蠻多不同的地方都可以設定，為避免混淆以下稍微解釋一下 :

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/expire-setting.jpg" >}}

- ##### **Expiration period**

  這個設定是針對 subscription 的設定: 若 subscription 沒有和任何 subscriber 產生 Pull/Push 甚至沒有連線達到一段時間，就會 inactive subscription，可以設定的區間目前是 `7 ~ 365 days`。當然也可以永遠 never expire

- ##### **Message retention duration**

  消息中間件有一個大功用就是保存 Message 達到「消減流量高峰」的目的，可以設定的區間目前是 `10 mins ~ 7 days`。 通常 Message 在 Acked 之後都會從 Message storage 中刪除，這邊也可以進一步設定連 acknowledged messages 都保存 retained

- ##### **Acknowledgement deadline**

  是在實作面上最需要注意的設定，可以設定的區間目前是 `10  ~ 600 seconds`。 因為有些 Message 處理時間可能會偏長，若設定 Acknowledgement deadline 太短會造成很多 Message 都是 Unacked 狀態導致重複投遞甚至重複消費

# Pub/Sub Pattern

Pub/Sub 的架構模式蠻多樣化的，也因如此 Message Service 在 GCP 產品上比較單一，比起其他雲端的多種不同 Message Service 服務，個人覺得 GCP 在這方面比較統一方便

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/pubsub-patterns.jpg" >}}

架構上特別注意 **Fan in (many-to-one)** 和 **Fan out (one-to-many)**，這裡指的是 **「Publisher」對上「Subscription」**。

> - 如果需要對 Message 執行不同的 pipe line 操作處理時， **Fan out** 就是必需的架構

> - 而對於 **(many-to-many)** 的形容比較是在 Load balanced 方面，是使用多個 Subscriber 大規模處理對應 Subscription 內收到的消息。

---

# Pub/Sub Comparison

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/real-usecases.jpg" >}}

Pub/Sub 消息傳遞服務在使用場景上非常多元，故會有很多類似的產品可以做比較，無論是對應到 GCP 平台的其他服務還是對比其他雲平台 AWS 、 Azure 的其他服務。

### Pub/Sub 和其他 GCP 類似產品的比較

- ##### Cloud Tasks

  Cloud Tasks 和 Pub/Sub 都可用於 Asynchronous Message-Passing，概念上相似，但主要差別是在 「Implicit invocation」 和 「Explicit invocation」 :

  > - **Pub/Sub** 屬於 Implicit invocation 隱式調用，因為 Pub/Sub 主要目的是把 Publisher 和 Subscriber 解耦，故 Publisher 是不會知道 Subscriber 的任何資訊的

  > - **Cloud Tasks** 屬於 Explicit invocation 顯式調用，這時 Publisher 是直接指定 Endpoint 來傳送消息，故 Cloud Tasks 適用的場景有: 定時任務觸發特 webhook、遠端 procedure 調用

- ##### Firebase

  根據官網所敘述，Pub/Sub 主要是用於 Service-To-Service 的訊息異步傳遞，而不是用於 End-User 或 IoT Clients 的通訊，所以若主要是讓 Mobile 或 Web-App 和 Service 溝通的話，**尤其是 Mobile 可以考慮 Firebase 系列展品**

- ##### Pub/Sub Lite
  {{< alert danger >}}
  目前是 **deprecated 產品**，故不要再使用它了，`2026/3/18` 會被關閉，建議將 Pub/Sub Lite 服務遷移到 Apache Kafka 或 Pub/Sub 。
  {{< /alert >}}

### Pub/Sub 和其他雲端類似產品的比較

讓獨立 App 應用之間進行 Asynchronous Requests 傳遞訊息主的話，對應到 AWS 和 Azure 的雲端服務分別是 :

> - Simple Notification Service (**SNS**)
> - Simple Queue Service (**SQS**)
> - Amazon MQ

> - Azure Service Bus Messaging
> - Azure Storage Queues

Pub/Sub 是一個強大服務，既支持**訂閱推送**模型也可以像**消息隊列**一樣使用。相比之下 AWS 提供兩個不同的消息傳遞服務設計: SNS 和 SQS ，其區別簡述如下：

- ##### SNS (分散式主題/訂閱)

  當生產者把消息發送給 SNS 服務後，會**將消息推送給多個訂閱者**如 : Short Message Service ( SMS 簡訊服務) 、 電子郵件服務 、 Simple Queue Service (SQS) 或 AWS Lambda 。因為一條消息會同時傳遞給多個訂閱者，故適合廣播通知，屬於 **Fan out 設計**

- ##### SQS (分散式消息隊列)

  SQS 是消息隊列，消息由一個生產者放入隊列中，不會主動發送給用戶端，由一個或多個消費者按 FIFO 順序拉取消息進行處理，適合任務隊列，供點對點的消息傳遞

  由於在 AWS SQS 中，並沒有支援 **fan-out** 的功能，想要實作類似的架構需要透過 SNS + SQS 整合才行，所以個人認為 Pub/Sub 比較接近 `SNS + SQS`：

  {{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/aws-fan-out-sqs.jpg" >}}

- ##### Amazon MQ

  另外 AWS 也提供托管的 Message Service 稱為 **Amazon MQ**，支援開源產品如 Apache ActiveMQ 和 RabbitMQ，適合那些使用已經使用標準消息傳遞協議（如 JMS、AMQP、MQTT 等）的應用，可將現有本地 Infra 從地端遷移到雲端來管理

- ##### Azure Service Bus Messaging

  是 Azure 提供的一個企業級 Message Service ，支持先入先出（FIFO）消息順序、消息排隊、發布/訂閱模式、延遲消息以及事務性消息處理

- ##### Azure Storage Queues

  算是 Azure Storage 的一部分功能，設計用於簡單的消息隊列，支持將大量小消息以非同步的方式存儲和傳遞，適合輕量級的應用間通信和後台任務處理。對比起來和 GCP 的 Cloud task 比較接近

以大數據 Streaming Data Ingest 為主的話， Pub/Sub 對應到其他的雲端服務是 :

> - Amazon Web Services (AWS) : **Amazon Kinesis**
> - Microsoft Azure : **Azure Event Hubs**

以上這些都是設計用來處理高吞吐量的 Streaming Data，偏向屬於專門 Data-Engineer 或 Data-Science 領域的雲端工具，下圖就舉例一個 GCP Big-Data Analysis 的一種經典架構 : 

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/example.jpg" >}}

{{< alert success >}}
相似的產品非常多，若真的想了解其中差異可能還會需要更多的實際使用才能比較，以上就先列出來
{{< /alert >}}

---

### 參考資料

- [What is Pub/Sub? ](https://cloud.google.com/pubsub/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [SQS 和 SNS 對比分析](https://juejin.cn/post/7039341353678929933)

- [拆解雲端 Message Service：Google Cloud Pub/Sub vs. AWS SQS 優劣分析](https://ikala.cloud/google-cloud-pub-sub-aws-sqs-comparison/)
