---
title: Java 技巧 - Java Arrays 方法

author: Aryido

date: 2022-09-09T19:59:56+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- java

tags:
- array

comment: false

reward: false
---
<!--BODY-->
> 刷題時很常出現 Array 的結構如 `int[]、char[]` 等等...，故在這邊條列一些常用的 Arrays 方法。
> 在使用過程也可以複習 java 提供的工具類，還有一些泛型的坑...

<!--more-->

---

主要介紹 java.lang.Object 的 java.util.Arrays 類。

#  Arrays.copyOf()
array 是 reference 傳遞的，想要實現 value 傳遞，則使用 **copyOf()**
{{< alert warning >}}
特別注意 **copyOf()** ，參數要傳 array 的 length 喔 !
{{< /alert >}}

```java
int[] nums = {1, 4, 3, 2, 6, 9, 8};
int[] newNums = Arrays.copyOf(nums, nums.length);

System.out.println(nums); // [I@6379eb
System.out.println(newNums); // [I@294425a7 ， 深拷貝，地址不一樣了

```
{{< alert warning >}}
特別注意， array 使用 toString() 方法，依然是列出地址 !
```java
System.out.println(nums.toString()); // [I@564fabc8
System.out.println(newNums.toString()); // [I@294425a7
```
{{< /alert >}}

---

# Arrays.toString()
前例知道直接使用 nums.toString()，會得到一個 reference 地址。故 Arrays.toString() 可以幫助我們快速列出內容。

Arrays.toString() 方法可以把 array 內容 print 出來。
```java
// 承前範例 nums = {1, 4, 3, 2, 6, 9, 8}
System.out.println(nums); // [I@564fabc8
System.out.println(Arrays.toString(nums)); // [1, 4, 3, 2, 6, 9, 8]
```

---

# Arrays.sort
默認升序

```java
Arrays.sort(nums)

// 以下會出錯，要用 Integer[] 才行
// Arrays.sort(ints, Collections.reverseOrder() );
```
{{< alert warning >}}
這就是坑...， Collections.reverseOrder() 有用到泛型，故要使用 Integer[] 才可以通過編譯。
```
Integer[] ints = {1, 4, 3, 2, 6, 9, 8};
Arrays.sort(ints, Collections.reverseOrder() ); //[9, 8, 6, 4, 3, 2, 1]
```
{{< /alert >}}

---

# 獲取 array 中最大值、最小值
最後需要 getAsInt() 方法是因為 stream 出來會是 optional 類型

```java
int minNum = Arrays.stream(nums).min().getAsInt();
int minNum = Arrays.stream(nums).max().getAsInt();
```

---


# Arrays.fill()
此方法可填充 value 至指定 index 範圍，若沒寫 index 範圍則代表全部置換

```java
// Arrays.fill(arrayname,value)
// Arrays.fill(arrayname ,starting index ,ending index ,value)
Arrays.fill(nums, 100);
```

---


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