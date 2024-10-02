---
title: GCP - Bigquery 概述 I - 架構

author: Aryido

date: 2024-08-09T23:08:24+08:00

thumbnailImage: "/images/google-cloud/bigquery/bigquery-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> BigQuery 是 Google 提供的一個無伺服器資料倉儲 (Serverless Data Warehouse)，其支持 ANSI SQL 來搜尋資料，所以只要會 SQL 語法就可以立即開始使用，且可高效率分析 TB、PB 等級的資料，故 Bigquery 也是企業級雲端大數據資料分析平台。對應到其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Athena**、**Redshift Spectrum**、**Redshift**
> - Microsoft Azure : **Azure Synapse Analytics**
>
> Google 在非常早期的時候，就有類似 BigQuery 其的服務就存在了，是自 2006 年以來一直在內部使用的 Dremel，後來隨著 GCP 雲端平台的產生，並於 2011 年以 BigQuery 為名被正式推出。目前是 GCP 分析資料的主力產品， Google 自家產品如搜尋引擎、 Gmail 等服務背後，其資料處理與分析的核心技術也和 Bigquery 息息相關。

<!--more-->

---

# Bigquery Architecture

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigquery/bigquery-architecture.jpg" >}}

BigQuery 服務利用了許多 Google 的技術如 :

- **Borg**: kubernetes 前身，由數台 VM 組成的集群
- **Dremel**: 可平行處理查詢的執行引擎，實現了一個 multi-level serving tree
- **Colossus**: Google 最新一代的分散式文件系統，GFS 的繼任者
- **Capacitor**: Column Oriented 儲存格式
- **Jupiter**: Google 的 Petabit 等級的高速網路，用於儲存和分析兩者溝通

BigQuery 架構很重要的是把 「compute」 和 「storage」 分離，這可以讓 BigQuery 根據需求獨立 scale up/down 其 storage 或 compute 資源，但也由於計算和存儲間的分離，故需要一個超高速網路讓其可以在幾秒鐘內將數 TB 及資料的傳輸於其間，而 Jupiter 是能夠提供 `1 PB/秒` 的頻寬的網路服務。

### Compute

首先介紹計算資源。當要在 BigQuery 內查詢資料時可能是利用 BigQuery Client 下一個 SQL 語法，接著 Client 會和 Dremel engine 互動， Dremel engine 作為 root server 會把**查詢重寫拆成小部分**，然後 parallel 並行分配給其下的 Dremel jobs，而 Borg 就會為這些多個 Dremel jobs 分配其計算容量。

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigquery/dremel-serving-tree.jpg" >}}

Dremel 會從 Colossus 儲存系統讀取相關資料，其因是完全在記憶體中執行 query ，且是使用 Jupiter 高速網路確保資料從 Colossus 讀取到 Dremel 內，故每個 Dremel Jobs 能以極快速度達成分析。最後所有 Dremel Jobs 做完分析之後，會把結果回傳給 Root ，然後 Root 會把每個運算分析的結果，整個收集合併起來，才回傳給 Client。

{{< alert success >}}
執行過程中需要的計算資源，都不需要自己控制分配，都是由 GCP 完全代管的，而底層是 BigQuery 利用 Borg 進行計算分配和處理容錯
{{< /alert >}}

### Storage

BigQuery 是把資料存儲在**分散式文件系統 Colossus 內**的，也是由 Google 提供完全代管，存儲容量大小的調整也是自動的，然後資料會在其中自動壓縮、encrypted ; 再來為了實現高可用，還會跨 zone 的 replica 來備份資料，進一步若在 BigQuery Dataset 內設定 **Multi-Region** 則還會跨 region 備份，可用於災難復原使 BigQuery 具有更高可用性。

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigquery/colossus-durability.jpg" >}}

BigQuery 是屬於 Column-Oriented databases ，更進一步說明: **BigQuery 的儲存是一種稱為 Capacitor 的格式**，壓縮方式類似如下:

```
c1 c2 c3             c1       c2      c3
--------             ----------------------
a  2  z     ====>    (3, a)   (2, 2)  (1, z)
a  2  x              (4, b)   (3, 1)  (3, x)
a  1  x                       (2, 3)  (2, y)
b  1  x                               (1, z)
b  1  y
b  3  y
b  3  z
```

