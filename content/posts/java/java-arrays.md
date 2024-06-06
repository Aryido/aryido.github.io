---
title: Java 技巧 - Java Arrays 方法

author: Aryido

date: 2022-09-09T19:59:56+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- language
- java

tags:
- array

comment: false

reward: false
---
<!--BODY-->
> 刷題時很常出現 Array 的結構如 `int[]、char[]` 等等...，故在這邊條列一些常用的 Arrays 方法。這次主要整理下 Java 中 Arrays 類的常用方法，在使用過程也可以複習 java 提供的工具類，還有一些泛型的坑...

<!--more-->

---

主要介紹 java.lang.Object 的 java.util.Arrays 類， 承前可知道 Arrays 類位於 java.util 包中。

#  Arrays.copyOf()
array 是 reference 傳遞的，想要實現 value 傳遞，則使用 **copyOf()**
{{< alert info >}}
 **copyOf()** 有參數要傳，分別是
- 要複製的 array
- 期望的 length
{{< /alert >}}

```java
int[] nums = {1, 4, 3, 2, 6, 9, 8};
int[] newNums = Arrays.copyOf(nums, nums.length);

System.out.println(nums); // [I@6379eb
System.out.println(newNums); // [I@294425a7 ， 深拷貝，地址不一樣了

// 期望的 length 可以比要複製的 array 的長度還短，不會報錯
int[] lessLengthArray = Arrays.copyOf(nums, nums.length - 3);
System.out.println(Arrays.toString( lessLengthArray )); //[1, 4, 3, 2]

```
{{< alert warning >}}
一樣再提醒， array 使用 toString() 方法，是列出地址 !
```java
System.out.println(nums.toString()); // [I@564fabc8
```

要列出內容，使用 Arrays.toString() 方法
```java
System.out.println(Arrays.toString( lessLengthArray )); //[1, 4, 3, 2]
```

{{< /alert >}}

---

#  Arrays.copyOfRange(T[] original, int from, int to)
拷貝指定起始位置和結束位置的陣列

```java
int[] nums = {1, 4, 3, 2, 6, 9, 8};
int[] newNums = Arrays.copyOfRange( nums, 2, 5 );
System.out.println(Arrays.toString( newNums )); // [3, 2, 6]

```

```java

// 如果超過原陣列長度，則會用 null 或者基礎類型的預設值進行填充。
int[] nums1 = {1, 4, 3, 2, 6, 9, 8};
int[] newNums1 = Arrays.copyOfRange( nums1, 2, 10 );
System.out.println(Arrays.toString( newNums1 )); // [3, 2, 6, 9, 8, 0, 0, 0]

Integer[] nums2 = {1, 4, 3, 2, 6, 9, 8};
Integer[] newNums2 = Arrays.copyOfRange( nums2, 2, 10 );
System.out.println(Arrays.toString( newNums2 )); // [3, 2, 6, 9, 8, null, null, null]

```

{{< alert danger >}}
起始位置 from index， 不能超出範圍，會報 java.lang.ArrayIndexOutOfBoundsException
{{< /alert >}}

{{< alert warning >}}
起始位置和結束位置設一樣的話，是複製給你一個空的 array
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
{{< alert danger >}}
這就是坑...， Collections.reverseOrder() 有用到泛型，故要使用 Integer[] 才可以通過編譯。
```
Integer[] ints = {1, 4, 3, 2, 6, 9, 8};
Arrays.sort(ints, Collections.reverseOrder() ); // [9, 8, 6, 4, 3, 2, 1]
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
int[] nums = {1, 4, 3, 2, 6, 9, 8};
Arrays.fill(nums, 11);
System.out.println(Arrays.toString( nums )); // [11, 11, 11, 11, 11, 11, 11]
```

```java
// Arrays.fill(arrayname ,starting index ,ending index ,value)
int[] nums = {1, 4, 3, 2, 6, 9, 8};
Arrays.fill(nums, 2, 5, 11);
System.out.println(Arrays.toString( nums ));
// index  [0, 1, 2 , 3 , 4 , 5, 6]
// origin [1, 4, 3 , 2 , 6 , 9, 8]
// array  [1, 4, 11, 11, 11, 9, 8]
```

{{< alert danger >}}
起始位置和結束位置都不能超出範圍，會報錯 java.lang.ArrayIndexOutOfBoundsException
{{< /alert >}}

{{< alert warning >}}
起始位置和結束位置一樣的話(不能超出範圍)，原 array 不會變動
{{< /alert >}}

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

# Arrays.equals(Object[] array1, Object[] array2)
判斷兩個陣列是否相等
- 陣列元素為**基本類型**時，依次比較值
- 陣列元素為**Object類型**時，依次調用元素的 equals（） 方法進行比較

```java
int[] nums1 = {1, 4, 3, 2, 6, 9, 8};
int[] nums2 = {1, 4, 3, 2, 6, 9, 8};
System.out.println( Arrays.equals( nums1, nums2 ) ); // true
```

{{< alert warning >}}
還有一個 Arrays.deepEquals()，就不詳細說明了
{{< /alert >}}

---

# Arrays.stream(T[] array)

返回陣列的 Stream，然後我們就可以使用 Stream 相關的許多方法了
```java
// int[] 轉成 List<Integer>
int[] nums = { 1, 4, 3, 2, 6, 9, 8 };
List<Integer> collect = Arrays.stream( nums )
				.boxed()
				.collect( Collectors.toList() );
System.out.println( collect ); // [1, 4, 3, 2, 6, 9, 8]
```

{{< alert danger >}}
特別注意很有多 java 方法都帶有**泛型**，所以 int[] 和 Integer[] 都可能會踩到各式各樣的坑。
{{< /alert >}}

---