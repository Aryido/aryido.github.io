---
title: "Overview Fargate"

author: Aryido

date: 2022-12-17T16:30:30+08:00

thumbnailImage: "/images/aws/fargate.jpg"

categories:
- aws

comment: false

reward: false
---
<!--BODY-->
> 2017年，AWS 推出了 Fargate，這是一個用於部署和管理容器的**模式**，代表用戶無需管理 EC2 基礎設施。 Fargate 的核心理念就是 **Serverless** ，讓 ECS 用戶專注於任務和服務定義，而不是管理集群基礎設施，Fargate 可配合用於 *ECS* 和 *EKS*。
<!--more-->

---
# 緣起
在過去，我們會需要考慮很多 container 要運行環境及機器，例如在雲端上必須要：
1. 創建一個 VPC
2. 在 VPC 裡創建兩個以上的 public subnet，分別放到 2 個 Availability Zone
3. 創建 EC2 instance 並安裝 Docker 然後拉下 Docker Image，然後運行 Container，一個 EC2 instance 會運行多個 Container
4. 創建 EC2 Auto Scaling Group 和 Load Balancer 將流量導到這些 EC2 Instance

但選擇了 Fargate 作為運行技術，則不需要再顧慮機器，一切交由 aws 管理，**我們甚至不知道 container 運行在哪台機器上**，直白上來說，我們在 aws console 的 EC2 UI 介面，根本沒有機器可以看到( ~~也別去想 SSH 進機器做事情~~ )。

{{< alert success >}}
Fargate 它抽象了基礎設施，使用戶能夠專注於應用程序的期望狀態。
{{< /alert >}}

使用 Fargate，我們只須想好 container 要怎麼定義和運行就好：
- 要用的 Docker Image
- 分配多少記憶體
- 幾顆 CPU
- Network 設定，例如 Container 要向外開哪個 port
- IAM 權限

故 AWS Fargate 被分類到是一種 **serverless 模式**

{{< alert info >}}
Fargate 類型的管理，AWS 保證其執行環境是單獨且隔離的，即 **應用資源並不會與其他在雲端上的客戶共享並被存取**。
{{< /alert >}}

讀過文件後，其實會知道 AWS Fargate 它不算是一種 Service ，它是 Service 的一種類型 type，這種類型可以應用到 ECS 和 EKS 上，代表著**不需要管理機器的集群的理念**。

---

# [AWS Fargate 定價](https://aws.amazon.com/tw/fargate/pricing/)
使用 AWS Fargate 無需預付費用，只需按使用的資源付費，就是對其容器化應用程式 vCPU、記憶體和儲存資源耗用量的費用，簡單計算如下，例如:

1. 使用 5 個 ECS 任務
2. 每個 ECS 任務使用美國東部維吉尼亞的Linux/ARM 定價:
    - 1 個 vCPU : 每 vCPU 秒 0.0000089944 USD
    - 2GB 記憶體 : 每記憶體每秒00000009889 USD
3. 暫時性儲存 10 GB: 每GB每秒 0.0000000308 USD
4. 每天執行 1 個小時
5. 持續執行一個月

### 月 CPU 費用
```
vCPU 總費用 = 任務數 x vCPU 數 x CPU-秒價格 x 每天 CPU 持續時間（秒）x 天數
```
vCPU 總費用 = 5 x 1 x 0.0000089944 x 3,600 x 30 = 4.86 USD

### 月 Memory 費用
```
記憶體總費用 = 任務數 x 記憶體數 (GB) x 記憶體價格 x 每天記憶體持續時間（秒）x 天數
```
記憶體總費用 = 5 x 2 x 0.0000009889x 3600 x 30 = 1.08 USD

### 月度暫時性儲存費用
```
暫時性儲存總計費用 = (任務數) x (額外的暫時性儲存 GB 數) x (每 GB 價格) x (每天記憶體持續時間 (秒)) x (天數)
```
暫時性儲存總計費用 = 5 x 10 x 0.0000000308 x 3600 x 30 = 0.18 USD

### 總共 Fargate 月費用
```
月度 Fargate 計算費用 = 月度 CPU 費用 + 月度記憶體費用 + 每月暫時性儲存費用
```
月度 Fargate 計算費用 = 4.86 USD + 1.08 USD + 0.18 USD = 6.12 USD

---

{{< alert success >}}
使用 Fargate 運行模式，就無需煩惱 :
- 定期需要更新底層的作業系統核心、軟體
- 漏洞修補以確保安全性和合規。

這些細項，一切由 AWS 託管。
{{< /alert >}}