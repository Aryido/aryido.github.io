---
title: GCP - Cloud Storage 概述

author: Aryido

date: 2024-07-21T20:56:57+08:00

thumbnailImage: "/images/google-cloud/gcs/gcs-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-storage-service

comment: false

reward: false
---

<!--BODY-->

> Cloud Storage 簡稱為 GCS ，是 Google 提供的 Object Storage，會把物件儲存在一個名叫 **Bucket** 的容器內，而 Bucket 還可以有類似劃分資料夾的功能稱 **Managed Folders**，可以針對這些資料夾加上個別的 IAM 來提供更精細的存取權限。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **Simple Storage Service (S3)**
> - Microsoft Azure : **Azure Blob Storage**
>
> Cloud Storage 是具有高性能 BLOB（Binary Large Object）儲存裝置，基本上可當成是**近乎無限大的檔案儲存空間**，不用預先配置容量也無需花費心力去管理容量，並提供 User 以 Google Web Console 介面 、SDK 或 RESTful API 的方式，簡單直觀地存取儲存的 Object。

<!--more-->

---

Object Storage 中比較重要的特徵，是**除了檔案本身之外還包含了該檔案的 metadata**。 metadata 的內容包含： 檔案大小或是自己設定的 key-value 等等，而且每一個 Object 可以有自己的 Access control 設定。Cloud Storage 是一個無伺服器應用，它的 API 可以承受全世界用戶達數億次的 request，穩定而且強大，再加上本身權限控管已經足夠精細，可以不需要再用 FTP 去做檔案的分享跟傳輸。

{{< image classes="fancybox fig-100" src="/images/google-cloud/gcs/block-file-object-storage.jpg" >}}

# Bucket

說到 Object Storage 就一定會提到 Bucket，每一個 object 都必須存放在某一個 Bucket 內。在創建 GCP Bucket 時也有些需要注意的地方：

### Bucket Name

GCP 的 Bucket name 必須 **globally unique**，需要在整個 GCP 服務中都不能和其他人公司或使用者重複，原因是 Bucket name 會成為 URL 的一部分，也因此 Bucket name 要符合語法上**有效的 DNS 名稱**，所以名稱有限制 :

