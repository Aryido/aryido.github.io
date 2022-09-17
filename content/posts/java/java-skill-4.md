---
title: "Java Arrays 方法"

author: Aryido

date: 2022-09-09T19:59:56+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- java

tags:
- java
- array

comment: false

reward: false
---
<!--BODY-->
> 刷題時很常出現 Array 的結構如 `int[]、char[]` 等等...，故在這邊條列一些常用的 Arrays 方法

<!--more-->
主要介紹 java.lang.Object 的 java.util.Arrays 類。

#  Arrays.copyOf()
array 是 reference 傳遞的，想要實現 value 傳遞，則使用 copy

```java
int[] nums = {1, 4, 3, 2, 6, 9, 8};
int[] newNums = Arrays.copyOf(nums, nums.length);
```

# Arrays.toString()
可以快速 print array。因為直接使用 nums.toString()，會得到一個 reference 地址 EX: [I@564fabc8
```java
Arrays.toString(nums);
System.out.println(Arrays.toString(nums));
```

# Arrays.sort
默認升序

```java
Arrays.sort(nums)
```

# 獲取 array 中最大值、最小值
最後需要 getAsInt() 方法是因為 stream 出來會是 optional 類型

```java
int minNum = Arrays.stream(nums).min().getAsInt();
int minNum = Arrays.stream(nums).max().getAsInt();
```

# Arrays.fill()
此方法可填充 value 至指定 index 範圍，若沒寫 index 範圍則代表全部置換

```java
// Arrays.fill(arrayname,value)
// Arrays.fill(arrayname ,starting index ,ending index ,value)
Arrays.fill(nums, 100);
```

# Arrays.deepToString()
打印多維array

```java
int[][] mat = new int[2][2];
mat[0][0] = 99;
mat[0][1] = 151;
mat[1][0] = 30;
mat[1][1] = 5;
System.out.println(Arrays.deepToString(mat)); //[[99, 151], [30, 5]]
```
---