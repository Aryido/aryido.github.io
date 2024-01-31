---
title: "Regular Expression 簡介"

author: Aryido

date: 2024-01-26T21:19:20+08:00

thumbnailImage: "/images/others/regular-exp.jpg"

categories:
- Monitoring

comment: false

reward: false

---
<!--BODY-->
>  Regular Expression 是一種強大的**字串匹配**、**字串查找**等操作工具，常簡寫爲 regex 、regexp 或 RE。這概念最初由 Unix 的 sed、grep 操作而普及開，它定義一系列**符號**來描述搜索的規則。
> 但在不同的 coding language 或者是不同 OS 中， 常發現 regex 都會有些差異，主要原因是演進過程中，出現 **POSIX** 與 **PCRE** 兩種 :
>
> - POSIX : 可以說是原初版本，主要用於 UNIX 系統的文本處理，grep 、sed 、awk 等都屬之
>
> - PCRE : 現代 coding language如 Python、Ruby、 C、C++、Java 都屬於 PCRE 派系。

<!--more-->

---

#  POSIX
80 年代，**POSIX (Portable Operating System Interface)** 標準公諸於世，它制定了不同的操作系統都需要遵守的一套規則，其中就包括正則表達式的規則。遵循 POSIX 規則的正則表達式，稱爲 POSIX 派系的正則表達式。分爲兩種標準:
- BRE（Basic Regular Expression) 基本正則表達式

- ERE（Extended Regular Expression) 擴展正則表達式

{{< alert info >}}
簡單來看差別的話，如果**使用 BRE 標準，需要對 [], (), | 符號進行轉義，而 ERE 不用**，故自己感覺使用 ERE 能書寫更簡單的 regex 。
{{< /alert >}}

---

#  PCRE
說到 **PCRE（Perl Compatible Regular Expressions）** 就要提到90 年代 Perl，在發展過程中，新增許多豐富的特性來加強 regex ，是很多現代 coding language 所使用的 regex lib 基礎(如 python 的 re lib)，所以絕大部分現在 code language 都使用 PCRE 標準。

{{< image classes="fancybox fig-100" src="/images/others/regular-exp-1.jpg" >}}

{{< alert danger >}}
regex 在處理 data 時很常使用，但在使用不同程式語言時，可以注意一下工具使用的 regex 是何種標準，避免遷移不同環境後運行結果不符合預期。
{{< /alert >}}

---

# 工具介紹
以下介紹一些實用 web 來檢測和學習 Regex :
- [Regex101](https://regex101.com/)

- [Regular Expression測驗](https://regexone.com)

- [fluent bit 的 PARSER 是使用和 Ruby regex 一樣的 lib，官方推薦用這個測試](https://rubular.com/)

Regex101 網站還蠻有用的，主要是在右側的 EXPLANATION 區域會顯示出對輸入的正則表達式的詳細解釋，另外還可以找一些別人寫好的 patterns。

---
### 參考資料

- [梳理正則表達式發展史](https://www.readfog.com/a/1655052731076939776)

- [正則表達式完整指南](https://www.readfog.com/a/1670444954621677568)


