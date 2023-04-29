---
title: "Avro"

author: Aryido

date: 2023-04-23T15:27:54+08:00

thumbnailImage: "/images/data-structure/avro-logo.jpg"

categories:
- data-structure

comment: false

reward: false
---
<!--BODY-->
> Avro 是一個 data serialization system ，是 Apache Hadoop 下的一個子項目，為跨多種程式語言和平台的**傳輸資料格式**。Avro 可使用 JSON 格式來描述 data structure，並且支持**架構演進**，保持向後、向前的相容性。
> Avro 也提供了編解碼和二進制格式，使得在高吞吐量的應用場僅中非常有用且高效。
>

<!--more-->

---

## Avro schema

Avro 序列化首先要先定義 avsc file ，通過 Schema 來定義 data structure 可增加通用性，目前 Java、GO 語言等等，都可以很方便的編譯生成 Avro 。以下是一個簡單  Avro schema 範例 :

```json
{
    "type": "record",
    "name": "Person",
    "fields": [
        {"name": "userName",        "type": "string"},
        {"name": "favouriteNumber", "type": ["null", "long"]},
        {"name": "interests",       "type": {"type": "array", "items": "string"}}
    ]
}
```

{{< alert success >}}
如果知道 protobuf 的話，這裡可以發現定義的 Avro schema 格式，並沒有在  其中使用**欄位標籤 (field tags)**
{{< /alert >}}

接下來拿個實例來分析一下:

``` json
{
    "userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
{{< image classes="fancybox fig-100" src="/images/data-structure/avro-encode.jpg" >}}

可以發現 encoded 後，關於 userName 各項其他資訊，只知道長度是 6，且後面是 UTF-8。並且在 encoded 中沒有任何其他資訊說明是一個 string。從 byte streaming 的觀點，並不知道 userName 會是甚麼型別，可能是一個整數或者完全是其他的東西。

所以能解析這個 byte streaming 的唯一方法是**通過與 schema 一起比對**。也就是說在 decoding 資料時，必須使用與寫資料時相同的 schema ，任何不匹配的欄位都有可能會造成 decoding 時不正確。

{{< alert warning >}}
這邊和 Protobuf 蠻不一樣的。 Protobuf encode 後，byte streaming 有 tag 和 type 等資訊。故不用再和 schema 做比對。
{{< /alert >}}

---

## Avro Schema 演變

Avro 提供了高效、跨語言、使用 schema 的序列化機制，但如果 schema 發生變化會怎樣? 他們如何解決情況呢 ?

故 Avro 產生了 reader 和 writer 模式兩種模式。
資料寫入與讀取的 schema 可以不同，這讓 schema 可以一直演變。

{{< image classes="fancybox fig-100" src="/images/data-structure/avro-readwrite.jpg" >}}
這裡可以發現， Avro 不會因為 2 邊欄位的順序不同造成問題，故 Avro 中重新排列 field 是沒有問題的。因為解析器是**按照名字**來匹配讀 field ，這就是爲什麼在 Avro 中不需要標籤號。但因爲 field 是按名稱匹配的，所以**改變 field 的名稱是個很大的問題**。

為了保持**演變前或後的一致性**，規則只能是: **新增**或**刪除**帶有**預設值**的 field

---

## 可擴展性

在某些程式語言，null 是可被允許的預設值，但它不是一個 Avro 用的欄位型態，故要使用另一種支援的方式： **union type 來讓預設值設為 null**。

- Avro encode 中，如果你想 **optional** 一個 field ，可以使用例如 ```union {null, long}```。
 {{< alert info >}}
 Avro 沒有像 Protocol Buffers 那樣的 required 和 optional 的標註，取而代之就是用 union 加預設值來實現。
 {{< /alert >}}

- Avro 只要給能給 default 值的話，可以任意添加 field 。
 {{< alert warning >}}
 默認值是必要的，因為**新 schema 的 reader 解析到用舊 schema 寫的記錄時**，因為舊 schema 並沒有該值，新 schema 的 reader 可以填入 default 值來代替，避免錯誤。(為疊代更新的情況)
 {{< /alert >}}


- 同理，要刪除一個 field，只要它以前有一個默認值的話就可以很方便的刪除。
 {{< alert warning >}}
 因為當要**使用舊 schema 的 reader 解析用新 schema 寫的記錄時**，因為新 schema 把該值刪除了，舊 schema 的 reader 可以填入 default 值來代替空。(為 rollback 的情況)
 {{< /alert >}}

承前的講解，如果某 field 沒有 default 值，以後將不能刪除該 field ; 如果要新增 field，定義 default 值，會較容易更新疊代和 rollback。

{{< alert info >}}
所以有種建議，是 avro schema 如果可能的話，**所有 field 都給默認值是可以的**
{{< /alert >}}

{{< alert success >}}
因為 Avro 是 java 寫的，故基本上 Java 能夠支持的 data structure 即爲 Avro 所支持的 data structure
{{< /alert >}}


前面有提到改變 field name 是有點麻煩的。但改變 field name 也不是不行，但是有點麻煩且解法 tricky。**需要把 reader 的 schema 改變後欄位名取 alias name (別名)**
{{< alert danger >}}
向後兼容沒問題，但向前兼容就無法兼顧了因為，舊 reader 遇到不認識的 schema 會忽略。
{{< /alert >}}

---
