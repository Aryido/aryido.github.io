---
title: "Java Skill 4"

author: Aryido

date: 2022-09-09T19:59:56+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- java

tags:
- java
- LeetCode

comment: false

reward: false
---
<!--BODY-->
> 刷題時很常出現 Array 的結構如 `int[]、char[]` 等等...，故在這邊條列一些常用的 Arrays 方法

<!--more-->
主要介紹 java.lang.Object 的 java.util.Arrays 類。

```java
int[] nums = {1, 4, 3, 2, 6, 9, 8};

// array 是 reference 傳遞的，想要實現 value 傳遞，則使用 copy。
int[] newNums = Arrays.copyOf(nums, nums.length);

// 快速 print array
// 直接使用 nums.toString()，會得到一個 reference 地址 EX: [I@564fabc8
Arrays.toString(nums);
System.out.println(Arrays.toString(nums));

// 默認升序
Arrays.sort(nums)

// 獲取 array 中最大值、最小值的簡單寫法
// 最後需要 getAsInt() 方法是因為 stream 出來會是 optional 類型
int minNum = Arrays.stream(nums).min().getAsInt();
int minNum = Arrays.stream(nums).max().getAsInt();

// Arrays.fill(arrayname,value)
// Arrays.fill(arrayname ,starting index ,ending index ,value)
// 此方法可填充 value 至指定 index 範圍，若沒寫 index 範圍則代表全部置換
Arrays.fill(nums, 100);

// 打印多維array
int[][] mat = new int[2][2];
mat[0][0] = 99;
mat[0][1] = 151;
mat[1][0] = 30;
mat[1][1] = 5;
System.out.println(Arrays.deepToString(mat)); //[[99, 151], [30, 5]]
```
---