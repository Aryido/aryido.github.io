---
title: 5. Longest Palindromic Substring

author: Aryido

date: 2022-09-14T22:30:14+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- java

tags:
- java
- LeetCode
- dynamic-programming

comment: false

reward: false
---
<!--BODY-->
> 這是一道常考題，也因為是 DP，難度也比較高。 看過令人膜拜的神解 Manacher's Algorithm，時間複雜度提升到了 O(n) ，但一般人還是熟悉一般 DP 解就好了...

<!--more-->
## 思路
{{< image classes="fancybox fig-100" src="/images/leetcode/leetcode5.jpg" >}}

我們維護一個2D-Matrix dp(如上圖)，其中 dp[i][j] 表示字符串區間 [i, j] 是否為回文串，當 i = j 時，只有一個字符，肯定是回文串。

再來是 Palindromic 的特性: 若字串最兩端一樣，且 dp[i+1][j-1] 也是 Palindromic 則，整個字串 Palindromic。

注意，在 for loop 時候，必須:
```java
//由上往下
for(int j = 1 ; j < length ; j++){
    for(int i = 0; i < j ; i++){
        ...
    }
}

//由右往左
for(int i = length - 1 ; i >= 0 ; i--){
    for(int j = length - 1; j >= i ; j--){
        ...
    }
}

```
因為在判斷 dp[i][j] 時，需要先有 dp[i+1][j-1] 的資訊，要不然會出錯。

*j-i <= 2* 的理由是因為前面判斷了*s.charAt(i) != s.charAt(j)*，故子字串只要長度小於等於二，就一定是回文。 再來只要 dp[i][j] == true，就表示 s.substring(i, length) 是回文。其中長度是 j-i+1 。

時間複雜度: O(N^2)

# 解答
```java
class Solution {
    public String longestPalindrome(String s) {
        if(s.length() == 1){
            return s;
        }


        // init
        int length = s.length();
        int maxLength = 1;
        int ansBeginIndex = 0;
        boolean[][] dp = new boolean[length][length];
        // dp's diagonal elements are always true
        for(int j = 0 ; j < length ; j++){
            dp[j][j] = true;
        }


        for(int j = 1 ; j < length ; j++){
            for(int i = 0; i < j ; i++){
                if(s.charAt(i) != s.charAt(j)){
                    dp[i][j] = false;
                }else{
                    if( j-i <= 2){
                        dp[i][j] = true;
                    }else{
                        dp[i][j] = dp[i+1][j-1];
                    }

                }

                if(dp[i][j] && j - i + 1 > maxLength){
                    maxLength = j - i + 1;
                    ansBeginIndex = i;

                }
            }
        }

        return s.substring(ansBeginIndex, ansBeginIndex + maxLength);
    }
}
```
---