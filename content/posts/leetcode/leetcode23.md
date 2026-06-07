---
title: "23. Merge k Sorted Lists"

author: Aryido

date: 2022-10-03T21:54:34+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetCode

tags:
  - linked-list
  - heap
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> [這題](https://leetcode.com/problems/merge-k-sorted-lists/)需求是要合併 k 個 linked-list 成一個大的 linked-list，**每個 linked-list 都是有序的**，且大的 linked-list 也必須是有序的，是 [LeetCode21. Merge Two Sorted Lists](https://aryido.github.io/posts/leetcode/leetcode21/) 的進階題，但本題可用 min heap 解題，還蠻巧妙的，故紀錄一下。

<!--more-->

---


# 思路

推薦的解法，是利用 min-heap 資料結構，首先 :

- 定義 `PriorityQueue` :
  - PriorityQueue 內放 Node
  - PriorityQueue 的排序，是看 Node 的 value ，且因為是 min-heap 所以由小排到大

- 因此得到
  ```java
  Queue<ListNode> heap = new PriorityQueue<>((n1,n2) -> n1.val - n2.val);
  ```

接下來把 k 個 linked-list 的 **First Node** 全都加入 heap 中， heap 會把 Node 自動排好序。

接下來開始對 heap 做 while 循環
- 把 poll 出來的元素加入解答 linked-list 中
- **再來把 poll 出來元素的 next 再加回 hea- p 中**(這邊很重要的是，有新的 Node 加入 heap 就會自動排好序了)

以此類推，直到 heap 中沒有元素了。此時 k 個linked-list 就合併為了一個新的大 linked-list，再返回首節點即可。

{{< alert warning >}}
特別注意 java 中， PriorityQueue 的 comparator 寫法，ListNode 是物件，故 PriorityQueue 一定需要 comparator
{{< /alert >}}

{{< alert info >}}
注意題目 Constraints，list 長度有可能是零，需要判斷下
{{< /alert >}}

# 解答

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode() {}
 * ListNode(int val) { this.val = val; }
 * ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
	public ListNode mergeKLists( ListNode[] lists ) {
		if ( lists == null ) {
			return null;
		}

		Queue<ListNode> heap = new PriorityQueue<>( ( n1, n2 ) -> n1.val - n2.val );
		for ( ListNode listNode : lists ) {
			if ( listNode == null ) {
				continue;
			}
			heap.offer( listNode );
		}

		ListNode fakeNode = new ListNode();
		ListNode curr = fakeNode;
		while (!heap.isEmpty()) {
			ListNode node = heap.poll();
			curr.next = node;
			curr = curr.next;

			if ( node.next != null ) {
				heap.offer( node.next );
			}
		}

		return fakeNode.next;
	}
}
```

{{< alert warning >}}
for-loop 把各個 linked-lists 加入時，要注意可能有些首節點是 null，就不要加入 heap。因此有 :

```java
if ( listNode == null ) {
	continue;
}
```

{{< /alert >}}

Python 也蠻多需要注意的地方，首先要熟悉 `heapq` 的操作：
- `heapq.heappush`
-  `heapq.heappop`

再來還要注意 `heapq` 是有比較大小的概念，但 ListNode 物件預設不能直接比較大小，所以會報錯。要解決這個問題最簡單的技巧，可以使用 `tuple`，因為 tuple 預設是會按照順序比較，例如說本題的 trick 可以寫成：

```python
for i, node in enumerate(lists):
    if node:
        heapq.heappush(heap, (node.val, i, node))
```
`()` 匡起來的就是一個 tuple，故 `heapq` 會先比較：
- `node.val`
- 如果 `node.val` 一樣，再比 i
- 前兩個都一樣才會比 node，而這裡的 i 已經保證不同 linked list 之間可以分出順序，所以不可能有機會去使用到 node 本體

```python
class Solution:
    def mergeKLists(self, lists):
        heap = []

        dummy = ListNode(None)
        cur  = dummy

        for i, node in enumerate(lists):
            if node:
                heapq.heappush(heap, (node.val, i, node))
        
        while heap:
            _, i, node = heapq.heappop(heap)

            cur.next = node
            cur = cur.next

            if node.next:
                node = node.next
                heapq.heappush(heap, (node.val, i, node))
        
        return dummy.next
```
{{< alert success >}}
看完 code 不難發現寫成 `(node.val, i, node)` 非常好用，前面的 `node.val` 和 `i` 用來比較大小決定順序，後面的 `node` 用來儲存資訊

{{< /alert >}}



### 時間空間複雜度

假設 k linked-lists 中，總共有 N 個資料元素

##### 時間複雜度: `O(Nlogk)`

- 遍歷所有資料需 `O(N)`
  - 每次插入資料到 heap 中，因為有 k 個要排序，所以花費 `O(logk)` 時間
  - 同時從 heap Pop 出資料 `O(1)`

故總共 `O(Nlogk)`

##### 空間複雜度：`O(k)`

針對 heap 的空間配置，會放入 K 個 Node

---

### 參考資料

- [Heap堆解题套路【LeetCode刷题套路教程6】](https://www.youtube.com/watch?v=vIXf2M37e0k&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=6)

- [思元的開發筆記](https://dev.twsiyuan.com/2017/11/leetcode-merge-k-sorted-lists.html)
