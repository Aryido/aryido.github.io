---
title: "1047. Remove All Adjacent Duplicates In String"

author: Aryido

date: 2023-02-27T23:47:08+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetcode

tags:
  - stack
  - python
  - multiple-pointers

comment: false

reward: false
---

<!--BODY-->

> [1047. Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/description/)

<!--more-->

---

Hint 提示是使用 stack，還蠻直觀的：
```python
class Solution:
    def removeDuplicates(self, s: str) -> str:
      stack = []

      for c in s:
        if not stack or stack[-1] != c:
          stack.append(c)
        else:
          stack.pop()
      
      return ''.join(stack)
```

雙指針寫法我自己是覺得不直觀，但就當作練習，核心想法：
- 先做出 chars 字母 array
- index: 指向一個可寫入字母的位置

```python
class Solution:
    def removeDuplicates(self, s: str) -> str:
        chars = list(s)
        index = 0
        for i in range(len(s)):
            chars[index] = s[i]

            if index > 0 and chars[index] == chars[index - 1]:
                index -= 1
            else:
                index += 1

        return ''.join(chars[:index])
```
以 s = `abbaca` 為例：
- `chars = [a(index)(i), b, b, a, c, a]` 
- `chars = [a, b(index)(i), b, a, c, a]`
- `chars = [a, b, b(index)(i), a, c, a]`，兩個 b 重複，index 回頭，變成 `chars = [a, b(index), b(i), a, c, a]`
- `chars = [a, a(index), b, a(i), c, a]`，兩個 a 重複，index 回頭，變成 `chars = [a(index), a, b, a(i), c, a]`
- `chars = [c, a(index), b, a, c(i), a]`
- `chars = [c, a(index), b, a, c, a(i)]`

最後答案是 "ca"