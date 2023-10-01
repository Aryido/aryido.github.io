---
title: "Merge Sort"

author: Aryido

date: 2023-09-04T21:03:48+08:00

thumbnailImage: /images/algorithm/merge-sort.jpg

categories:
- algorithm

tags:
- recursion

comment: false

reward: false
---
<!--BODY-->
> **歸併排序 (Merge Sort)** 算是比較優秀的排序算法，因為時間複雜度是 ```O(N log N)```；而選擇排序、冒泡排序、和插入排序時間複雜度則是```O(N^2)```。 Merge Sort 的基本思想是**分治法 (Divide and conquer)**，是將原問題分解為規模較小的子問題，然後逐一解決這些子問題之後，**合併這些子問題的答案**，並建立原問題的答案。

<!--more-->
---

# 思考
按照 Recursion Template 來設計 mergeSort 這個函數 :


### {{< hl-text blue >}}
定義 Recursion 函數
{{< /hl-text >}}

mergeSort 這個函數的作用是將 List 中 ```[l, r)``` 區間內的元素進行排序。使用起始下標、終止下標可以用來表示，要處理的是屬於**原 List** 還是**子 List**，滿足逐步縮小問題規模的思想。

{{< alert success >}}
符合**原問題和子問題都可以調用的形式**
{{< /alert >}}
### {{< hl-text blue >}}
Base case處理
{{< /hl-text >}}

再來檢查待排序 List 內元素個數，看看是否小於等於 1，即  ```r-l <= 1 ```。如果是的話就是 Base case ! 代表 array 只有一個元素，那麼不用排序了，因為這本身就是有序，直接返回。

{{< alert success >}}
**基礎情況是為了避免遞歸無限繼續下去無法停止**
{{< /alert >}}


### {{< hl-text blue >}}
遞迴調用
{{< /hl-text >}}
若 List 並非只有一個元素，則會取 *l* 和 *r* 中間的那個元素，即  ```int m=（l+r）/2```**(java 會向下取整)**，然後調用遞迴函數，分別對 array 中:
- ```List([l, m))``` 元素進行排序
- ```List([m, r))``` 元素進行排序

{{< alert info >}}
因為對原本大的 List 排序，以及對切半後的小 List 排序，其實都是一樣的問題。以這種思考去看，可以自然知道為何使用遞迴來解，會變得比較簡單的原因了。
{{< /alert >}}

### {{< hl-text blue >}}
構造當前層狀態答案
{{< /hl-text >}}

調用 merge 函數，來合併這兩個已經排序好的 List。

{{< alert warning >}}
merge 步驟其實 Merge Sort 內是最重要的一個步驟
{{< /alert >}}

---

# Merge Sort

```java
public class MergeSortExample {

	public void mergeSort( List<Integer> arr, int left, int right ) {
		if ( right - left <= 1 ) {
			return;
		}

		int mid = (left + right) / 2;

		mergeSort( arr, left, mid );
		mergeSort( arr, mid, right );

		merge( arr, left, right );

	}

	public void merge( List<Integer> arr, int left, int right ) {
		int mid = (left + right) / 2;

		// Note that mid is not included in L and is the starting point for R
		List<Integer> L = new ArrayList<>( arr.subList( left, mid ) );
		List<Integer> R = new ArrayList<>( arr.subList( mid, right ) );

		int i = 0, j = 0;
		int k = left;
		while (i < L.size() && j < R.size()) {
			if ( L.get( i ) <= R.get( j ) ) {
				arr.set( k, L.get( i ) );
				i++;
			} else {
				arr.set( k, R.get( j ) );
				j++;
			}
			k++;
		}

		while (i < L.size()) {
			arr.set( k, L.get( i ) );
			i++;
			k++;
		}

		while (j < R.size()) {
			arr.set( k, R.get( j ) );
			j++;
			k++;
		}
	}
}
```
---

# 時間空間複雜度
假設 array 內共有 ```N``` data。要分析 Merge Sort 的時間空間複雜度的話，可以先觀察 mergeSort function ，然後專注在 **merge 合併操作**和 **Recursion Tree**。

### 時間複雜度 : ```O(N logN)```

觀察 MergeSort Function ，內部有幾個操作:

- {{< hl-text blue >}}
Base case 計算:
{{< /hl-text >}}

	當 Array 長度是 1 時，本身就是有序的，判斷這件事情，是```O(1)```的操作

- {{< hl-text blue >}}
遞歸調用
{{< /hl-text >}}

	調用前需要做一些預處理，就是要取得中間 index。這個操作也只是```O(1)```的操作而已。再來調用遞歸函數，**這是屬於下一層的事情**

- {{< hl-text red >}}
merge 合併操作
{{< /hl-text >}}

  這是這一階層分析中最重要的部分 !
	- 首先分析第一層，由於需要循環 n 次，所以時間複雜度為 ```O(N)```。
	- 接下來我們分析第二層，有兩個長度為 ```n/2```的 List1 、 List2 需要處理 :
    	- 處理第一個  List1 的時候，需要循環```n/2```次
    	- 同理第二個 List2 也需要循環```n/2```次

		故在第二層，總共還是需要循環 n 次，時間複雜度為```O(N)```。

	不難發現，無論是第幾層，每一層的合併操作都需要 ```O(N)``` 時間複雜度才能完成

	{{< alert info >}}
簡單說明的話，其實就是每層都還是要合併 N 個元素，故每一層時間複雜度均為 ```O(N)```。
	{{< /alert >}}


由於每進入下一層， array 的長度變成一半，所以我們大約需要```log N```層，就可以將 array 的長度變為 1 並開始返回結果。總結上述分析， Merge Sort 的時間複雜度為 ```O(N*(深度)) = O(N logN)```


### 空間複雜度 :  ```O(N)```
- recursion tree 的深度是 ```log N```，故 recursion stack 的空間需花費 ```O(logN)```

- 在 merge function 合併操作的時候，我們有額外開存儲空間來合併兩個已排序的 sub-array，該存儲空間和和元素個數成正比，故空間複雜度為 ```O(N)```。另外注意，這個 ```O(N)``` 的臨時空間僅在該層遞迴中使用並不會傳到下層，故不需要乘上深度。

總結上述分析， Merge Sort 的空間複雜度為 ```O(N)```

{{< alert warning >}}
Mergesort 的缺點之一就是在合併子序列時，需要額外的空間依序插入排序資料，**並非 in-place** 排序
{{< /alert >}}


---

### 參考資料

- [排序算法精华2 归并排序](https://www.youtube.com/watch?v=KAgkvtKMbwY)

- [合併排序 Mergesort](https://rust-algo.club/sorting/mergesort/)



