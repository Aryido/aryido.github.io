---
title: "15. 3Sum"

author: Aryido

date: 2023-02-09T22:56:05+08:00

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

> [是 Two Sum 的一種另類進階](https://leetcode.com/problems/3sum/description/)，要從 nums 中找出和為 `0` 的三個 element ，由於答案可能有多種，故答案會是個 **List of list** 。特別注意，題目中有提到不能有兩個內容一樣的 list。
>
> 因為整個題目並沒有對 _nums_ 的 _index_ 有任何要求，故可以**把 nums 排序**，為解題拓開另一種思路。

<!--more-->

---

# 多指針解法

本題使用多指針法是最優的解，可進一步減少空間複雜度。若參考 [Two Sum](https://leetcode.com/problems/two-sum/description/) 使用 hash-table ，空間複雜度會比較高。

### 思路

比較優的解法的使用重點，是想到可以對原 **nums** 進行排序，這樣對指針的移動就會清楚很多。 這裡也可以注意，不用遍歷到最後一個，而是到倒數第三個就可以了(因為是 3 sum)。 那指針該如何移動呢？

- 三數和小於 `0`，則將左邊那個指針 _j_ 右移，使得三數和增大一些。
- 三數和大於 `0`，則將右邊那個指針 _k_ 左移，使得三數和變小一些。
- 三數和等於 `0`，為答案之一，要記錄下來。

題目有特別提到，**must not contain duplicate triplets**，故需要思考一下指針若遇到重複數字怎麼辦 ? 常見的處理方式是跳過 :

```python
if i > 0 && nums[i] == nums[i-1]: continue

while j < k && nums[j] == nums[j + 1]: j++

while j < k && nums[k] == nums[k - 1]: k--
```

另外可以加個 `if(nums[i] == 0) break;` 判斷來優化一些。因為把 **nums** 進行由小到大排序了，所以若是`nums[i] == 0`，代表之後的 num 都大於等於零，又因為三數大於零的數加起來，基本一定大於等於零，故可以直接 break，省略許多判斷。

### 解答

```java
class Solution {
	public List<List<Integer>> threeSum( int[] nums ) {
		Arrays.sort( nums );
		List<List<Integer>> ans = new ArrayList<>();
		for ( int i = 0; i < nums.length - 2; i++ ) {
			int j = i + 1;
			int k = nums.length - 1;

			while (j < k) {
				if ( nums[i] + nums[j] + nums[k] == 0 ) {
					List<Integer> list = List.of( nums[i], nums[j], nums[k] );
					ans.add( list );
					if ( nums[i] == 0 ) {
						break;
					}
					while (j < k && nums[j] == nums[j + 1]) j++;
					while (j < k && nums[k] == nums[k - 1]) k--;
					j++;
					k--;
				} else if ( nums[i] + nums[j] + nums[k] > 0 ) {
					k--;
				} else {
					j++;
				}
			}
			while (i < k && nums[i] == nums[i + 1]) i++;
		}
		return ans;
	}
}
```

{{< alert info >}}
題目的條件有 `-10^5 <= nums[i] <= 10^5`，故三數和不會超過`Integer.MAX_VALUE`。因為這個條件所以不用擔心數字相加會發生甚麼問題~
{{< /alert >}}


```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        
        n = len(nums)
        ans = []
        for i in range(n-2):

            if i > 0 and nums[i] == nums[i-1]: 
                continue

            if nums[i] > 0: 
                break
            
            j = i + 1
            k = n - 1
            while j < k :

                s = nums[i] + nums[j] + nums[k]

                if s == 0:
                    ans.append([nums[i], nums[j], nums[k]])
                    j +=1
                    k -=1
                elif s < 0:
                    j +=1
                else: 
                    k -=1
                    
                while j < k and nums[j]== nums[j+1]:
                    j += 1
                
                while k > j and nums[k]== nums[k-1]:
                    k -= 1
        return ans

```

### 時間空間複雜度

##### 時間複雜度: `O(N^2)`

- 排序的時間複雜度為 `O(NlogN)`
- for loop 遍歷 nums，且內部有一個 while loop 再遍歷 nums，是 `O(N^2)`

##### 空間複雜度：`O(1)`

演算法過程只需要存儲 `i, j, k` ，花費`O(1)`空間而已。

---

也可是參考[Two Sum](https://leetcode.com/problems/two-sum/description/) 使用 hash-table 來解題，空間複雜度會比較高: `O(n)`

### 解答

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()

        res = set()
        n = len(nums)
        for i in range(n - 2):
            
            if i > 0 and nums[i] == nums[i-1]:
                continue

            seen = set()
            for j in range(i+1, n):
                
                target = -nums[i] - nums[j]
                
                if target in seen:
                    res.add((nums[i], target, nums[j]))
                else:
                    seen.add(nums[j])

        return [list(t) for t in res]

```

{{< alert warning >}}
- `seen = set()` 作為 hash-table ，記錄「掃過的數字」`nums[j]`，要放在尋找 target 迴圈的外面
- 還要特別處理重複的答案，這邊使用的手法是：
	- `res` 使用 `set()` 用來避免重複的 triplets
	- 由於 `cannot use 'list' as a set element (unhashable type: 'list')`，所以 `res` 要使用 
		`res.add((nums[i], target, nums[j]))`，代表 set 內是加入 **tuple\`()\`** 而不能是 **list\`[]\`**
- 最後把 tuple 轉為答案所需的 list:  `[list(t) for t in res]`

{{< /alert >}}

---

### 參考資料

- [[LeetCode]15. 3Sum 中文](https://www.youtube.com/watch?v=2tbi1W7ce1c&t=378s)
- [[LeetCode 解題紀錄] 15. 3Sum](https://medium.com/%E6%8A%80%E8%A1%93%E7%AD%86%E8%A8%98/leetcode-%E8%A7%A3%E9%A1%8C%E7%B4%80%E9%8C%84-15-3sum-196ddd83efd8)