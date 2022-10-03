---
title: 把 List of array 轉成 2D-array

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

這邊想補充一下，同理我想把 List<Integer> 轉成 int[]，卻遇到一些問題。 看source code 知道 java.util.List<E> 有用到泛型，故無法用 int 代替 Integer。
因為泛型在編譯時，會進行類型擦除，最後只保留原始類型。而原始類型只能是Object類及其子類，故不能使用基本數據類型。
```java
List<Integer> list1 = List.of( 1, 2, 3, 55, 78, 465, 354131, 12, 6 );

//int[] integers = list1.toArray( new int[list1.size()] ); 編譯錯誤

Integer[] integers = list1.toArray( new Integer[list1.size()] ); // 正常運行，但會是Integer array

```

---

# Exercise
## [LeetCode57. Insert Interval](https://leetcode.com/problems/insert-interval/)

---