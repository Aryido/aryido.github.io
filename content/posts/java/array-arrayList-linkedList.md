---
title: "Array & ArrayList & LinkedList 選擇場景"

author: Aryido

date: 2022-09-17T12:49:55+08:00

thumbnailImage: "/images/java/java-bean-logo3.jpg"

categories:
- language
- java

tags:
- array
- linked-list

comment: false

reward: false
---
<!--BODY-->
> Array 是 Java 中的基本功能，而 ArrayList 是 Collection 的一部分； ArrayList 和 LinkedList 都是 Java 中的集合類型，它們都實現了 List 接口。基本特徵簡單如下 :
> - Array 是一個有固定大小的，每次創建都需要設定，而且在創建後，是不能再更改大小
> - ArrayList ，是一個有浮動大小的Array，且適用於需要快速訪問集合中的元素的場景。
> - LinkedList 適用於頻繁插入和刪除元素的場景。
>
> 如果需要實現隊列或棧等數據結構，也可以選擇 LinkedList。
>
<!--more-->

---

# Array 和 LinkedList

Array 是一組內存中，連續的 data :
- 能用 index 直接搜尋 >> ```O(1)``` access
- **加入 data 時要注意擴容問題**
- 刪除 data  會需要後面所有 data 的 index

LinkedList
內存中不一定連續的 data :
-  不能用 index 直接搜尋，只能遍歷 >> ```O(n)``` access
- 刪除 data 要先遍歷找到目標 >> ```O(n)``` access

# Array 和 ArrayList
比較常忘記的部分，是資料結構(Data type)的類別有所限制 :
- Array 可以包含 primitive data types 和 object entities
- ArrayList 只可以包含 object entries ，**但不支持 primitive data types**

```java
// array 允許 primitive data types 和object entities
int[] array = new int[10];
Integer[] array1 = new Integer[10];

// ArrayList 允許 object entities
ArrayList<Integer> arrL1 = new ArrayList<>();
```

{{< alert danger >}}

```java
// ArrayList 不允許 primitive data types
// 當運行以下一句的code時，會出現error
ArrayList<int> arrL = new ArrayList<int>();
```

{{< /alert >}}

{{< alert warning >}}

Primitive data types : **byte**, **short**, **int**, **long**, **float**, **double**, **boolean**, **char**

{{< /alert >}}

# ArrayList 和 LinkedList

在 [**leetcode 332. Reconstruct Itinerary**](https://leetcode.com/problems/reconstruct-itinerary/) ，答案須回傳的類型是 **List**。題目的解題架構，歸類到圖論題的 **Post-order traversal on Edges** ，所以添加答案要往第一個元素插入。 這裡就是一個可以使用 **LinkedList** 而非 **ArrayList** 的好情境 !

---

## Interface List<E> 方法
### set(int index, E element)

- **set(int index, E element)**:  替換 array 中指定位置的元素。
- **返回值**: 是在指定位置**過去舊的**元素

```java

ArrayList<Integer> list = new ArrayList<>();
list.add(15);
list.add(20);
list.add(25);
list.add(22);
// list = [15, 20, 25, 22]
Integer int = list.set( 1, 55 ); // int = 20
// list = [15, 55, 25, 22]

```

### add(int index, E element)
- **add(int index, E element)**: 插入到指定位置，然後*其他指定位置之後所有元素，向後移一個位置*。
- 返回值: 沒有返回值

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(15);
list.add(20);
list.add(25);
list.add(22);
// list = [15, 20, 25, 22]

list.add( 1, 1 ); // void
// list = [15, 1, 20, 25, 22]
```

---

{{< alert info >}}

使用 **LinkedList** 的 add 插入效率會比 ArrayList 的 add 插入效率還要好。

{{< /alert >}}

 因為 ArrayList 的 add 底層實作，首先考慮擴容，故可能要複製一個新的 array ，且指定位置之後的元素要往後一個位置。
而 **LinkedList** 直接把 node 的 reference 更改就達成插入了。

---

### 參考資料

- [Java - Array 與 ArrayList 的分別](https://ithelp.ithome.com.tw/articles/10229699)

- [Linked List链表题型解题套路和模板](https://www.youtube.com/watch?v=0czlvlqg5xw&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=4)

