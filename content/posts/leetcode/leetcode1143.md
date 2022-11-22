---
title: 1143. Longest Common Subsequence

author: Aryido

date: 2022-11-22T22:46:42+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- dp

comment: false

reward: false
---
<!--BODY-->
> 這題是求最長相同的**子序列**，可用 Dynamic Programing 來做，最難的還是想出狀態函數。這裡使用 2D-dp ，其中 dp[i][j] 表示 :
> - text1 的前 i 個字符
> - text2 的前 j 個字符
>
> 的最長相同的子序列的字符個數
<!--more-->

---
**Subsequence** 和 **Substring**，是有差別的
- Subsequence可以跳位置，但是順序還是要保持一致
- Substring 一定要連續字符

{{< alert info >}}
Longest Common Subsequence 以下簡稱 LCS
{{< /alert >}}

## 思路
我們先定義 State 函數
{{< alert success >}}
State(i,j) :

Longest Common Subsequence of [0,i] for text1 and [0,j] text2
{{< /alert >}}

從兩字串最後面開始，若二者最後面對應位置的字符相同，表示當前的 LCS 長度 + 1，所以可以用 dp[i-1][j-1] + 1。否則錯位比較，可以分別從 text1 或者 text2 去掉一個當前字符，那麼 LCS 長度就是 dp[i-1][j] 和 dp[i][j-1]，取二者中的較大值來更新。

# 解答
```java
class Solution {
    Integer[][] memo;
    public int longestCommonSubsequence(String text1, String text2) {
        memo = new Integer[text1.length()][text2.length()];
        return dfs(text1.length() - 1, text2.length() - 1, text1, text2);
    }

    private int dfs(int i, int j, String text1, String text2){
        if(i < 0 || j < 0){
            return 0;
        }

        if(memo[i][j] != null){
            return memo[i][j];
        }

        int res = 0;
        if(text1.charAt(i) == text2.charAt(j)){
            res =  dfs(i - 1, j - 1, text1, text2) + 1;
        }else{
            res = Math.max(dfs(i - 1, j, text1, text2), dfs(i, j - 1, text1, text2));
        }

        return memo[i][j] = res;
    }
}
```
{{< alert warning >}}
DP 題的 memo 常用技巧，就是 Integer[][]，這樣可以用位置是否為 null 來判斷是否有算過。int[][]會再全部位置上初始0
{{< /alert >}}


{{< alert warning >}}
為什麼 *i < 0* 或 *j < 0* 要 return 0 了?

可以想成是如果和一個空 string 比較，LCS長度自然是 0 ，會比較好理解
{{< /alert >}}


{{< alert danger >}}
return 室友返回值的，用上```memo[i][j] = res``` 簡化 Code
{{< /alert >}}


---