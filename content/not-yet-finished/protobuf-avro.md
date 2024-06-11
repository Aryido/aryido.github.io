---
title: "Protobuf & Avro"

author: Aryido

date: 2023-09-02T20:05:20+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- data-structure

tag:
- data-exchange

comment: false

reward: false
---
<!--BODY-->
> 在現實情境中，data 的設計總是會不斷變化。故我們都會希望可以變成**能夠快速添加或刪除一些 field**，但不影響太多系統的設計。目前 Protobuf 和 Avro 都支持 schema reversion，它允許你在不同的時間獨立地更新系統的不同組件，而不用擔心兼容性問題。
>

<!--more-->

---

# 緣起
為了進程、程式、伺服器之間通信，人們定義了一批**資料交換格式**以應對不同的場景。過去主流的幾種資料交互的格式，有 :

#### XML
在蠻早期時是很受觀迎的，因為格式定義嚴格清楚，解析起來十分方便。但它也非常冗餘，因爲需要**成對的閉合標籤**。在 json 格式的興起之下逐漸式微。

#### json
由於 XML 的繁瑣冗餘，json 逐漸取代 XML，目前 Web project 最流行的資料交互的格式，主要還是 json；而瀏覽器對於 json 支持也非常好，有很多內建的 function 支持。json 使用了鍵值對的方式，不僅壓縮了一定的空間，同時也具有可讀性。

#### protobuf & avro
這些都是後來興起的

整體來說，在做 data pipeline 時，隨著系統逐漸複雜及高流量，可能會經歷了幾個階段的 :

- 一開始只使用簡單的序列 module 且綁定語言平台，例如 Java serialization，然後想要開始做一些資料分析決定使用 python ，故開始**各自**定義其 JSON 資料格式

- 隨著資料量加大，JSON 有點太冗長，解析傳送起來太慢，決定開始使用二進制。 這時常選擇 Protocol Buffers 或 Avro，他們都提供了高效、跨語言、使用 schema 來定義 data 的序列化，並可以統一生成 code。

---

### 參考資料

- [Avro、Protobuf 和 Thrift 中的模式演變](https://www.readfog.com/a/1664563032555098112)

- [Protobuf 與 JSON 的差異，其帶來的效能提升有多明顯？](https://insights.rytass.com/protobuf-%E8%88%87-json-%E7%9A%84%E5%B7%AE%E7%95%B0-%E5%85%B6%E5%B8%B6%E4%BE%86%E7%9A%84%E6%95%88%E8%83%BD%E6%8F%90%E5%8D%87%E6%9C%89%E5%A4%9A%E6%98%8E%E9%A1%AF-c3e799efa81d)

https://www.syscom.com.tw/ePaper_New_Content.aspx?id=425&EPID=205&TableName=sgEPArticle


https://www.nebula-graph.com.cn/posts/backward-compatibility


https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html