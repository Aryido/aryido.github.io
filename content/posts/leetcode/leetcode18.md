---
title: "18. 4Sum"

author: Aryido

date: 2023-02-15T22:17:40+08:00

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

> Leetcode 幾個數字題 [15](https://aryido.github.io/posts/leetcode/leetcode15/)、[16](https://aryido.github.io/posts/leetcode/leetcode16/)、[18](https://leetcode.com/problems/4sum/description/)，基本上套路都是一樣的，可以一起複習，重點仍然是 :
>
> 1. 排序 array
> 2. 避免紀錄重複項
>
> 由於可以預期 ~~還會出 5 Sum...~~ 之類的，本題主要是引出「來想一種 KSum 的解法」，通用任意的數位。

<!--more-->

---

# 思路

特別注意下，題目有特別提到:

- Constraints: `-10^9 <= nums[i] <= 10^9` 數字是十億級別，故在 Java 中會遇到超過 int 上限而發生**溢位(arithmetic overflow)**，進而導致錯誤。這裡簡單轉成 long 型相加來解決。

- **unique quadruplets**，解決方式一樣可以使用 while 寫法來去除重複答案

# 解答

```java
class Solution {
	public List<List<Integer>> fourSum( int[] nums, int target ) {
		List<List<Integer>> ans = new ArrayList<>();
		if ( nums.length == 4 ) {
			long sum = (long) nums[0] + (long) nums[1] + (long) nums[2] + (long) nums[3];
			if ( sum == target ) {
				List<Integer> list = List.of( nums[0], nums[1], nums[2], nums[3] );
				ans.add( list );
			}
			return ans;
		}

		Arrays.sort( nums );
		for ( int i = 0; i < nums.length - 3; i++ ) {
			for ( int j = i + 1; j < nums.length - 2; j++ ) {
				int k = j + 1;
				int l = nums.length - 1;
				while (k < l) {
					long sum = (long) nums[i] + (long) nums[j] + (long) nums[k] + (long) nums[l];
					if ( sum == target ) {
						List<Integer> list = List.of( nums[i], nums[j], nums[k], nums[l] );
						ans.add( list );

						while (k < l && nums[k] == nums[k + 1]) k++;
						while (k < l && nums[l] == nums[l - 1]) l--;
						k++;
						l--;
					} else if ( sum > target ) {
						l--;
					} else {
						k++;
					}
				}
                while (j < l && nums[j] == nums[j + 1]) j++;
                while (i < l && nums[i] == nums[i + 1]) i++;
			}
		}
		return ans;
	}
}
```

特別看下 _for loop_ 的終止條件，因為從圖像上去思考的話，在 _j_、 _k_、 _l_ 到達最尾端時，剛剛好佔了三格。所以 _i_ 最多只需要到*倒數第四格*，也就是不超過 `nums.length - 3`。
基本上以此模板，可以推論 N SUM 的話，_for loop_ 的終止條件: `nums.length - 3`

接下來附上 python 的解法：
```python
class Solution:
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        n = len(nums)
        nums.sort()
        ans = []

        for i in range(n - 3):
        	if nums[i] * 4 > target:
                break

            if i > 0 and nums[i] == nums[i - 1]:
                continue

            for j in range(i + 1, n - 2):
                if j > i + 1 and nums[j] == nums[j - 1]:
                    continue

                k, l = j + 1, n - 1

                while k < l:
                    cur = nums[i] + nums[j] + nums[k] + nums[l]

                    if cur == target:
                        ans.append([nums[i], nums[j], nums[k], nums[l]])
                        k += 1
                        l -= 1

                        while k < l and nums[k] == nums[k - 1]:
                            k += 1
                        while k < l and nums[l] == nums[l + 1]:
                            l -= 1

                    elif cur < target:
                        k += 1
                        while k < l and nums[k] == nums[k - 1]:
                            k += 1
                    else:
                        l -= 1
                        while k < l and nums[l] == nums[l + 1]:
                            l -= 1

        return ans
```

---

# 時間空間複雜度

### 時間複雜度: `O(N^3)`

- nums 排序的時間複雜度為 `O(NlogN)`。
- 過程中有 2 層 for loop 遍歷 nums且內部還有一個 while loop 故算 `O(N^3)`

### 空間複雜度：`O(1)`

演算法過程主要只需要存儲 `i, j, k, l`，故為 `O(1)`

---

# K Sum

通過之前的題目，可以得出對於 KSum 來說，就是要固定 K-2 的個 index，剩下兩個 index 藉由排序過後的 nums 來進行雙指標選擇。
如果要進一步寫出通用解法，就不能手動加 K-2 個 for 迴圈了，所以用遞歸來做 :


```python
class Solution:
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        nums.sort()
        return self.k_sum(nums, target, 0, 4)
    
    def k_sum(self, nums: List[int], target: int, index: int, k: int)-> List[List[int]]:
        result = []
        n = len(nums)

		# 不夠取 k 個數
        if index >= n:
            return result
        
		# 最小值都太大，或最大值都太小，就不用再算了
        if nums[index] * k > target or nums[-1] * k < target:
            return result
        
        if k == 2:
            return self.two_sum(nums, target, index)
        
        for i in range(index, n  - k + 1):
            if i > index and nums[i] == nums[i - 1]:
                continue
			
			# 固定 nums[i]，剩下做 k-1 Sum
            for arr in self.k_sum(nums, target - nums[i], i + 1, k - 1):
                result.append([nums[i]] + arr)
                
        return result



    def two_sum(self, nums: List[int], target: int, index: int) -> List[List[int]]:
        result = []
        n = len(nums)
        k, l = index, n - 1

        while k < l:
            cur = nums[k] + nums[l]

            if cur == target:
                result.append([nums[k], nums[l]])
                k += 1
                l -= 1

                while k < l and nums[k] == nums[k - 1]:
                    k += 1
                while k < l and nums[l] == nums[l + 1]:
                    l -= 1

            elif cur < target:
                k += 1
                while k < l and nums[k] == nums[k - 1]:
                    k += 1
            else:
                l -= 1
                while k < l and nums[l] == nums[l + 1]:
                    l -= 1
        
        return result
```

再來是練習使用 `set` 來去除重複:
```python
class Solution:
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        nums.sort()
        return [result for result in self.k_sum(nums, target, 4)]
    
    def k_sum(self, nums: List[int], target: int, k: int)-> List[set[int]]:
        result = set()

        n= len(nums)

        if k == 2:
            l = 0
            r =  n - 1
            while l < r:
                cur = nums[l] + nums[r]
                if cur == target:
                    result.add((nums[l], nums[r]))
                    l += 1
                    r -= 1
                elif cur < target:
                    l += 1
                else:
                    r -= 1

        else:
            for i in range(n - k + 1):
                for arr in self.k_sum(nums[i+1:], target - nums[i], k-1):
                    result.add((nums[i],) + arr)
        
        return result
```


---

# 時間空間複雜度

### 時間複雜度: `O(N^(k-1))`

- nums 排序的時間複雜度為 `O(NlogN)`。
- 遞迴關係：
	- 2Sum 用雙指針是 `O(N)`
	- 3Sum 固定一個 index，下降維度變成 2Sum，所以是 `O(N*N)`
	- 4Sum 固定一個 index，下降維度變成 3Sum，所以是 `O(N*N^2)`

kSum 的時間複雜度是 `O(N^(k-1))`

### 空間複雜度：`O(k)`

遞迴深度為 `O(k)`

---

### 參考資料

- [[LeetCode]18. 4Sum 中文](https://www.youtube.com/watch?v=kUW2_6xOiZs&t=152s)

- [[LeetCode] 18。4Sum 四数之和](https://www.cnblogs.com/grandyang/p/4515925.html)
