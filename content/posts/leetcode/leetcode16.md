---
title: "16. 3Sum Closest"

author: Aryido

date: 2023-02-08T21:56:12+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetCode

tags:
  - multiple-pointers
  - array
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> [第 16 題](https://leetcode.com/problems/3sum-closest/description/)跟 15 題相似，又增加了些許難度。題目敘述一樣也很簡單 : 求 nums 內最接近 _target_ 值的三數和。優化關鍵點一樣是，_把 nums 排序_，這樣就可以確定指針滑動方向。
> {{< image classes="fancybox fig-100" src="/images/leetcode/16.jpg" >}}

<!--more-->

---

# 思路

因為是求最接近 _target_ 值的三數和，而沒有要關注其 _index_，故可以考慮將 **nums 排序**。排序之後可以有甚麼好處呢 ? 好處是在計算第 _i_ 值、第 _j_ 值、第 _k_ 值加總後，**指針移動情況可以掌握** :

- 加總後等於 target：直接返回 target 值即可
- 加總後小於 target：代表加總值不夠大，**因為排序，固可將 _j_ 點往右移動，尋更大的值**
- 加總後大於 target：代表加總值太大，**因為排序，固可將 _k_ 點往左移動，尋找更小的值**

另外可以再稍稍優化的部分是，當`nums[i]*3 > target` 的時候，在 `nums`已經排過序的情況下，後面的數字只會越來越大，故距離 target 一定會比 `nums[i] + nums[i+1] + nums[i+2]` 還要更大，所以不必再往後看了。只要判斷`nums[i] + nums[i+1] + nums[i+2]` 和當前答案，返回較小的就可以了。

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
			if ( nums[i] * 3 > target ) {
				int sum = nums[i] + nums[i + 1] + nums[i + 2];
				if ( Math.abs( sum - target ) < closest ) {
					return sum;
				}
			}

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

特別思考下 _for loop_ 的終止條件，是 `nums.length - 2`。從圖像上去思考的話，在 _j_、 _k_ 到達最尾端時，剛剛好佔了兩格。所以 _i_ 最多只需要到*倒數第三格*，也就是不超過 `nums.length - 2` !  另外附上 Python 解法：


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

                if nums[i]*3 >= target:
                    if abs(cur-target) < abs(ans - target):
                        ans = cur

                    return ans

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
