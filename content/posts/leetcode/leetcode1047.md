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

> [1047. Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/description/)，最直觀的還是 stack 方式解答。

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
- 先初始化一個長度和 s 一樣的 array
  {{< alert info >}}
  用雙指標，chars 必須先有空間。例如只寫 `chars = []`，又直接 `chars[index]`，會發生 IndexError
{{< /alert >}}
- index: 指向下一個可寫入字母的位置，也代表「目前答案長度」
- for-loop 遍歷字串，每個字母放入 index 位置，比較 index 和 index-1 位置的字母，從而調整 index 的走向

```python
class Solution:
    def removeDuplicates(self, s: str) -> str:
        chars = [ "_" for _ in range(len(s))]
        
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
- `chars = [a(index)(i), _, _, _, _, _]` 
- `chars = [a, b(index)(i), _, _, _, _]`
- `chars = [a, b, b(index)(i), _, _, _]`，兩個 b 重複，故 index 會回頭，變成 `chars = [a, b(index), b(i), _, _, _]`
- `chars = [a, a(index), b, a(i), _, _]`，兩個 a 重複，index 回頭，變成 `chars = [a(index), a, b, a(i), _, _]`
- `chars = [c, a(index), b, a, c(i), _]`
- `chars = [c, a(index), b, a, c, _(i)]`

最後答案是 `''.join(chars[:index])` 為 `ca`，時間空間複雜度均為 `O(N)`。

{{< alert info >}}
雙指標寫法也是在模擬 stack：
- index 左邊的數字，相對於 stack 內保留下來的字元
- index - 1 相對於 stack top
{{< /alert >}}
