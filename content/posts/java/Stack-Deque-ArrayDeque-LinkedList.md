---
title: Stack、Deque、ArrayDeque、LinkedList 介紹

author: Aryido

date: 2022-09-05T00:54:48+08:00

thumbnailImage: "/images/java/java-bean-logo2.jpg"

categories:
- language
- java

tags:
- stack

comment: false

reward: false
---
<!--BODY-->
> Java 現在 Stack 類已經**不建議**使用。現在推薦的是，使用雙端隊列接口 Deque 取代 Stack。 Deque 是 interface，有兩個常用的 implement :
> - **ArrayDeque**
> - **LinkedList**
>
> 簡單來介紹和比較一下。

<!--more-->

---

# Stack
{{< image classes="fancybox fig-100" src="/images/java/stack.jpg" >}}
Stack 是一種後進先出 (LIFO) 的容器，常用的操作為 push / pop / peek。 現在 Java **建議使用**雙端隊列接口 Deque，並用其實現類 ArrayDeque / LinkedList 來進行 Stack 的初始化。

```java
//以下兩個都可以

Deque<Integer> stack1 = new LinkedList<>();

Deque<Integer> stack2 = new ArrayDeque<>();

```

使用 Deque 有一個很大的好處是， Deque 在轉成 ArrayList 時，或者使用 stream 時， Deque 會保持 **後進先出** 的語義:
```
Deque<Integer> deque = new ArrayDeque<>();

deque.push(1);
deque.push(2);

List<Integer> list1 = new ArrayList<>(deque); //[2, 1]
List<Integer> list2 = deque.stream().collect(Collectors.toList()); //[2,1]

```

---

## Deque 實作該用 ArrayDeque 還是 LinkedList ？

{{< alert info >}}
其實都行，但官方建議使用ArrayDeque
{{< /alert >}}

- ArrayDeque: 底層是 array，容量不夠時需要擴容和數組拷貝，通常容量不會填滿，會有空間浪費。
- LinkedList: 底層用 Node 串聯，每次 push 都需要 new Node 節點，並且 node 節點裡面有 prev 和 next 成員，也會有額外的空間佔用。

---

# 整理:
{{< alert warning >}}
- Stack: 此類已經不建議使用，因為也是從棄用的 Vector 類繼承， Vector 底層是用 array 實現且 Thread 安全。
- Deque: 是 interface ，雙端隊列接口。
- ArrayDeque: 實現類，建議優先使用 ArrayDeque 來代替 Stack。
- LinkedList: 注意 LinkedList 是Deque interface 的另一種實現類。
{{< /alert >}}

![class圖展示](/images/java/collection-tree.jpg)
---