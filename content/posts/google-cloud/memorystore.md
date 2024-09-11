---
title: GCP - Memorystore 概述

author: Aryido

date: 2024-08-07T16:58:29+08:00

thumbnailImage: "/images/google-cloud/memorystore/memorystore-logo.jpg"

categories:
  - cloud
  - gcp

tags:

comment: false

reward: false
---

<!--BODY-->

> Memorystore 是 GCP 提供的完全託管 In-Memory Database 服務，可以輕鬆地在 GCP 上建立 Redis 和 Memcached 等著名的開源 **Caching-Engines**，專門提供**毫秒級的低延遲資料存取/寫入**，並減輕管理 Database 的部署 deployment、Replica、容錯移轉等等維運事項。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Amazon ElastiCache**
> - Microsoft Azure : **Azure Cache**
>
> Memorystore 也是屬於 NoSQL，由於常用於快取情境故也會被稱呼為**快取資料庫**。目前看起來主流是使用 Redis，故建議在大部份的使用場景上，優先考慮使用 Redis。

<!--more-->

---

「應用程式效能」已成為現在應用架構中的核心議題之一，當 Data 或 Query 快速增加時，往往會出現 response time 顯著提高的現象，在這個時候可以於「應用服務」跟「資料庫」的中間層，再做一個快取資料庫服務，把使用頻率高的資料存儲在快取中，減少傳統 DB 負擔並大大加速應用程式的回應速度。 Memorystore 快取技術是一種基於 Redis 和 Memcached 等開源項目的**雲端快取服務** :
{{< image classes="fancybox fig-100" src="/images/google-cloud/memorystore/cluster-redis-memcached.jpg" >}}

