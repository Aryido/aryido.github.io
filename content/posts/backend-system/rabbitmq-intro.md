---
title: "RabbitMQ 簡介"

author: Aryido

date: 2025-06-20T23:22:49+08:00

thumbnailImage: "/images/backend-system/rabbitmq-logo.jpg"

categories:
  - backend-system

tag:
  - data-exchange

comment: false

reward: false
---

<!--BODY-->

> 之前有介紹了 Message Queue 和其常見協定 MQTT 、 AMQP ，而 RabbitMQ 就是一款基於 AMQP 訊息傳遞協定實現的輕量級開源服務，由 Erlang 語言開發，能夠跨進程的傳遞訊息，且對多數主流程式語言如 Python、Java 等都有官方或社群開發 lib。
> {{< image classes="fancybox fig-100" src="/images/backend-system/rabbitmq-architecture.jpg" >}}
> 再來為了方便運維與監控，RabbitMQ 有內建一套 Web 介面，使用者可透過此介面管理檢視 queue 健康狀態，還可以處理使用者權限等操作

<!--more-->

---

# RabbitMQ

RabbitMQ 生產者將訊息透過 Channel 傳送到 Exchange，再來 Exchange 決定將訊息分發到哪個 queue ，最後消費者從 queue 中接收訊息。

{{< image classes="fancybox fig-100" src="/images/backend-system/rabbitmq-workflow.jpg" >}}

### 虛擬主機 (vhost)

vhost 擁有自己的權限機制，一個 broker 內可以開設多個 vhost，用於不同用戶的權限隔離 ; vhost 之間是也完全隔離的 ; 而一個 vhost 內也可以有若干個 exchange 和 queue，同一個 vhost 裏面不能有相同名稱的 exchange。

當 RabbitMQ 運行時，預設會產生一個 virtual host 叫做 /，然後如果不特別調整的話，所有的 Queue 都是創建在這個 / 的 virtual host 裡面，而 user 預設也是被設定成能存取 / ，所以當多個不同的使用者，使用同一個 RabbitMQ server 提供的服務時，可以分割出多個 vhost，讓每個使用者在自己的 vhost 建立 exchange 和 queue 。

### Exchange

Exchange 是在 Producer 與 Queue 之間，用來接收消息，根據路由鍵 routing-key 轉發 Message 到綁定的 Queue。Exchange 不具備 Message 存儲能力，主要有不同類型 type 可選擇，簡單舉例有 :

- Direct : 轉發 Message 到指定的 Queue
- Topic : 可按照 regular exp 來匹配轉發 Message
- Fan-out : 轉發 Message 到**所有**綁定的 Queue，類似於廣播發送

而 Exchange 和 queue 之間的關係連結稱為 binding ， binding 資訊被儲存到 exchange 中的查詢表中，用於 message 的分發依據，聲明 binding 關係的時候，會使用 RoutingKey 參數。

---

# RabbitMQ 設計模式

上面有提到 Exchange 有不同的 type ，那接下來根據 [RabbitMQ Tutorials 官方的範例](https://www.rabbitmq.com/tutorials)，以下說明幾種：

### Routing (也稱為 Direct 模式)

Exchange type 為 direct，特性是 Exchange 與 Queue 的 binding 還會帶上 routing key，Producer 傳送訊息到 Exchange 時也會帶上 routing key 這個參數，可以多重綁定，故可以達成把 Message 給多個指定 queue 但不像 Publish/Subscribe 全部都廣播出去，因此可以達到選擇性訊息分流，不同 Consumer 只需要接受到特定 routing 的訊息。

{{< image classes="fancybox fig-100" src="/images/backend-system/rabbitmq-routing.jpg" >}}

實際舉例的話，可以將 info、error、warning 這三個 routing key，綁到記錄一般 Log 的 Queue 上，然後再將 error 這個 routing key，再綁定到另一條記錄 Error Log 的 Queue 上，這樣子就可以實作出一份帶有全部 Log 的 Queue、以及一份只有 Error Log 的 Queue 了。

### Topics

Exchange type 為 topic，也透過 routing key 來分流訊息，差別在 topic 的特性能夠**模糊綁定**而非固定的 routing key，能使用 regular exp 設定 .(dot) 、 \*(star) 、 #(bash) 匹配到了多個 Queue。

### Publish/Subscribe (也稱為 Fanout 模式)

Exchange type 為 fanout，由於 fanout 的特性，Exchange 會把訊息廣播給所有綁定的 Queue，每個 Consumer 就會接收到相同的訊息。因此當有另外的系統需要同步接收訊息，只需增加一組 Queue + Producer，綁定這個 Exchange 即可。

{{< alert warning >}}
如果生產者將消息發送到沒有綁定隊列的交換機上，消息將丟失
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/backend-system/rabbitmq-exchange-types.jpg" >}}

### Work Queues

Worker 模式會有多的 Consumer 從同一個 Queue 取出訊息，加速訊息處理的速度。因此只要連接同一個 Queue，就可以在多台機器上 Consumer 平行處理。當有多個 Consumer 時，如何均衡 Consumer 消費模式，主要有兩種模式：

- **輪詢模式分發**：按順序輪詢分發，每個消費者獲得相同數量的消息
- **公平分發**：根據費能力公平分發，處理快的處理的多，處理慢的處理的少，按勞分配

---

### 參考資料

- [[DATA] 訊息佇列 02 - RabbitMQ 簡介與 5 種設計模式](https://enzochang.com/rabbitmq-introduction/)
- [一文搞懂 RabbitMQ 常用模式](https://www.readfog.com/a/1669788280201252864)
- [RabbitMQ 介紹（二）- RabbitMQ 用法介紹](https://kucw.io/blog/2020/11/rabbitmq/)
- [Message Queue 簡介(以 RabbitMQ 為例)](https://godleon.github.io/blog/ChatOps/message-queue-concepts/)
- [探索 RabbitMQ 的內部架構](https://mybaseball52.medium.com/researching-on-rabbitmq-architecture-and-concepts-476e2d912575)
- [RabbitMQ 與訊息確認](https://yoziming.github.io/post/220204-gulimall-18-rabbitmq/)
- [The RabbitMQ Management Interface](https://www.cloudamqp.com/blog/part3-rabbitmq-for-beginners_the-management-interface.html)
- [RabbitMQ 學習筆記](https://kangshitao.github.io/2021/10/26/rabbitmq/#1-1-MQ%E7%9B%B8%E5%85%B3%E6%A6%82%E5%BF%B5)
