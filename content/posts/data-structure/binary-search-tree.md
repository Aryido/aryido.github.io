---
title: Binary Search Tree

author: Aryido

date: 2022-09-11T14:53:52+08:00

thumbnailImage: "/images/data-structure/BST.jpg"

categories:
- data-structure

tags:
- binary-search-tree
- DFS

comment: false

reward: false
---
<!--BODY-->

> 二元搜尋樹（英語：Binary Search Tree），也稱為有序二元樹（ordered binary tree）或排序二元樹（sorted binary tree）。 從 wiki 上得到的時間與空間複雜度 :
>|  演算法  |  平均  |  最差  |
>|:---:|:---:|:---:|
>| 空間 | O(n) | O(n) |
>| 搜尋 | O(log n) | O(n) |
>| 插入 | O(log n) | O(n) |
>| 刪除 | O(log n) | O(n) |

<!--more-->

---

# 定義
- 以左邊節點 ( left node ) 作為根的子樹 ( sub-tree ) 的所有值都小於根節點 ( root )
- 以右邊節點 ( right node ) 作為根的子樹 ( sub-tree ) 的所有值都大於根節點 ( root )
- 任意節點 ( node ) 的左、右子樹也分別符合 BST 的定義

{{< alert warning >}}
承上定義也可以知道，不存在任何鍵值 ( key/value ) 相等的節點。因為如果有相等值的節點就會違反第一點或第二點。
{{< /alert >}}

---

# 特性
Binary Search Tree 是基於 binary search 而建立的資料結構，所以搜尋的時間複雜度能達成 O(log n)。但不是說使用了 BST 做搜尋就一定可以穩定 O(log n)，搜尋的最差情況是在 O(n) ，關鍵就**平衡**，也就是所謂樹高。因為二元搜尋樹的查詢複雜度取決於深度。為了實現更高效的查詢，產生了平衡樹。

---

## Depth First Traversal
### Pre-Order：適合使用在想要 Copy Tree 時使用
    由上往下，由左至右，依序輸出所有內容。

### In-Order：由小到大依序迭代
    從最底部開始，先把左邊的輸出，再輸出右邊的，如此所有的值就會由小到大依序排列。

### Post-Order： 適合使用在刪除節點時
    由下往上，由左至右。

---

# Exercise
## [LeetCode230. Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

---
# Reference
[wiki](https://zh.wikipedia.org/zh-tw/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9)

[資料結構大便當 — binary search tree](https://medium.com/@Kadai/%E8%B3%87%E6%96%99%E7%B5%90%E6%A7%8B%E5%A4%A7%E4%BE%BF%E7%95%B6-binary-search-tree-3c40be3204e)

[PJCHENder Binary Search Tree](https://pjchender.dev/dsa/dsa-bst/)

