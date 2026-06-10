---
title: "Quick Sort"

author: Aryido

date: 2023-09-05T02:13:59+08:00

thumbnailImage: /images/algorithm/quick-sort/logo.jpg

categories:
  - algorithm

tags:
  - recursion

comment: false

reward: false
---

<!--BODY-->
> Quick Sort ，如原本名字所示速度非常快，又稱分割交換排序法。在比較好情形下，時間複雜度為 `O(nlogn)`，往往比 Merge Sort、 Heap sort 等排序算法更快，其基本思想也是**分治法 (Divide and conquer)**。
> 基於 Quick Sort 思想也衍生出了 Quick Selection，在排序序列的同時，選擇出序列中「第 K 小」或是「第 K 大」的元素，因為不需要把整個 array 的排序做完，故其平均時間複雜度為 `O(n)`。

<!--more-->

# 思想
{{< image classes="fancybox fig-100" src="/images/algorithm/quick-sort/pivot.jpg" >}}

以上圖為例來說明，一開始 array 中的元素是亂序的，我們首先在 array 中隨機選擇一個元素，標記為黃色，稱它為 pivot。

接著將其他元素進行一次化分，「**小於 pivot**」 的元素標為藍色，把藍色元素挪到左邊 ;「**大於 pivot**」的元素標為紅色，紅色元素挪到右面。這樣做之後大概會變成如下圖樣子：
{{< image classes="fancybox fig-100" src="/images/algorithm/quick-sort/blue-red.jpg" >}}

{{< alert warning >}}
此時我們只關心藍色元素「**小於 pivot**」; 紅色元素「**大於 pivot**」，至於藍色元素和紅色元素的「內部順序，先用不在意」
{{< /alert >}}

接下來如何給這兩個分成藍紅的子數組排序呢 ? 因為是類似的問題。因此我們可以用同樣的方法，持續遞迴，直到最后一層，此時數組長度是 1 ，單一元素本身就有序的，因此直接返回，而最後最後當遞迴操作完成的時候，整個數組剛好就完成排序了。

# Pseudo Code
```text
# 將 A[l..r] 進行排序
function QUICKSORT(A, l, r)
    if r <= l
        return
    q = PARTITION(A, l, r)
    QUICKSORT(A, l, q - 1)
    QUICKSORT(A, q + 1, r)


# 將數組 A[l..r] 進行隨機劃分
function PARTITION(A, l, r)
    p = [l, r] 之間的隨機整數
    SWAP(A[p], A[r])
    i = l
    for j = l to r - 1
        if A[j] <= A[r]
            SWAP(A[i], A[j])
            i = i + 1
    SWAP(A[i], A[r])
    return i
```

希望對 Array `A` 中 `l` 到 `r` 的閉區間內的元素進行排序：

- 如果 `r <= l`，則說明子數組中只有一個元素，不用做事情直接返回

- 再來開始對這個 array 進行 Partition，其 function 返回值是 `q`，代表了 Pivot 的 index，內容具體做法 :
	- 是 `l` 到 `r` 的閉區間內，**隨機選擇一個元素作為 Pivot**，然後
	- 把 Pivot 拿到數組的最右邊，即將 `A[p]`和`A[r]`交換，會變成下圖(黃色是 Pivot)：
		{{< image classes="fancybox fig-100" src="/images/algorithm/quick-sort/pivot-right.jpg" >}}
	- 設計 `i, j` 兩個指針，一開始都在最左邊的 `l` ，接下來 for-loop `j` 檢查每一個元素。如果:
		- `A[j] <= Pivot`，就將 `A[i]` 和 `A[j]` 互換，並且讓 `i` 和 `j` 指針都前進
		- 否則就只讓 `j` 指針前進
	- 上面 for-loop 完之後，最後再把 `A[i]`和 `Pivot` 互換
		
上面做法為什麼這樣實行完之後，就可以分成兩種 array 呢？ 換一種方式解釋一下，上面做法其實是把 `l` 到 `r` 分成了幾個部分:
{{< image classes="fancybox fig-100" src="/images/algorithm/quick-sort/lijr.jpg" >}}
- `l` 到 `i` 左閉右開區間 : 所有的元素都小於等於 Pivot
- `i` 到 `j` 左閉右開區間 : 所有的元素都大於Pivot
- `j` 到 `r` 左閉右開區間 : 代表還沒看到的部分
- r 位置代表 `Pivot`

最後當 for-loop 結束的時候：
- `[l, i)` : 內所有的元素都小於等於 Pivot
- `[i, r)` : 內所有的元素都大於 Pivot
- `i` index 剛好就是分界點

最後，我們將 `A[i]`和 `A[r]` 互換一下，這樣便將 Pivot 放到了兩類 array 的中間位置。以下給出 python 實現：

```python
import random

def quick_sort(nums: list[int]) -> list[int]:
    quick_sort_range(nums, 0, len(nums) - 1)
    return nums


def quick_sort_range(nums: list[int], left: int, right: int):
    # 如果區間長度 <= 1，就不用排序
    if right <= left:
        return

    # 先做 partition，拿到 pivot 最終位置
    pivot_index = partition_range(nums, left, right)

    # 遞迴排序 pivot 左右兩邊
    quick_sort_range(nums, left, pivot_index - 1)
    quick_sort_range(nums, pivot_index + 1, right)


def partition_range(nums: list[int], left: int, right: int) -> int:
    # 在 [left, right] 之間隨機選一個 pivot
    random_index = random.randint(left, right)

    # 先把 pivot 換到最右邊
    swap(nums, random_index, right)

    pivot_value = nums[right]
	store_index = left
    for scan_index in range(left, right):
        if nums[scan_index] <= pivot_value:
            swap(nums, store_index, scan_index)
            store_index += 1

    # 最後把 pivot 放回中間正確位置
    swap(nums, store_index, right)

    return store_index


def swap(nums: list[int], i: int, j: int):
    nums[i], nums[j] = nums[j], nums[i]
```


# 時間空間複雜度

### 時間複雜度 : `O(N*logN)`
Quick Sort 的時間複雜度非常有意思。最壞的情況是發生在 partition 挑選的 pivot 時總是選到最大或最小值，此刻時間複雜度為 `O(N^2)` 
{{< alert info >}}
這種情形好發於已排序或接近排序完成的資料上。
{{< /alert >}}

最佳情況則發生在每次 pivot 都可以順利選到序列的中位數（median），如此一來，每次遞迴分割的序列長度都會減半，此情況時間複雜度為`O(N*logN)`


### 空間複雜度 : `O(logN)`
本 quick sort 使用了遞迴版本，雖然容易理解，但會影響到空間複雜度。因為每次都需要申請兩個子數列的記憶體空間，遞迴的深度越多，需要記憶體空間就越大。


---

### 參考資料

- [【排序算法精华3】快速排序 (上）](https://www.youtube.com/watch?v=duln2xAZhBA)

- [[演算法] 學習筆記 — 12. 快速排序法 Quick Sort](https://medium.com/@amber.fragments/%E6%BC%94%E7%AE%97%E6%B3%95-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-12-%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F%E6%B3%95-quick-sort-841420575b24)

- [【Day26】\[演算法\]-快速排序法Quick Sort](https://ithelp.ithome.com.tw/articles/10278644)