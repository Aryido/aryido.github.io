---
title:  GCP - Cloud CDN 概述

author: Aryido

#date: 2023-07-27T23:26:32+08:00
date: 2024-06-24T23:26:00+08:00


thumbnailImage: "/images/google-cloud/lb/cdn/cdn-logo.jpg"

categories:
- cloud
- gcp

tags:
- load-balancer

comment: false

reward: false
---
<!--BODY-->
> CDN 全名為 Content Delivery Network，是一種透過分散在不同地區的 server，用離使用者**最近的伺服器**來傳送快取內容。而 Google Cloud CDN，就是借助 Google 分佈在**全球各地**的網路節點，將內容以快取(Cache)形式預先儲存，以達到最快速的內容交付。以下是一個 CDN 的全球分布架構，如圖所示歐洲使用者可以從荷蘭的節點獲取資料，而不必透過跨大西洋電纜，到位於美國 host server 拿取資料。
> {{< image classes="fancybox fig-100" src="/images/google-cloud/lb/cdn/cdn-global.jpg" >}} **Cloud CDN 會需要與 GCP-Load-Balancer 搭配使用**，故建議可以先熟習 GCP 負載平衡器的基本用法和觀念。
<!--more-->

---

# 架構及工作原理
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/cdn/cdn-architecture.jpg" >}}

user 向 GCP-Load-Balancer 請求時，請求首先會送到盡可能距離使用者最近的 Google Frontend （GFE），這時如果有做以下設置:
- 已啟用 CDN
- 已配置 GCP-Load-Balancer 的 url-map 將流量路由到指定 Google Backend

則 GFE 會使用 Cloud CDN。接下來會看**有無命中緩存**來決定接下來的處理:

- ##### Cache hits :

  如果 GFE 在 Cloud CDN Cache 中查詢到對應 key ，GFE 會**直接從 Cloud CDN Cache 內回應使用者 key 對應的內容**，不需要原始 server 處理請求。

- ##### Cache miss :

  如果 GFE 無法從 Cache 中找到請求所需內容， GFE 會將請求轉發給 GCP-Load-Balancer ，然後再將請求轉發到某 server 處理。

{{< alert success >}}
Cloud CDN 不會執行任何網址重定向。 Cloud CDN  Cache 位於 GFE，故

- **無論是否啟用 Cloud CDN，請求的網址都保持不變**
- **無論 Cache hits or miss，網址都保持不變**

因此在 GCP，開啟 CDN 設定是非常快速方便的。
{{< /alert >}}

# Cloud CDN Backends
在設定 Google Cloud CDN 時，會需要啟用 Cloud Load Balancing 服務 (require)，其中特別注意**Cloud CDN 也有各種類型的 Load-Balancer-Backend 可選取**來當作快取的儲存位置，CDN 內容可以來自以下類型的 Backends:
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/cdn/cdn-backends.jpg" >}}
於 CDN 的 backend，這裡特別介紹下 Internet Network Endpoint Groups（NEGs) for external backends

### Internet NEGs with external backends
基於 Network Endpoint Group 的官網介紹，我們知道有了 NEG ，則 Google Cloud load balancers 可以提供基於 VM 的 instance group 、 serverless 和 containerized 等等以上 **workloads** 層面的負載平衡功能，也就是說 NEG 提供了更 granular level 、更精細的層級來分配流量，如下圖所示 : 
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb/cdn/neg-lb.jpg" >}}
Network Endpoint Group 提供 GKE 分散流量到 **Pod-level**，而不只是一般的 **VM-level**。

> 那 Internet NEGs with external backends 和 CDN 有什麼關係呢？

因為蠻常 CDN 的存儲位置可能位於地端自有機房，或者是在其他的雲端平台如 AWS 等等，這時就需要設定外部網路端點，而 Cloud Load Balancing 支援將流量代理到在 Google Cloud 之外的外部 backend ，這時就是用 NEG，而整體結構大概可以參考下圖：
{{< image classes="fancybox fig-100" src="/images/google-cloud/dns-cdn/internet-neg-cdn.jpg" >}}
這樣 Cloud CDN 就可以透過設定的 NEG 輕鬆支援外部來源的快取。

{{< alert info >}}
這功能非常的方便，因為不用把所有的資料，都放到 GCP 雲端上面，放在地端也沒關係，一樣可以使用快取功能。
{{< /alert >}}

### Cloud CDN Terraform

接下來我們用比較簡單的 GCS 的 Bucket 來做為 CDN Load-Balancer-Backend 的範例，參考 [GoogleCloudPlatform-terraform-large-data-sharing-java-webapp](https://github.com/GoogleCloudPlatform/terraform-large-data-sharing-java-webapp)。 關鍵部分是 :
- 通過在後端存儲桶上設置```enable_cdn = true```來啟用CDN
- 需要公開 backend 存儲桶的**訪問權限**
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

# 總結

##### 減少延遲、提升負載量、減少頻寬成本，指標分析也方便

當使用者與主要提供服務的伺服器不在同一地區時，能夠有效地利用多個 CDN 節點分佈於不同地區的優勢，減少使用者訪問內容時的延遲 ; 並且由多個 CDN 能處理更多 request，故能接收更多流量以提升負載量 ; 最後 CDN 的快取(Cache)能有效降低主要 server 流量，從而降低主機成本。


因為 Google Cloud CDN 是 Google Cloud 提供的功能，故和其他雲端服務整合的也很不錯，所以還有一些額外的好處： **Cloud CDN 與 Cloud Monitoring 還有 Cloud Logging 緊密整合**。提供可用的詳細延遲指標及原始 HTTP 請求記錄檔。另外 user 也可將 monitoring data 匯出至 Cloud Storage 或 BigQuery 進一步分析。


---

### 參考資料
- [Cloud CDN overview](https://cloud.google.com/cdn/docs/overview#removing_content_from_the_cache)

- [CDN 是什麼？Google Cloud CDN 用途、架構完整介紹](https://blog.cloud-ace.tw/networking-website/cdn/cloud-cdn-intro/)

- [使用 Terraform 配置 Google Cloud CDN](https://medium.com/cognite/configuring-google-cloud-cdn-with-terraform-ab65bb0456a9)

- [Serving Assets via CDN with Google Cloud](https://hackersandslackers.com/cdn-google-cloud/)

- [Cloud CDN 連接外部來源 Custom Origin](https://ikala.cloud/cloud-cdn-custom-origin/)

- [什麼是負載平衡？原理、6大 GCP Load Balancer 完整介紹](https://blog.cloud-ace.tw/networking-website/load-balance/gcp-load-balancer-introduction/)