在 BigQuery 內 Table 的每個 Column 都會被單獨儲存，並以 Capacitor 格式寫入 Colossus 內，當被發送到 Colossus 進行永久儲存時，所有內容都會被加密並會開始備份冗余。

BigQuery 也可以串接使用外部資料源例如 Bigtable、Cloud Storage 和 Google Drive 等等，這時在執行 query 時， BigQuery 會將外部資料動態載入 Dremel 的 in-memory 來進行運算，此時外部資料**不會被 BigQuery 儲存**。

{{< alert warning >}}
一般來說 BigQuery 使用外部資料源運行的查詢速度，會比資料儲存在 BigQuery 內部表運行的速度還要「慢」。如果對性能有要求，那麼在運行查詢之前建議將資料導入先 BigQuery 內
{{< /alert >}}

# Resource hierarchy

BigQuery 也有所謂的 Resource hierarchy ，此用來管理 BigQuery 的 permissions、quotas、billing 等等，比較特別的是還多了個名為 **Dataset** 的分組。

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigquery/organization-resource.jpg" >}}

### Dataset

Dataset 是用來管理和控制 BigQuery 資源，比喻上可以想成是一個資料庫。為什麼可以這樣想呢？因為 BigQuery 資源如 tables、views、functions、procedures 都是建立在 DataSet 中，和傳統資料庫概念上很相像呢。

{{< image classes="fancybox fig-100" src="/images/google-cloud/bigquery/dataset.jpg" >}}

要建立 Dataset 時需指定 Location ，且創建之後**無法更改**，再加上當我們要創建 table 時， table 也是會歸屬在 Dataset 之下的，故需要思考一下 [location requirement](https://cloud.google.com/bigquery/docs/locations#data-locations)。然後 tables 透過相同的 Dataset 名稱管理在一起，在 BigQuery 查詢表格操作上，需要加上 Dataset 名稱來識別資源位置，格式為: `projectname.datasetname`。

### Project

每個 Dataset 都會關聯一個 Project，而 Project 內可以有多個不同 location 的 Dataset ， UI 顯示上 Explorer Panel 內的 Project、Dataset、資料表會呈階層式排列。

BigQuery 的每個執行動作均稱為 Job，而 Job 本身會關聯到一個 Project ，主要是因為 quotas 計費是以 Project 為單位。雖然 Job 本身是連結到一個 Project，但是 Job 在執行動作時的資料 table 來源，可以是來自多個不同 Project 的 Datasets。

{{< alert warning >}}
tables、views、functions、procedures 都是建立在 DataSet 中。**但 Connections 和 jobs 是例外**，這兩個與專案相關聯而不是 DataSet
{{< /alert >}}

### Organization / Folder

Organization 概念上經常表示**一家公司 Company**，雖然不需要 Organization 也可使用 BigQuery，但還是建議創建 Org 用來集中控制 BigQuery 資源 ; 而 Folder 概念上經常表示**公司的部門 Departments**，然後 Folder 可以有多個層次，且其會自動繼承其 parent folder。

如果希望 organization 中的 Departments 使用不同的 Billing accounts，那建議為每個 Departments 開不同 Project ，然後在 organization level 創建 Billing account ，再將它與 Project 關聯。

---

### 參考資料

- [BigQuery overview](https://cloud.google.com/bigquery/docs/introduction)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [A Deep Dive Into Google BigQuery Architecture: How It Works [2024 Updated]](https://panoply.io/data-warehouse-guide/bigquery-architecture/)

- [BigQuery Admin reference guide: Storage internals](https://cloud.google.com/blog/topics/developers-practitioners/bigquery-admin-reference-guide-storage)

- [Inside Capacitor, BigQuery’s next-generation columnar storage format](https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format)

- [Google 數據分析核心技術：BigQuery 深度介紹及初步教學](https://event.livehouse.in/gcp/whitepaper/c85c42ebce9d625faa9c5059cab4c3f2d462b4dccaa2f3647aad832623aa6b4c.pdf)
