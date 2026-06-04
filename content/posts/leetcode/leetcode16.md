---
title: "16. 3Sum Closest"

author: Aryido

date: 2023-02-08T21:56:12+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetcode

tags:
  - multiple-pointers
  - array
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> [第 16 題](https://leetcode.com/problems/3sum-closest/description/)跟 15 題相似，又增加了些許難度。題目敘述一樣也很簡單 : 求 nums 內**最接近** _target_ 值的三數和。因為是求**最接近** _target_ 值的三數和，而解答也沒有要關注其 _index_，故還是可以考慮將 **nums 排序**，這樣就可以確定指針滑動方向。
> {{< image classes="fancybox fig-100" src="/images/leetcode/16.jpg" >}}

<!--more-->

---

# 思路

排序之後可以在計算第 _i_ 值、第 _j_ 值、第 _k_ 值加總後，**掌握下一步指針應該移動的方向** :

- 加總後等於 target： 本題求最接近的和。若相等一定是最接近的，所以直接返回
- 加總後小於 target： 代表加總值不夠大，可以在嘗試大一些。**又因為 nums 已經排序了，故可將 _j_ 點往右移動，尋找更大的值看看**
- 加總後大於 target： 代表加總值太大，需要縮小一些，**又因為 nums 已經排序了，故可將 _k_ 點往左移動，尋找更小的值看看**

這邊選擇不寫上「跳過重複項」的判斷，原因是本題不會因為重複項導致答案錯誤，也可使 code 更加簡潔 ; 而且加上跳過的判斷，直接從 Runtime 看起來，還讓速度更慢了。

---

# 解答

```java
class Solution {
	public int threeSumClosest( int[] nums, int target ) {
		if ( nums.length == 3 ) {
			return nums[0] + nums[1] + nums[2];
		}

		Arrays.sort( nums );
        
		int result = nums[0] + nums[1] + nums[2];
		int closest = Math.abs( result - target );
		for ( int i = 0; i < nums.length - 2; i++ ) {
			int j = i + 1;
			int k = nums.length - 1;
			while (j < k) {
				int sum = nums[i] + nums[j] + nums[k];
				int diff = Math.abs( sum - target );
				
                if ( diff < closest ) {
					result = sum;
					closest = diff;
				}

				if ( sum == target ) {
					return target;
				} else if ( sum < target ) {
					j++;
				} else {
					k--;
				}
			}
		}
		return result;
	}
}
```

特別思考下 _for loop_ 的終止條件，是 `nums.length - 2`。從圖像上去思考的話，在 _j_、 _k_ 到達最尾端時，剛剛好佔了兩格。所以 _i_ 最多只需要到*倒數第三格*，也就是不超過 `nums.length - 2` ! 另外附上 Python 解法：

```python
class Solution:
	
    def threeSumClosest(self, nums: List[int], target: int) -> int:
        if len(nums) == 3:
            return nums[0] + nums[1] + nums[2]

        n = len(nums)
        nums.sort()

        ans = nums[0] + nums[1] + nums[2]
        for i in range(n-2):
            j = i + 1
            k = n - 1

            while j < k:
                cur = nums[i] + nums[j] + nums[k]

                if cur == target:
                    return target
                elif cur > target:
                    k -= 1
                else:
                    j += 1

                if abs(cur-target) < abs(ans - target):
                    ans = cur

        return ans
```

{{< alert warning >}}
Java 和 Python 一些用法要記一下，例如

- array 的排序
  - Java: `Arrays.sort(nums);`
  - Python: `nums.sort()`

- 絕對值
  - Java: `Math.abs();`
  - Python: `abs()`

{{< /alert >}}

---

# 時間空間複雜度

### 時間複雜度: `O(N^2)`

- 排序的時間複雜度為 `O(NlogN)`
- 過程中有 for loop 遍歷 nums，且內部有一個 while loop 再遍歷 nums，為 `O(N^2)`

### 空間複雜度：`O(1)`

本題的解法，演算法過程只需要存儲 `i, j, k, closest` 而已，故花費`O(1)`空間而已。

---

### 參考資料

- [[LeetCode]16. 3Sum Closest 中文](https://www.youtube.com/watch?v=vDrUqaPCVyk&t=152s)

- [Grandyang 16. 3Sum Closest](https://www.cnblogs.com/grandyang/p/4510984.html)
