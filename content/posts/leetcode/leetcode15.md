---
title: "15. 3Sum"

author: Aryido

date: 2023-02-09T22:56:05+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- java
- multiple-pointers
- array

comment: false

reward: false
---
<!--BODY-->
> 是 Two Sum 的一種另類進階，從 nums 中找出和為 ```0``` 的三個  element ，並組成 **List of list** 。特別注意，不能有兩個內容一樣的 list。因為整個題目並沒有對 *nums* 的 *index* 有任何要求，故可以**把 nums 排序**，為解題拓開另一種思路。
<!--more-->

---

# 思路
比較優的解法重點，大概就是想到可以對原 **nums** 進行排序。然後開始 *for loop* 排序後的 *nums* 。這裡注意 ! 不是遍歷到最後一個停止，而是到倒數第三個就可以了。 接著處理

- 三數和小於 ```0```，則將左邊那個指針 *j* 右移，使得三數和增大一些。
- 三數和大於 ```0```，則將右邊那個指針 *k* 左移，使得三數和變小一些。
- 三數和等於 ```0```，為答案之一，要記錄下來。

題目有特別提到，**must not contain duplicate triplets**，故需要思考一下指針若遇到重複數字怎麼辦，比較好的方式是以下寫法 :
```
while (i < k && nums[i] == nums[i + 1]) i++;
while (j < k && nums[j] == nums[j + 1]) j++;
while (j < k && nums[k] == nums[k - 1]) k--;
```

另外可以加個 ```if(nums[i] == 0) break;``` 判斷來優化一些。因為把 **nums** 進行由小到大排序了，所以若是```nums[i] == 0```，代表之後的 num 都大於等於零，故可以用```if(nums[i] == 0) break;```，因為後面三數加起來，基本一定大於等於零，可省略許多判斷了~

---

# 解答
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
題目的條件有 ```-10^5 <= nums[i] <= 10^5```，故三數和不會超過```Integer.MAX_VALUE```。因為這個條件所以不用擔心數字相加會發生甚麼問題~
{{< /alert >}}

---

# 時間空間複雜度

### 時間複雜度: ```O(N^2)```

- 排序的時間複雜度為 ```O(NlogN)```
-  for loop 遍歷 nums，且內部有一個 while loop 再遍歷 nums，是 ```O(N^2)```

### 空間複雜度：```O(1)```
依照本題寫法，演算法過程只需要存儲 ```i, j, k``` ，理所當然花費```O(1)```空間而已。

---

# 參考資料

- [[LeetCode]15. 3Sum 中文](https://www.youtube.com/watch?v=2tbi1W7ce1c&t=378s)
