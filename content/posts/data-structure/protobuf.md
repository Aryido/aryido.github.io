---
title: "Protobuf"

author: Aryido

date: 2023-04-28T22:49:35+08:00

thumbnailImage: "/images/data-structure/protobuf.jpg"

categories:
- data-structure

comment: false

reward: false
---
<!--BODY-->
> Protobuf ，全稱 Protocol Buffers， 是一種輕量級的資料交換格式，適合高性能，對響應速度有要求的 data 傳輸場景。最初由 Google 開發，現在已成為一個開源項目。Protobuf 的核心思想是定義資料格式，然後使用**生成的 code** 在不同的平台上編碼和解碼。缺點 profobuf 是因為二進制格式，data 本身不具有可讀性，需要解碼後才能看得懂資料內容。
>

<!--more-->

---

##  Protobuf schema

想探討一下 Protobuf 實際上是如何將 data 編碼成 byte array ，這也有助於理解 Protobuf 如何處理 revision 的，其中核心就是 ```.proto```檔，以下是一個簡單 Protobuf schema 範例 :

```
syntax = "proto3";
message Person {
    optional string user_name        = 1;
    optional int64  favourite_number = 2;
    repeated string interests        = 3;
}
```
{{< alert success >}}
Protobuf 有不同的版本， proto2 和 proto3 ，現在一般用 proto3 。其中 proto3 已經去掉了 required 這個類型
{{< /alert >}}
接下來拿個實例來分析一下:

``` json
{
    "userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-encode.jpg" >}}

看一下 encode 後二進制表示法結構，可發現整個實例就是用**多個 field 組成**的，每個 field 都有一個 tag，後面會接上 field 的 type。

Protobuf 在解析時，主要都是看 tag 號。如果看到一個在其 schema 中沒有定義的 tag 號，就沒有辦法知道這個 field 名字叫什麼。但是可以知道是什麼 type ，因爲每個 field 的起始 byte 中，都會有一個 3 個 bit 位，這是代表 field 的 type。

{{< alert info >}}
即使解析器不能準確地解析出這個 field ，它也能算出需要跳過多少個 byte，以便找到 record 中的下一個 field。
{{< /alert >}}


例如 userName ，表明該 field 是一個字串，它後面是該字串的長度，這裡表示用了 6 個 byte，然後是該字串的 UTF-8 編碼 ； favouriteNumber 的 type 表明是一個整數，後面是一可變長度的數字編碼； 比較特別的是，interests， 因為沒有 array type ，故這裡用**出現多次 tag**，以代表一個多值 field 。

---

## 可擴展性

- 只要 Protobuf schema 是給一個新的 tag 號，就可以自由添加 field

-  Protobuf schema 中， optional field 、 repeated field 之間，編碼其實沒有區別

可以將 optional 改爲 repeated ；同理 repeated 也可以自由更改成 optional 。
{{< alert warning >}}
解析器如果要的是一個 optional field ，但在一條記錄中多次看到相同的 tag號，它就會**只會保留最後一個值**
{{< /alert >}}

optional field 如果沒有值的話，根本不會出現在編碼中，也可以說是代表該 tag 號的 field 根本不存在編碼中，因此從 schema 中刪除這類 field 是安全的。
{{< alert danger >}}
雖然從 schema 中刪除 optional field 是安全的，但要注意絕對不能在未來，讓另一個 field 不小心使用到已刪除的 optional field 的 tag 號。

因為可能有已經存儲的 data 但它的 schema 是舊的，不小心讓另一個 field 使用重複的標籤號，會造成解析出錯。
{{< /alert >}}


- Protobuf schema 可以簡單的**重新命名 field**

Protobuf 在解析時，主要都是看 tag 號，在二進制序列化中並不存在 field 名稱的標示，在 Protobuf schema 中， field name 其實不太重要。

{{< alert success >}}
Protobuf 每個 field 的區分方法，是用 tag 號來做主要設計。 和 Avro 很不一樣。
{{< /alert >}}

---

## 常見問題

- 可不可以在某個 protobuf 結構的中間加一個 field，然後把後面所有的 field tag 數字都加 1。

答案是不行 ! 會有這樣的想法，大概是把 protobuf 想成跟 json 類似，都是用 field 名來標識 data 的 Key，所以順序和字段標識不重要。**但這是錯誤的**。 protobuf field tag 非常重要，爲了保持兼容性，**定義好之後是不能修改的**。增加新的 field 只能用新的 tag 數字。
