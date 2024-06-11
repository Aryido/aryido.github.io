---
title: "322. Coin Change"

author: Aryido

date: 2023-12-24T23:14:10+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- java
- dp

comment: false

reward: false
---
<!--BODY-->
> 這題是著名的**零錢兌換**問題，給我們一些可用的硬幣面額，問**最少**能用幾個硬幣來兌換出來。標準解法是用 Dynamic Programming 。比較需要知道的地方，是初見時都會想使用 Greedy 方法來做，也就是從最大的 coin 面額拿最多的硬幣開始，如果這個數目無法滿足，那麼從最大的面額數目減一，再重複步驟。但這個解法是錯的 ! 因為**不保證大面額硬幣拿最多的數目就是最佳解**...
>
>
<!--more-->

---

# 思路

這題的大問題就是 F(amount) = MinCoins
可以推論 F(amount - coins[i]) = MinCoins-1
F(amount) = F(amount - coins[i]) + 1
這樣就可以縮小問題了。



# 解答
```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount+1];
        Arrays.fill(dp, 10001);
        dp[0] = 0;

        for(int i = 1 ; i <= amount ; i++){
            for(int coin : coins){
                if(i - coin >= 0 ){
                    dp[i] = Math.min(dp[i], dp[i-coin]+1);
                }
            }
        }

        return dp[amount] ==  10001 ? -1 : dp[amount];
    }
}
```

```java

```

---

# 時間空間複雜度

### 時間複雜度 : ```O(N^2)```
兩層迴圈

### 空間複雜度 : ```O(N^2)```
2-D陣列

---

{{< alert info >}}
example
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/leetcode/logo.jpg" >}}

---

# Vocabulary

{{< alert info >}}
**denomination** [dɪ͵nɑməˋneʃən] :

- 1.名稱[C]
- 2.命名[U]
- 3.宗派，教派[C]
Among Christians there are many denominations. 基督教中有許多教派。
- 4.（貨幣等的）面額；（度量衡等的）單位[C]
stamps of different denominations 面額不同的郵票

{{< /alert >}}

---
### 參考資料

- [[LeetCode]5. Longest Palindromic Substring 中文](https://www.youtube.com/watch?v=ZnzvU03HtYk)

- [[Day 5] Leetcode 322. Coin Change (C++)](https://ithelp.ithome.com.tw/articles/10262309)

- [贾考博 LeetCode 322. Coin Change - 无限金钱!](https://www.youtube.com/watch?v=aQhNCYN5TsU)

- [【小小福讲Leetcode】LeetCode 322. Coin Change 两种方法详细解答](https://www.youtube.com/watch?v=EM9YWv1hBSk)