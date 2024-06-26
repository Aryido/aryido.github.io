---
title: CIDR 介紹

author: Aryido

date: 2022-11-13T17:37:27+08:00

thumbnailImage: "/images/others/cidr.jpg"

categories:
- develop

tags:
- network

comment: false

reward: false
---
<!--BODY-->
> 無類別域間路由（ Classless Inter-Domain Routing ，簡稱 CIDR ）是為了避免造成 IP 位址的大量浪費，於是出現的一種技術。CIDR重點有：
> 1. 多變長度子網路遮罩 (Variable-Length Subnet Mask，VLSM)
> 2. 路由匯總 （Route Summarization）(暫不介紹)

<!--more-->

---

# 緣起
網際網路通訊協定（ IP 協定），對於位址的分配是由 32 個位元，分成 :
- Class A
- Class B
- Class C

32 個位元以每 8 個位元一組來區隔做不同的Class，而每一個類別依據這樣的區隔有不一樣的 IP 容納數量。

在早期的時候要做 IP 分配，只能是 8bits、16bits 或 24bits 為一組，這使得可分配主機數量從最小的 256 個，要再升一級，會直接跳到 65536 個。這樣的方式類別之間的差距過大，而造成 IP 位址的大量浪費。因為對大部分的企業來說 256 個太少，而 65536 個又太多，很不方便，因此 IETF 就在 RFC 1519 定義了新的分配 IP 的方式，就是 CIDR（Classless Inter-Domain Routing）。

{{< alert info >}}
網際網路工程任務組（Internet Engineering Task Force，縮寫：IETF）是一個開放的標準組織，負責開發和推廣網際網路標準（Internet Standard，縮寫:STD），特別是構成TCP/IP協定族（TCP/IP）的標準。
{{< /alert >}}

---
# CIDR 標記法
CIDR 標記法舉例像是這樣 192.168.1.0/20 ，其中字尾 /20 指的是子網路遮罩單位的數量，也就是代表從頭開始 20bits，從最前面算來有 20 個 1，其他為 0：

```
11111111.11111111.11110000.00000000
```
這種方式使得分配 IP 的彈性變比較大，而這樣的設計也被 IPv6 給延用了。

---
# 不同的遮罩長度與所能分配的IP位址個數
{{< image classes="fancybox fig-100" src="/images/others/subnet-mask-table.jpg" >}}

{{<alert info>}}
 CIDR 標記法還蠻常使用的，例如在設定 GCP Firewall 規則的時候，限制來源 IP 範圍的欄位，要我們以 CIDR 標記法輸入。
{{</alert>}}

{{<alert danger>}}
如果是要設定成任一 IP 皆能通過防火牆就設成 0.0.0.0/0，不過這樣等同於在網路上裸奔很危險。
{{</alert>}}

---
# 情境
假設現在分配到的網段是：

    172.16.32.0/20

打算分配給四個不同的部門，每個部門都有自己的網段，每個部門都要規劃 55 個 IP 位址。

# 分配方式
承前圖知道要能夠分配 55 個 IP 位址，至少必須使用 26 個位元當作 subnet mask。 接下來只把網段後面的兩個部分轉換成二進位表示:

**172.16.00100000.00**000000

因為網路遮罩個數是 26，故粗體顯示的地方是屬於**網路位址**的部分。接下來分段:

- **172.16.00100000.00**000000/26 = *172.16.32.0/26*

- **172.16.00100000.01**000000/26 = *172.1.32.64/26*

- **172.16.00100000.10**000000/26 = *172.16.32.128/26*

- **172.16.00100000.11**000000/26 = *172.16.32.192/26*

在以上分段可知
- 172.16.32.1 ~ 172.16.32.62 是在同一個子網路

- 172.16.32.65 ~ 172.16.32.126 是在同一個子網路

- 172.16.32.129 ~ 172.16.32.190 是在同一個子網路

- 172.16.32.193 ~ 172.16.32.254 是在同一個子網路

每一個子網路都是 62 個 ip 可用。

---
# Reference
[無類別區隔路由CIDR技術 依需求善用有限IP位址](https://www.netadmin.com.tw/netadmin/zh-tw/technology/0B9B631F987A45439061B6629F63DD07?page=1)