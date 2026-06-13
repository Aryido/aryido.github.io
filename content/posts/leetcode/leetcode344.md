---
title: "344. Reverse String"

author: Aryido

date: 2023-01-16T22:26:58+08:00

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

> [344. Reverse String](https://leetcode.com/problems/reverse-string/)，為反向雙指針題型。

<!--more-->

---

```python
class Solution:
    def reverseString(self, s: List[str]) -> None:
        """
        Do not return anything, modify s in-place instead.
        """
        left = 0
        right = len(s) - 1

        while left < right:
            self.swap(s, left, right)
            left += 1
            right -= 1
    
    def swap(self, s: List[str], left: int, right:int):
        s[left], s[right] = s[right], s[left]
```

當然在實務工作上，還是直接使用 `s.reverse()` 是最好的
```python
class Solution:
    def reverseString(self, s: List[str]) -> None:
        s.reverse()
```

---

### 參考資料

- [Array题型：双指针Two Pointers套路【LeetCode刷题套路教程2】](https://www.youtube.com/watch?v=86GHTcY0K4I)