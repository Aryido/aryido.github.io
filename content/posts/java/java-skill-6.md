---
title: ArrayList & LinkedList 使用小優化情境

author: Aryido

date: 2022-09-17T12:49:55+08:00

thumbnailImage: "/images/java/java-bean-logo3.jpg"

categories:
- java

tags:
- java
- list

comment: false

reward: false
---
<!--BODY-->
> 在 *leetcode 332. Reconstruct Itinerary* 時，答案須回傳 **List<String>**，因為題目是 **Post-order traversal on Edges** ，所以添加答案要往第一個元素插入。 這裡就是一個可以使用 **LinkedList** 而非 **ArrayList** 的好情境!
<!--more-->

## Interface List<E> 方法說明
### set(int index, E element) 和 add(int index, E element) 差別

- **set(int index, E element)** 替換 array 中指定位置的元素。返回在指定位置**之前**元素

- **add(int index, E element)** 插入到指定位置，然後*其他指定位置之後所有元素向後移一個位置*。沒有返回值。

使用 **LinkedList** 的 add 效率會比 ArrayList 的 add 還要好。 因為 ArrayList 的 add 底層實作，首先考慮擴容，故可能要複製一個新的 array ，且指定位置之後的元素要往後一個位置。
而 **LinkedList** 直接把 node 的 reference 更改就達成插入了。

---