---
title: "23. Merge k Sorted Lists"

author: Aryido

date: 2022-10-03T21:54:34+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- java
- heap

comment: false

reward: false
---
<!--BODY-->
>這題需求是要合併 k 個 linked-list 成一個大的 linked-list，**每個 linked-list 都是有序的**，且大的 linked-list 也必須是有序的，是 LeetCode21. Merge Two Sorted Lists 的進階題，但本題可用 min heap 解題，還蠻巧妙的，故紀錄一下。
<!--more-->

---

## 思路
最直觀的暴力解法，是把所有的 linked-list 的 Node value ，塞到一個新 array list 中然後**排序**，最後再做出新的 linked-list。假設 k Lists 中，共有 N 個 Node:
- 所有資料要塞到一個 list 中做排序演算法，故空間複雜度```O(N)```
- 遍歷所有資料，並塞入到陣列需花費```O(N)```，再來使用排序算法 Quick Sort，一般情況需花費 ```O(NlogN)``` 時間，故時間複雜度```O(NlogN)```

| 解法             | 空間複雜度 | 時間複雜度 |
|------------------|----------|----------|
| 暴力排序法       | O(N)     | O(NlogN) |

推薦的解法，是利用 min-heap 資料結構，首先 :
- 定義 ```PriorityQueue``` :
  - PriorityQueue 內放 Node
  - PriorityQueue 的排序，是看 Node 的 value ，且因為是 min-heap 所以由小排到大

- 因此得到
    ```java
    Queue<ListNode> heap = new PriorityQueue<>((n1,n2) -> n1.val - n2.val);
    ```

接下來把 k 個 linked-list 的 **First Node** 全都加入 heap 中， heap 會把 Node 自動排好序。然後把 poll 出來的元素加入解答 linked-list 中；**再來把 poll 出來元素的 next 再加回 heap 中**，這邊很重要的是，有新的 Node 加入  heap 就會自動排好序。下次仍繼續 poll 操作，以此類推，直到 heap 中沒有元素了。此時 k 個linked-list 就合併為了一個新的大 linked-list，再返回首節點即可。

{{< alert warning >}}
特別注意 java 中， PriorityQueue 的 comparator 寫法，ListNode 是物件，故 PriorityQueue 一定需要 comparator，**由小到大**、**由大到小** 不要寫錯了。
{{< /alert >}}

{{< alert info >}}
注意題目 Constraints，list 長度有可能是零，代表一個 corner case 需要判斷下!
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

{{< alert info >}}
注意 edge case 的判斷
{{< /alert >}}

{{< alert warning >}}
for-loop 把各個 linked-lists 加入時，要注意有些首節點是 null，不要加入 heap
{{< /alert >}}

---

# 時間空間複雜度
假設 k linked-lists 中，總共有 N 個資料元素
### 時間複雜度: ```O(Nlogk)```
遍歷所有資料需 ```O(N)```，每次插入資料到 heap 中，因為會排序所以花費 ```O(Logk)``` 時間排序，從 heap Pop 出資料 ```O(1)```，故總共 ```O(Nlogk)```

### 空間複雜度：```O(k)```
針對 heap 的空間配置，會放入 K 個 Node

---
# Vocabulary

{{< alert info >}}
**ascending**[əˋsɛndɪŋ]

adj.上升的；ascend的動詞現在分詞、動名詞
{{< /alert >}}

{{< alert info >}}
**descending** [dɪˋsɛndɪŋ]

adj.下降的；descend的動詞現在分詞、動名
{{< /alert >}}


---

### 參考資料

- [Heap堆解题套路【LeetCode刷题套路教程6】](https://www.youtube.com/watch?v=vIXf2M37e0k&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=6)

- [思元的開發筆記](https://dev.twsiyuan.com/2017/11/leetcode-merge-k-sorted-lists.html)
