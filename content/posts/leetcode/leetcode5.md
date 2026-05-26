---
title: "5. Longest Palindromic Substring"

author: Aryido

date: 2022-09-14T22:30:14+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
  - leetcode

tags:
  - dp
  - java
  - python

comment: false

reward: false
---

<!--BODY-->

> **[最長回文子串 (Longest Palindromic Substring)](https://leetcode.com/problems/longest-palindromic-substring/description/) 算是一個常考題**。Palindrome 的定義是順讀逆讀，字母都一樣的詞語，比如範例給的 「 bab 」、 「 bb 」 等等，實際單字如 「 level 」 也是一個的 Palindromic 單字。
>
> 先熟悉「 Dynamic Programming - 2D matrix 」解和「 中心擴散法 (Expand Around Center) 」就好了，還有聽過最佳解 Manacher's Algorithm，時間複雜度可優化到了 **O(n)**，[之後再花其他篇幅去補充](https://aryido.github.io/posts/algorithm/manacher/)。

<!--more-->

---

# 動態規劃法

先使用動態規劃來解題吧。

### 思路

{{< image classes="fancybox fig-100" src="/images/leetcode/5/5-1.jpg" >}}

維護一個 2D-Matrix ，用來紀錄子問題狀態，其中 `dp[i][j]` 表示字符串區間 `[i, j]` 是否為回文串。明顯地 : 只有一個字符，肯定是 Palindrome，對應 2D-Matrix ，當 `i = j` 時，子問題是表示為只有一個字符，故 `dp[i][i] = True` ，代表可以先把表格對角線表填完(如上圖) 。

再來是 Palindromic 字串的定義，「**字串的對稱位置的兩端，字母要一樣**」，故可用 `dp[i+1][j-1]` 表示**內縮一層的單字**，他一定也要是 Palindromic：

{{< image classes="fancybox fig-100" src="/images/leetcode/5/5-4.jpg" >}}

承上述，可利用些特性寫出**狀態方程式**

> `dp[i][j] = dp[i + 1][j - 1]`

再來要特別注意 **Palindromic 字串在 DP 表格之間的依賴關係**，無論如何都要注意**填寫表格的順序** :

{{< image classes="fancybox fig-100" src="/images/leetcode/5/5-2.jpg" >}}

下面給出**上面表格**的正確的遍歷方式:

```java
// 逐 column 計算
for(int j = 1 ; j < length ; j++){
    for(int i = 0; i < j ; i++){
        ...
    }
}
```

{{< image classes="fancybox fig-100" src="/images/leetcode/5/5-5.jpg" >}}

因為在判斷 `dp[i][j]` 時，需要先有 `dp[i+1][j-1]` 的資訊，以錯誤的順序填表，在參考「縮小的狀態 `dp[i+1][j-1]` 時」，會因為還未填寫正確狀態而發生錯誤，所以 loop 的方式有所限制。

### 解答

```java
class Solution {
    public String longestPalindrome(String s) {
		// 一個單字，肯定 palindromic
        if ( s.length() == 1 ) {
			return s;
		}

		int length = s.length();
		boolean[][] dp = new boolean[length][length];
		for ( int i = 0; i < length; i++ ) {
			// 初始化表個對角線資訊
			dp[i][i] = true;
		}

		String ans = s.substring( 0, 1 );
		int maxLength = 1;
		// 依照範例表格 i,j 定義，以下使用「上半」部，由「左column」 到 「右column」; 上到下的方式 loop
		for ( int j = 1; j < length; j++ ) {
			for ( int i = 0; i < j; i++ ) {
				if ( s.charAt( i ) == s.charAt( j ) ) {
					if ( j - i == 1 ) {
						// 這種情況可以直接判斷了
						// 避免內縮去查詢時，跑去找沒有初始化的表格「下半」部
						dp[i][j] = true;
					} else {
						dp[i][j] = dp[i + 1][j - 1];
					}
				}

				int currentLength = j - i + 1;
				if ( dp[i][j] && currentLength > maxLength ) {
					maxLength = currentLength;
					ans = s.substring( i, j + 1 );
				}
			}
		}

		return ans;
    }
}
```

Python 就完全換個方式填表，假設我的 2D-matrix `i`, `j` 是和上面範例表格 schema 相反的話 ：

```python
class Solution(object):
    def longestPalindrome(self, s):
        if len(s) == 1:
            return s

        dp = [[False] * len(s) for _ in range(len(s))]
        for i in range(len(s)):
            dp[i][i] = True

        max_length = 1
        ans = s[0]
		# 以下使用「下半」部，由 「上row」 到 「下row」; 右到左的方式 loop
        for j in range(1, len(s)):
            for i in range(j-1, -1, -1):
                if s[i] == s[j]:
                    if j - i <= 2:
                        dp[i][j] = True
                    else:
                        dp[i][j] = dp[i+1][j-1]

                current_len = j - i + 1
                if dp[i][j] and current_len > max_length:
                    max_length = current_len
                    ans = s[i : j + 1]

        return ans
```

{{< alert warning >}}

- Python 中 2D-Matrix 初始化：

```python
dp = [[0] * cols for _ in range(rows)]
```

- 在 Python 中 [`range(start, stop, step)`](https://www.py101.org/learn-python-range.html) 是**不包含 stop 本身的**，

```python
for i in range(j-1, 0, -1):
	...

# 所以當你寫 0 的時候，`i` 最多只會走到 1 就停止了，完全沒有執行 i = 0 的情況
```

{{< /alert >}}

### 時間空間複雜度

動態規劃把大問題拆成小問題，例如：想知道 「 ababa 」 是不是回文，只要**看頭尾**且**中間的**是不是回文，這種子問題之間的「轉移關係」是考核動態規劃基本功的蠻不錯的題目。

##### 時間複雜度 : `O(N^2)`

兩層迴圈

###### 空間複雜度 : `O(N^2)`

2-D 矩陣，為了紀錄某步驟執行結果(動態規劃)，此解法記憶體消耗比較大

---

# 中心擴散法 (Expand Around Center)

### 思路

DP 做法的缺點是「空間複雜度為 `O(N^2)`」， 能不能把優化呢？，這時候可以用**中心擴散法**。Palindrome 的特性是「左右對稱」，所以我們可以窮舉字串中的每一個字元，把它當作「中心點」然後同時往左右兩邊擴散，直到左右兩邊的字元不相等為止。
要考慮所謂中心點，有兩種可能：

- 奇數長度回文： 以單一字元為中心，例如 "aba" 的中心是 "b"
- 偶數長度回文： 以兩個字元的間隙為中心，例如 "abba" 的中心是 "bb"

這時候雖然時間複雜度一樣是 `O(N^2)`，因為窮舉 `N` 個中心點，每個中心點可擴散約 `N/2` 次 ; 但空間複雜度變成`O(1)` ! 因為只需要記錄幾根指針（left, right）和目前最大長度，完全不用開任何二維陣列！

### 解答

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        if len(s) == 1:
            return s

        ans = s[0]

        for i in range(len(s)):
            odd_left, odd_right = self.get_indices(s, i, i)
            even_left, even_right = self.get_indices(s, i, i+1)

            current_best = s[odd_left : odd_right + 1] if (odd_right - odd_left) > (even_right - even_left) else s[even_left : even_right + 1]

            if len(current_best) > len(ans):
                ans = current_best

        return ans

    def get_indices(self, s:str, left:int, right: int) -> tuple[int, int]:
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left = left -1
            right = right + 1

        return left + 1, right-1
```

{{< alert warning >}}

- 因為 Python 切片是「 左閉右開 」的，並不包含右邊界，故才寫例如 `odd_right + 1` ，這樣才能抓到最後一個字元
- `current_best` 要跟在 `ans` 回文比大小
- `return left + 1, right-1` 是要還原合法邊界。當while 迴圈跳出時，此時的 left 和 right 已經不合法，所以要還原回去
- `left -= 1` 等價 `left = left -1 `
  {{< /alert >}}

上面是回傳 index 方式，但其實也可以直接回傳當前位置找到的最大 Palindrome。
另外還可以用了一個資料前處理的小技巧：

> 在字串的頭尾和中間，每隔一個字母就插入一個原字串不存在的符號 `#`

這麼做的目的是可以「不用分兩個迴圈」寫，不用管給定字串的字母數，為奇數或者為偶數。例如 :

'bb'和'bab'，字母數前者是偶數，後者是奇數，這樣檢查時的起點會不同，但在插入 `#` 之後分別變成 `#b#b#` 和 `#b#a#b#`，字母數都是奇數， 最後再將插入的 `#` 消除即可。由於只在演算法開頭和結尾各做一次，故也不會影響時間複雜度。

```python
class Solution:
    class Solution:
    def longestPalindrome(self, s: str) -> str:
        if len(s) == 1:
            return s
        
        s = '#' + ('#').join(c for c in s) + '#'
        n = len(s)
        max_length = 1
        ans = s[0]

        for i in range(n):
            palindrome =  self.get_the_index_max_palindrome(s, i, i)
            
            if len(palindrome) > max_length:
                max_length = len(palindrome)
                ans = palindrome
        
        return ans.replace('#', '')
    
    def get_the_index_max_palindrome(self, s: str, left: int, right: int) -> str:
        while left >=0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        
        return s[left+1 : right]

```

---

### Vocabulary

{{< alert info >}}
**palindrome** [ˋpælɪn͵drom]

**palindromic** [ˌpælɪnˈdrɑmɪk]

(只會出現在考題的單字，記得怎麼念就好)

n.[C] 回文字

{{< /alert >}}

---

### 參考資料

- [[LeetCode]5. Longest Palindromic Substring 中文](https://www.youtube.com/watch?v=ZnzvU03HtYk)

- [Manacher's Algorithm](https://medium.com/hoskiss-stand/manacher-299cf75db97e)

- [Day 2: 按照規律的方式填表可以解決大部分的問題！](https://ithelp.ithome.com.tw/articles/10215365)

- [LeetCode in Python 5 Longest Palindromic Substring｜动态规划 [On Strings style] - Michelle小梦想家](https://www.youtube.com/watch?v=nSFWpXuNfyw)
