---
title: Java 泛型注意事項 - List of array 轉成 2D-array

author: Aryido

date: 2022-09-01T08:11:32+08:00

thumbnailImage: "/images/java/java-bean-logo1.jpg"

categories:
- java

tags:
- array

comment: false

reward: false
---
<!--BODY-->

> 把 List of array 轉成 2D-array
```java
List<T[]> list = new ArrayList();

// 記得 list.size() 後面還有個[]
T result[][] = list.toArray(new T[list.size()][]);
```

<!--more-->

---

注意 ! 當想把 **List<Integer> 轉成 int[]時**，會遇到一些問題。 看 source code 知道 java.util.List<E> 有用到**泛型**，故無法用 int ，要使用其 wrapper class ，也就是 Integer 才行。

因為泛型在編譯時，會進行類型擦除，最後只保留原始類型。而原始類型只能是 Object 類及其子類，故不能使用基本 data structure 類型。
{{< alert info >}}
Java 中的 data structure 類型有 **8 種**，**最好背下來**，分別是：
- byte
- short
- int
- long
- float
- double
- boolean
- char
{{< /alert >}}
```java
List<Integer> list = List.of( 1, 2, 3, 55, 78, 465, 354131, 12, 6 );

//int[] integers = list.toArray( new int[list1.size()] ); 編譯錯誤

Integer[] integers = list .toArray( new Integer[list1.size()] ); // 正常運行，但會是Integer array

```

---

# Exercise
## [LeetCode57. Insert Interval](https://leetcode.com/problems/insert-interval/)

---