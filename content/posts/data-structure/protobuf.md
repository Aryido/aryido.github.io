---
title: "Protobuf 簡介"

author: Aryido

#date: 2023-04-28T22:49:35+08:00
date: 2024-04-26T00:44:35+08:00

thumbnailImage: "/images/data-structure/protobuf.jpg"

categories:
- data-structure

tag:
- data-exchange

comment: false

reward: false
---
<!--BODY-->
> Protobuf ，全稱 Protocol Buffers ，是一種輕量級的資料交換格式，語⾔中⽴且平台無關，適合高性能，對響應速度有要求的 data 傳輸場景，也因為有**資料壓縮**的能力，故也可用於資料儲存。最初由 Google 開發的「可擴展的序列化資料結構」，現在已成為一個開源項目。 Protobuf 的核心思想是**先定義好資料 schema** ，然後可根據平台或語言**生成對應的 code**，使用生成好的 code 就可以讀取或寫入各種資料流。 Profobuf 需要注意的缺點是**為二進制格式**，故 data 本身不具有可讀性，需要反序列化後才能看得懂資料內容。
>
> Protocol Buffers 雖然是個好東西，但並非是個用來完全取代 JSON 的解決方案，JSON 仍有其可讀性高、易操作及通用性高等優點。在多數 API 設計的場景之下，JSON 仍然是最好的選擇。

<!--more-->

---

Protocol Buffers 做為一種**資料交換格式**，主要有兩個面相：

- 定義了一種介面描述語言 (Interface Description Language，簡稱 IDL)，用來描述需要交換的資料結構
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
    optional int32  age       = 3;
    repeated string interests = 5;

    reserved 2, 4;
    reserved "foo", "bar";
}
```
- 一個 `.proto` 檔案中，可以定義一個或多個message
- `optional string name = 1;` 此稱為 **field**
- **field** 內我們有看到 `optional`、 `repeated`...etc，這些稱為 **label**
- **field** 內我們有看到 `string`、`int64`...etc，這些稱為 **type**
  {{< alert info >}}
  Message 也可以是一個 **type**。Protobuf 也允許一個 Message 可以是另一個 Message 的 type，使得其能夠表達更為複雜的資料結構。
{{< /alert >}}

- 再來我們看到每個 field 都有定義一個數字，我們稱 **field number**，它的作用是在二進制格式中，用來識別（identify）欄位，一定要 unique，最小值為 1

- **Reserved Field**:

  當刪除或註解掉 message 中的一個 field 時，將來其他開發人員在更新 message 定義時，可能不小心重複使用先前的 field number，將會導致嚴重的問題如資料損壞。一種避免問題產生的方式就是使用 Reserved Field，保留 field number 或 field name。 如果將來有人用了，在編譯 proto 的時候，編譯器就會報錯
  {{< alert warning >}}
  注意不要把 field number 和 field name 一起混用，要分開寫。
{{< /alert >}}


# Protobuf 編譯器

一個 `.proto` 檔，提供了一個標準通用的資料結構和格式定義，可以使用 **Protobuf 編譯器**，輕易生成 class類別或 struct結構體，讓我們可以在不同的語言之間，序列化和反序列化相同的結構資料。
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-code-gen.jpg" >}}
```
protoc --version //libprotoc 3.20.1

// 將當前目錄中的所有 .proto 進行編譯以產生 java class
protoc --java_out=./ *.proto
```

# Style Guide of .Proto
- 文件名建議全小寫字母加下底線（lower_snake_case）
- Message、Enums 使用所有單字首字母大寫(PascalCase)
- field name 建議使用全小寫字母加下底線（lower_snake_case）
- Enums 的 field name 使用全大寫加下底線 (CAPITALS_WITH_UNDERSCORES)

```
message SongServerRequest {
  optional string song_name = 1;
}

enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

- string 使用雙引號包起來
- repeated 類型的 field name 使用複數形式

---

## 版本兼容性
Protobuf 支援對 Message 進行擴展，可以在不破壞現有程式碼的情況下擴充 `.proto`，但是需要滿足一些以下條件，這裡先不詳細解釋其原理，後續會再慢慢說明，如下:
- **添加新的 field，其 field number 絕對不能重複之前的**
- 已存在 field number 不能變更
- int32、uint32 int64、uint64、bool是兼容的，可以直接轉換而不會直接使解碼完全壞掉。
  {{< alert warning >}}
但注意 64 位元數字，用 32 位元讀取會產生截斷
{{< /alert >}}
- sint32、sint64 是兼容的


---
### 參考資料

- [Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3/)

- [深入protobuf（Protocol Buffers）原理：简化你的数据序列化](https://zhuanlan.zhihu.com/p/667573873)

- [Protobuf简介与实例](https://github.com/ShaoQiBNU/Protobuf)

- [protobuf](https://www.cnblogs.com/hgzero/p/17240848.html)

- [Protobuf 简介与入门](https://juejin.cn/post/7086810236593373214)