- Bucket name 必須以數字或字母來開頭和結尾
- 只能包含英文小寫、數字、破折號（-）、底線（\_）和點（.），且注意包含點（.）的名稱會需要[驗證](https://cloud.google.com/storage/docs/domain-name-verification)，需參考官網說明
- 名稱**不能**有空格
- 名稱先簡單注意長度限制，[限制長度是 3 ～ 63 個字元](https://cloud.google.com/storage/docs/buckets?_gl=1*qzj2l6*_ga*Mjk3MDYwNi4xNzE4MjU5OTM1*_ga_WH2QY8WWF5*MTcyMTk4NjA4MS42My4xLjE3MjE5ODYyMjIuNTYuMC4w#naming)
- 不能有類似 google 的字眼

{{< alert warning >}}
刪除儲存桶後，任何人都可以在新儲存桶中重複使用其名稱，但注意名稱 release 會需要一些時間，有可能會長達數週不會被 release，並不是刪除後就可以馬上建立同樣名稱的 Bucket。
{{< /alert >}}

### Bucket Location

Bucket 的 Location 是在創建存儲桶時就要設置的 建立 Bucket 完成之後，就**無法更改**其 Location 了，如果還是有移動 Location 的需求，解法是新建 Bucket 然後將資料移至這個新建的 Bucket。Location 的類型決定定價的方式，有 「multi-region」、「dual-region」、「region」可以選。

# Storage Class

Storage Class 是一個 metadata 附加在每個 Object 上，依據自身‘需求， GCS 提供不同的儲存方案，此會影響 Object 的 Availability 和定價，可以讓儲存價格最佳化。

建立 Bucket 時，可以在 Bucket 級別上預設 Storage Class ，當 Object 加入 Bucket 時，除非 Object 自身有另外指定 Storage Class ，否則就會使用 Bucket 預設 Storage Class 來儲存。

對於每種 Class 都提供到達 99.999999999%，稱 11 個 9 的 durability。
{{< image classes="fancybox fig-100" src="/images/google-cloud/gcs/storage-class.jpg" >}}
目前最新的 Cloud Storage 功能上還有新增一個新選項為 **Autoclass**，由 GCP 自動推薦 Storage Class。
{{< alert warning >}}
Minium storage duration，意指 object 最少需要存放的天數。例如 nearline storage 最少要放 30 天，如果**沒有放超過時間**就刪除、替換或移動該 object，就要另外付錢
{{< /alert >}}

補充一下 Archive storage，它適用於資料長期歸檔、online backup 和 災難恢復 disaster recovery 的資料備份。 Archive 和 Coldest 都屬於儲存不頻繁訪問的資料，但不同處是 Archive 中的資料可以在毫秒內訪問，因為 disaster recovery 時間是關鍵點， Archive storage 提供資料再需要時可以快速取得。

### [Object Lifecycle Management](https://cloud.google.com/storage/docs/lifecycle)

Cloud Storage 使用時會有一些常見的案例需求，例如

- 為 Object 設置存留時間 Time to Live(TTL)，超過後自動刪除
- 保留 Object 的非當前的 3 個版本，作為簡單的版本控制或留存
- 超過 1 年的 Object 的 Storage Class 自動改為 Coldline 來節省錢

為了支援以上這些案例，Cloud Storage 也提供了 Object Lifecycle Management 功能可以配置。

{{< alert info >}}
Object Lifecycle Management 的依據是完全基於 Object 的創建時間。例如有一些 files 在 Bueckt 中，希望 3 個月後要轉換成 Coldline 儲存，然後在創建之日起一年後自動刪除，則配置為:

- action to 90 days >> Coldline
- action to 365 days >> Delete

**不需要去思考 `365-90=275`** 這種事情。

{{< /alert >}}

# Protect Object Data

有 Data protection 設定，可以客製化設定如： Custom soft delete policy 、 Object Versioning 、Retention 等等，來防止誤刪檔案或用於恢復被覆蓋的 Object ; 再來還有 Data encryption 設定，encryption 進階還可以用 Cloud KMS 來進行 key 的管理。

# Cloud Storage 和[其他雲端產品的結合](https://cloud.google.com/storage/docs/google-integration)

Object Storage 特性上經常是用來當作「 網站靜態圖片」或者「 影片 」的儲存庫使用。舉例來說如果所有的影片都放在一個 Web Server 上，當 user 很多人來看影片的時候，主機 Loading 可能就會不堪負荷。這時常用的解法就會是把影片放到 Cloud Storage，自己 Web Server 就只是簡單 HTML 直接引用 Cloud Storag url，資料是從 Cloud Storage 出去而不是透過自己的 Web Server 再出去，可有效減少主機 Loading，這是一個**主機效能優化**很常見且重要的概念。

簡單上傳資料的話使用 gcloud storage 就足夠了，但若有大量資料傳送需求且需要排程設定，可以參考 [Storage Transfer Service](https://cloud.google.com/storage-transfer/docs/overview)。
{{< image classes="fancybox fig-100" src="/images/google-cloud/gcs/transfer-ways.jpg" >}}
1T 以下的資料用 gcloud storage 就可以了，大於 1T 可以考慮使用 Storage Transfer Service。

- ### Load Balancer & CDN Bucket
  GCS 可以用來儲存靜態網頁，也提供使用 Load Balancer 的掛載方式，讓 GCS 作為 Load Balancer 的 Backend，進階還可以搭配上 Load Balancer 的 CDN 功能來進行 Cache 讓存取更加快速。可以參考 [GoogleCloudPlatform-terraform-large-data-sharing-java-webapp](https://github.com/GoogleCloudPlatform/terraform-large-data-sharing-java-webapp)

---

# [「gcloud-cli」V.S 「gsutil」](https://cloud.google.com/blog/products/storage-data-transfer/new-gcloud-storage-cli-for-your-data-transfers)

先說結論:

> 以使用 gcloud-cli 的 storage 功能為優先，gsutil 並不是 google 官方推薦的 CLI

可以直接從官方網站中 gsutil 頁面看到類似的敘述。

{{< image classes="fancybox fig-100" src="/images/google-cloud/gcs/gsutil.jpg" >}}

因為新的 gcloud storage CLI 提供了顯著的性能改進（參照 title 連結說明），也保持一致的方式來管理所有 Google Cloud 資源，不用因為 cloud storage 而特別使用一個特定 lib 。這邊也特別注意一下，儘管 Google 都已經在很多 GCS 官方文件上都註明一個醒目的警告標語說:

`gsutil 不是 GCS 推薦的 CLI，用戶應該改用 gcloud`

而且相關資訊已經公告從 2022 開始，直到今天也算過蠻久的了，但現在很多 LLM 語言模型還是會推薦首選 gsutil 而不是 gcloud。畢竟 gsutil 從 2016 年就開始發展了，所以至今有太多範例都還是使用 gsutil，因此 LLM 模型都有非常大的機率採用 gsutil 而不是 gcloud，這也是 LLM 現在蠻容易遇到的問題之一...

---

# Practice

> Cloud Storage bucket 中有一個 object 要分享給外部公司，但該 object 包含 sensitive data ，需要在**四小時後移除對該內容的訪問權限**。外部公司沒有 Google 帳戶，因此無法授予基於特定用戶的訪問權限，該怎麼做 ?
>
> - A. Create a signed URL with a four-hour expiration and share the URL with the company. **(O)**
> - B. Set object access to 'public' and use object lifecycle management to remove the object after four hours.
> - C. Configure the storage bucket as a static website and furnish the object's URL to the company. Delete the object from the storage bucket after four hours.
> - D. Create a new Cloud Storage bucket specifically for the external company to access. Copy the object to that bucket. Delete the bucket after four hours have passed.

這題考點是需要知道 Cloud Storage 有 Signed URL 功能，可以創建一個具有四小時過期的 [Signed URL](https://cloud.google.com/storage/docs/access-control/signed-urls)，並將該 URL 分享給外部公司，這是最符合需求的解決方案，因為外部公司沒有 Google 帳戶， Signed URL 也可以設定 `X-Goog-Expires` 過期時間。

B,C 選項都會讓資料 Public 出去，由於資料是 sensitive data 故絕對不能這樣做。 D. 選項其實可能也是一種解法但方式比較複雜，如果有大量外部分享需求，才會需要往這方向做。

---

### 參考資料

- [Product overview of Cloud Storage](https://cloud.google.com/storage/docs/introduction)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [Google Cloud — 資料的儲存與處理服務](https://jason-kao-blog.medium.com/google-cloud-%E8%B3%87%E6%96%99%E5%84%B2%E5%AD%98%E6%9C%8D%E5%8B%99%E7%B0%A1%E4%BB%8B-55dad31811e0)

- [Generative AI Experiments: LLM Knowledge Freshness & GSUTIL vs GCLOUD – The Dangers Of LLMs For Fast-Moving Fields](https://blog.gdeltproject.org/generative-ai-experiments-llm-knowledge-freshness-gsutil-vs-gcloud-the-dangers-of-llms-for-fast-moving-fields/)

- [【最佳實踐】4 種方法，確保 Cloud Storage 中的資料隱私與安全性](https://ikala.cloud/4-ways-to-ensure-privacy-and-security-on-cloud-storage/)

- [Cloud Storage 教學―使用方式、費用節省訣竅完整介紹](https://blog.cloud-ace.tw/infrastructure/data-backup/talk-about-cloud-storage/)
