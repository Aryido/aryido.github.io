---
title: GCP - Bigtable 概述 II - 儲存模型

author: Aryido

date: 2024-07-31T15:28:33+08:00

thumbnailImage: "/images/google-cloud/bigtable/bigtable-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> Bigtable 是對有使用 GCP 雲端的人經常會被問到有沒有使用過的一個服務，我想是因為 Google 在 2003 ~ 2006 年間連續發表了幾篇很有影響力的技術文章：
>
> - [The Google File System (GFS)](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf)
> - [MapReduce: Simplified Data Processing on Large Clusters](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf)
> - [Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/bigtable-osdi06.pdf)
>
> 其中主角 Bigtable 則是其技術集大成者，其基於「 Chubby 的 Paxos 演算法提供分散式鎖服務 」、「 SSTable 作為存儲格式 」、「 GFS 儲存模式 」等等技術所建構出的分散式存儲系統，又加上可以在低延遲的情況下支持高讀寫吞吐量，故也是 Google 開發的大規模並行計算框架 MapReduce 理想的 source 來源。最後 `2015/5` 變成了 GCP 的一個雲端產品 Bigtable，提供強大的分散式儲存功能。

<!--more-->

---

在 google 官網上 Bigtable 的定義是一個 sparsely populated table，其中 sparse 意思是如果某 Column 未在特定 Row 中使用，就不會佔用任何空間。 Bigtable 可以輕鬆擴展到「 數十億 Rows 」 或有「 數千個 Columns 」，且能夠存儲 PB 級的資料量，適合 High-Throughput 場景的服務。

{{< alert success >}}
Apache Hadoop 系列中 HBase 一般被認為是 Bigtable 的開源實現，甚至連 API 都可共用，適用場景也大部分重合 ; Cassandra 也是以 Bigtable 原始論文為模型實現出來的技術，不過在部分設計思路上有很大不同。
{{< /alert >}}

承前篇 Bigtable Architecture 的介紹，對於部署一個 Bigtable ，其整體是稱為 Bigtable Instance 。 Instance 最前面有 Front-end Server 控制路由 ; 且由一到多個 Clusters 組成 ; 每個 Cluster 內也有一到多個 Nodes ; 還有儲存組件 Colossus 用來儲存中 SSTable 與 Log，整體來說架構為：

```
Bigtable = 「 Front-end Server Pool 」 + 「 Cluster(s) 」 + 「 Colossus 」
```

# Bigtable storage model

Bigtable 對於如重啟節點、擴展、資料備份等等都是自動管理的，故對使用者來說重要的是: **需要掌握 Table 的設計**，以根據訪問模式和查詢要求來 dirven 設計 ：

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigtable/bigtable-storage-model.jpg" >}}

Bigtable 將資料存儲在可大規模擴展的 Table 中，雖然提到一個在關聯式資料庫常常使用的 Table 這個名詞，但實際上 Bigtable 指的 Table 根本和關聯式資料庫的「表」沒有任何關係，它的物理存儲形式其實是一個**排序 Map**，每個 Bigtable 的 Table 都是一個 **sorted key-value map** 。Bigtable 將所有資料視為 byte strings，存儲邏輯可以表示為：

`(row:string, column:string, time:int64) → string`

Table 由 Rows 組成，每 Row 都由 Cloumns 組成。然後再次提醒: **雖然有一些術語如 row、column、table 等等，但它和傳統 RDB 那些概念是不太一樣的**。

### Row

Row 通常描述單個 Entity。 **Row 只有一個 value 會被編入索引 index ，這個索引 index 值就被稱為 Row-Key** ，其類似於 RDB 的 primary key 主鍵，資料會按順序排序儲存，故從 Bigtable 中檢索資料可以使用 Row-Key 指定特定的 Row，特別注意 Bigtable 中沒有 secondary indexes，因此 Row-Key 的設計非常重要。Bigtable 通過 Row-Key 的字典順序來組織資料，Table 中的每個 Row 都可以動態分區，每個分區叫做一個 Tablet，Tablet 是資料分佈和負載均衡調整的最小單位。

{{< alert danger >}}
GCP Bigtable 的 Hard limits，Row-Key 不能超過 4 KB
{{< /alert >}}

想要讓 Bigtable 有更好的寫入性能，重點是在儘可能均勻平均的寫入不同的 Node 內，故建議 Row-Key 選擇是比較偏向亂序的 ; 同時對有相關的資料進行分組讓他們連續的放在一起。

{{< alert warning >}}
注意， BigTable 支援單 Row 事務，可以對存儲在某個 Row-Key 下面的資料執行原子的 CRUD 操作，但不支援通用的跨 Row 的事務
{{< /alert >}}

### Column

首先介紹 Column-Family ，它是多個 Columns 的群組， Bigtable 是結構化（Structured）資料，故在定義表 Table 時就是**一定要定義的**。

再來 Column-Family 下面是 Column ，可以有多個且可動態添加，也是按字母順序被排序存放的。 Column Key 的格式為 `family：qualifier` ，以 Qualifier 作為唯一標示名稱，且承上也可看出 Column-Family 會作為每個 Column-Key 的**前綴**。

{{< alert warning >}}
GCP 建議 Qualifier 名稱大小不要超過 16 KB ，且在單一 Table 中 Column-Family 不要超過 100 個，但基本上單一 column 數量並沒有限制。
{{< /alert >}}
Column-Family 通常很少變化，其內部的 Column 一般存儲相同類型的資料，可方便對數據進行壓縮。不同的 Column-Family 可以設置不同的訪問許可權，**是 Bigtable 中 Access Control 的基本單元**。

