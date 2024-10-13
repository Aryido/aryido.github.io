---
title: GCP - Pub/Sub 概述 I - 架構

author: Aryido

date: 2024-08-17T23:18:06+08:00

thumbnailImage: "/images/google-cloud/pubsub/pubsub-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> 

<!--more-->

---

https://cloud.google.com/pubsub/docs/subscriber



# Message Lifecycle

接下來專注在描述一下 Message 的 consume 流程：

{{< image classes="fancybox fig-100" src="/images/google-cloud/pubsub/message-lifecycle.jpg" >}}

當 Publisher 向指定 Topic 發送消息後，首先會先把 Pub/sub 會把該 Message 儲存起來，然後再來是會把該 Message **傳送給所有有訂閱該 Topic 的 Subscriptions** ，再來 **Subscriptions 又會把 Message 送給所有有訂閱該 Subscription 的 Subscriber**。

### Message Status Type

當 Message 送到 Subscriber 來進行處理時，對於判定 Message 到底「**有沒有處理**」與「**處理成功與否**」是非常很重要的，故來說明此幾種狀態：

##### Acknowledged (Acked) Message - 已確認消息

當 Subscriber 處理完 Message 之後，會對其 Subscription 回覆 Acked 表明已成功處理 Message。 當**所有 Subscription** 都有確認收到 Acked 時就會從存儲中刪除該 Message ，而個別 Subscription 判定是 Ack 的方式是： **至少一個 Subscriber 有回覆 Acked**，就當作是該 Message 有成功處理，然後則該 Message 就會從 Message-Storage 中異步刪除。

{{< alert success >}}
因為一個 Subscription 可以有多個 Subscriber 來**並行同時**處理資料，且 Message 只會送給一個 Subscriber 來處理，故只要有一個 Subscriber 表示該 Message Acked 處理完成，則 Subscription 就認為該 Message 成功消費這樣的想法是合理的
{{< /alert >}}

##### Negatively acknowledged (Nacked) Message - 否定確認消息

形容這個狀態的名稱只是看英文會有點疑惑，我個人會稱做「**確認處理失敗消息**」。如字面上的意思所述， Message 在處理過程中發現了問題無法繼續，例如 :「Message 格式錯誤」、「Subscriber 認為處理太久而自己取消流程」等等，若屬於**使用者能自己 try-catch 的情形**，就會用 Nacked 回應保證這條消息並**沒有成功消費**。 在 Nacked 的情況，可以選擇 Retry-Policy 有 ：

> - Retry immediately
> - Retry after exponential backoff delay

依照上面 Policy 的設定時間間隔， Message 會重新傳遞來重試，若重試成功就變成 Acked ; 重試失敗則重複 Nacked 流程。

{{< alert info >}}
Retry immediately 也就是通過設定 ackDeadline 為 0 來實現
{{< /alert >}}

Nacked 的時候要注意**避免一直重複 Retry** 的情況，像是如果是發生「Message 格式錯誤」的情形，那麼應該是永遠不可能會成功 Acked 的，故這時通常還要設定**死信隊列（Dead Letter Queue, DLQ）**，且在 Pub/Sub 還可以直接設定 `max-delivery-attempts` 最大嘗試投遞次數，超過次數上限後，這些 Message 會被自動移入指定的 DLQ，避免一直重複 Retry。

##### Unacknowledged messages (Unacked) Message - 未確認消息

由於 Message Service 本質上會解耦生產端和消費端，故為了確保 Subscriber 消費的速度能跟上 Publisher 產生 Message stream ，我們會監控**消息積壓(Message-Backlog)**，例如說：

- Unacknowledged messages -`subscription/num_undelivered_messages`
- Oldest unacknowledged message age - `subscription/oldest_unacked_message_age`

然後若看到 `num_undelivered_messages` 和 `oldest_unacked_message_age` 都在穩步上升，就知道 **Subscriber 消費跟不上消息增加速度**。從 Pub/Sub Monitoring 的指標命名下也可以比較能反映出 **Unacked** 的定義： 「**Subscription 收到 Acked 或 Nacked ，就稱為 Unacked**」。發生這種事情的經典情況可能是 :

1. 在 Subscriber 的內存中排隊等待處理中，英文上也會稱 **Outstanding**，此單字除了傑出的、卓越的，還可以表示「未支付的」、「未完成的」

