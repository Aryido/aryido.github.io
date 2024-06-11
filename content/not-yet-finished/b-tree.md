---
title: B Tree

author: Aryido

date: 2023-11-25T16:23:31+08:00

thumbnailImage: /images/data-structure/b-tree-logo.jpg

categories:
- data-structure

tags:
- tree

comment: false

reward: false

---

<!--BODY-->
> B-tree 是一種自平衡的樹，能夠保持資料的有序性，能讓搜尋、插入、刪除都在 logN 時間內完成。B-tree 是一個廣義化的 binary search tree，可以擁有多於 2 的 sub-Node。
> B-tree 與自平衡 binary search tree 最大不同，是 B-tree 做了讀寫操作做的優化，減少定位記錄時所經歷的中間過程，從而加快存取速度。常應用在 database 和 file-system 實現上。

<!--more-->

---

# B-tree 定義

B-tree 是一顆**多路平衡查找樹((Multi-way Search Tree)**，它可以高效的存儲和查找資料。描述 B-tree 時需要指定它的階數，階數表示了一個 Node 最多有多少個 sub-Node ，一般用字母 m 表示階數。而 Node 內會稱為 **record** 的東西，為 key 和其對應的 data 的鍵值對。

### 一顆 m 階的 B-tree 定義如下：

- 每個 Node 最多有 m-1 個 key。

- Root 最少可以只有 1 個 key。

- 每個 Node 中的 key，都按照從小到大的順序排列；每個 key 的左子樹中的所有 key 都小於它，而右子樹中的所有 key 都大於它。

- 所有 Leaf 都位於同一層，或者說 Root 到每個葉 Leaf 的長度都相同。

{{< alert info >}}
多路平衡查找樹 (Multi-way Search Tree) 是一種樹狀資料結構，其中每個節點可以有**多個子節點**。
{{< /alert >}}

{{< alert success >}}
當 m 取 2 時，就是我們常見的二叉搜索樹。
{{< /alert >}}

### 範例

{{< image classes="fancybox fig-100" src="/images/data-structure/b-tree.jpg">}}

上圖是一顆階數爲 4 的 B-tree。每個結點中存儲了一些 record ，另外還有 sub-Node 內的指針。

{{< alert info >}}
在實際應用中的 B-tree 的階數 m 都非常大（通常大於 100），所以即使存儲大量的數據，B-tree 的高度仍然不會太高。
{{< /alert >}}

{{< alert success >}}
在 database 中就是用 B-tree 結構(最精確說明是**B+tree**)作爲索引
{{< /alert >}}

---

# B-tree 性質
根據 B 樹的性質可以得出結論：

- Node 的 sub-Node 爲 key 數量 + 1；
- root 沒有 key 的話，相當於沒有子樹，也即是 empty-tree；若 root 有 key ，則 sub-tree 必大於兩棵；

- Node 中 key 自左向右遞增， key 兩側均有指向 sub-tree 的指針。
  - 左邊指針所指 sub-tree 的所有 key 均小於該 key
  - 右邊指針所指的 sub-tree 的所有 key 均大於該 key


---
### 參考資料

- [解密 B 樹與 B - 樹：構建穩定可擴展的數據結構](https://www.readfog.com/a/1718792051177394176)

