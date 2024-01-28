---
title: GCP Cloud CDN

author: Aryido

date: 2023-07-27T23:26:32+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp

comment: false

reward: false
---
<!--BODY-->
> CDN 全名為 Content Delivery Network，是一種透過分散在不同地區的 server，用離使用者**最近的伺服器**來傳送快取內容。而 Google Cloud CDN，就是借助 Google 分佈在**全球各地**的網路節點，將內容以快取(Cache)形式預先儲存，以達到最快速的內容交付。以下是一個 CDN 的全球分布架構，如圖所示歐洲使用者可以從荷蘭的節點獲取資料，而不必透過跨大西洋電纜，到位於美國 host server 拿取資料。
> {{< image classes="fancybox fig-100" src="/images/google-cloud/cdn-global.jpg" >}} **Cloud CDN 會需要與 GCP-Load-Balancer 搭配使用**，故建議可以先熟習 GCP 負載平衡器的基本用法和觀念。
<!--more-->

---

# Cloud CDN 架構及工作原理
{{< image classes="fancybox fig-100" src="/images/google-cloud/cdn.jpg" >}}

user 向 GCP-Load-Balancer 請求時，請求首先會送到盡可能距離使用者最近的 Google Frontend （GFE），這時如果:
- 已啟用 CDN
- 已配置 GCP-Load-Balancer 的 url-map 將流量路由到指定 Google Backend

則 GFE 會使用 Cloud CDN。接下來會看**有無命中緩存**來決定接下來的處理:

> #### Cache hits :
> 如果 GFE 在 Cloud CDN Cache 中查詢到對應 key ，GFE 會**直接從 Cloud CDN Cache 內回應使用者 key 對應的內容**，不需要原始 server 處理請求。

> #### Cache miss :
> 如果 GFE 無法從 Cache 中找到請求所需內容， GFE 會將請求轉發給 GCP-Load-Balancer ，然後再將請求轉發到某 server 處理。

{{< alert success >}}
Cloud CDN 不會執行任何網址重定向。 Cloud CDN  Cache 位於 GFE，故

- **無論是否啟用 Cloud CDN，請求的網址都保持不變**
- **無論 Cache hits or miss，網址都保持不變**

因此在 GCP，開啟 CDN 設定是非常快速方便的。
{{< /alert >}}

---

## Cloud CDN and Terraform
在設定 Google Cloud CDN 時，會啟用幾個的 GCP 服務：

> - Cloud DNS (optional)
> - Load-balancer (require)
> - Cloud CDN  (require)
> - **Backend  (require)**

其中特別注意根據 GCP-Load-Balancer ， **Cloud CDN 也有各種類型的 Backend 可選取** :
{{< image classes="fancybox fig-100" src="/images/google-cloud/cdn-sources.jpg" >}}
這裡我們用 GCS 的 Bucket 來做為 CDN Cache 獲取來源的範例吧 ! 參考 [GoogleCloudPlatform-terraform-large-data-sharing-java-webapp](https://github.com/GoogleCloudPlatform/terraform-large-data-sharing-java-webapp)。 關鍵部分是 :
- 通過在後端存儲桶上設置「enable_cdn=true」屬性來啟用CDN
- 只啟用存儲桶可公開訪問權限，
```Terraform
# ------------------------------------------------------------------------------
# CREATE A STORAGE BUCKET
# ------------------------------------------------------------------------------

resource "google_storage_bucket" "main" {
  project       = var.project_id
  name          = var.name
  location      = var.location
  labels        = var.labels
  force_destroy = true
}

resource "google_storage_default_object_acl" "policy" {
  bucket = google_storage_bucket.main.name
  role_entity = [
    "READER:allUsers", //非常重要 ! 確保每個 user 都有權限取得 bucket 內的檔案
  ]
}

# ------------------------------------------------------------------------------
# CREATE A BACKEND CDN BUCKET
# ------------------------------------------------------------------------------

resource "google_compute_backend_bucket" "cdn" {
  project     = var.project_id
  name        = "lds-cdn-java"
  bucket_name = var.bucket_name
  enable_cdn  = true // 啟動CDN
  cdn_policy {
    cache_mode        = "CACHE_ALL_STATIC"
    client_ttl        = 3600
    default_ttl       = 3600
    max_ttl           = 86400
    negative_caching  = true
    serve_while_stale = 86400
  }
  custom_response_headers = [
    "X-Cache-ID: {cdn_cache_id}",
    "X-Cache-Hit: {cdn_cache_status}",
    "X-Client-Location: {client_region_subdivision}, {client_city}",
    "X-Client-IP-Address: {client_ip_address}"
  ]
}
```

---

# Cloud CDN 優點
使用 CDN 的好處，簡單來說: **優化網頁速度與效能，提供了一種可靠的方式來向使用者提供內容，從而降低 I/O 成本和內容交付延遲。**

- 減少延遲:

  {{< alert info >}}
當使用者與主要提供服務的伺服器不在同一地區時，能夠有效地利用多個 CDN 節點分佈於不同地區的優勢，減少使用者訪問內容時的延遲。
{{< /alert >}}

- 提升負載量:
  {{< alert info >}}
由多個 CDN 處理多位使用者的請求，能接收更多流量以提升負載量。
{{< /alert >}}

- 減少頻寬成本:
  {{< alert info >}}
CDN的快取(Cache)能有效降低主要服務所需提供的資料與流量，從而降低傳輸成本。
{{< /alert >}}

接下來因為 Cloud CDN 與 Google Cloud 相互整合，所以

- 指標分析方便
  {{< alert info >}}
Cloud CDN 與 Cloud Monitoring 還有 Cloud Logging 緊密整合，提供立即可用的詳細延遲指標及原始 HTTP 請求記錄檔，使用戶能深入瞭解實際情況。另外用戶也可將記錄檔匯出至 Cloud Storage 或 BigQuery 進一步分析。
{{< /alert >}}

---

### 參考資料
- [Cloud CDN overview](https://cloud.google.com/cdn/docs/overview#removing_content_from_the_cache)

- [CDN 是什麼？Google Cloud CDN 用途、架構完整介紹](https://blog.cloud-ace.tw/networking-website/cdn/cloud-cdn-intro/)

- [使用 Terraform 配置 Google Cloud CDN](https://medium.com/cognite/configuring-google-cloud-cdn-with-terraform-ab65bb0456a9)

- [Serving Assets via CDN with Google Cloud](https://hackersandslackers.com/cdn-google-cloud/)

