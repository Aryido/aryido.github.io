---
title: "80. Remove Duplicates from Sorted Array II"

author: Aryido

date: 2023-01-20T23:27:59+08:00

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

> [80. Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/description/)是 [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)的進階，比較重要的也是要求 in-place 操作。

<!--more-->

---

原始 array 有一個 **non-decreasing order** 的特性其實非常重要 :

- 由於本題的數字最多可以重複兩次，代表直接從 `index=2` 就可以了
- index 指向「下一個可以寫入的位置
- 因為每個數最多保留 2 次，所以用 `nums[index - 2]` 來比較。因為 array 是 **non-decreasing order** 才可以這樣做。

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        index = 2
        for i in range(2, len(nums)):
            if nums[i] == nums[index - 2]:
                continue
            else:
                nums[index] = nums[i]
                index += 1

        return index

```

所以可以想到如果題目改成「**限制不超過 Ｎ 個 duplicate**」的話，可以寫出通用解法：

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        return self.removeOverNDuplicates(nums, 2)

    def removeOverNDuplicates(self, nums: List[int], n: int) -> int:
        index = n
        for i in range(n, len(nums)):
            if nums[i] == nums[index - n]:
                continue
            else:
                nums[index] = nums[i]
                index += 1

        return index
```

---

### 參考資料

- [贾考博 LeetCode 80. Remove Duplicates from Sorted Array II](https://www.youtube.com/watch?v=JimP_qCjb0Q)
