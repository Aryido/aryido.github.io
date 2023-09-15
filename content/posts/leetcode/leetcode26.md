---
title: "26. Remove Duplicates from Sorted Array"

author: Aryido

date: 2022-12-25T15:05:58+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- java
- multiple-pointers

comment: false

reward: false
---
<!--BODY-->
> 這道題要我們從**有序數組**中去除重複項，題目難度雖然被歸為 easy 等級，但在**條件限制**上的討論，蠻多東西可以釐清討論的。 英文方面寫得蠻長，記得要看到最後因為有寫一些限制如 :
> - *O(1)* extra memory
> - The relative order of the elements should be kept the same.
>
> 所以**不能用 Set 或另開 array 去寫**。另外也花了些篇幅去說明，只要原 array 前面長度部分內，有把所有不重複數字列出來就好，不需要在意後面 array 的元素和 array 的長度。


<!--more-->

---
兩個重點條件限制 :
- *O(1)* extra memory

- The relative order of the elements should be kept the same.

以下使用各種方式來解題，當作練習

# 思路
使用**快慢指針**來遍歷 array :
- **lower** 和 **faster** 都從 index 0 開始
- **lower**之前的 index 都是處理好的數字

接下來開始循環邏輯 :

- 若 ```faster == 0``` ，則為初始狀態，**快慢指針**都前進。

- 因為定義 **lower** 之前的 index 均為處理好的資料，故若條件 ```nums[lower - 1 ] != nums[faster]``` ，則 **快慢指針**都前進，並把 ```nums[lower]``` 換成  ```nums[faster]```

{{< alert info >}}
很自然的，可以把 ```faster == 0``` 和 ```nums[lower - 1 ] != nums[faster]``` 條件併在一起。因為初始狀態 :
-  ``` faster = 0 ```
-  ``` lower = 0 ```

這樣 ```nums[lower] = nums[faster]``` 改動其實等於沒改。
{{< /alert >}}

---

# 解答
```java
class Solution {
	public int removeDuplicates( int[] nums ) {
		int faster = 0;
		int lower = 0;
		while (faster < nums.length) {
			if ( faster == 0 || nums[lower - 1] != nums[faster] ) {
				nums[lower++] = nums[faster++];
			} else {
				faster++;
			}
		}
		return lower;
	}
}
```
{{< alert success >}}
``` nums[lower++] = nums[faster++];``` 是個蠻簡潔不錯的寫法，可以在賦值完之後，才再讓指針前進。想提醒自己這個部分，才特別註記筆記這題的。
{{< /alert >}}

{{< alert warning >}}
思考了一下，這題的條件 :
- The relative order of the elements should be kept the same.

好像沒有用，因為雙指針的題型，會改變 order 的是**反向指針**，這題應該不能用反向指針去解題...
{{< /alert >}}

---

# 其它思路
一樣是使用**快慢指針**來遍歷 array ，最開始時兩個指針都指向第一個數字，但
- 兩個指針指的數字相同，則快指針向前走一步
- 如果不同，則慢指針先向前一步，把慢指針對應 index 數字換成快指針對應 index 數字，再來快指針前進

可以發現這樣的邏輯下，慢指針的位置，就代表**目前置換過的 index** 。當快指針走完整個數組時，慢指針當前位置，就代表**最後更新的位置**，故 ```lower + 1``` 就是數組中不同數字的個數。

# 解答
```java
class Solution {
	public int removeDuplicates( int[] nums ) {
		int faster = 0;
		int lower = 0;
		while (faster < nums.length) {
			if ( nums[lower] == nums[faster] ) {
				++faster;
			} else {
				nums[++lower] = nums[faster++];
			}
		}
		return lower + 1;
	}
}
```

上面邏輯也可以換成 for loop 也可以解 !

```java
class Solution {
	public int removeDuplicates( int[] nums ) {
		int lower = 0;
		for ( int faster = 0; faster < nums.length; faster++ ) {
			if ( nums[faster] != nums[lower] ) {
				nums[++lower] = nums[faster];
			}
		}

		return lower + 1;
	}
}
```
---

# 時間空間複雜度
### 時間複雜度: ```O(N)```
遍歷 nums 需 ```O(N)```

### 空間複雜度：```O(1)```
因為禁止開額外空間，此題強制 ```O(1)```

---
# Vocabulary

{{< alert info >}}
[**In-place**](https://en.wikipedia.org/wiki/In-place_algorithm)

特殊片語，代表直接對輸入的 data-structure 進行操作，不能另開額外空間。

in-place algorithm 會應用在一些不希望大量增加記憶體使用量，是拿時間換取空間的概念。
{{< /alert >}}


---
### 參考資料

- [Grandyang](https://www.cnblogs.com/grandyang/p/4329128.html)
