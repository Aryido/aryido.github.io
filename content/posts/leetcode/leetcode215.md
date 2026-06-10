---
title: "215. Kth Largest Element in an Array"

author: Aryido

date: 2022-10-03T21:54:34+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetCode

tags:
  - heap
  - python

comment: false

reward: false
---

<!--BODY-->
> [尋找第 k 大的數字](https://leetcode.com/problems/kth-largest-element-in-an-array/description/)，注意是排序後的第 k 個最大元素，而不是第 k 個不同的元素。解題上是經典的使用 **priority_queue** 的題目，但對於這個問題，還可以使用 quick select ，也是個蠻好的解法。

<!--more-->

---

# 思路
假設 n 代表 array 的長度，而 k 就是要取第 k 個最大元素：

### **直接排序（Sorting）** 

時間複雜度是: `O(nlogn)` 此為最簡單的 brute force 解，就是先將陣列排序，再取出第 k 個元素。由於在同一個 array 排序，故空間複雜度為 `O(1)`。

### **Heap 資料結構**
- **Max Heap** `O(klogn)`
  - 先把原 array 使用 `heapify` 建成 Heap，這個操作的時間複雜度是 `O(n)`。由於預設是 min-heap，所以可用負號技巧轉成 max-heap
  - 再取出 k 次堆頂元素，因為每次取出最大值都要做一次 `O(logn)` 的調整，所以總共是`O(klogn)`
  {{< alert warning >}}
如果不是用 heapify，而是把 n 個元素一個一個 push 進 heap，則每次 push：`O(logn)`， n 次就是 `O(nlogn)`
  {{< /alert >}}
  - 因為要另外空間儲存原始 array 資料，故空間複雜度為 `O(n)`

- **Min Heap** `O(nlogk)`
  - 維護一個大小為 k 的 min-heap，故空間複雜度為 `O(k)`
  - 如果新元素大於或等於堆頂元素，就加入min-heap，數亮超過時要調整 heap 的大小
  - 最後堆頂元素就是第 k 大

  因為我們每次加入堆中的，都是較大的元素，所以最後 Min Heap 的 top，雖然是 Heap 裡面最小的元素，但它已經大於 array 中全部其他沒留在 heap 裡的元素：換句話說，它就是整個 array 中的第 k 大元素。


---

# 解答

先給 Heap 的解法：

### Max Heap

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        nums = [-x for x in nums]
        heapq.heapify(nums)

        ans = 0
        for _ in range(k):
            ans = -heapq.heappop(nums)

        return ans      
```
heapq 預設是 min heap，技巧上可以先把 nums 的值都加上負號，這樣概念上就是轉為 max heap 了，最後答案記得再把負號轉回去。

### Min Heap
```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        heap = []

        for x in nums:
            if len(heap) < k:
                heapq.heappush(heap, x)
            elif x > heap[0]:
                heapq.heapreplace(heap, x)

        return heap[0]
  
```
當 heap 還沒滿 k 個時，都要把 `nums[i]` 放入，等到 heap 滿了之後再做比較。這裏就是使用 `heapreplace` 的好時機。


### Quick Select Algorithm

比較特別的是，這題主要是考察 Quick Select ，可以把時間複雜度平均下降到 `O(n)`，當然最壞的情況會是 `O(n^2)`，會用到了快速排序 Quick Sort 的思想。

核心是每次都要先找一個「中樞點 Pivot」，然後遍歷其他所有的數字，像這道題需要從大往小排：
- 把大於中樞點的數字放到左半邊
- 把小於中樞點的放在右半邊

這樣中樞點是整個數組中第幾大的數字就確定了。如果:
- **正好是** `k-1`，那麼直接返回該位置上的數字
- **大於** `k-1`，說明要求的數字在左半部分，更新右邊界，再求新的中樞點位置
- **反之**，則更新右半部分，求中樞點的位置；

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        n = len(nums)

        target = n - k

        left_index = 0
        right_index = n - 1
        while left_index <= right_index:
            index = self.partition(nums, left_index, right_index)

            if index == target:
                return nums[target]
            elif index > target:
                right_index = index - 1
            else:
                left_index = index + 1


    def partition(self, nums: List[int], left_index: int, right_index: int) -> int:
        pivot_index = random.randint(left_index, right_index)
        pivot = nums[pivot_index]

        self.swap(nums, pivot_index, right_index)

        i = left_index
        for j in range(left_index, right_index):
            if nums[j] <= pivot:
                self.swap(nums, i, j)
                i += 1
        
        self.swap(nums, i, right_index)
        return i

    
    def swap(self, nums: List[int], i:int, j:int):
        nums[i], nums[j] = nums[j], nums[i]  

```
`target = n - k` 為什麼呢 ？ 可以思考一下，nums 是小到大排序，那麼：
- 第 1 大 index 是 : `n - 1`
- 第 2 大是 index 是 : `n - 2`

以此類推，第 k 大 index 就是 : `n - k`。接下來思考一下， `index > target` 代表什麼意思，可以先想成是下圖：
```text
0    1    2    3    4    5    6
|----|----|----|----|----|----|
       target     index
```

因為 `index > target`，如圖 index 一定落在 target 的右邊，代表答案不可能在右側，答案一定在左半邊，所以把 `right_index` 縮小成 `index - 1`

雖然 quick selection 解法很不錯，但是現在這種一般寫法是會 **Time Limit Exceeded** 的，原因是有 test_case 是 `[1,2,3,4,5,1,....(too many 1),-5,-4,-3,-2,-1 ]`， array 裡有很多和 pivot 一樣的值時，會導致 partition 每次只縮小一點點，時間退化到 `O(n^2)`。

這時候就要稍微進階一點使用 **3-way partition** 技巧，partition function 修改成會回傳一個區間 `[lt, gt]`，意義上其區間內值都是一樣的，這樣遇到很多重複值時，可以一次全部排除:

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        n = len(nums)
        target = n - k

        left_index = 0
        right_index = n - 1

        while left_index <= right_index:
            lt, gt = self.partition(nums, left_index, right_index)

            if target < lt:
                right_index = lt - 1
            elif target > gt:
                left_index = gt + 1
            else:
                return nums[target]

    def partition(self, nums: List[int], left_index: int, right_index: int) -> tuple[int, int]:
        pivot_index = random.randint(left_index, right_index)
        pivot = nums[pivot_index]

        lt = left_index
        i = left_index
        gt = right_index

        while i <= gt:
            if nums[i] < pivot:
                self.swap(nums, lt, i)
                lt += 1
                i += 1
            elif nums[i] > pivot:
                self.swap(nums, i, gt)
                gt -= 1
            else:
                i += 1

        return lt, gt

    def swap(self, nums: List[int], i: int, j: int) -> None:
        nums[i], nums[j] = nums[j], nums[i]
```
修正的重點是：
- 原本 partition 只回傳一個 index，現在改成回傳 tuple `(lt, gt)`
- `nums[lt:gt+1]` 這一段內容都是 pivot
- 如果 target 落在這段裡，就直接找到答案

### 時間空間複雜度

假設 nums 總共有 n 個資料元素

##### 時間複雜度: `O(n)`

quick select 跟 quick sort 最大差別是 quick sort 每次 partition 後，左邊要排且右邊也要排 ; 但 quick select 每次 partition 後，只會選則一邊做，所以平均為: `O(n)`

{{< alert warning >}}
quick select 和 quick sort，兩個是不一樣的
{{< /alert >}}


##### 空間複雜度：`O(1)`
若用遞迴，則平均空間 `O(logn)`，最差是 `O(n)`

---

### 參考資料

- [Heap堆解题套路【LeetCode刷题套路教程6】](https://www.youtube.com/watch?v=vIXf2M37e0k&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=6)

- [思元的開發筆記](https://dev.twsiyuan.com/2017/11/leetcode-merge-k-sorted-lists.html)

- [[LeetCode] 215. Kth Largest Element in an Array 数组中第k大的数字](https://www.cnblogs.com/grandyang/p/4539757.html)
