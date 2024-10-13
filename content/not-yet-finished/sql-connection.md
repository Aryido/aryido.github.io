---
title: GCP - Cloud SQL 連線

author: Aryido

date: 2024-08-03T23:33:55+08:00

thumbnailImage: "/images/google-cloud/sql/sql-logo.jpg"

categories:
  - cloud
  - gcp

tags:

comment: false

reward: false
---

<!--BODY-->



<!--more-->

---


# Cloud SQL 的連線

Cloud SQL 開起來之後，因為它概念上並不是給一個**裝了 DB 的主機**讓你操控，故沒有 SSH 連進去的功能，我們沒有辦法直接進入到它的 VM 底層。在考慮如何連接到 Cloud SQL Instance 時有很多方式和其優缺點：

- ##### 是否讓 DB 可以從 public internet 連線，還是只要使用 private VPC
  - Internal: VPC-only (Private) IP address
  - External: internet-accessible (Public) IP address
  
  {{< alert warning >}}
Cloud SQL Instance 可以同時具有 External 和 Internal 位址。
{{< /alert >}}

- ##### 是打算寫自己的 connection code，還是使用例如其他連接工具
  Database connections 都會消耗資源，故如需自己寫程式來連線 DB 都要注意**連接管理**，減少超過 Cloud SQL 連接限制的可能。

  - Cloud SQL Auth Proxy 
  - [Database administration](https://cloud.google.com/sql/docs/mysql/introduction#database_administration) 等 DB Client 端產品，比如說常見的產品有 `MySQL Workbench`、`SQL Server SSMS` 等等，可以來觀看 DB 資訊。

  目前很不錯解決方案是使用  [Cloud SQL Auth Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy) 進行連線，此解決方案提供 IAM permissions 使用 service account 的 credential 來連線。 Cloud SQL Auth Proxy 是 GCP 官方提供且推薦的認證連線方式。

- ##### Authenticate
  單純 Granting access 給 application 並**不會**自動啟用 user account 使之可以連線 DB ，還是需要 configure default user account
  - Built-in database authentication 使用 `username/password`
  - 是否要求 SSL/TLS 進行加密，或者允許未加密連線
  

{{< image classes="fancybox fig-100" src="/images/google-cloud/sql/cloudsql-auth-proxy.jpg" >}}


---

### 參考資料

- [Cloud SQL overview](https://cloud.google.com/sql/docs/introduction)

- [Database Configurations with Google Cloud SQL](https://www.youtube.com/watch?v=q6noaMAnk5s)

- [PostgreSQL 與 MySQL 適用的雲端資料庫 – Cloud SQL 介紹](https://blog.cloud-ace.tw/database/cloud-sql-intro/)

- [雲端資料庫教學：以 Cloud SQL for MySQL 實現自動分流](https://blog.cloud-ace.tw/database/cloud-sql-for-mysql-tutorial/)

- [【Explanation】GCP 新手村 — CloudSQL ](https://medium.com/@kellenjohn175/how-to-guides-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-cloudsql-27835f93a2e7)
