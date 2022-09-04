---
title: "Java Skill 2"

author: Aryido

date: 2022-09-05T00:54:48+08:00

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
Java中的Stack、Deque、ArrayDeque、LinkedList簡單介紹。

<!--more-->
# Stack
Stack 是一種後進先出 (LIFO) 的容器，常用的操作 push / pop / peek。

現在 Java **建議使用**雙端隊列接口 Deque，並用實現類 ArrayDeque / LinkedList 來進行初始化。

有一個很大的好處是使用 Deque 在轉成 ArrayList 或者使用 stream 的時候，保持***後進先出***的語義
```
Deque<Integer> deque = new ArrayDeque<>();

deque.push(1);
deque.push(2);

List<Integer> list1 = new ArrayList<>(deque); //[2, 1]
List<Integer> list2 = deque.stream().collect(Collectors.toList()); //[2,1]

```
## 該用 ArrayDeque 還是 LinkedList？
- ArrayDeque: 底層是 array，容量不夠時需要擴容和數組拷貝，通常容量不會填滿，會有空間浪費；

- LinkedList？: 底層用 Node 串聯，每次 push 都需要 new Node 節點，並且 node 節點裡面有 prev 和 next 成員，也會有額外的空間佔用。

# 整理:
- Stack: 此類以不建議使用，因為從Vector類繼承(底層是用 array 實現且線程安全)。
- Deque: 是interface，雙端隊列接口。
- ArrayDeque: 實現類，建議優先使用Deque的實現類來代替Stack。
- LinkedList: Deque interface 的另一種實現類。

![class圖展示](/images/java/collection-tree.jpg)
---