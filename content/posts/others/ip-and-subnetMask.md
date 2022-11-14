---
title: IP and Subnet Mask 介紹

author: Aryido

date: 2022-11-07T22:55:14+08:00

thumbnailImage: "/images/others/ip-address-definition.jpg"

categories:
- network

comment: false

reward: false
---
<!--BODY-->
> IP (Internet Protocol) 是電腦的地址。IP 位址在系統中是一個 32 位元的數字，但為了方便人類讀寫，每一個位元組會被轉換成一個十進位的數字。
>
> IP 位址可以分為 Network ID 和 Host ID，為了讓電腦可以判斷出 IP 位址的 Network ID 及 Host ID，必須靠子網路遮罩 (Subnet Mask) 的幫忙。
<!--more-->

---

# Public IP Address

所有的**公有 IP 位址**是由 IANA (Internet Assigned Numbers Authority) 組織授權給各地區的單位分派，在台灣則是由 TWNIC (http://www.twnic.net.tw) 所管理。

## Class A
理論上Ａ級的範圍是從 0.0.0.0 到 127.255.255.255。它的第一個位元組由 IANA 所分派，後面的三個位元組可以自行運用。

例如，麻省理工學院的 WWW 主機 web.mit.edu，它的IP位址是 18.69.0.27，很明顯的，是屬於Ａ級的。

## Class B
理論上Ｂ級的範圍是從 128.0.0.0 到 191.255.255.255。它的前二個位元組是由 IANA 所指派，而後面二個位元組可以自行運用。

例如國內各大專院校所配發的IP位址，就是隸屬於Ｂ級網路之中。

## Class C
理論上Ｃ級的範圍是從 192.0.0.0 到 223.255.255.255。它的最後一個位元組可以自行運用。一個 Class C 最多可以有 256 個 IP 位址。

{{<alert warning>}}
為什麼是「理論上」呢？因為有些 IP 位址另有特殊用途。譬如說:
- 大部份系統都指定 127.0.0.1 為 loopback 位址
- 如果有一個 Class C 是 210.202.102.x，則 210.202.102.0 是 Network ID，代表網路本身，而 210.202.102.255 是用來做為網路廣播，這些 IP 都不可以用在電腦的 IP 上。
{{</alert>}}

---

# Private IP Address

可想而知，如果每世界上每一台電腦都需要一個 IP，一定會不夠用。所以除了一般可以在 Internet 上使用的 IP 之外，還有所謂的 Private IP Address (私有 IP)。RFC 1597 所定義的**私有 IP** 範圍如下：

## Class A
私有 IP：10.0.0.0 到 10.255.255.255，也就是說可以使用 10.x.x.x 這範圍的 IP。

## Class B
IP：172.16.0.0 到 172.31.255.255，也就是從 172.16.x.x 到 172.31.x.x，共有 16 個 Class B 的 IP。

## Class C
IP：192.168.0.0 到 192.168.255.255，也就是 192.168.0.x 到 192.168.255.x，共有 256 個 Class C 的 IP。

---

# Subnet Mask(子網路遮罩)
IP 位址分為二個部份
- 一個是由 IANA 或 TWNIC 或 ISP 所分派的固定部份，稱之為 **Network ID**。
- 以及可以自行運用的部份，稱之為 **Host ID**。

為了讓電腦可以判斷出 IP 位址的 Network ID 及 Host ID，必須靠子網路遮罩 (Subnet Mask) 的幫忙。因為每個網路都可以再切割為更小的子網路，例如前二個位元組是 Network ID，因此子網路遮罩為 255.255.0.0。

{{<alert info>}}
電腦會將 IP 和 Netmask 做 AND 運算，運算結果就是 Network ID
{{</alert>}}

當我們在表示一個子網路時，我們可以使用 140.115.0.0/255.255.0.0 來表示。然而，我們也常看到另一種表示方式：
- 140.115.0.0/16。
意思是子網路遮罩中，高位元的部份，前 16 個都是 1。

- 140.115.0.0/20。
意思是子網路遮罩中，高位元的部份，前 20 個都是 1。

---
