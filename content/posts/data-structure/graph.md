---
title: Graph 介紹

author: Aryido

date: 2022-09-13T21:37:08+08:00

thumbnailImage: "/images/data-structure/graph.jpg"

categories:
- data-structure

tags:
- graph
- DFS
- BFS
- tree
- list
- best first search

comment: false

reward: false
---
<!--BODY-->

> Graph 是廣義化的 LinkedList，是內存中不一定連續的資料，每個節點會一個或多個 Reference 指向其他節點
> - 可能有環
> - 分無向圖和有向圖
> - 沒有固定入口
> - 可能有多個入口

<!--more-->
## Tree
Graph 也是廣義化的 Tree，抽象程度 Graph >> Tree >> LinkedList。是內存中不一定連續的資料，每個節點會一個或多個 Reference 指向其他節點
- 節點分成 Parent、Child
- 可看作是無環圖
- 只有一個入口


## DFS(Depth-First Search)
DFS會選一 Reference ，一口氣到遞歸到最下層再沿途回去，沿途若還有其他條路可以走，也會一口氣到遞歸到最下層，最後回到原點。回到原點之後再選另一 Reference，如此往復。基本上不會允許訪問一節點多次，故需要檢查重複。
時間複雜度:
{{< alert info >}}
**O(N * k), k = max(pre-order-time,  post-order-time)**
{{< /alert >}}


## BFS(Breadth-First Search)
因為是先找該節點所有的 Reference，故適合尋找最短路徑，但要注意找最短路徑只適用於 Uniform Cost(每條 edge 的 Weight 一樣)。基本上也不會允許訪問一節點多次，故也需要檢查重複。
時間複雜度:
{{< alert info >}}
**O(V + E)**
{{< /alert >}}

## BFS(Best-First Search)
雖然也稱 BFS，但實際名稱不一樣，是Best-First Search。  BFS 是針對 Non-uniform cost graph 的一種算法，核心思想是優先展開最'優'的點。如何每次快速計算最優路徑呢 ? 可以使用 Heap 這資料結構來幫忙。
最有名找最短路徑的算法是 Dijsktra's Algorithm。時間複雜度:
{{< alert info >}}
**O((V + E) * logV)**
{{< /alert >}}

---