2. Subscriber 處理比較久時間而超過 deadline 時間限制，故 Message 會是 Unacked 的狀態，而這種又可以分成兩種：

   > 1. 這則 Message 處理一直都需要很大的計算資源導致 consumer 直接崩潰，沒辦法做任何回應，也沒有消費成功
   > 2. 雖然 Subscriber 處理比較久超過 deadline，但還是成功消費完成了

3. Subscriber 其實已經處理完成了，但是因為網路問題而使得 Subscription 沒有收到回應，故 Message 也會是 Unacked 的狀態

Unacked 的 Message 也會持續被重複投遞直到 `retention duration` 過期或者成功回覆才會停止，例如說 `2.1` 的情況發生會有很高的機率產生**一直重複投遞**，故也需要設定 Retry-Policy 和 DLQ 處理錯誤情形。

{{< alert danger >}}
只要有了重試，會發現在上述情形 `2.2`、`3` 之下是會發生**重複消費**問題，因為實際上是有消費成功，但 Subscription 卻因故沒收到回應導致 Retry，這是使用 Messaging Service 時很令人頭疼的... 要避免重複消費的系統設計會是另一個大主題，基本上會是運用**唯一 Message ID** 來達成，這邊先略過，後續有機會再補充
{{< /alert >}}





---

### 參考資料

- [What is Pub/Sub? ](https://cloud.google.com/pubsub/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Google Cloud Pub/Sub 介紹](https://yinghao1019.github.io/gcp-pubsub/)

https://medium.com/@louijose/maximizing-efficiency-in-messaging-systems-google-pub-sub-vs-partition-based-models-e8552e7822e8

https://tachingchen.com/tw/blog/google-cloud-pubsub-introduction/


你想在 Cloud Run 上部署一個應用程序來處理來自 Cloud Pub/Sub 主題的消息。你希望遵循 Google 推薦的最佳實踐。你應該怎麼做？

A. 1. 創建一個使用 Cloud Pub/Sub 觸發器的 Cloud Function。2. 每次有消息時，從 Cloud Function 調用 Cloud Run 上的應用程序。 B. 1. 為 Cloud Run 使用的服務帳戶授予 Pub/Sub Subscriber 角色。2. 為該主題創建一個 Cloud Pub/Sub 訂閱。3. 讓你的應用程序從該訂閱中拉取消息。 C. 1. 創建一個服務帳戶。2. 為該服務帳戶授予 Cloud Run Invoker 角色，用於你的 Cloud Run 應用程序。3. 創建一個使用該服務帳戶的 Cloud Pub/Sub 訂閱，並使用你的 Cloud Run 應用程序作為推送端點。 D. 1. 將你的應用程序部署在 Cloud Run on GKE 上，並將連接性設置為 Internal。2. 為該主題創建一個 Cloud Pub/Sub 訂閱。3. 在與你的應用程序相同的 Google Kubernetes Engine 集群中，部署一個容器，將消息傳遞給你的應用程序。




B. 讓 Cloud Run 應用程序拉取 Pub/Sub 消息
這個解決方案將 Pub/Sub 訂閱設置為“拉取”模式，並授權 Cloud Run 的服務帳戶拉取消息。雖然這是可行的，但 Google 更推薦使用 Pub/Sub 的“推送”模式，因為這樣可以更簡化消息處理的流程，不需要應用程序持續拉取消息並進行管理。

不推薦：這種拉取模式在 Cloud Run 中不是最佳實踐，推送模式會更有效率和易於管理。

C. 使用推送訂閱將消息推送到 Cloud Run
這個選項採用了推送訂閱，將 Pub/Sub 訂閱消息直接推送到 Cloud Run 應用程序，這是 Google 推薦的最佳實踐。使用推送訂閱意味著 Cloud Pub/Sub 可以自動將消息推送到指定的 URL（Cloud Run 的端點），無需應用程序手動拉取，這樣可以減少運行時的複雜性。給服務帳戶授權 Cloud Run Invoker 角色則保證了安全性，只有授權的帳戶可以調用 Cloud Run 應用。

推薦：這是 Google 的最佳實踐，使用 Cloud Pub/Sub 推送訂閱到 Cloud Run，減少了中間層和手動操作，並且更符合事件驅動架構。