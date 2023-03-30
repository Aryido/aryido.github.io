---
title: Google Cloud Firestore 簡介

author: Aryido

date: 2023-03-27T22:18:10+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp

comment: false

reward: false

---
<!--BODY-->
> Cloud Firestore 是 Google 提供的一款雲端 NoSQL 資料庫。資料模型上簡單分成了 collection 、 document 結構，本質上也是 JSON 格式，但使用起來更直觀。Firestore 還具有即時同步資料的功能，也就是說一旦資料庫有異動，資料便會自動同步到相關的用戶端上（如手機 App）
>
<!--more-->

## Firebase 和 Firestore 的關係
Firebase 是一個平台 BAAS 服務，提供了多種功能如: database 、身份驗證、應用程式部署等等...，併入 Google 後， BAAS 的功能更加地完備了。 而 Firebase 提供比較新的數據庫解決方案就是 Cloud Firestore，即 **Firestore 是 Firebase 平台的一個雲端數據庫服務**。

{{< alert info >}}
Firebase 是一個**平台服務**，有提供兩種數據庫解決方案，支持實時數據同步：
- Firestore (較新)
- Realtime Database
{{< /alert >}}

---

## Firestore

有分成 :
- Native mode
- Datastore mode

目前每個 gcp project ，**只允許一個 Firestore**，強制預設名稱是 **(default)**，注意括號不能省略 !

簡單整理表格 :

|                  | Native Mode                  | Datastore Mode |
|:----------------:|:----------------------------:|:----------:|
| 資料類型         | document、collections        |  entity、kind |
| Datastore v1 API | 不支援                       | 支援   |
| Firestore v1 API | 支援                        | 不支援   |
| 即時更新          | 支援 listen document 的功能 | 不支援   |

{{< alert danger >}}
一個 project 只能選擇一種 mode。
{{< /alert >}}

在 Cloud Firestore 中，存儲的基本單位是 document。其中 :
- collection 內會包含 document
- document 內可以包含 collection 和欄位 fields
- field 只能存在 document 內。

{{< alert success >}}
document 是一個 lightweight record ，可包含欄位fields，也可以嵌套另一個 collection。
{{< /alert >}}

展示範例如下 :
{{< image classes="fancybox fig-100" src="/images/google-cloud/firestore-example.jpg" >}}

---
## 心得
因為每個 project ，只允許一個 Firestore，所以**沒有刪掉整個庫來代替刪除資料**的操作，對於測試有點不太方便；即使關閉 api ，下次 enable api 後，資料仍然會在 Firestore 裡面。另外看了一下 [Firestore CLI](https://cloud.google.com/sdk/gcloud/reference/firestore) ， 會發現能用操作很少，**跟 terraform 的整合甚至奇差無比...**。舉例來說 :

- 用 terraform 建立 Firestore database ，只能在全新乾淨從來沒開過 Firestore 的 project 做。當 terraform destroy 之後，再 terraform apply 想建立 Firestore 就會出問題，會回報 Firestore 已存在。

- terraform 建立其他名稱的 Firestore database 會失敗
