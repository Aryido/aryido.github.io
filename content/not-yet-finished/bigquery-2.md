---
title: GCP - Bigquery 概述 II - 

author: Aryido

date: 2024-08-11T2:18:34+08:00

thumbnailImage: "/images/google-cloud/bigquery/bigquery-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> BigQuery 除了可以把資料以 columnar storage format 儲存進 BigQuery 內後進行查詢分析，也**支持直接查詢外部資料來源**如 GCS、Bigtable 、 Spanner/SQL 、 Google Sheets 等等

<!--more-->

---



### Metadata
BigQuery 在儲存空間部分，還有保存 Metadata。 Metadata 包含的內容有 table schema 資訊、 table expiration time 過期時間 等等。比較重要的是 Metadata 的保存是不需要付費的，並且 BigQuery 還會使用 Metadata 來私下優化查詢，這部分似乎是不可見的。

### Storage
BigQuery 雖然也可以處理非結構化資料，但**還是以結構化資料處理為主要**，故儲存的大部分資料還是以 Table Data 為主，詳細還分成以下幾種類型：

##### Bigquery Storage

> - **Standard Tables**: 
> 
> 每 Table 具有 schema，並且 schema 中的每一 column 都具有 data type

> - **Table clones**: 
> 
> 是 lightweight copies ，還是可以寫入的，只儲存 Standard Table 的變化

>  - **Table snapshots**: 
> 
> 是 read-only 的不支持寫入，也只儲存 Standard Table 的變化而已

> - **Materialized views**: 
> 
> 是 BigQuery 會定期預先計算的查詢結果，會緩存在 BigQuery 內用來提高 performance

##### BigLake Stroage
BigQuery 除了 query 儲存在自己 BigQuery 內的資料，其實也可以 query 外部的 datasource 如 Cloud Storage。提供兩種不同的機制來查詢外部 data，有分成 「External tables 外部表」和「federated queries 聯合查詢」：

- **External tables** 有幾種類型的 External table：
  - BigLake tables 
  - BigQuery Omni tables
  - Object tables
  - Non-BigLake external tables

External tables 其實就類似於 standard Tables，一樣也有 table schema 且儲存在 BigQuery 內，但真實資料儲存在外部，故 BigQuery 只會針對 table metadata 的儲存收費，外部的儲存資料由他們自己儲存服務的定義收費

- **Federated queries**
Federated queries 會使用 BigQuery Connection API 將查詢語句發送到 AlloyDB、Spanner 或 Cloud SQL ，並將結果用 temporary table 返回。




Managed 託管: BigQuery 是以使用量為基礎，根據每個月分析的資料量來計費，只需為使用的「儲存量」和「計算資源」付費，有關價格詳情，請參閱 BigQuery 價格

Durable 耐用: 有提供 `99.9999999999%` ，11 個 9 的 annual durability 年耐用性 ; 還有跨 zone replicate 可執行災難復原


BigQuery 將 table 整理到一個 logical containers 稱為 datasets



---

還有提供機器學習等 built-in 內置功能來管理和分析。





 





它很特別 它雖然是結構化資料 可是它不是資料庫 它不是拿來做什麼 Insert, delete, update
雖然它有這個語法 可是它不適合來這樣做 它本身沒有索引
也沒有 Key 就是它跟資料庫的用法不一樣 它不是拿來做什麼資料庫
效能調校 然後什麼關聯式 沒有, 都不是這樣做 雖然它可以
可是它不是拿來做這樣的用途 另外一方面是說 它的計費方式是有兩種
一個是你存多少資料 一個是你每次分析的時候你查詢了 多少資料
所以說我們今天就像前面講的 我如果做 Insert, delete, update
我做這種語法的時候 它的資料量它會用整個 Table 去算
即使你只動了一筆資料 它還是會用整個 Table 來算 因為它要整個 Table 掃描
所以它不適合做交易處理的動作 再來就是我們資料在寫入的時候
我們可以整筆匯入, 批次匯入資料 或者我們可以支援串流寫入
什麼叫串流寫入 就是資料是一次一筆進來 可是你每分每秒都會進來
你可能每秒一百筆, 一千筆 就是一直寫一直寫一直寫一直寫 它不是一次一萬筆進來
不是一口氣倒進來 它是每分每秒在 產生這種串流的寫入方式




Table
Table即為BigQuery資料實際存放的表格，透過持續的儲存，可以提供給使用者進行查詢使用。表格的設計支援自動依日期為單位的命名方式，即可將舊有的資料以日期另外封存，一方面可以降低每日資料查詢的數量，另一方面可以應用GCP上的BigQuery存放優勢。

View
View為某查詢的語句進行封裝的物件，例如可以將某常用查詢語句建立成View表，未來開發者可以透過該表格做快速的查詢，可以簡化查詢語句的複雜度。

Job
BigQuery的每個執行查詢的動作，均稱為Job。開發者可以透過BigQuery的指令或是web console查詢、監控目前Job的執行狀況。

