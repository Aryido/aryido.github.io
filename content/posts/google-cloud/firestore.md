---
title: GCP - Firestore 概述

author: Aryido

#date: 2023-03-27T22:18:10+08:00
date: 2024-07-27T18:02:56+08:00

thumbnailImage: "/images/google-cloud/firestore/firestore-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> Firestore 是 Google 提供的一款雲端全代管無伺服器的 **NoSQL 資料庫**，scale out 取向的設計會自動多區域資料複製 replication ，也有強一致性 query 和 transaction 支援。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **DocumentDB**
> - Microsoft Azure : **Cosmos DB**
>
> Firestore 特別的一點是有提供 **realtime listeners 即時監聽器**來同步 Firestore 資料庫和 client apps 之間的資料，同時也有提供 **offline support 離線支援**，也就是說一旦雲端的 Firestore 有異動，資料便會自動同步到用戶端上 ; 另一方面當用戶端無法上網時會先存取資料在自己用戶端上，等到可以上網之後會跟雲端的 Firestore 資料庫互相同步。

<!--more-->

---

# 歷史

在查詢 Firestore 相關資料或教學時，會發現很多名詞例如：「 Firebase 」、「 Realtime Database 」、「 Datastore」等等經常出現，那這些是什麼呢？有什麼關係呢？ 以下來簡單說明一下：