上圖可看到 Memorystore 有提供簡單的代管 [Redis](<(https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview)>) 和 [Memcached](https://cloud.google.com/memorystore/docs/memcached/memcached-overview) ，進一步還有代管 [Redis Cluster](https://cloud.google.com/memorystore/docs/cluster/memorystore-for-redis-cluster-overview)：

# Memorystore for Redis/Memcached

Redis/Memcached 兩者都是 In-Memory Database，都可以提供低延遲、高效率的資料存取，非常適合拿來當快取資料庫使用，以下對兩者個別做簡單介紹 :

### Memorystore for Redis

其因為符合 Redis 協議，故可以將使用 Redis 的應用程式直接遷移到 Memorystore 而無需更改任何程式碼。有支援兩種 Tier 分別是 Basic 和 Standard :

- ##### Basic Tier:

  便宜但沒有 High Availability(HA)，因為沒有 Replication，故 Redis 只會部署在**單一個 Region 的其中一個自己選的單一 Zone 內**，但優點是會比較便宜。

- ##### Standard Tier:

  會使用 Replication 來達成自動故障轉移（Automatic Failover）來實現 High Availability(HA) ，可提供 `99.9%` 的可用性 SLA ; 若進一步啟動 Read-Replicas 還可以分散式讀取（Distributed Reads）來增加回應速度。

  {{< image classes="fancybox fig-100" src="/images/google-cloud/memorystore/standard-with-readonly.jpg" >}}

值得注意的是 Memorystore for Redis **大多數的配置參數在創建後是無法更改的**(如上述的 Tier) ，若想更換 Tier 只能新建其它新的 Instance 並依靠資料 import 和 export 來轉移資料。

{{< alert danger >}}
Memorystore for Redis 目前支援 RDB Snapshots 和 Exporting Data，但**不支援 AOF 持久化**
{{< /alert >}}

{{< alert info >}}
##### 補充

RDB 全名 Redis DataBasa File，是 Redis 在指定的時間，會將此刻整個 DB 資料做成 Snapshot，就是將這些資料寫成一份 `.rdb` 檔案。

AOF 全名 Append-Only File
Comment 。 AOF 會專門儲存**資料的操作**，這些操作的紀錄會儲存到一份名為 `.aof` 檔案中。
{{< /alert >}}

### Memorystore for Redis 和 Memorystore for Memcached 簡單比較

Memorystore for Memcached 也為 GCP 完全託管的開源 Memcached 服務，目前和 Redis 比較大的不同點，是缺少了例如 backup、replication、pub-sub 等等功能，還有是 **Memcached 缺少比較複雜的 data-type 如 Geohash**。

> - Memcached 用 multi-thread 處理請求 ; Redis 只有 main thread，但在 `Redis 6.0` 之後在**網路 I/O**的部分也使用了 multi-thread 增加了速度

> - Memcached 只支援簡單的資料型別。

簡單整理來說：
- 如果使用的 data-type 比較簡單，僅是 key-value 形式就可以**選擇 Memcached**
- 若需要複雜的資料型態，或需要做讀寫分離和保持比較高的可用性，就**選擇 Redis**

{{< alert success >}}
multi-thread 比較能善用 CPU，例如 AWS EC2 的 CPU 和 Memory 是同步成長的，為了用更多 memory 而租高級的型別，這時盡量使用 Redis 6.0 之後版本或者 Memcached，才不會會浪費一堆沒在用的 CPU
{{< /alert >}}
目前網路上我看到比較多的主流是使用 Redis，故可優先考慮使用 Redis，資料也比較好找。

# Memorystore for Redis Cluster

Memorystore for Redis Cluster 具有零停機時間擴展功能，且官方描述其 throughput 比單純 Redis 高出 60 倍，還有可跨 Zone 放置 Primary 和 Replica，並自動管理故障轉移，以上大大的提高了可用性和降低了操作複雜性。

Memorystore for Redis Cluster 基於**開源 Redis 版本 7.x** ，支援大部分 Redis command library，但還是要注意有 [Blocked commands 例如 ACL 相關命令](https://cloud.google.com/memorystore/docs/cluster/supported-commands#blocked_commands)。以下做一些基本術語介紹：

{{< image classes="fancybox fig-100" src="/images/google-cloud/memorystore/cluster.jpg" >}}

Memorystore for Redis Cluster Instance 代表一個 Redis Cluster 單元，在 [GCP 官方文件](https://cloud.google.com/memorystore/docs/cluster/memorystore-for-redis-cluster-overview#instances)中有表示「Instance」和「Cluster」是指同一件事情。

Instance 由一組大小相等的 Shards 組成，每個 Shard 包含: 
> - 一個 Primary node 簡稱 Primary 
> - `0 ~ 2` 個 Replica nodes 簡稱 Replicas

Memorystore 會自動跨 Zone 分配 Primary/Replica 避免都在同 Zone 。

Shard 的多寡定義了 Cluster 的 Throughput 和 Memory 大小，目前看起來最多可以有 125 Shards ; 而 Replica 會提供更高的 Availability ; 每個 shard 最多可以有 2 個 replicas，在啟用副本的情況下 Cluster 可提供 `99.99%` 的 SLA。

{{< alert danger >}}
由於 replicas 的設定是 Per-Shard 為單位，故 Shard 越多開啟對應的 Replicas 複製也就成倍數增加，故要注意一下費用
{{< /alert >}}

# 建議 & 結語

一般來說 in-memory 資料庫，如果碰到一些狀況重啟，則資料會通通不見，因為**資料都只在放在暫存記憶體**中。為了避免此情況，會建議**啟用高可用**，而實現方式的話 Redis Cluster 有提供 Sharding 和 Replica 來使之高可用 ; 另外 Redis 使用 RDB 和 AOF 來備份復原。

---

### 參考資料

- [Memorystore Office Doc](https://cloud.google.com/memorystore?hl=en)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Redis vs Memcached 比較](https://medium.com/jerrynotes/redis-vs-memcached-%E6%AF%94%E8%BC%83-15d2ba829da7)

- [Understanding Google Cloud Memorystore for Redis Cluster: A Fully Managed Redis Service](https://medium.com/google-cloud/understanding-google-cloud-memorystore-for-redis-cluster-a-fully-managed-redis-service-6de56ba0ea2c)

- [Redis 和 Redis Cluster 概念筆記](https://medium.com/fcamels-notes/redis-%E5%92%8C-redis-cluster-%E6%A6%82%E5%BF%B5%E7%AD%86%E8%A8%98-fdc19a3117f3)
