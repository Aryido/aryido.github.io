---
title: "21. Merge Two Sorted Lists"

author: Aryido

date: 2022-10-03T20:43:23+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetCode

tags:
  - linked-list
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> [LeetCode21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)這道題目主要是熟悉 linked-list 相關重點，例如：
> - dummy head
> - 當新 Linked List 加完所有的節點後，需要返回其頭節點的 next，即 `dummy.next`
> 
> 這些技巧在後續解決 Linked List 題目時會反覆用到。

<!--more-->

---

# 思路
最直觀的暴力解法是：

- 把所有的 linked-list 的 Node value ，塞到一個新 array list 中然後**排序**
- 新建一個 dummy node，用於存儲合併後的鏈表，並用一個指針 cur 來構建新鏈表

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        val_list1 = []
        while list1 is not None:
            val_list1.append(list1.val)
            list1 = list1.next
        
        val_list2 = []
        while list2 is not None:
            val_list2.append(list2.val)
            list2 = list2.next

        val_list = val_list1 + val_list2
        val_list.sort()

        dummy = ListNode(None)
        cur = dummy

        for val in val_list:
            cur.next = ListNode(val)
            cur = cur.next

        return dummy.next
        
```
{{< alert danger >}}
一開始用 `val_list = [node.val for node in list1] + [node.val for node in list2]` 有報錯，因為
`'ListNode' object is not iterable`，相關說明可以參考 [Python : Iterable, Iterator, Generator 綜合介紹](https://aryido.github.io/posts/python/iterable-iterator-generator/)
{{< /alert >}}

假設 k Lists 中，共有 N 個 Node:

- 所有資料要塞到一個 list 中做排序演算法，故空間複雜度`O(N)`
- 遍歷所有資料，並塞入到陣列需花費`O(N)`，再來使用排序算法 Quick Sort，一般情況需花費 `O(NlogN)` 時間，故時間複雜度`O(NlogN)`

上解法在理論上時間複雜度還可以優化。由於鏈表本身是遞增排序的，處理過程中只需按照大小順序逐步拼接兩個鏈表即可，所以推薦的解法如下：

# 解答

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        if list1 is None:
            return list2
        
        if list2 is None:
            return list1
        
        dummy = ListNode(None)
        cur = dummy

        while list1 and list2:
            list1_node_val = list1.val
            list2_node_val = list2.val

            if list1_node_val < list2_node_val:
                cur.next = list1
                cur =  cur.next
                list1 = list1.next
            else:
                cur.next = list2
                cur =  cur.next
                list2 = list2.next
        
        if list1 is None:
            cur.next = list2
        
        if list2 is None:
            cur.next = list1

        return dummy.next
```

假設 k Lists 中，共有 N 個 Node:

- 僅使用了常數空間來存儲指針，故空間複雜度`O(1)`
- 要遍歷兩個鏈表的所有節點，故時間複雜度`O(N)`


還有遞歸解法，寫起來更簡潔:
```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        if list1 is None:
            return list2
        
        if list2 is None:
            return list1
        
        list1_node_val = list1.val
        list2_node_val = list2.val

        if list1_node_val < list2_node_val:
            list1.next = self.mergeTwoLists(list1.next, list2)
            return list1
        else:
            list2.next = self.mergeTwoLists(list1, list2.next)
            return list2
```
假設 k Lists 中，共有 N 個 Node:

- 遞歸深度 N，故空間複雜度`O(N)`
- 要遍歷兩個鏈表的所有節點，故時間複雜度`O(N)`


### 整理

| 解法       | 空間複雜度 | 時間複雜度 |
| ---------- | ---------- | ---------- |
| 暴力排序法 | `O(N)`     | `O(NlogN)` |
| 迭代合併法 | `O(1)`     | `O(N)`     |
| 遞迴合併法 | `O(N)`     | `O(N)`     |


---

### 參考資料

- [\[LeetCode解題攻略\] 21. Merge Two Sorted Lists](https://vocus.cc/article/67550b14fd8978000178ca22)

- [圖解LeetCode(入門篇): 21. Merge Two Sorted Lists](https://ithelp.ithome.com.tw/m/articles/10344658)
