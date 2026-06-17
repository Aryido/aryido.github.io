---
title: "283. Move Zeroes"

author: Aryido

date: 2023-01-18T00:04:14+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetcode

tags:
  - multiple-pointers
  - python

comment: false

reward: false
---

<!--BODY-->

> [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/description/)，要求 in-place 操作，使用快慢指針來解題。

<!--more-->

---

```python
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        index = 0
        for num in nums:
            if num != 0:
                nums[index] = num
                index += 1
        
        for i in range(index, len(nums)):
            nums[i] = 0
```

{{< alert danger >}}
Python 沒有 ++ 這種遞增運算子。
- `index++` 會直接 syntax error
- `++index`，雖然不會報錯，但它不是「遞增」的意思，是連續做兩次正號的意思
{{< /alert >}}


---

### 參考資料

- [Move Zeroes - LeetCode 283](https://www.youtube.com/watch?v=4w_jTUw_XO8)