---
title: "2460. Apply Operations to an Array"

author: Aryido

date: 2023-01-16T22:16:58+08:00

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

> [2460. Apply Operations to an Array](https://leetcode.com/problems/apply-operations-to-an-array/description/) 和 [26. Remove Duplicates from Sorted Array](https://aryido.github.io/posts/leetcode/leetcode26/) 是類似題。

<!--more-->

---

```python
class Solution:
    def applyOperations(self, nums: List[int]) -> List[int]:
        n = len(nums)

        for i in range(n - 1):
            if nums[i] == nums[i + 1]:
                nums[i] *= 2
                nums[i + 1] = 0

        index = 0
        for i in range(n):
            if nums[i] != 0:
                nums[index] = nums[i]
                index += 1

        while index < n:
            nums[index] = 0
            index += 1

        return nums
```
---

# 時間空間複雜度

### 時間複雜度: `O(N)`

算遍三次 nums ，還是 `O(N)` 複雜度

### 空間複雜度：`O(1)`

都沒有開闢額外空間