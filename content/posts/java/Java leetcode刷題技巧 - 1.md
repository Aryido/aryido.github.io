---
title: Java leetcode刷題技巧 - 1

author: Aryido

date: 2022-09-02

thumbnailImage: "/images/java/java-bean-log.jpg"

categories:
- java

tags:
- java
- LeetCode

comment: false
reward: false
---

刷題時候有些轉換會忘記或是寫出bug，在這邊想做一些簡單紀錄。 閱讀約3分鐘

<!--more-->

# [LeetCode57](https://leetcode.com/problems/insert-interval/)

在寫這題時，因為要回傳 intervals 的數量不太確定，故使用了`ArrarList`
```java
 List<int[]> newIntervals = new ArrayList<>(); // unknow ans's length
```
但最後答案回傳型別為`int[][]`，故必須把 List of array 轉成 2D-array。

generalize 寫法如下:
```java
List<T[]> list = new ArrayList();

// 記得 list.size() 後面還有個[]
T result[][] = list.toArray(new T[list.size()][]);

```


---