Column-Family 這個方式可以很有效率的讀取一群經常被放在一起讀取的資料，實務上例如街道地址，城市，鄉鎮，郵遞區號這幾個欄位就是經常會被一起讀取的，所以可以將這幾個 column 組成一個 Column-Family 。

{{< alert info >}}
Column 可以在 row 中**不被使用**，這也是 BigTable 被稱為 sparse 的重要原因
{{< /alert >}}

### Cell

Cell 就是 Row 與 Column 的交集處，而且 Cell 中的值會有多個版本，版本按時間戳進行索引，按 Timestamp 的降序排序（最新的資料排在最前面），每個 Cell 都有一個唯一的 Timestamp 。

{{< alert warning >}}
GCP 建議在單一 cell 中存儲不超過 10 MB ; 在整個 Row 中存儲不超過 100 MB
{{< /alert >}}
{{< alert danger >}}
[Hard limits](https://cloud.google.com/bigtable/quotas#limits-data-size): 單一 cell 中存儲不超過 100 MB ; 在整個 Row 中存儲不超過 256 MB
{{< /alert >}}

為了減輕多個版本的管理產生的資料冗余，Bigtable 可以通過設置指定例如

- 只有最新的 （only new enough）
- 保留最新的 N 個版本 （only last n）
- 只保留過去 7 天寫入的版本

以上可以對廢棄版本的資料，自動進行垃圾收集 Garbage collection，其是一個內建 built-in, asynchronous background process，由於無法自控，故在真的被系統刪除前還是會出現在 Read Results，所以會需要 filter，而篩選規則是：[不包括與垃圾回收 policy 相同目標資料](https://cloud.google.com/bigtable/docs/garbage-collection#data-removed)。

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigtable/data-size.jpg" >}}

### 總結

- 每個 Row 可以不用設計完整的 Column，保留很大的擴展彈性
- 不需要填滿每個 Row 的每個 Column
- 幾乎可以處理所有類型的資料，因為 BigTable 將其所有資料都視為字串
- k-v 模型看起來簡單，卻可以表達成很多形式：
  | rowkey | info:name | info:age | meta:status |
  |--------|-----------|----------|-------------|
  | 1 | 小明 | 19 | 1 |
  | 2 | 小紅 | 17 | 0 |
  | 3 | 小剛 | 13 | 1 |

  像上面的一個常見的表格，在 Bigtable 中可以使用如下方式表示：

  ```
  table1:1:info:name   -> 小明
  table1:1:info:age    -> 19
  table1:1:meta:status -> 1
  ```

  上面 3 條記錄共同構成原表中的 rowkey 為 1 的 row，形成一個 entity

另外特別注意，[Table 屬於整個 Bigtable Instance，而不是屬於任一 Cluster 或 Node](https://cloud.google.com/bigtable/docs/instances-clusters-nodes#instances)，所以其實沒有將 Table 分配給某個指定 Cluster 這種功能，也沒有在不同 Cluster 內的 Table 存不同資料這種事情。

# 設計範例

Bigtable 論文給出的例子 Webtable 如下圖，需求是想保存大量的 web 和網頁相關的 metadata，這些資料會被不同的專案使用。接下來是對應 Table 設計:

> - Row-Key: URL
> - contents: 也是 colume family ，存放頁面 html 內容
> - anchor: 是 Column-Family ，儲存引用了這個頁面的 anchor（HTML 錨點）的文本

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigtable/webtable.jpg" >}}

Bigtable 表內部存儲了大量的 web 相關信息，如圖中所示，代表 CNN 首頁被 cnnsi.com 和 my.look.ca 以上兩個給引用了，因此會有 `anchor:cnnsi.com` 和 `anchor:my.look.ca`， qualifier 是引用這個網頁的 anchor 名字，而對應的 value 是連結的文本（link text）; 另外可以發現 contents column 有三個版本，時間戳分別為 `t3, t5, t6` ;

在 Webtable 中，每 row 以**反轉的 URL** 作為 row-key 而 value 內容是存儲 web html，而反轉 URL 的原因是為了讓類似名稱的子域名網頁能聚集在一起，可以利用 Locality 提高查詢效率。
{{< alert info >}}
將多個 column family 組織到一個 locality group，每個 tablet 會為每個 locality group 產生一個單獨的 SSTable。 locality group 詳細研究後是可以增進 Bigtable 很多效能的，可以多多參考
{{< /alert >}}

例如 `maps.google.com/index.html` 頁面在儲存時， Row-Key 就是把字反過來變成 `com.google.maps/index.html`，其會把來自相同網域的頁面儲存到**連續的 Row**，會使那些針對「 主機 」和「網域」的分析（host and domain analyses）非常有效率。

---

### 參考資料

- [Bigtable overview](https://cloud.google.com/bigtable/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Cloud Bigtable – 完全代管的 NoSQL 資料庫服務 (一)](https://ikala.cloud/google-cloud-bigtable-intro-1/)

- [Paper Notes: Bigtable – A Distributed Storage System for Structured Data](https://distributed-computing-musings.com/2022/09/paper-notes-bigtable-a-distributed-storage-system-for-structured-data/)

- [Essential Cloud Infrastructure: Core Services - Summary Notes - Part 2](https://www.polarsparc.com/xhtml/Essential-GCP-Serv-2.html)

- [《Bigtable》论文笔记](http://hecenjie.cn/2020/02/04/%E3%80%8ABigtable%E3%80%8B%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0/)

- [Google 经典系统赏析 | GFS, Big Table & MapReduce](https://blog.acecodeinterview.com/google/)

- [[译] [论文] Bigtable: A Distributed Storage System for Structured Data (OSDI 2006)](https://arthurchiao.art/blog/google-bigtable-zh/)
