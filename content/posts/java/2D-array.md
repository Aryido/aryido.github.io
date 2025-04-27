---
title: Java 泛型注意事項 - List of array 轉成 2D-array

author: Aryido

date: 2022-09-01T08:11:32+08:00

thumbnailImage: "/images/java/java-bean-logo1.jpg"

categories:
  - language
  - java

tags:
  - array

comment: false

reward: false
---

<!--BODY-->

> 把 List of array 轉成 2D-array
>
> ```java
> List<T[]> list = new ArrayList();
> // 記得 list.size()
> // 後面還有個[]
> T result[][] = list.toArray(new T[list.size()][]);
>
> // 實際 example
> List<int[]> listOfIntegers = List.of( new int[] { 1, 2 }, new int[] { 3, 55, 65 } );
> int[][] array2D = listOfIntegers.toArray( new int[listOfIntegers.size()][] );
> ```

<!--more-->

---

注意 ! 當想把 **List of Integer 轉成 int[]時**，會遇到一些問題。

```java
List<Integer> list = List.of( 1, 2, 3, 55, 78, 465, 354131, 12, 6 );

//int[] integers = list.toArray( new int[list.size()] ); 編譯錯誤

Integer[] integers = list.toArray( new Integer[list.size()] ); // 正常運行，但會是Integer array

```

去看 java.util.Lis source code 發現 java.util.List<E> ，**有用到泛型**，故無法用 int ，要使用其 wrapper class ，也就是 Integer 才行。因為泛型在編譯時，會進行類型擦除，最後只保留原始類型。而原始類型只能是 Object 類及其子類，故不能使用基本類型。
{{< alert warning >}}
Java 中的**基本類型**有 **8 種**，**最好背下來**，分別是：

- byte
- short
- int
- long
- float
- double
- boolean
- char
  {{< /alert >}}

---

因為 java 有 **wrapper class** 的概念，所以要特別注意**到底是 List 還是 Array**，例如

- **2D case**
  - `List<List<Integer>> -> int[][]`
  - `List<int[]> -> int[][]`
- **1D case**
  - `List<Integer> -> int[]`

這些類型在轉換時，若有需要呼叫 api ，都要**特別注意是否會因為泛型或基本型造成編譯錯誤**。

## 轉換範例

- Stream API
  ```java
  List<List<Integer>> list = Arrays.asList(
  Arrays.asList(1, 3, 2),
  Arrays.asList(1, 2, 2),
  Arrays.asList(1, 2)
  );

      int[][] arr = list.stream()
                      .map(l -> l.stream().mapToInt(Integer::intValue).toArray())
                      .toArray(int[][]::new);

      System.out.println( Arrays.deepToString(arr)); // [[1, 3, 2], [1, 2, 2], [1, 2]]
      ```

  {{< alert info >}}
  可以特別留意

- **mapToInt** 方法
- **Arrays.deepToString()** 方法
  {{< /alert >}}

---

# Exercise

## [LeetCode57. Insert Interval](https://leetcode.com/problems/insert-interval/)

---