- ### Firebase 和 Firestore 的關係

  Firebase 原本是一間獨立的**公司**，提供 mobile application 平台服務 BaaS (Backend as a Service)，支援很多手機開發的應用，而當時 Firebase 平台服務的資料庫是「 Realtime Database 」。後續 Firebase 公司被 Google 收購，而 Firebase 併入 Google 後兩邊一起對「 Realtime Database 」做了改進並推出 Firestore ，所以 Firebase 提供比較新的數據庫解決方案就是 Firestore 。

  {{< alert info >}}
  Firebase 平台雖然有了 Firestore 資料庫可以選，但其實舊的 Realtime Database 也還是可以使用。如何選擇可以參考[ Firebase 官方說明](https://firebase.google.com/docs/database/rtdb-vs-firestore)，主要只是為了保證兼容性，故仍然推薦優先使用新的服務 Firestore。
  {{< /alert >}}

- ### Datastore 和 Firestore 的關係

  當時 Google 在 GCP 平台上原本也有研發自己的 NoSQL 稱為 「 Datastore 」，但後續 Google 收購了 Firebase 公司，就整合自己的 Datastore 和 Firebase 公司的 Realtime Database ，最後 Datastore 進行了品牌重塑（Rebranding），產生的下一代產品就是 Firestore。
  {{< alert info >}}
  在 GCP Firestore 中可以選擇 Native mode 和 Datastore mode ，而這個 Datastore mode 的存在也比較像是為了兼容舊版的 Datastore，故也推薦優先使用新的 Native mode。
  {{< /alert >}}

- ### 「 Datastore 」和「 Realtime Database 」的關係

  **Datastore 和 Realtime Database 都是 Firestore 的前身**。目前簡單查資料看起來當時 Google 的 Datastore 似乎沒有<即時同步>和<離線模式>這種功能 ; 而 Firebase 的 Realtime Database 當時在 Availability 一定比不上雲供應商的彈性。故兩個合併算是互相補足一些功能，最後合力推出 Firestore ，並且都可以在自己的平台上使用。
  {{< alert success >}}
  Firestore 可由 GCP 或者 Firebase 提供。GCP Console 網頁上可以看到的 Firestore 服務 ; 而 Firebase 中使用的資料庫也可以看到 Firestore
  {{< /alert >}}

以上說明可以統整出一些關鍵：

- Firebase 原本是一間公司，提供的 BaaS 平台名稱也叫 Firebase
- Firebase 並不是資料庫，是一個 BaaS 平台
- Google 的資料庫產品: Datastore 是 Firestore 的前身
- Firebase 的資料庫產品: Realtime Database 也是 Firestore 的前身
- 無論是 Firebase 還是 GCP 平台上，都建議新的開發專案都使用 Cloud Firestore。

# Data Model

{{< image classes="fancybox fig-100" src="/images/google-cloud/firestore/structure-data.jpg" >}}

Firestore 是 document 的 NoSQL 資料庫，資料模型上分成了 collection 、 document 結構。資料存儲在 document 而由 collection 組織管理， document 內還可以包含另一個 subcollection 形成 nested 嵌套結構。

Firestore 中，存儲的基本單位是 document :

- collection 內會包含 document
- document 內可以包含 collection 和欄位 fields
- field 只能存在 document 內。

{{< alert success >}}
document 是一個 lightweight record ，可包含欄位 fields，也可以嵌套另一個 collection。
{{< /alert >}}

展示範例如下 :
{{< image classes="fancybox fig-100" src="/images/google-cloud/firestore/firestore-example.jpg" >}}

# 心得

當年第一次使用 Firestore 是在 `2022 ~ 2023`， Firestore 心得文初次誕生於 `2023-03-27`。那時對 Firestore 這項服務的心得，雖然其他方面沒有深入測試，但從 DevOps 角度來看是真的**蠻糟的**... 因為那時每個 GCP Project ，只允許存在唯一一個 Firestore，對於測試環境的建置來說很不方便；即使把 Firestore Serive API 關閉，下次重新 enable api 後，資料仍然會在 Firestore 裡面。另外那時看了一下 Firestore CLI 官方文件，會發現能用操作很少

> - **沒有刪掉整個庫來代替刪除資料**的操作
> - 一個 project 只能選擇一種 mode

最後跟 terraform 的整合甚至**奇差無比**... 舉例來說 :

> - 用 terraform 建立 Firestore database ，只能在全新乾淨從來沒開過 Firestore 的 project 做。當 terraform destroy 之後，再 terraform apply 想建立 Firestore 就會出問題，會回報 Firestore 已存在。
> - terraform 建立其他名稱的 Firestore database 會失敗，強制預設名稱是 (default)，注意括號不能省略

**2023 年那時候真心覺得雲端全代管無伺服器的 NoSQL 資料庫解決方案不要選 GCP Firestore ，它是我自己使用過 GCP 服務中經驗最讓人不舒服的**。

但 `2024/8` 有機會再回去簡單看一下，發覺那個超級詬病的： **一個 GCP Project ，只允許存在唯一一個 Firestore** 的限制好像已經被移除了，從 GCP Console 上可以看得出來我有成功建置新的 database 如下圖
{{< image classes="fancybox fig-100" src="/images/google-cloud/firestore/multi-database.jpg" >}}
雖然還沒使用 terraform 嘗試再重新建立 Firestore ，但可能已經有修正相關問題了，後續再找時間測試吧。

---

### 參考資料

- [Firestore overview ](https://cloud.google.com/firestore/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [前端工程師的救星，Firebase 的簡介與實作 TodoList](https://medium.com/ho-japan/%E5%89%8D%E7%AB%AF%E4%BA%BA%E7%9A%84%E6%95%91%E6%98%9Ffirebase%E7%9A%84%E7%94%A8%E9%80%94%E8%88%87%E5%AF%A6%E4%BD%9Ctodolist-c7af49fe3104)

- [認識 Cloud Firestore：Cloud Native NoSQL 資料庫完整介紹](https://ikala.cloud/cloud-firestore-cloud-native-nosql-introduction/)

- [Firebase 是什麼＆與 GCP 的差別](https://ithelp.ithome.com.tw/articles/10222962)

- [建置與管理 Google Cloud 的儲存服務](https://jason-kao-blog.medium.com/%E5%BB%BA%E7%BD%AE%E8%88%87%E7%AE%A1%E7%90%86google-cloud%E7%9A%84%E5%84%B2%E5%AD%98%E6%9C%8D%E5%8B%99-2bc15acf2187)

- [Firebase wiki](https://zh.wikipedia.org/zh-tw/Firebase)
