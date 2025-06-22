---
title: Message Queue 簡介 - MQTT & AMQP

author: Aryido

date: 2025-06-16T14:58:37+08:00

thumbnailImage: "/images/backend-system/rabbitmq-logo.jpg"

categories:
  - backend-system

tag:
  - data-exchange

comment: false

reward: false
---

<!--BODY-->

> 為了要把系統架構解耦，改為異步分散式處理，有時會決定使用 Message Queue 的設計來達成目標，其指的是應用程序之間通過在 Message Service 而不是直接調用彼此通訊。對於其落地的產品，常見的開源工具有 **Kafka**、**RabbitMQ** 等等或者雲端服務 **GCP-Pub/Sub** 和 **AWS-SQS** 等等。作為兩個子系統之間的通信中間層，會需要依靠協定保持有序且有效率的方式實現資訊交換，而目前廣泛使用的協定有 **MQTT** 和 **AMQP** ，以下也會簡單介紹一下。

<!--more-->

---

# Message Queue

{{< image classes="fancybox fig-100" src="/images/backend-system/mq-multi.jpg" >}}

該架構中基本上可以抽象成幾個角色：
- **生產者** : 名稱可稱為 Producer、Publisher、Generator 等等，負責將訊息傳送出去
- **消費者** : 名稱可稱為 Consumer、Subscriber、Worker、Receiver 等等，能主動拿取或被動接收訊息
- **消息中間件**: 中間的部分稱呼也有很多種：如 Broker、Queue、Bus 或者直接使用實現產品的名稱 RabbitMQ、Kafka 等等

Message Queue 在系統架設設計時很常出現，兩個最常見的用途分別是 :

- **任務緩衝** : 譬如碰到有任務必須運算很久的 CPU Bound 任務，這時 Queue 可以讓任務進行「排隊」，使用者也可以先去做其他事達成非同步功效

- **系統解耦** : Producer 和 Consumer 並不需要知道彼此，只需要記得 Queue 在哪裡即可，故可以架設在不同主機，也不需要使用相同語言開發，最重要是可以很方便依據需求進行水平擴展

---

在電腦網路中，不同的設備需使用 **Protocol 協定**來建立通用語言，這些協定定義了交換資料的格式和規則。其中 AMQP 和 MQTT 都是定義了分散式系統之間訊息傳遞協定，儘管它們的用途相似，但在底層原理和用例上有一些所不同：

# MQTT : Message Queuing Telemetry Transport

MQTT 以輕量級著稱，由 IBM 在 1999 年推出，是為了在不穩定的網路中可靠地通訊，例如設備通常在資源有限、低頻寬、非穩定的網路環境中運行。至於是多輕量呢？ 從網路上查到的資料，MQTT 有 2bytes 的固定 header 而 AMQP 的 header 大小為 8bytes，所以在某些 iot 低功耗、低頻寬的設備，可能會以 MQTT 為首選。而且 MQTT 的設計更基於發布/訂閱的訊息模式 Publish-Subscribe Pattern，故解耦性會再強一些。

{{< image classes="fancybox fig-100" src="/images/backend-system/mqtt.jpg" >}}

{{< alert info >}}
MQTT 當時的優點是在於以低開銷、低頻寬佔用的即時通訊協議，故其在小型 iot 設備等方面有較廣泛的應用
{{< /alert >}}

---

# AMQP : Advanced Message Queuing Protocol

AMQP 是一種開放標準的應用層協議，由摩根大通於 2003 年創建，AMQP 是一種點對點協議，具有確定的發送者和接收者，實現生產者和消費者之間的直接通訊。其重點是 Message 首先會發佈到 **Exchange 交換器**，其作為路由代理，可處理複雜路由規則，將 message 轉送到對應的 queue，保證訊息被準確地傳輸，然後消費者再從 queue 中檢索訊息。

{{< image classes="fancybox fig-100" src="/images/backend-system/amqp.jpg" >}}

AMQP 的一個最關鍵特性是其訊息確認機制，它確保了訊息傳遞的可靠性。當訊息從生產者發送到消費者時，消費者需要發送一個確認回應(ACK)，表明訊息已被接收並處理。這種機制減少了訊息丟失的可能性。像 RabbitMQ 就是基於 AMQP 傳遞協定。

{{< alert info >}}
AMQP 所採用的模式是將訊息傳送到交換器（Exchange），此支援複雜的路由和分發策略，使 AMQP 適用於複雜的分散式系統。
{{< /alert >}}

---

### 參考資料

- [MQTT 與 AMQP：物聯網通訊協定對比](https://www.emqx.com/zh/blog/mqtt-vs-amqp-for-iot-communications)

- [AMQP vs MQTT: Messaging protocols compared](https://www.cloudamqp.com/blog/amqp-vs-mqtt.html)

- [訊息佇列 01 - Message Queue 介紹與實際應用](https://enzochang.com/message-queue-introduction/)

- [\[Kafka 和 RabbitMQ 有何區別？\]](https://aws.amazon.com/tw/compare/the-difference-between-rabbitmq-and-kafka/)

- [Message Queue 簡介(以 RabbitMQ 為例)](https://godleon.github.io/blog/ChatOps/message-queue-concepts/)
