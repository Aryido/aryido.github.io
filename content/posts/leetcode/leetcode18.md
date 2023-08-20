---
title: "18. 4Sum"

author: Aryido

date: 2023-02-15T22:17:40+08:00

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
> Leetcode 幾個數字題，15、16、18，基本上套路都是一樣的(~~甚至可以預期可能還會出 5 Sum...~~)，整體的解法都差不多，可以一起複習，重點仍然是 :
> 1. 排序 array
> 2. 避免的重複項
>
> 以 3Sum 此基礎上，再加了一個循環而已。

<!--more-->

---
# 思路
特別注意下，題目有特別提到:
- Constraints: ```-10^9 <= nums[i] <= 10^9``` 數字是十億級別，故在 Java 中會遇到超過 int 上限而發生**溢位(arithmetic overflow)**，進而導致錯誤。這裡簡單轉成 long 型相加來解決。

- **unique quadruplets**，解決方式一樣可以使用 while 寫法來去除重複答案 :
    ```
    while (k < l && nums[k] == nums[k + 1]) k++;
    while (k < l && nums[l] == nums[l - 1]) l--;

    while (j < l && nums[j] == nums[j + 1]) j++;
    while (i < l && nums[i] == nums[i + 1]) i++;
    ```

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

{{< alert warning >}}
特別思考下 *for loop* 的終止條件 ```nums.length - 3```。

從圖像上去思考的話，在 *j*、 *k*、 *l* 到達最尾端時，剛剛好佔了三格。所以 *i* 最多只需要到*倒數第四格*，也就是不超過 ```nums.length - 3```。

基本上以此模板，可以推論 :
- ```N SUM : nums.length - (N-1)```

{{< /alert >}}

---
# 時間空間複雜度

### 時間複雜度: ```O(N^3)```

nums 排序的時間複雜度為 ```O(NlogN)```。再來是過程中有 2 層 for loop 遍歷 nums且內部還有一個 while loop 故算 ```O(N^3)```

### 空間複雜度：```O(1)```
演算法過程主要只需要存儲 ```i, j, k, l```，故為 ```O(1)```

---

### 參考資料

- [[LeetCode]18. 4Sum 中文](https://www.youtube.com/watch?v=kUW2_6xOiZs&t=152s)