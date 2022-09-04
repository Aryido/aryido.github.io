---
title: Java Skill 1

author: Aryido

date: 2022-09-01T08:11:32+08:00

thumbnailImage: "/images/java/java-bean-log.jpg"

categories:
- java

tags:
- java
- LeetCode

comment: false

reward: false
---
<!--BODY-->

把 List of array 轉成 2D-array
```java
List<T[]> list = new ArrayList();

// 記得 list.size() 後面還有個[]
T result[][] = list.toArray(new T[list.size()][]);
```

<!--more-->
# Exercise
## [LeetCode57. Insert Interval](https://leetcode.com/problems/insert-interval/)

---