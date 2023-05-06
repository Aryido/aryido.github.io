---
title: "Protobuf 序列化詳解"

author: Aryido

date: 2023-05-02T23:10:23+08:00

thumbnailImage: "/images/data-structure/protobuf.jpg"

categories:
- data-structure

comment: false

reward: false
---
<!--BODY-->
> 要了解 Protobuf 的編碼和序列化知識，首先需要了解一些儲備知識點。
> - Varint 編碼
> - Zigzag 編碼
> - Wire Type 類型
> - T-L-V 儲存方式
>
> 可以更加理解 protobuf。
>
<!--more-->

---

Protobuf 是通過 Varint 和 Zigzag 編碼，大大減少了 field 值佔用 byte 。以下簡單說明介紹一下這兩種編碼 :

## Varint 壓縮編碼

Varint 編碼原理是**用一個或多個 byte 序列化整數來表示數字**，例如值越小的數字，使用越少的 byte 數表示。因此，可以通過**減少表示數字的字節數**，來進行資料壓縮。 Varint 編碼後每個 byte 的**最高位**都有特殊含義：
-  1: 表示後續的 byte 也是數字的一部分
-  0: 表示本 byte 是最後一個字節，剩餘 7 位都用來表示數字。

{{< alert info >}}
每個字節的最高一位只是標識位，沒有具體數字含義。

在實際場景中小數字的使用率遠遠多於大數字，因此通過Varint編碼對於大部分場景都可以起到很好的壓縮效果。
{{< /alert >}}

例如:
- 數字 1，只需要一個 byte 就可以表示 : ```[0]000 0001```
- 數字 300 ，會比較複雜，以下推導:
    ```
    300
    = 256 + 32 + 8 + 4
    = 0000 0001 0010 1100 (二進制)
    =  0000010  0101100 （每7位代表一個字節）
    =  0101100  0000010 （低位在前，故交換）
    = `1`0101100 `0`0000010 （第8位按規則補 1 或 0）
    ```

## Zigzag 編碼

Zigazg 編碼的出現，是為了解決 Varint 對**負數**編碼效率低的問題。由於負數的補碼表示法很大（最高位是符號位爲 1），直接使用 Varint 編碼，會佔用較多的 byte 。這種情況使用 ZigZag 編碼，先轉換成比較小的正數，再使用 Varint 編碼，這樣最終生成的 data 會佔用較少的 byte 。

簡單來說， Zigazg 編碼原理對於負數，是使用 :
- 補碼的最高位，**移位**
- 符號位保持不變，剩下位**取反**

這樣就能對負數更好地進行壓縮。詳細暫時略~

{{< alert info >}}
如果提前知道 field 值是可能取負數的時候，可採用 sint32/sint64 類型，不要使用 int32/int64。因為採用 sint32/sint64 數據類型表示負數時，會先採用 Zigzag 編碼再採用 Varint 編碼，從而更加有效壓縮數據。
{{< /alert >}}

## Wire Type
Wire Type 是 google 爲 protobuf 專門定義的類型，用來生成 Tag，指定某個欄位的值在序列化時所使用的編碼方式，以下是 WireType 定義 :

```
enum WireType {
    WIRETYPE_VARINT = 0,
    WIRETYPE_FIXED64 = 1,
    WIRETYPE_LENGTH_DELIMITED = 2,
    WIRETYPE_START_GROUP = 3,   //deprecated
    WIRETYPE_END_GROUP = 4,     //deprecated
    WIRETYPE_FIXED32 = 5
};
```

| WireType | 定義 | 編碼方式 |
| -------- | ---- | -------- |
| 0        | Varint | 值被編碼成可變長度的 byte array |
| 1        | Fixed64 | 固定 64 位元的二進制 byte array |
| 2        | Length-delimited | byte array 前綴加上其長度，用來序列化字符串、嵌套的 message 或二進制數據 |
| 3        | Start group | 已廢棄|
| 4        | End group | 已廢棄 |
| 5        | Fixed32 | 固定 32 位元的二進制 byte array |


每個 Wire Type 可以對應多個具體的 data 類型，例如一個使用 Varint 編碼的欄位可以是一個 int32、int64、uint32、uint64、bool、enum、sint32 或 sint64。在解析 Protobuf data時，序列化和反序列化都是基於 proto 檔的，所以可以明確知道 type ，故會根據欄位的 WireType 以及其定義的類型來進行解碼。
{{< alert success >}}
通過不同的Wire Type使用不同的儲存方式，來最大化的節省空間。
{{< /alert >}}

{{< alert info >}}
因爲每個 field 的起始 byte array中，都會有一個 3 個 bit 位，這是代表 field 的 type。

為甚麼 3 個 bit 就足夠了呢 ? 因為 WireType 只有 6 種
{{< /alert >}}


## T-L-V 儲存方式
為 Tag - Length - Value 的簡稱，其原理是以**標籤-長度-值**，來表示單個 data ，最終將所有 data 拼接成一個 byte array，從而實現資料存儲的功能。

### Tag

Tag 就是，```tag 號碼 + Wire Type``` 的 Varint 編碼格式。最少 1 字節。
- 因爲 Wire Type 只有 6 個定義，所以使用 3bit 完全夠用，Tag 的最低三位表示 Wire Type
- 高位表示 tag 號碼，佔用的長度，取決於號碼的大小。

### Length
Length 是可選的。只有 Wire Type = 2 時，才需要 Length。例如 Varint 編碼就不需要存儲 Length，此時簡化爲 T-V 存儲方式。

### Value
特別說明，有一些有嵌套的 data，T-L-V 表示，其實就是在 Value 內，放一個 T-L-V 形成嵌套模式。


T-L-V 存儲方式的優點是:
- 不需要分隔符就能隔開 field ，各個 field 存儲得非常緊湊，存儲空間利用率非常高。
- 如果某個 field 沒有被設置值，那麼該 field 在序列化時的是完全不存在的，相應 field 在解碼時會被設置爲 default 值。


---

##  proto3 和 proto2 差別
主要差別是 proto3 通過 packed 來修飾 repeat 字段。 proto2在數字類型，例如 int32、int64 等等的 repeated field 沒有高效地編碼。

沒有 packed 的 repeat 數字，標瑪模式都是 Tag-Value 接續 Tag-Value 持續下去，但其實它們有着一樣的 tag 號碼和 Wire Type，所以可以巧妙的使用 T-L-V 存儲方式，Length 代表 Value 列表的**總字節長度**，不需要每個 Value 前面放個相同的 Tag，極大的節省了空間。
