---
title: 877. Stone Game

author: Aryido

date: 2022-11-25T23:36:44+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dp

comment: false

reward: false
---
<!--BODY-->
> 石頭遊戲，兩個人輪流選石頭，Alex 先選，每次只能選開頭或結尾，最終獲得石頭總數多的人獲勝。 乍看之下不好想到可以用 DP 解，但其實可用一個 2D-state 去描述遞迴的狀態。 這題一開始會好奇是因為負評倒讚很多，個人是感覺能從 Game Theory 單純想出這結論也是蠻厲害的...
<!--more-->

---
題目中的條件
- 有偶數堆
- 石子總數為奇數個

因為是輪流拿，再根據以上條件，則可以分成兩大堆，且兩大堆的數量一定不一樣。 先拿得可以掌控自己是拿到*奇數堆*或*偶數堆*，這剛好是兩堆，又因為兩堆一定有一個比較大，故先拿得一定可以拿到比較大的。 >> **先手必勝**

所以解答有人直接返回 **true**(把刷題當成 Game 來玩...)

## 思路
用兩個指針 *i* 和 *j*，分別指向頭和尾的位置。當選取了頭時，則 *i++*，若選了尾時，則 *j--* (未完成)
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
**objective** [əbˋdʒɛktɪv] : 不要只記得 object 是物體

adj. 客觀的，如實的，無偏見的;
n. 目的，目標；

The objective of the game is to end with the most stones

{{< /alert >}}

{{< alert info >}}
**optimally** [ɑptəməli]

adv. 最佳地；最適地；最優地;

To be optimally effective, the drug should be taken with food.

{{< /alert >}}

---