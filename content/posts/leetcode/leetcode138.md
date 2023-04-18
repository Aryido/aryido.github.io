---
title: "138. Copy List with Random Pointer"

author: Aryido

date: 2023-04-11T23:40:06+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> deep copy 是程式初新手常會有疑問的地方，這邊剛好可以藉由解題來複習指標的概念。另外這題有很巧妙的解法，就是利用 HashMap 來建立原 node 和拷貝 node 之間的映射 ! 也可以使用遞迴的解法，寫起來相當的簡潔。
<!--more-->

---

## deep copy & shallow copy
設想有一個**物件 ListNode**，當我們使用 ```ListNode a = new ListNode(1)``` 時，內存發生了什麼 ?
- 對於等號右邊 ```new ListNode(1)```，具體來說就是在**內存中的 heap 內**，開闢了一個內存空間給 ```new ListNode(1)```。
- 對於等號左邊， a 代表是物件的 reference，是一個指針，是在**內存中的 stack 內**， a 其實拿的是 ```new ListNode(1)``` 在內存中的地址。

{{< image classes="fancybox fig-100" src="/images/leetcode/deep-and-shallow-copy.jpg" >}}

---

## 思路
這題的關鍵點/難點在於 random 的複製和查找。扣除掉這點，只要遍歷原本的linked list，把每個 node 的 val 取出來，再創建新的 node 把 val 放入，再移動到 node 的 next ，如法炮製，再把前一個 新node 指到這一個新 node 就可以了。大概如下 :
```java
Node cur = head;
Node new_head = new Node(head.val);
Node new_cur = new_head;
while(cur != null){
    cur = cur.next;
    Node new_node = new Node(cur.val);
    new_cur.next = new_node;
    new_cur = new_cur.next;
}
return new_head;
```
但現在多了一個 random，要如何找到新的 random 呢？

第一次遍歷生成所有新 node ，同時建立一個**原 node 和新 node 的 HashMap** ；再來第二次遍歷就給 random 和 next 賦值。最巧妙的就是建立的 HashMap ，已經暗示了 random 要如何找到，查找時間是常數級，大大加快了速度。

# 解答
```java
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null){
            return null;
        }

        Map<Node,Node> map = new HashMap<>();

        Node cur = head;
        while(cur != null){
            map.put(cur, new Node(cur.val));
            cur = cur.next;
        }

        Node cur2 = head;
        while(cur2 != null){
            map.get(cur2).next = map.get(cur2.next);
            map.get(cur2).random = map.get(cur2.random);
            cur2 = cur2.next;
        }

        return map.get(head);
    }
}
```

{{< alert warning >}}
最關鍵的地方就是:
```
map.get(cur2).random = map.get(cur2.random);
```
運用了 HashMap ，並找到複製的 random node。
{{< /alert >}}

---