Slots
BigQuery在運作上，會持續需要使用到大量的運算資源，基本上運算資源也是虛擬主機群所提供，而每個提供運算的服務器，即是Solt



---

### 參考資料

- [BigQuery overview](https://cloud.google.com/bigquery/docs/introduction)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)


- [Day 30: BigQuery 的下一座山頭](https://ithelp.ithome.com.tw/articles/10308507)




http://www.digitalmoon.cn/ask/article/2.html



You are creating an environment for researchers to run ad hoc SQL queries. The researchers work with large quantities of data.  Although they will use the environment for an hour a day on average, the researchers need access to the functional environment at any time during the day. You need to deliver a cost-effective solution. What should you do?
A. Store the data in Cloud Bigtable, and run SQL queries provided by Bigtable schema.
B. Store the data in BigQuery, and run SQL queries in BigQuery.
 
C. Create a Dataproc cluster, store the data in HDFS storage, and run SQL queries in Spark.
D. Create a Dataproc cluster, store the data in Cloud Storage, and run SQL queries in Spark.


你正在為研究人員創建一個環境，以便他們可以執行即席的 SQL 查詢。這些研究人員處理大量數據。雖然他們平均每天只使用一個小時的環境，但他們需要隨時可以訪問功能性環境。你需要提供一個具有成本效益的解決方案。你應該怎麼做？

A. 將數據存儲在 Cloud Bigtable 中，並運行由 Bigtable 架構提供的 SQL 查詢。
B. 將數據存儲在 BigQuery 中，並在 BigQuery 中運行 SQL 查詢。
C. 創建一個 Dataproc 集群，將數據存儲在 HDFS 存儲中，並在 Spark 中運行 SQL 查詢。
D. 創建一個 Dataproc 集群，將數據存儲在 Cloud Storage 中，並在 Spark 中運行 SQL 查詢。

正確答案：B. Store the data in BigQuery, and run SQL queries in BigQuery.
分析：
A. 將數據存儲在 Cloud Bigtable 中，並運行由 Bigtable 架構提供的 SQL 查詢
Cloud Bigtable 是一個 NoSQL 資料庫，適合用於處理時間序列數據和其他非結構化的大數據集。但它並不提供直接的 SQL 查詢功能，而是需要特定的 API 來進行數據檢索。因此，它並不適合處理研究人員所需的即席 SQL 查詢。
不推薦：Bigtable 不適合運行傳統 SQL 查詢，它是一個 NoSQL 資料庫。
B. 將數據存儲在 BigQuery 中，並在 BigQuery 中運行 SQL 查詢
BigQuery 是 Google Cloud 的無伺服器數據倉庫服務，專為處理大規模數據集和運行高效的 SQL 查詢而設計。它可以按需擴展，並且是即時查詢環境的理想選擇。由於研究人員需要隨時訪問數據，但每天只使用一小時，BigQuery 是一個具有成本效益的解決方案，因為你只需為實際使用的查詢量付費，而不需要維護任何基礎設施。
推薦：BigQuery 是大規模數據查詢的最佳選擇，並且成本效益高，符合研究人員的使用模式。
C. 創建一個 Dataproc 集群，將數據存儲在 HDFS 存儲中，並在 Spark 中運行 SQL 查詢
Dataproc 是 Google Cloud 的 Hadoop 和 Spark 托管服務，它可以用於批處理和即時處理。然而，HDFS 是一種分佈式文件系統，適合持續運行的大數據處理環境。對於僅使用一小時並且需要隨時訪問的環境，Dataproc 集群需要手動管理和維護，並且運行成本較高，不如 BigQuery 成本效益高。
不推薦：維護 Dataproc 集群和 HDFS 存儲的成本較高，不適合短時間高效使用。
D. 創建一個 Dataproc 集群，將數據存儲在 Cloud Storage 中，並在 Spark 中運行 SQL 查詢
這個選項與選項 C 類似，使用 Dataproc 和 Spark 進行查詢，但數據存儲在 Cloud Storage 中，而不是 HDFS。這會降低部分存儲成本，因為 Cloud Storage 比 HDFS 更靈活。然而，Dataproc 集群依然需要持續運行或啟動，對於間歇性查詢需求，運營和維護成本較高。
不推薦：雖然 Cloud Storage 降低了存儲成本，但 Dataproc 集群的運行成本依然較高。
結論：
選項 B 是正確答案。BigQuery 是 Google Cloud 上為大規模數據分析設計的無伺服器解決方案，支持高效 SQL 查詢，並且按照查詢次數和數據量計費。這非常符合研究人員隨時需要查詢數據但每天只使用一小時的需求，並且具有較高的成本效益。

---

Your company is hosting 10TB of customer data in BigQuery. The CTO of the company has decided to use this data and build some analytics on top of data which they have. At the end of the first month there was a huge spike in the bill due to use of BigQuery and CFO was not happy with the same. He has asked you to cut down the cost. How can you achieve this?

A. Use GROUP BY clause
B. Use composite keys to query the data
C. Instead of using SELECT *, query only required columns
D. Use JOINS in the query to fetch data

