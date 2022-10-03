---
title: 23. Merge k Sorted Lists

author: Aryido

date: 2022-10-03T21:54:34+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- heap

comment: false

reward: false
---
<!--BODY-->
>這題是合併 k 個 linked-list，且每個linked-list 都是有序的，最終要合併成一個大的 linked-list ，且也必須是有序的。用 heap 解題還蠻巧妙的，故紀錄一下...
<!--more-->

---

## 思路
利用min-heap，首先把 k 個 linked-list 的首元素都加入 heap 中，會自動排好序。然後把 poll 出來的元素加入解答 linked-list 中，然後把取出元素的 next 再加回 heap 中，下次仍繼續 poll 操作，以此類推，直到堆中沒有元素了，此時 k 個linked-list 就合併為了一個新的大 linked-list，返回首節點即可。

# 解答
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        // check edge case
        if(lists == null ){
            return null;
        }
        if(lists.length == 1 && lists[0] == null ){
            return null;
        }

        // init create min-heap and answer node
        ListNode fakeNode = new ListNode();
        Queue<ListNode> heap = new PriorityQueue<>((n1,n2) -> n1.val - n2.val);
        // put all listNode into heap
        for(ListNode listNode : lists){
            if(listNode == null){
                continue;
            }
            heap.offer(listNode);
        }

        ListNode curr = fakeNode;
        // poll first node and add this node to fakeNode next (while loop)
        while(!heap.isEmpty()){
            ListNode node = heap.poll();
            curr.next = node;
            curr = curr.next;

            if(node.next != null){
                heap.offer(node.next);
            }

        }


        return fakeNode.next;
    }
}
```

{{< alert warning >}}
- 注意 edge case 的判斷
- ListNode 是物件，heap 需要comparator
- for-loop 把各個 linked-lists 加入時，要注意有些首節點是 null，不要加入 heap
{{< /alert >}}


---