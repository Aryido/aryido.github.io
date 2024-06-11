---
title: "Elasticsearch 基礎介紹"

author: Aryido

date: 2024-01-21T22:47:34+08:00

thumbnailImage: "/images/monitoring/elasticsearch.jpg"

categories:
- Monitoring

comment: false

reward: false

---
<!--BODY-->
> Elasticsearch 是一款開源的分佈式搜索引擎，也是一個分佈式資料庫。底層是基於 Lucene 開發，另外 ES 來提供了 RESTful API 風格的接口、支持分佈式、可水平擴展。除了進行全文檢索( full-text search )；倒排索引 (Inverted index)，也支持聚合以及排序操作。
<!--more-->

---

# 前言
因為 Elasticsearch 是基於 Lucene ，故先介紹一下 Lucene 的基本知識:
- 基於 java 編寫的，開源的全文檢索引擎
- 原生並不支持水平擴展

# 基礎概念介紹
Elasticsearch 的搜尋方式，可以分為兩種：
- 全文檢索 (full-text search)
- 倒排索引 (Inverted index)



{{< image classes="fancybox fig-100" src="/images/monitoring/es-index-type-doc.jpg" >}}

## index
index 相當於關係型數據庫中的一個Database 或 Table

## type ( Elasticsearch 7 之後廢除 Type 的支持)
在 6.x 之前， index 可以被理解爲關係型數據庫中的相當於關係型數據庫中的一個 Database ；而 type 則可以被認爲是 Database 中的 Table。

使用 type 允許我們在一個 index 裏存儲多種類型的數據。type 的存在從某種程度上可以減少 index 的數量，但是 type 存在以下限制：

- 不同 type 內的 field 需要保持一致。例如一個 index 下的不同 type 內，有兩個名字相同的 field，則它們的類型和配置必須相同。

- 只在某個 type 裏存在的 field，在其他沒有該字段的 type 中也會消耗資源。

- 一個 type 中的文檔會影響另一個 type 中的文檔的得分。

以上限制要求我們，只有同一個 index 的中的 type 都有類似的映射 (mapping) 時，才勉強適用 type 。否則，使用多個 type 可能比使用多個 index 消耗的資源更多。這大概也是爲什麼 ES 決定廢棄 type 這個概念。

{{< alert warning >}}
ES 多個版本可能出現破壞性變更，例如，在 6.x，ES 不允許一個 Index 中出現多個 Type
{{< /alert >}}

{{< alert info >}}
在 Elasticsearch 7 中使用默認的_doc 作爲 type
{{< /alert >}}

## document
index 中的單條記錄稱爲 document ，可以理解爲 Table 中的 row data。 而一個 document 內會包含 :
- _index： document 所屬的 index 名稱。

_type：已默認爲_doc，之後會移除

_id ： document 的主鍵。在寫入的時候，可以指定該 Doc 的 ID 值，如果不指定，則系統自動生成一個唯一的 UUID 值。

_score： 在查詢時 ES 會根據一些規則計算得分，並根據得分進行倒排。除此之外，ES 支持通過 Function score query 在查詢時自定義 score 的計算規則。

- _source： document 的原始資料

## field
一個 document 會由一個或多個 field 組成，field 是 ES 中數據索引的最小定義單位，列舉部分常用的類型。
{{< alert warning >}}
在 ES 中，沒有數組類型，任何字段都可以變成數組。
{{< /alert >}}


| Elasticsearch | MongoDB   | SQL    |
|---------------|-----------|--------|
| index         | collection| table  |
| document      | document  | row    |
| field         | field     | column |
| Inverted index| index     | index  |



## mapping
mapping 是定義 document 的結構，包含的所有 field 信息。









---
### 參考資料

- [Elasticsearch 基礎入門詳文文](https://www.readfog.com/a/1666818628507504640k)

- [Elasticsearch 介紹 – 基本概念篇](https://www.omniwaresoft.com.tw/techcolumn/elastic-techcolumn/elasticsearch-basic-concept/)
