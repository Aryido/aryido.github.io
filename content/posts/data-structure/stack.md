---
title: "Stack"

author: Aryido

date: 2022-09-05T00:54:48+08:00

thumbnailImage: "/images/java/java-bean-logo2.jpg"

categories:
  - data-structure

tags:
  - stack
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> Stack 是一種「**後進先出 LIFO**」的容器 (Last if First Out)，適用於需要紀錄**之前**的狀態或是值的情況，必要的時候可以回到之前的樣子。對於 Java 和 Python 的 Stack 使用可以記憶一下：
> 
> - Java 推薦是**雙端隊列 Deque** 取代 Stack
> - Python 更簡單的可直接將 List 作為 Stack 使用
> 
> {{< image classes="fancybox fig-100" src="/images/java/stack.jpg" >}}

<!--more-->

---

{{< alert info >}}
題外話，對於 Dynamic Programming 是需要紀錄所有狀態，所以通常會用 Array 或者 HashTable ; 而 Stack 只能記錄前一個狀態。
{{< /alert >}}

單純使用 list 的話，對於操作兩端元素，是無法做到時間效率均 `O(1)` 的，使用 list 操作 `pop(0)` 或 `insert(0, the_value)` 就會有 `O(n)` 的時間成本，因為要重新對記憶體裡的 list 排順序。操作上 stack 只對頂元素進行操作，所以從 stack 新增/移除 資料的時間複雜度都是 `O(1)`。

# Java 的 Stack 使用

對於 Java 現在的 Stack 類，已經**不建議使用**了。因為 Stack 繼承 Vector 類，其已經棄用。現在推薦使用雙端隊列接口 Deque，其發音為 deck 又稱 **double-ended queue**，它是一個 interface，有兩個 implement，常用的操作為 push / pop / peek :

- **ArrayDeque**
- **LinkedList**

![class圖展示](/images/java/collection-tree.jpg)

```java
//以下兩個都可以
Deque<Integer> stack1 = new LinkedList<>();
Deque<Integer> stack2 = new ArrayDeque<>();
```
Deque 該用 ArrayDeque 還是 LinkedList ？

> 官方建議使用 ArrayDeque

使用 Deque 有一個很大的好處是: Deque 在轉成 ArrayList 時，或者使用 stream 時， Deque 會保持「**後進先出**」的語義:

```
Deque<Integer> deque = new ArrayDeque<>();

deque.push(1);
deque.push(2);

List<Integer> list1 = new ArrayList<>(deque); //[2, 1]
List<Integer> list2 = deque.stream().collect(Collectors.toList()); //[2,1]

```

- ArrayDeque: 底層是 array，容量不夠時需要擴容和數組拷貝，但通常容量不會填滿，會有些空間浪費
- LinkedList: 底層用 Node 串聯，每次 push 都需要 new Node 節點，並且 node 節點裡面有 prev 和 next 成員，也會有額外的空間佔用

# Python 的 Stack 使用

Python 使用 Stack 推薦的做法，是直接利用 list 或套件 `collections.deque`。
```python
stack = [3, 4, 5]
stack.append(6)
stack.append(7)
# stack = [3, 4, 5, 6, 7]

stack.pop() # 7
# stack = [3, 4, 5, 6]

stack.pop() # 6
stack.pop() # 5
# stack = [3, 4]

# Python Stack 若要實現 peek，是使用 索引 -1 來取得 top 元素
stack[-1]


from collections import deque
queue = deque(["John", "Michael"])
queue.append("Terry")        
queue.append("Graham")
queue.appendleft("Eric")

queue.popleft() # 'Eric'
queue.popleft() # 'John'
queue.pop() # 'Graham'             
# queue = deque(['Michael', 'Terry'])
```
有時候也有可能會要你 python 實作 Stack:
```python
class Stack():
    def __init__(self):
        self.container = []

    def push(self, data):
        self.container.append(data)

    def pop(self):
        return self.container.pop()

    def size(self):
        return len(self.container)

    def isEmpty(self):
        return self.container == []
```


---

### 參考資料

- [Stack堆栈解题套路【LeetCode刷题套路教程5】](https://www.youtube.com/watch?v=D_MHAZGtByY&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=5)

- [5.1.1. 將 List 作為 Stack（堆疊）使用](https://docs.python.org/zh-tw/3/tutorial/datastructures.html#using-lists-as-stacks)

- [Python 效能之鬼！你應該學會使用的 deque ！](https://myapollo.com.tw/blog/python-deque/)