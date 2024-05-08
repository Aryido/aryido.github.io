---
title: "Protobuf - 序列化反序列化詳解"

author: Aryido

#date: 2023-05-02T23:10:23+08:00
date: 2024-05-01T22:10:30+08:00

thumbnailImage: "/images/data-structure/protobuf.jpg"

categories:
- data-structure

tags:
- data-exchange

comment: false

reward: false
---
<!--BODY-->
> 現在越來越多的服務應用使用 Protobuf 來作為資料交換的格式，它被廣泛應用於 RPC 調用和資料存儲。 Protobuf 語言中立、平臺中立，只要定義好一份 **.proto** 檔案，就可以生成**不同的程式語言**來處理資料的序列化或反序列化。要了解 Protobuf 序列化/反序列化，首先需要了解一些知識點 :
> - Varint Encoding
> - Zigzag Encoding
> - Wire Type 類型
> - T-L-V 儲存方式
>
> 熟悉這些可以更加理解 protobuf，也能避免錯誤使用，以及更好的優化性能。本章節會實際把前面學到的知識點一次用上，用實際案例來了解 Protobuf - Serialization。
<!--more-->

---

之前有對 [protobuf 做過基礎的介紹](/posts/data-structure/protobuf-encoding-sturcture/)，而  Protobuf 是通過 Varint 和 Zigzag 來大幅減少了 Value 佔用的儲存空間，可參考前面 [Varint & Zigzag Encoding 介紹](/posts/algorithm/varint-zigzag-encoding/)。Protocol Buffers 做為一種**資料交換格式**，主要有兩個面相：

- 定義了一種介面描述語言 (Interface Description Language)，用來描述需要交換的資料結構
- 定義了一種 serialization-deserialization 模式

本篇介紹 Protobuf  的 serialization-deserialization。

---

# Serialization
Protobuf encode 後二進制表示法結構，就是**多個 field 組成**的。下面我們就針對不同的 WireType，來一個個分析，Protocol Buffers 不同 WireType 的序列化方式 :

### WireType = 0, 1, 5

