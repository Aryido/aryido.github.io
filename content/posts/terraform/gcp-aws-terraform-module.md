---
title: GCP load-balancer 的 terraform Module

author: Aryido

date: 2022-10-29T23:17:25+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- cloud

tags:
- aws
- gcp

comment: false

reward: false
---
<!--BODY-->
> 目前使用 AWS 和 GCP terraform module 的感想，其實我覺得都還可以。但這邊特別覺得 GCP load-balancer module，我個人感覺寫得真的不好，有很多地方應該可以寫得更好，讓使用者體驗更棒的，但他們並沒做到...，也讓我思考了其實一昧 module 化是否有必要呢 ? 讓我列出來一些我簡單比較和缺點吧。

<!--more-->

---

## AWS
{{< image classes="fancybox fig-100" src="/images/aws/alb.jpg" >}}
最簡單需要的 terraform resource 有:
1. Listener
2. Target group
3. health-check
4. auto-scaling group

{{< alert success >}}
**共 4 個**
{{< /alert >}}

## GCP
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb.jpg" >}}
最簡單需要的 terraform resource 有:
1. forwarding rule
2. Target proxy
3. URL map
4. backend service
5. health-check
6. Bankends
7. instance group manager
{{< alert success >}}
**共 7 個**
{{< /alert >}}

---

# 功能對比
GCP 的 terraform resource 數量比 AWS 多一些，但這我覺得是設計差異罷了，畢竟 GCP 網路層的設計和 AWS 非常不一樣，這邊並沒有優劣之分。

| AWS      | GCP      |
| -------- | -------- |
| **Listener** | **forwarding rule** + **Target proxy** + **URL map** = **frontend** |
| **health-check** | **health-check**     |
| **Target group** + **auto-scaling group** | **instance group manager** +  **Backend service** + **Bankends**|

---

# Terraform 和 雲平臺 console 對應關係
這邊我自己感覺，對於初學者會點 GCP console 的 load-balancer 設定後，要讓他直接上手寫 load-balancer 的 GCP terraform 還有點難度。

舉例來說， GCP console 中取了 Load-balancer 名字，但它對應到 terraform 哪個 resource 呢? 文件我看了一下似乎沒有寫，只能手動測看看。下圖左邊是 GCP console ，右邊是 terraform 簡單的架構
{{< image classes="fancybox fig-100" src="/images/google-cloud/lb-console-and-tf.jpg" >}}
以 GCP console 來看 他很貼心地幫我們分類了一下，但面對底層 API ， GCP terraform 沒有直接一個叫做 Frontend 的 resource , 所以相關功能對應要多看一下文件。

---

# GCP module 部分參數冗長
某些 GCP terraform module 使用後反而會讓你覺得 code 更加冗餘了!? 這方面其實也有一些人有提出來，像是 [health-check](https://github.com/terraform-google-modules/terraform-google-vm/issues/153)，在使用 module 時，反而需要填更多參數。

這部分在[Load Balancer Terraform Module](https://github.com/terraform-google-modules/terraform-google-lb-http)時，更加災難的發生...。
```
module "gce-lb-http" {
  source            = "GoogleCloudPlatform/lb-http/google"
  version           = "~> 4.4"

  project           = "my-project-id"
  name              = "group-http-lb"
  backends = {
    default = {
      description                     = null
      protocol                        = "HTTP"
      port                            = var.service_port
      port_name                       = var.service_port_name
      timeout_sec                     = 10
      enable_cdn                      = false
      custom_request_headers          = null
      custom_response_headers         = null
      security_policy                 = null

      connection_draining_timeout_sec = null
      session_affinity                = null
      affinity_cookie_ttl_sec         = null

      health_check = {
        check_interval_sec  = null
        timeout_sec         = null
        healthy_threshold   = null
        unhealthy_threshold = null
        request_path        = "/"
        port                = var.service_port
        host                = null
        logging             = null
      }

      log_config = {
        enable = true
        sample_rate = 1.0
      }

      groups = [
        {
          group                        = var.backend
          balancing_mode               = null
          capacity_scaler              = null
          description                  = null
          max_connections              = null
          max_connections_per_instance = null
          max_connections_per_endpoint = null
          max_rate                     = null
          max_rate_per_instance        = null
          max_rate_per_endpoint        = null
          max_utilization              = null
        },
      ]

      iap_config = {
        enable               = false
        oauth2_client_id     = null
        oauth2_client_secret = null
      }
    }
  }
}
```
以上我們看到的那些 **null** 參數，**都不能拿掉**，會報錯誤；或是我們希望自建 health-check resource，再傳入到這個 module 裡，似乎也是無法辦到。以上大概就是我會直接放棄使用這個 module 的原因。

---

# module 間功能重複
例如 GCP Load Balancer module 和 img module 整合沒很好，有功能重複。

Load Balancer module 必填 health_check ，會把 health_check 建出來； 但 img module 也可以建 health_check 。如果從 img module 建出 health_check ，也無法用 reference 方式，把它關聯給 Load Balancer module。

---

# 其他
文件很多路徑有問題
- [Managed Instance Group 的 readme](https://github.com/terraform-google-modules/terraform-google-vm/tree/master/modules/mig)
- [gcp load-balancer output](https://github.com/terraform-google-modules/terraform-google-lb-http)
