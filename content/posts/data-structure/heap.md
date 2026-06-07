---
title: Heap 資料結構

author: Aryido

date: 2022-09-14T05:06:03+08:00

thumbnailImage: "/images/data-structure/heap.jpg"

categories:
  - data-structure

tags:
  - heap
  - python

comment: false

reward: false
---

<!--BODY-->
> Heap 的定義是「父節點和子節點之間滿足固定大小關係的樹」。當所有父節點都**大於**其子節點時，就稱其為「Max Heap」; 當所有父節點都**小於**其子節點時，就稱其為「Min Heap」，[附上一個 Heap 圖參考連結](https://www.geeksforgeeks.org/dsa/heap-data-structure/)。雖然 Heap 本身定義上**只關心節點之間的順序規則**，所以可以有很多種類例如： d-ary heap、binomial heap、Fibonacci heap 等等，但很多時候教材講的 Heap，其實默認會是 complete binary heap ，也就是除了滿足 heap property 還滿足了：
> - **complete**: 代表同一階層要由左到右排列，不能跳過
> - **binary**: 每個 node 最多只能有兩個 child
> 
> Heap 常被作為 Priority Queue 的實作方式，且在一些圖論(例如 Dijkstra)或排序(Heap sort)的演算法中扮演極其重要的地位。
<!--more-->

---

{{< alert warning >}}
資料結構上的 Heap 和 討論記憶體時所提到的 Heap ，這兩個完全不一樣。

- 資料結構的 Heap 是一種樹狀資料結構，常拿來實作 Priority Queue 

- 記憶體裡的 Heap，是指程式執行時的一塊「動態記憶體區域」，特性是：
    - 執行時才配置大小
    - 生命週期不一定跟函式結束同步
    - 需要手動釋放，或交給 GC（Garbage Collector）
{{< /alert >}}

由於我們經常默認默認會是 「**complete binary heap**」 ， 這也是為什麼 binary heap 很適合用 array 表示，因為完全二元樹的結構很緊密，中間不會亂空。這個 Tree 的 root 節點代表整個 Heap 裡的最大（max heap）或者最小值（min heap）。
```text
        100
       /   \
     19     36
    /  \   /  \
  17   3 25    5
 /  \
9    15  ....
....
```
可以用一個 list 來表示，由上到下，由左至右的排列在 list 中：

{{< image classes="fancybox fig-100" src="/images/data-structure/heap-list.jpg" >}}

從上圖中可以發現，「**Heap 不是完全排序過的資料結構**」，只有部份有序 ; 而且同一個 level 中，其實 node 之間並沒有什麼特別的關係。

# 特性

假設根節點的 `index = 0`，若某節點在索引 i：
- 左子節點在 `2i + 1`
- 右子節點在 `2i + 2`
- 父節點在 `(i - 1) / 2 取整數`

舉例來說 `node 19` 的 index 是 `1`：
- `node 19` 的 left child `node 17` 的 index 就是 3
- `node 19` 的 right child `node 12` 的 index 就是 4

若已知 `node 17` 的 index 就是 3 ; `node 12` 的 index 就是 4 ，他們的 parent node index 就是 1





# Python 的 Priority Queue

Python 的 Priority Queue 放在名為 `heapq.py`的模組中，LeetCode 環境已經先將它 import 進來了，所以可以直接應用，想要看完整的說明可以查 [heapq --- 堆積佇列 (heap queue) 演算法](https://docs.python.org/zh-tw/3/library/heapq.html)文件。

使用時注意以下事情：
- 預設是個 `min-heap`
- 如果要用 `max-heap`，一個簡單的方法是把數值正負反轉
- 增加 element 就用 `heappush(pq, element)`
- 取出(刪除)最小值就用 `heappop(pq)`
- 如果 list 中一開始有資料，可以用 heapify 來初始化。會在線性時間內將 list 轉為 heap，且過程不會申請額外記憶體
- 插入與刪除的時間複雜度都是 `O(logN)`


Priority Queue 最好的運用的時機，是要針對一組流數據，沒有固定長度的資料群中找出最小值(或最大值) ; 如果資料不會變動，那用排序就足夠，也就是：
- **Online** Algorithm（using Heap）：
    針對 stream data，可以隨時根據需求 scalable

- **Offline** Algorithm（using Sorting）：
    針對固定長度資料，每次 scale 後要重新計算



# 題目

[排隊買飲料](https://tioj.ck.tp.edu.tw/problems/1999)題目，限制條件：
- 每個店員一次只能服務一名顧客
- 店員必須按照順序接客

假如所有的店員製作飲料的速度都是每分鐘 1 杯，服務完這些客人至少需要幾分鐘?

```python
import heapq

# n = 5 排隊等待的客人數量
# m = 2 店員數量
# time_list = [4, 2, 3, 1, 5] 購買的飲料數量

def service_time(n, m, time_list):
    k = min(n, m)

    # 前 k 位顧客先分配給 k 位店員
    heap = time_list[:k]
    heapq.heapify(heap)

    ans = max(heap) if heap else 0

    # 剩下顧客依序交給最早空下來的店員
    for t in time_list[k:]:
        earliest = heapq.heappop(heap)
        finish = earliest + t
        if finish > ans:
            ans = finish
        heapq.heappush(heap, finish)

    return ans
```
以 `time_list = [4, 2, 3, 1, 5]` 為範例，進入迴圈時要處理 `[3, 1, 5]`

- 第 1 輪
    - 目前顧客需要 3 分鐘，`t = 3`
    - 目前 heap: `[2, 4]`
    - 最早空下來的店員，`heapq.heappop(heap)` 是 2，表示這位店員在第 2 分鐘空出來
    - `finish = earliest + t = 2 + 3 = 5`
    - `ans` 更新成 5 ; 店員新的忙碌結束時間放回 heap，heap 變成： `[4, 5]`

- 第 2 輪
    - heap 最後變成 `[5, 5]`，意思是兩位店員都會在第 5 分鐘空下來

- 第 3 輪
    - 目前顧客需要 `t = 5` 分鐘時間 ; 目前 heap `[5, 5]`，代表 `earliest = 5`
    - 這位店員從第 5 分鐘開始做，做 5 分鐘：`finish = 5 + 5 = 10`，所以 ans 應該更新為 10
    - 放回 heap 變成 `[5, 10]`

所以所有顧客飲料全部做完時間是 10 mins。

---

### 參考資料

- [Heap （堆）](https://wdv4758h.github.io/notes/algorithm/heap.html)

- [來征服資料結構與演算法吧 | 搞懂 Binary Heap 的排序原理](https://medium.com/starbugs/%E4%BE%86%E5%BE%81%E6%9C%8D%E8%B3%87%E6%96%99%E7%B5%90%E6%A7%8B%E8%88%87%E6%BC%94%E7%AE%97%E6%B3%95%E5%90%A7-%E6%90%9E%E6%87%82-binary-heap-%E7%9A%84%E6%8E%92%E5%BA%8F%E5%8E%9F%E7%90%86-96768ea30d3f)

- [Heap Sort 的步驟](https://medium.com/@Kadai/%E8%B3%87%E6%96%99%E7%B5%90%E6%A7%8B%E5%A4%A7%E4%BE%BF%E7%95%B6-binary-heap-ec47ca7aebac)

- [Python-LeetCode 581 第五招 優先佇列 Priority Queue/Heap](https://hackmd.io/@bangyewu/ryLbEED23/%2FQPaqO5-HRF-PLuoBjlKwwA)