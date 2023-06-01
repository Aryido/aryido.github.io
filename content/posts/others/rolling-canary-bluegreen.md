---
title: "Deployments: Rolling vs Canary vs Blue-Green"

author: Aryido

date: 2023-05-17T00:26:34+08:00

thumbnailImage: "/images/others/devops.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 現今應用程式發展迅速， app 的更新也變得越來越頻繁，在微服務、DevOps、Cloud-native 的迭代過程中，最終都需要上線。上線就需要部署；需要部署就意味著有修改；修改則意味著有風險，要如何在**盡量不影響 user 的前提下，讓 app 升版呢** ? 這時就有一些**部屬策略**可以考慮。對於 Deployment Strategies 有一些基本的專有名詞和觀念，例如 :
> - Recreate
> - Rolling
> - Blue-Green
> - Canary
>
> 對於應該使用哪種 Deployment Strategy 、它們的工作原理、優缺點等等，以下會做些基本介紹。
<!--more-->

---

## Canary Deployment (金絲雀部署)
Canary Deployment 策略，是先將**一小部分流量**路由到**新版本**，而**大部分流量**繼續由**舊版本**提供服務。 如果在 Canary Deployment 期間檢測到問題，可以將**新版本** rollback 回**原始版本**，出問題基本上也只是影響到一小部分 user 或 server 而已。Canary Deployment 可以在**重大更改發佈到整個系統之前進行 prod 測試**，從而最大限度地降低風險，並確保複雜分散式系統中的高可用性。

{{< alert info >}}
Canary Deployment 策略，可允許在 prod 環境中測試新版本。故通常用於需要頻繁更新的 app 。
此策略允許在對生產環境影響最小的情況下，測試新功能，並有助於在問題影響整個系統之前識別問題。
{{< /alert >}}

### 注意事項
- 最後可能不停止舊版本，不同版本應用共存。
- 常常會設置路由權重，例如 90% 的用戶維持使用舊版本，10% 的用戶嚐鮮新版本。
- Canary Deployment 經常與 **A/B test** 一起使用。讓大部分用戶繼續用**A舊版本**，一部分用戶開始用**B新版本**，如果用戶對B新版本，沒有什麼反對意見，就可以逐步擴大B新版本數量。


---

## Blue-Green Deployment (藍綠部署)
Blue-Green Deployment 可最大限度減少風險和停機時間。**正在運行的舊版本是藍色**；**將要上線的新版本是綠色**。
雖然會同時運行兩種版本，但只在測試階段而已，將要上線的新版本基本上並不會給 user看到。**最後將流量從舊版本切換到新版本時，等價於新版本部屬完畢**。

{{< alert info >}}
藍綠部署的階段，需要平時 server 的數量的兩倍。因為涉及運行兩種 app 在相同的環境，一個用作正在提供服務的原始 app，另一個是新版本。新版本在與生產環境切換之前，先經過測試，確保最後切換流量時沒有錯誤。
{{< /alert >}}

### 注意事項
- 當切換到新版本時，需要妥當處理進行到**一半的 business 任務**；特別是在應用之間有 message queue 時，需要注意 data reversion。
- Blue-Green Deployment 對於增量升級有比較好的支持，但是對於涉及 database table 變更等等不可逆轉的升級，並不合適。
- Blue-Green Deployment 在大型的微服務系統很難用，主要是考慮到**服務器成本**，以及改動涉及到的中間件過多。

---

## Rolling Deployment (滾動部署)

Rolling Deployment 是一種以**漸進的方式**部署新版本的策略，在發生錯誤時也可以自動 rollback 。**更新版本創建新 pod 時，同時逐步縮減舊版本 pod** ，可確保服務不會中斷。這種部署方式相對於 Blue-Green Deployment ，更加節約資源，因為不需兩倍 server 資源，可以部分部署，例如每次只取出 20% 進行升級。

{{< alert info >}}
pod 一增一減就是滾動部署的特色 ! 非常適合在部署期間，需要零停機時間的應用程式。逐步推出應用程式的新版本，同時確保舊版本仍在運行，從而降低停機風險，並允許在出現問題時輕鬆回滾。
{{< /alert >}}

### 注意事項
- 要注意一下 rollback 時間。例如需要更新 100 個實例，每次更新 10 個實例，每次部署需要5分鐘。當滾動發佈到第80個實例時，發現了問題，rollback 時間就相當可觀了。
- 如果部署期間，系統自動 scale up or down ，則要注意到底是使用哪個版本。

---

## Strategy 總簡介

#### 重建(Recreate) :
{{< alert success >}}
將 v1 版本下線後讓 v2 版本上線
{{< /alert >}}


#### 滾動部署(Rolling-update) :
{{< alert success >}}
v1 版本緩慢換掉，更新至 v2 版本
{{< /alert >}}


#### 藍綠部署(Blue/Green Deployment):
{{< alert success >}}
先將 v2 版本部屬到正式環境，經過測試後再把流量從 v1 版本切換至 v2 版本
{{< /alert >}}


#### 金絲雀部屬(Canary):
{{< alert success >}}
逐步將正式環境的流量從 v1 版本轉移到 v2 版本
{{< /alert >}}
