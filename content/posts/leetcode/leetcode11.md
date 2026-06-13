---
title: "11. Container With Most Water"

author: Aryido

date: 2023-01-17T03:36:09+08:00

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

> [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)是一個運用雙指標的算法

<!--more-->

---

# 思路
最簡單的是暴力解，就是把一個一個的面積遍歷出來比較
```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        result = 0
        for i in range(n):
            left = i 
            right = n - 1

            while left < right:
                h = min(height[left], height[right])
                l = right - left
                area = h * l
                if area > result:
                    result = area
                
                right -= 1

        return result
```
不過會跳出 Time Limit Exceeded 超過時間複雜度限制，失敗的 case 是：
```text
[0,1,2,3,4,5,...,10000,9999,9998,...,4,3,2,1]
```

但其實指針的移動是可以知道的，因為 area 的公式是 `h*l` ，那麼根據公式可以知道如果要求出最大體積，`h` 或 `l` 應該是盡可能的大。而由於 `l` 只能縮小，所以 `h` 不變就不可能超過原來的值，所以就只能嘗試變動比較矮的指針，故：
- 如果左邊的隔板比右邊矮，就收縮左邊
- 如果右邊的隔板比左邊矮，就收縮右邊

收縮滑窗的做法，直接把暴力解的 while-loop 幹掉了，直接下降一個時間複雜維度。

# 解答
```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        result = 0
        
        left = 0 
        right = n - 1
        while left < right:
            l = right - left
            h = min(height[left], height[right])
            area = h * l
            if area > result:
                result = area

            if height[left] > height[right]:
                right -= 1
            elif height[left] < height[right]:
                left += 1
            else:
                right -= 1
                left += 1

        return result
```

### 時間空間複雜度
假設整數陣列 `height` 長度為 `N` :

##### 時間複雜度: `O(N)`
這題最主要是可以優化到 `O(N)`，由 Hint2. 的說法：
```text
Try to use two-pointers. Set one pointer to the left and one to the right of the array. 
Always move the pointer that points to the lower line.
```
關鍵是可以知道要讓哪個指針移動，這題是每次都移動較矮那條線的指標就可以了。

##### 空間複雜度：`O(1)`

都是臨時變數，所占用的記憶體空間固定，都是常數級別 `O(1)`

---

### 參考資料

- [Array题型：双指针Two Pointers套路【LeetCode刷题套路教程2】](https://www.youtube.com/watch?v=86GHTcY0K4I)

- [[LeetCode] 11. Container With Most Water 盛最多水的容器](https://www.cnblogs.com/grandyang/p/4455109.html)

- [\[LeetCode 筆記\] 11. Container With Most Water](https://ithelp.ithome.com.tw/articles/10312733)

- [Container With Most Water - LeetCode 11](https://www.youtube.com/watch?v=PecOvPjS96w)