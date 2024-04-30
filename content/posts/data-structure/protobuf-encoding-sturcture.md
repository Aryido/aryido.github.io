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
> 隨者網路傳輸、頻寬與硬體的設備的改善和增強，其實我們傳遞資料量越來越大、越來越複雜，這時我們也不再只是追求能夠將資料互相傳遞完成，而更加要求**即時傳遞**與**接收大量的資料**，故勢必會需要強化序列化即壓縮的的技術。本章節介紹 protobuf 編碼後的 byte array 結構，說明為何它可以壓縮資料，實現高效率。

<!--more-->

---

來看看負十進制值 `-128`
- 以 2’s Complemen 表示
    ```
    -128
    => 128 二進制 `10000000`
    => Complemen 得 `01111111`
    => +1 `10000000 `
    => 此值可以儲存在 1 byte 中
    ```


- 在 XML 或 JSON 中是屬於文字編碼，則需要更多個 bytes。用 UTF-8 編碼，`-128` 需要四個 bytes，每個字元一個 byte
    ```
    '-'、'1'、'2','8' (ASCII 字符)
    = 45、49、50、56 (十進制)
    0x2d、0x31、0x32、0x38 (十六進制)
    = 每個字母至少需要 1 byte，故共 4 bytes
    ```
    {{< alert info >}}
因為 XML 和 JSON 還添加了標記字符，如 `<>` 或者 `{}`，故明顯可知文字編碼的壓縮性明顯低於二進位編碼。
    {{< /alert >}}



目前雖然 JSON、XML 仍在 Web 服務等資料交換中占主導地位，這也是Protobuf 回到二進制編碼系統來優化的原因。

# Encoding 結構
首先定義一個 `.proto`
```
syntax = "proto3";
message Author {
    optional string name      = 1;
    optional int32  age       = 2;
    repeated string interests = 3;
}
```

當一個資料實例如:
```
{
  "author": {
    "name": "henry",
    "age": 27,
    "interests": ["diving"]
  }
}
```

被 protobuf Author Message 編碼後，其每個 field 會被表示成的結構訊息，**粗略表示**大概會像這樣: `(field number, type, payload)`，更進一步說明，Protobuf 實現的資料存儲和傳輸格式，
是以**二進制**格式表示，編碼後所得到的結構，會是如下圖:
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-encode-data.jpg" >}}


呈上結構，我們必須要知道 protobuf 已經做了一些事情和一些名詞意義 :
> 1. 編碼後 Type 會轉換成 WireType
> 2. 承上`(field number, WireType)` 通常會被分為一組來聲明，被稱為 **Tag**
> 3. 會使用 payload 這個名詞，是因為這部分的內容物，可能是 :
>       - 都是被**壓縮**的值
>       - 包含 length 和其被**壓縮**的值
>
>   所以只用比較中性的字眼 payload 來說明避免混淆。

以上這種結構被稱為 **Tag-Length-Value**，簡稱(TLV)。接下來針對 WireType 和 Tag-Length-Value 來說明。

---

# WireType
Protobuf 對 Type 做了專門的分類和編碼，把 `string`、`int64`...等等在 `.proto` 內各式不同的 type，對應成一個 Protobuf 中的 enum，稱為 `WireType`，故每個 `WireType` 有自己的 **field number**。不同的 WireType 對應的 **Payload** 格式不同，可以通過序列化的 WireType 的值，知道後續的 data **儲存方式**。

{{< alert success >}}
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

對於不同 WireType ，**編碼後的<結構>也不一樣**，故根據 WireType 可以把 protobuf 的 **Payload** 格式分成以下幾類:
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

從上述的介紹，應該有發現一件特別的事情，.proto 中定義的 Message Author 內所有**key名稱**，也就是 name、age、interests，**都不在 T-L-V 內**。protobuf 根本沒有把這些東西編碼進去 byte array 內! 而是僅僅使用了 ```field number + WireType```，故序列化-反序列化時都需要參考 .proto 檔案。

{{< alert warning >}}
這一點相比我們熟悉的 json 很不一樣，因為 json 是會把 key 訊息傳送出去的。
所以對於 protobuf 來說沒有 .proto 文件是無法序列化-反序列化成功的。但省略**key名稱**，加上緊湊，除了壓縮儲存空間，還可以讓編碼解碼速度更快。
{{< /alert >}}

另外從圖中也可發現這樣**一堆 T-L-V 緊湊串在一起**的結構，這種存儲方式還有個優點，就是**不需要分隔符就能分隔開字段**，減少了分隔符的使用。

---
### 參考資料

- [Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3/)

- [protobuf 是怎麼序列化的](https://www.readfog.com/a/1668729653630701568)

- [深入 ProtoBuf - 编码](https://www.jianshu.com/p/73c9ed3a4877)

- [如何使用 Protobuf 做数据交换](https://linux.cn/article-11600-1.html)