{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-wiretype-015.jpg" >}}

- int32: `WireType = 0`，其 Value 是數字，用 Varint 編碼，因此編碼自帶長度訊息，**最多會使用到 8 個 bytes**。

- double: `WireType = 1`，其 Value 固定 8 bytes，是採用**雙精確度浮點數**的編碼方式 `IEEE 754`。

- float: `WireType = 5`，其 Value 固定 4 bytes。

{{< alert info >}}
以上這幾種類型，都不需要再多一個 byte 來說明 Length，故更好的利用了空間。
{{< /alert >}}

### WireType = 2

這種 WireType 會多了一個 Length，用於標識 Value 的長度，其中 Value 又區分成幾種類型：

- String 類型，是使用 UTF8 編碼
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-wiretype-2-string.jpg" >}}

- 嵌套另一個 Protobuf Message 類型
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-wiretype-2-message.jpg" >}}

- 有 repeat label
{{< image classes="fancybox fig-100" src="/images/data-structure/protobuf-wiretype-2-repeated.jpg" >}}
`proto2` 才會需要寫上 `packed=true`；在 `proto3` 中數字類如 int32、int64 等等有標上 repeated 的 field 默認使用 packed 編碼，**會專門針對數字類型做的一個編碼空間優化策略**，它會把 Tag 完全一樣的 value 打包在一起。
  {{< alert success >}}
沒有 packed 的 repeat 數字，標瑪模式都是 Tag-Value 接續 Tag-Value 持續下去，但其實它們有着一樣的 tag，所以可以巧妙的使用 T-L-V 存儲方式，Length 代表 Value 列表的**總字節長度**，不需要每個 Value 前面放個相同的 Tag，極大的節省了空間。
{{< /alert >}}

{{< alert warning >}}
如果原始資料的某個 field 沒有被附加上值，那麼序列化後的 byte array 就會完全不存在相關資訊。相應 field 在解碼時會被設置爲**默認值**。
{{< /alert >}}


# Deserialization
直接來練習一下吧! 我是拿這個網站 [zddhub](https://zddhub.com/note/2021/07/25/protobuf.html) 內範例題目，首先有一個新的 `.proto` 檔定義如下:
```
message Award {
  int64 id = 1;
  string code_book = 4;

  message Bonus {
    repeated int32 indexes = 24;
  }

  Bonus bonus = 128;
  double magic = 2048;
}
```

有個資料已經被 protobuf 編碼為二進制 :
```
00001000 10110111 01001010 00100010 00011110 01100001
01100010 01100011 01100100 01100101 01100110 01100111
01101000 01101001 01101010 01101011 01101100 01101101
01101110 01101111 01110000 01110001 01110010 01110011
01110100 01110101 01110110 01110111 01111000 01111001
01111010 00101100 00100001 00111111 00100000 10000010
00001000 00101011 01010010 00000100 00000101 00000000
00001010 00000100 11000010 00000001 00100010 00011000
00001110 00010100 00011101 00000000 00010001 00000100
00011101 00000000 00010110 00000100 00010010 00001110
00001100 00000100 00011011 00011101 00010110 00000100
00000010 00000111 00000000 00010011 00011101 00001100
00000100 00011100 00011101 00011001 00000011 00000011
00000111 00010100 00000001 10000001 10000000 00000001
00000000 00000000 00000000 00000000 00000000 10000000
00100100 01000000
```
由於 string 直接用 ASCII 碼存儲，直接查看二進制就解密了，所以做了一個 code_book ，透過下標對應到密碼本上。例如 `indexes = [7, 4, 11, 11, 14, 29, 22, 14, 17, 3, 27] = hello word`，那這邊例題密碼本稍微不太一樣，以下實際來解密它 :

因為 Protobuf 編碼是依照 T-[L]-V 串聯而成，第一個 byte 一定是 tag。

- 第 1 個 byte `00001000`，最高位元是 `0`，表示該 tag 僅以一個位元組表示。 `field number = 1`，`wiretype = 0` 代表 Varint ，故接下來是 value。
- 第 2 個 byte `10110111`，最高位元為 `1`，表示該 value 超過一個 byte ，繼續看下一個。
- 第 3 個 byte `01001010 `，此時最高位元為 `0`，該 value 以兩個 byte 表示，把順序反過來為，並去掉最高位標示 :  `1001010 0110111` 為一個 value 的表示。
- 第 4 個 byte `00100010`，最高位元是 `0`，表示該 tag 僅以一個位元組表示。 field number 是 `00100`代表 `4`，`wiretype = 2` 代表 T-L-V ，故接下來是 length。
- 第 5 個 byte `00011110`，代表長度為 30，故接下來 30 個，即第 6 ~ 35 個 byte 都是 value :
  ```
  01100001
  01100010 01100011 01100100 01100101 01100110 01100111
  01101000 01101001 01101010 01101011 01101100 01101101
  01101110 01101111 01110000 01110001 01110010 01110011
  01110100 01110101 01110110 01110111 01111000 01111001
  01111010 00101100 00100001 00111111 00100000
  ```
  看起來是代表 `a~z,!? `，**注意最後一個有空白鍵**。
- 第 36 個 byte `10000010`，為一個新資料的 tag，因最高位元為 `1`，表示該 tag 超過一個 byte，繼續看下一個。
- 第 37 個 byte `00001000 `，因最高位元為 `0`，表示該 tag 結束，以兩個 byte 表示。把順序反過來為，並去掉最高位標示 :  `0001000 0000010` 為一個 tag 的真實表示。其中 `wiretype = 2`，是 T-L-V ，故接下來是**長度**；而 field number = `10000000` = 128。
- 第 38 個 byte `00101011` = 43，故接下來 43 個，即第 39 ~ 81 個 byte 都是 value :
  ```
  01010010 00000100 00000101 00000000
  00001010 00000100 11000010 00000001 00100010 00011000
  00001110 00010100 00011101 00000000 00010001 00000100
  00011101 00000000 00010110 00000100 00010010 00001110
  00001100 00000100 00011011 00011101 00010110 00000100
  00000010 00000111 00000000 00010011 00011101 00001100
  00000100 00011100 00011101 00011001 00000011 00000011
  00000111 00010100 00000001
  ```
- 第 39 個 byte `01010010` 代表 tag，`wiretype=2`，知道下一個 byte 代表長度，且 `field number = 10`，但沒有在 `.proto`中定義，故等等之後 value 會省略不解析
- 第 49 個 byte `00000100` = 4，承上接下來 4 個 byte 都捨棄不解析，所以 50~53 byte :
  ```
  00000101 00000000
  00001010 00000100
  ```
  都不用解析，丟掉他們。
  {{< alert info >}}
Protobuf 在解析時，主要都是看 `field number`。如果看到 Protobuf byte array 內有一個資料 T-[L]-V ，但是 schema 中沒有定義，其實並不會怎樣。雖然沒有辦法知道這個 field 名稱叫什麼，但是可以知道是什麼 type，同時也可以算出需要跳過多少個 bytes，以便找到的下一個 T-[L]-V。
{{< /alert >}}

- 第 54 個 byte `11000010` 代表 tag，但因最高位元為 `1`，表示該 tag 超過一個 byte，繼續看下一個。
- 第 55 個 byte `00000001`，也代表 tag，因最高位元為 `0`，表示該 tag 結束，故 tag 以兩個 byte 表示，把順序反過來為，並去掉最高位標示 : `0000001 1000010`，代表 `field number = 24`；`wiretype = 2`，然後又因為有 repeated ，故接下來代表長度，然後其後面 byte 都是指 value。

- 第 56 個 byte `00100010` ，代表長度 34。

- 接下來 34 個都是單字對應
  ```
  00011000
  00001110 00010100 00011101 00000000 00010001 00000100
  00011101 00000000 00010110 00000100 00010010 00001110
  00001100 00000100 00011011 00011101 00010110 00000100
  00000010 00000111 00000000 00010011 00011101 00001100
  00000100 00011100 00011101 00011001 00000011 00000011
  00000111 00010100 00000001
  ```
  [解析完之後，以數字表示如下，並加上空白](https://tools.yeecord.com/zh-tw/base-converter/bin/dec):
  ```
  (24 14 20) (0 17 4) (0 22 4 18 14 12 4 27) (22 4 2 7 0 19) (12 4 28) (25 3 3 7 20 1)
  ```
  對上數字翻譯為:
  >  `you are awesome! wechas le? zddhub` (wechas 代表中文微信)


- 第 82 個 byte 為 tag， `10000001`，因最高位元為 `1`，表示該 tag 超過一個 byte ，繼續看下一個。
- 第 83 個 byte `10000000`，因最高位元為 `1`，繼續看下一個。
- 第 84 個 byte `00000001  `，因最高位元為 `0`，tag 結束。去掉最高位標示並重新排列後 `0000001 0000000 0000001`，故 field number 為 `100000000000` = 2048
 `wiretype = 1` ，下 8 個 bytes 表示為 double
- 第 85 ~ 93 個 byte 都是 value:
  ```
  00000000 00000000 00000000 00000000 00000000 10000000
  00100100 01000000
  ```
  為 [1.025E1 = 10.25](https://www.binaryconvert.com/result_double.html?)

至此全部分析完，以上是整個反序列化的講解。

{{< alert success >}}
在解析 Protobuf data 時，序列化和反序列化都是基於 `.proto` 檔的，所以可以明確知道 type ，故會根據欄位的 WireType 以及其定義的類型來進行解碼。
{{< /alert >}}

---
### 參考資料

- [protobuf 是怎麼序列化的](https://www.readfog.com/a/1668729653630701568)

- [Protocol Buffers](https://zddhub.com/note/2021/07/25/protobuf.html)


- [android 序列化Image 序列化 protobuf](https://blog.51cto.com/u_16213700/7706668)

- [protobuf优缺点及编码原理](https://www.cnblogs.com/niuben/p/14212711.html)