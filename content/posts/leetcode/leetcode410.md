---
title: 410. Split Array Largest Sum - dynamic programming

author: Aryido

date: 2022-11-27T20:26:22+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dp

comment: false

reward: false
---
<!--BODY-->
> 這題真的蠻難的，一開始看題目我也覺得很繞口，給了一個非負數的 nums 和一個 m 代表把 nums 分成 m 個 group 且 每個 group non-empty 並取 m 個 group 中的最大值。但注意，前面只是代表一種**切法**而已，我們是要找所有可能**切法**之中的最小值。看一下 Related Topics 發現可以用 Binary Search 和 DP 求解，也是一道高頻題目。
<!--more-->

---

這邊用題目給的例子來舉例，會更清楚 :  ```nums = [7,2,5,10,8], k = 2```，分成兩組有多少種分法呢?
- [7][2,5,10,8] >> 25
- [7,2][5,10,8] >> 23
- [7,2,5][10,8] >> 18
- [7,2,5,10][8] >> 24

因為要取分法裡面的最小值，所以我們要的答案是 **18**

---

## 思路
State:  為一個二維 dp，其中 dp[m][n] 表示將 nums 中前 n 個數字分成 m 組，取 Largest sum。

Base Case:
- (1,n) 代表只有 1 組 group，Largest sum 就是 sum(n)，要回傳 sum(n)

- *m > n* 代表 Invalid，因為不可能把 n個數字分成比 n 還多的非空 group，故不考慮這個狀態。
{{< alert info >}}
不考慮 *m > n* 這個狀態，這裡我們 return *-1* ，表示不可能的情況，因為題目每個數字都是非負的，故總和一定大於 *0*。
{{< /alert >}}

再來考慮一下中間狀態，回憶一下 [139. Word Break](https://leetcode.com/problems/word-break/) 的思路，首先切一刀分成左右兩邊:
- right 是不可在分割的
- left 是要繼續遞迴的

因為切一刀，假定 left 有 k 個數字； right 就有 n-k 個數字。可以假定 right 就是一個 group ，而 left 就只剩 m-1 個 group。

所以 right 是一組 group，值為 sum(k,n) ； 而 left 就是新的狀態，且需要分成 m-1 個 group。 要回傳的答案就是 left 和 right 做對比，選大的。

當前前面整個分析，只是其中一種切法而已，我們要做的是用 for loop 遍歷所有切法，然後取最小值。

{{< alert warning >}}
如何快速計算 sum(k,n) 呢 ?

可以使用 **prefix sum** 這個方式，可參考 [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)
{{< /alert >}}

# 解答
```java
# 反向 DP
class Solution {
    Integer[][] memo;
    int[] prefix;

    public int splitArray(int[] nums, int k) {
        int n = nums.length;
        memo = new Integer[k + 1][n + 1];
        prefix = new int[n + 1];
        for(int i = 0 ; i < n ; i++){
            prefix[i+1] = prefix[i] + nums[i];
        }
        return dfs(nums, k , n);
    }

    private int dfs(int[] nums, int m, int n){
        if(m == 1){
            return prefix[n];
        }

        if(m > n){
            return -1;
        }

        if(memo[m][n] != null){
            return memo[m][n];
        }

        int res = Integer.MAX_VALUE;
        for(int k = 1; k < n ; k++){
            int left = dfs(nums, m - 1, k);
            if(left == -1){
                continue;
            }
            int right = prefix[n] - prefix[k];
            int sub = Math.max(left, right);

            res = Math.min(res,sub);
        }

        return memo[m][n] = res ==  Integer.MAX_VALUE ? -1 : res;
    }
}
```
{{< alert warning >}}
```java
int[] nums = [7,2,5,10,8]
int n = nums.length;
int[] prefix = new int[n + 1];
for(int i = 0 ; i < n ; i++){
    prefix[i + 1] = prefix[i] + nums[i];
}
```
以上定義可知 *prefix[i]* 代表 nums 前 i 個值得sum。

- *prefix[0]* = nums 前 0 個值得 sum = 0
- *prefix[1]* = nums 前 1 個值得 sum = prefix[0] + nums[0] = nums[0]
- *prefix[2]* = nums 前 2 個值得 sum = prefix[1] + nums[1] = nums[0] + nums[1]
- *prefix[n]* =  nums 前 n 個值得 sum = prefix[n - 1] + nums[n - 1] = (nums[0] + nums[1] + ... + nums[n-1] = nums全部總和

所以 *sum(k,n)* 可以簡單寫成 *prefix[n] - prefix[k]*，這就是我們 *int[] prefix = new int[n + 1];* 要 padding 的原因，讓表示方便好看。
{{< /alert >}}

{{< alert info >}}
會使用```int res = Integer.MAX_VALUE;```是因為是求所有切割法的最小值，用一個上界來避免選到錯誤的 res
{{< /alert >}}

---