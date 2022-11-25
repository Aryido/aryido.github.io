---
title: 877. Stone Game

author: Aryido

date: 2022-11-25T23:36:44+08:00

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
> 石頭遊戲，兩個人輪流選石頭，Alex 先選，每次只能選開頭或結尾，最終獲得石頭總數多的人獲勝。乍看之下不好想到可以用 DP 解，但其實可用一個 2D-state 去描述遞迴的狀態。
<!--more-->

---

## 思路
用兩個指針 *i* 和 *j*，分別指向頭和尾的位置。當選取了頭時，則 *i++*，若選了尾時，則 *j--*
# 解答
```java
class Solution {
    Integer[][] memo;
    public boolean stoneGame(int[] piles) {
        int size = piles.length;
        memo = new Integer[size][size];
        return dfs(0,size-1,piles) > 0;
    }

    private int dfs(int i, int j, int[] piles){
        if( i > j ){
            return 0;
        }
        if( i == j ){
            return piles[i];
        }

        if(memo[i][j] != null){
            return memo[i][j];
        }

        int res;
        res = Math.max(
            piles[i] - dfs(i + 1,j,piles),
            piles[j] - dfs(i,j - 1,piles)
        );
        return memo[i][j] = res;
    }
}
```

---
# Vocabulary

{{< alert info >}}
**objective** [əbˋdʒɛktɪv]

adj. 客觀的，如實的，無偏見的;
n. 目的，目標；

The objective of the game is to end with the most stones

{{< /alert >}}

---