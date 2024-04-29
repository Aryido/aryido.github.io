---
title: "Protobuf - Encoding 結構介紹"

author: Aryido

date: 2024-04-28T17:56:27+08:00
#date: 2023-04-28T22:49:35+08:00

thumbnailImage: "/images/data-structure/protobuf.jpg"

categories:
- data-structure

tag:
- data-exchange

comment: false

reward: false
---
<!--BODY-->
> Protobuf ，全稱 Protocol Buffers ，是一種輕量級的資料交換格式，適合高性能，對響應速度有要求的 data 傳輸場景。最初由 Google 開發的「可擴展的序列化資料結構」，現在已成為一個開源項目。Protobuf 的核心思想是**先定義好資料 schema** ，然後可根據平台或語言**生成對應的 code**，使用生成好的 code 就可以讀取或寫入各種資料流。
>
> Profobuf 需要注意的缺點是**為二進制格式**，故 data 本身不具有可讀性，需要反序列化後才能看得懂資料內容。

<!--more-->

---

Protocol Buffers 做為一種**資料交換格式**，主要有兩個面相：

- 定義了一種介面描述語言 (Interface Description Language)，用來描述需要交換的資料結構
- 定義了一種 serialization-deserialization 模式

本章節主要說明其中的 Interface Description Language，介紹一些專有名詞。

# Protobuf schema

首先介紹 ```.proto```檔，以下是一個簡單 Protobuf schema 範例 :
{{< alert success >}}
Protobuf 有 proto2 和 proto3 版本，現在一般都使用 proto3，如果不指定 **proto3**，預設使用會 **proto2** ，要寫的話一定要寫在最前面第一行。
{{< /alert >}}
```
syntax = "proto3";
message Author {
    optional string name      = 1;
    optional int32  age       = 2;
    repeated string interests = 3;
}
```
- `optional string name = 1;` 此稱為 **field**
- **field** 內我們有看到 `string`、`int64`...etc，這些稱為 **type**
- **field** 內我們有看到 `optional`、 `repeated`...etc，這些稱為 **label**
- 再來我們看到每個 field 都有定義一個數字，我們稱 **field number**，它的作用是在二進制格式中，用來識別（identify）欄位，一定要 unique


當一個資料實例被 Author Message 編碼後，其結構訊息**粗略表示**大概會像這樣:

`(field number, type, payload)`

接下來還需說明一些事情 :
> - 編碼後 Type 會轉換成 WireType
> - 承上`(field number, WireType)` 通常會被分為一組來聲明，被稱為 **Tag**
> - 會使用 payload 這個名詞，是因為這部分的內容物，可能是 :
>   - 都是被壓縮的值
>   - 包含 length 和其被壓縮的值
>
>   所以只用比較中性的字眼 payload 來說明避免混淆。

以上這種結構被稱為 **Tag-Length-Value**，簡稱(TLV)。接下來針對 WireType 和 Tag-Length-Value 來說明


# WireType
Protobuf 對 Type 做了專門的分類和編碼，對應為 WireType，會把 `string`、`int64`...等等，在 `.proto` 內各式不同的 type，對應成一個 Protobuf 中的 enum，稱為 `Wire Type`，每個 `WireType` 有自己的 **field number**。不同的 WireType 對應的 **Payload** 格式不同，可以通過序列化的 WireType 的值，知道後續的 data **儲存方式**。

{{< alert info >}}
WireType 的作用是告訴解析器 **payload 的格式**，表示這個編碼中，下一個 byte 到底是代表 length 還是 value。
{{< /alert >}}

| WireType | 定義            | 編碼方式                                    | Used For                                    |
| -------- | --------------- | ------------------------------------------- | --------------------------------------------|
| 0        | Varint          | 值被編碼成可變長度的 byte array               | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1        | Fixed64         | 固定 64 位元的二進制 byte array              | fixed64, sfixed64, double                    |
| 2        | Length-delimited| byte array 前綴加上其長度，用來序列化 string、嵌套的 message 或  byte array | string, bytes, embedded messages, packed repeated fields |
| 3        | Start group     | deprecated                                      | group start (deprecated)                     |
| 4        | End group       | deprecated                                      | group end (deprecated)                       |
| 5        | Fixed32         | 固定 32 位元的二進制 byte array              | fixed32, sfixed32, float                     |

對於不同 WireType ，**編碼後的結構也不一樣**，故根據 WireType 可以把 protobuf 的 **Payload** 格式分成以下幾類:
> - Varint/Zigzag `(WireType  = 0)`
> - Value
>   - `(WireType  = 1)`，固定 8 個 bytes
>   - `(WireType  = 5)`，固定 4 個 bytes

以上為 T-V 格式。

> - Length + Value `(WireType  = 2)`，會需要 Length 來記錄 Value 的 bytes 長度，但 Value 也需要區分幾種類型：
>   - String 類型，使用 UTF8 編碼
>   - 嵌套另一個 protobuf Message 類型
>   - 有 repeat label

以上是 T-L-V 格式。無論如何，基本結構如下圖:

{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-structure.jpg" >}}

通過不同的 WireType 會使用不同的 payload 格式，來最大化的節省空間。基本上就分成 T-V 或 T-L-V 兩種。那接下來說一下剛剛一直提到 T-[L]-V 儲存格式吧~

# Tag-Length-Value(TLV)
## Tag
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-tag-structure.jpg" >}}

Tag 由 ```field number + WireType``` 組成，Tag 的最低三位表示 WireType。配合 WireType 表格，也可以知道為什麼 tag 的圖中，WireType 只需要用 3 個 bit，因為 WireType 目前只有 6 種，所以使用 3 個 bit 完全夠用。

所有的 Tag 都是 Varint，其計算公式為
`Tag = (field number << 3) | WireType number`

## Length
Length 是可選的，只有 `WireType = 2` 時，才需要 Length。 而 Length 是可選的原因，是因為看 Tag 內的 WireType 訊息，就能知道 value 的編碼是甚麼。

像是 `WireType = 0, 1, 5` 時，payload 是使用 Varint 編碼，這種編碼方式是自帶 length 的資訊的，故不需要再另外多儲存一個 Length ，可以更加節省空間。

---

# 補充整理

protobuf 編碼後所得到的結構，會是如下圖:
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-encode-data.jpg" >}}

**一堆 T-L-V 緊湊串在一起**的結構，這種存儲方式還有個優點，就是**不需要分隔符就能分隔開字段**，減少了分隔符的使用。


從上述的介紹，應該有發現一件特別的事情，.proto 中定義的 Message Author 內所有**key名稱**，也就是 name、age、interests，**都不在 T-L-V 內**。protobuf 根本沒有把這些東西編碼進去! 而是僅僅使用了 ```field number + WireType```，故編碼解碼時都需要參考 .proto 檔案。

{{< alert warning >}}
這一點相比我們熟悉的 json 很不一樣，所以對於 protobuf 來說沒有 .proto 文件是無法解出來的。但省略**key名稱**，加上緊湊，除了壓縮儲存空間，還可以讓編碼解碼速度更快。
{{< /alert >}}

---
### 參考資料

- [Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3/)

- [protobuf 是怎麼序列化的](https://www.readfog.com/a/1668729653630701568)

- [深入 ProtoBuf - 编码](https://www.jianshu.com/p/73c9ed3a4877)
