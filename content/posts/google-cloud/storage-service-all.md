---
title: GCP 儲存服務 - 總整理

author: Aryido

date: 2024-08-13T23:58:17+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-storage-service

comment: false

reward: false
---

<!--BODY-->

> 在前面幾個章節有介紹了 GCP 常見的 Storage Service ，有提供了各種高可靠且高可擴展存儲服務產品，雲端化直觀的提供了「減輕管理服務基礎架構」的負擔，特別是是硬體角度的管理。從比較 High-Level 角度來討論的話，雲存儲的服務類型分成
>
> - **Block Storage** : Compute-engine 的 Persistent Disk
> - **Object Storage** : [Cloud Storage](https://aryido.github.io/posts/google-cloud/cloud-storage/)
> - **File Storage** : [Filestore](https://aryido.github.io/posts/google-cloud/filestore/)
> - **SQL** : [Cloud SQL](https://aryido.github.io/posts/google-cloud/sql/)、[Cloud Spanner](https://aryido.github.io/posts/google-cloud/spanner/)
> - **NoSQL** : [Firestore](https://aryido.github.io/posts/google-cloud/firestore/)、[Bigtable](https://aryido.github.io/posts/google-cloud/bigtable-1/)、[Memorystore](https://aryido.github.io/posts/google-cloud/memorystore/)
> - **Data Warehouse** : [Bigquery](https://aryido.github.io/posts/google-cloud/bigquery-1/)
>
> {{< image classes="fancybox fig-100" src="/images/google-cloud/storage-service-summary/storage-services.jpg" >}}
> 在現今的雲端計算時代，儲存和管理大量資料變得更加重要，以上這幾種儲存服務需根據業務來選擇，因為這也會決定到訪問和管理組資料的難易程度，故以下對其做一些廣義的整理和筆記來幫助和選擇適合的儲存服務。

<!--more-->

---

# Data Type :

為了正確的選擇適當的儲存技術，通常需要對資料的**結構**和**使用場景**有一定的了解，以下簡單介紹結構化資料 :

> - **Structured 結構化資料** : 如果資料可以以 Column 和 Row 等格式進行組織，則稱為結構化數據。結構化資料主要存儲在 Relational DB 內，對應 GCP 儲存服務為 Cloud SQL 、 Cloud Spanner

> - **Semi-Structured 半結構化資料** : 結構資料通常會被組織成 key-value pairs 、 document 、 wide-column 的形式

> - **Unstructured 非結構化資料** : 它是一個 sequence of bytes 資料，例如 Video 、 圖像檔、聲音檔等等，其資料作為「物件」存儲，最標準的是會使用 Cloud Storage

{{< alert success >}}
Structured 與 Semi-Structured 資料，都會有一些 Schema 設計來定義儲存資料格式，雖然近年來來 Relational DB 與 NoSQL 的界線越來越模糊，但這邊還是簡單介紹分類一下
{{< /alert >}}

- ### 從資料型態概念上出發來做產品選擇

  {{< image classes="fancybox fig-100" src="/images/google-cloud/storage-service-summary/select1.jpg" >}}

# GCP 服務簡介和比較

### Cloud Storage & Filestore & GCP Persistent Disks

從比較物理層面的儲存類型來說的話，儲存類型分成 : **Block Storage**、**Object Storage**、**File Storage** 這三種類型的儲存: 

- ##### Cloud Storage
  Cloud Storage 會在 Bucket 內儲存非結構化的 blob 資料如圖片、影片等，其服務具有高度可擴充性、為完全託管式服務，屬於標準 Object Storage，其特色除了前面提到的 Bucket 管理介面之外，Object 都會有一個 **metadata 元數據** 一起存儲。另外還可以根據自己服務對於**資料存取**特性，選擇保存的類型來 Archive，最佳化存儲成本 : 
  {{< image classes="fancybox fig-100" src="/images/google-cloud/storage-service-summary/storage-class.jpg" >}}
  - **Pros**: 巨大的可擴充性，使用物件存儲，可擴充性幾乎沒有限制
  - **Cons**: 物件的存儲對於**事務**並不理想，且與檔存儲或塊存儲相比，寫入資料的過程稍慢

  {{< alert info >}}
  由於 Block/File storage 成本較高，大量數據且不太會修改資料的話，建議用 Object storage 來存儲
  {{< /alert >}}

- ##### Filestore
  Filestore 顧名思義是對到 File storage，大多數人都會在電腦的使用中熟悉到它例如：創建一個名為「trip」的 folder ，然後在此 folder 下添加另一個名為「animal」的 folder，將旅行遇到喜歡的動物照片放入其中。這種將文件放到「資料夾」和其「子資料夾」的層次結構管理，並使用資料夾/檔案路徑訪問就是 File storage 的特色。雖然日常使用中表現良好，但隨著資料量的增長，性能會下降蠻快的 : 
  - **Pros**: 使用者友好的管理，可輕鬆創建、管理和刪除自己的檔 ; 還提供簡單的訪問管理，以自定義誰可以訪問檔、查看檔和更改檔
  - **Cons**: 檔、資料夾和目錄越多，查找和訪問就越慢

  {{< alert info >}}
  從 Management 來考慮的話 File Storage 最為簡單好用，但它通常不太適合處理大數據存儲，也是這原因讓 Object Storage 逐漸發展起來
  {{< /alert >}}

- ##### GCP Persistent Disks
  Block Storage 是儲存系統的**基礎**，實際上 Object/File storage 都是 Block Storage 上的一個應用，常聽到的 SSD/HDD 就是標準的 Block Storage，直接關聯到 Disk 磁碟讀取，故可以非常快速地檢索/寫入資料。
  {{< alert info >}}
  Block Storage 將資料視為一系列固定大小的 chunks 或 blocks，這些「塊」不需要連續存儲在磁碟上可以分散，每當使用者請求此資料時，底層存儲系統會將塊合併在一起提供。由於有分「塊」的概念，故名稱上就稱作 Block Storage
  {{< /alert >}}
  - **Pros**: 快速的效能
  - **Cons**: 需要為所有分配的存儲空間付費，故價格比較價格昂貴

### Firestore & Bigtable  

Firestore 跟 Bigtable 一樣是屬於 NoSQL 資料庫。 Firestore 它適合儲存相對較小的資料，且價格便宜非常多 ; 而 Bigtable 屬於是企業級 NoSQL 大數據資料庫服務。

|                     | Firestore       | Bigtable                                            |
| ------------------- | --------------- | --------------------------------------------------- |
| **Type**            | NoSQL document  | NoSQL Wide Column                                   |
| **Transactions**    | Yes             | Single-Row                                          |        
| **Capacity**        | Terabytes       | Petabytes                                           |
| **Best for**        | Getting started | Flat data, heavy read/write events, analytical data |

- ##### Firestore
  Firestore 預設是「強一致性的」，以反映數據的最新更新，因為它在設計時考慮了與客戶端之間提供即時數據同步，故適合需要即時更新的 Mobile 和 Web 應用程式。另外 Firestore 它也可以在不需要立即一致性的資料的情況下，提供「終將一致性」，是以靈活地選擇的

- ##### Bigtable
  Bigtable 是一個分散式系統，採用「最終一致性」，它更看重高輸送量和低延遲，而不是即時一致性。另外 Bigtable 也經常配合其他大數據處理工具和框架如 Apache Hadoop 和 Apache Spark 一起使用，用於大規模分析和批處理

Firestore 與 Bigtable 相比，不需要預置資料庫實例（Bigtable 要設定 Node）、也支援 ACID 原子事務，但 Bigtable 強大在於提供更強大的可擴展性和 PB 量級資料的處理。

{{< alert warning >}}
還是額外提一下，Filestore 和 Firestore 名稱和發音都非常相似，但是為完全不一樣的服務。Filestore 是 NFS 儲存類型 ; 而 Firestore 是 Document NoSQL database，注意不要看錯了
{{< /alert >}}

### Cloud SQL & Cloud Spanner & BigQuery

|                     | Cloud SQL              | Cloud Spanner                          | BigQuery                                |
| ------------------- | ---------------------- | -------------------------------------- | --------------------------------------- |
| **Type**            | RDB - OLTP  | RDB - OLTP                  | Warehouse - OLAP                   |
| **Transactions**    | Yes                    | Yes                                    | No                                      |
| **Capacity**        | Up to ~10TB            | Petabytes                              | Petabytes                               |
| **Best for**        | Basic web applications | Very Large-scale database applications | Interactive querying, offline analytics |


Online transaction processing(OLTP) 是指我們常用的關聯式資料庫的**交易處理**Cloud SQL、Cloud Spanner 都可提供該能力。Cloud SQL 是 GCP 中提供的代管 Relational Database 服務，適用於較小資料量 ; 而 Cloud Spanner 是 GCP 下的全球分散式 Relational Database 系統，用於需要高度一致性和可擴展性的大數據資料儲存。 

Online analytical processing（OLAP） 主要針對**分析讀取**進行最佳化，BigQuery 是代表產品。

{{< alert warning >}}
也提一下，Bigtable 和 BigQuery 名稱也是非常相似的，但為完全不一樣的服務。 Bigtable 是企業級 NoSQL 大數據資料庫 ; 而 BigQuery 是 OLAP DataWarehouse
{{< /alert >}}

### 從服務角度上出發來做產品選擇
{{< image classes="fancybox fig-100" src="/images/google-cloud/storage-service-summary/select2.jpg" >}}

---

### 參考資料

- [Google Professional Cloud Architect Day 4 Q/A Review: Storage Services](https://k21academy.com/google-cloud/gcp-training-day-4/)

- [GCP 零基礎入門 (18) - 數據儲存服務總結](https://ithelp.ithome.com.tw/m/articles/10333191)

- [Object Storage vs Block Storage vs File Storage](https://cloud.google.com/discover/object-vs-block-vs-file-storage?hl=en)

- [Google Cloud 儲存選項](https://jayendrapatil.com/google-cloud-storage-options/)
