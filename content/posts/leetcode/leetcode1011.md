---
title: 1011. Capacity To Ship Packages Within D Days

author: Aryido

date: 2022-10-15T19:18:43+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- binary-search

comment: false

reward: false
---
<!--BODY-->
這題是 Google 面試題，在 Hide Hint 中表示可以使用 binary-search 解決，剛開始覺得蠻 tricky 的，但仔細思考會覺得 binary-search 很符合這題目。

<!--more-->

---
用傳送帶運送包裹，要求在days天內把全部包裹運送完。weights給出了每個包裹的重量
- 必須按 weight s的順序來運送包裹，不可排序。
- ship 每次運送可以盡可能多運包裹但是總重量不能超過運力上限

現在問要保證在days天運送完全部包裹的傳送帶的最小運力上限是多少 ?

---

## 思路
首先要想出
- *運力上限不能小於 weights 裡的最大值*

要不然該 weight 最重的貨物會沒辦法運送。

接下來思考運力的上限
- *weights 全部加總就是 運力的上限*

代表於一天之內一次全部載走

確定了這個解的上限和下限，我們要做的就是從 **weights 最大值** 和  **weights總和** 的區間範圍內，找到一個最小的且滿足條件的值，這才想到了 binary-search。

另一個關鍵在於，**把 weights 轉換成天數**，要有這個轉換才能跟 days 比較。

---

# 解答
```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int left = 0;
        int right = 0;
        for(int weight : weights){
            if(weight > left){
                left = weight;
            }
            right += weight;
        }

        while(left < right){
            int mid = left + (right - left ) / 2 ;
            int target = cal(weights, mid);
            if(target <= days){
                right = mid;
            } else{
                left = mid + 1;
            }
        }
        return left;
    }

    public int cal(int[] weights, int mid){
        int costday = 1;
        int shipWeight = 0;
        for(int weight : weights){
            if(shipWeight + weight <= mid){
                shipWeight += weight;
            } else {
                shipWeight = weight;
                costday ++;
            }
        }
        return costday;
    }
}
```
{{< alert warning >}}
cal 函數會判斷該 input 運力，會花多少時間把貨物載完。

- 注意 costday 初始化是 1。
- 當貨物載不下去後會換船，shipWeight 會有**初始重量**，不要忘記了~

{{< /alert >}}

{{< alert warning >}}
對於運力，需要判斷多少天，這要和題目給的 days 比較，需要仔細一點:

- 如果 *target < days*, 代表運力太少了，運力一定是需要增加， 那麼一定是 *l= target + 1*
- 如果 *target == days*, 那有可能當前的運力是答案， 因為可能運力減少以後就不符合要求了，所以這裡設置 *r = target*
- 如果 *target > days*, 這時候也應該設置*r = mid*, 因為運力減少以後也有可能不符合要求
{{< /alert >}}

---

補充一個完全使用解題模板得出來的解答，解題模板因為都要在一個 array 上操作，所以多做了把**運力 array 化的動作**，時間會很慢就是了XDD...

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int left = 0;
        int right = 0;

        for(int weight : weights){
            if(weight > left){
                left = weight;
            }
            right += weight;
        }

        int[] arr = new int[right - left + 1];
        int range = right - left + 1;
        for(int i = 0 ; i < range ; i ++ ){
            arr[i] = left;
            left++;
        }

        int l = -1;
        int r = arr.length;

        while(l + 1 < r){
            int mid = l + ( r - l ) / 2 ;
            int target = cal(weights, arr[mid]);
            if(target <= days){
                r = mid;
            } else{
                l = mid;
            }
        }

        return arr[r];
    }

    public int cal(int[] weights, int mid){
        int costday = 1;
        int shipWeight = 0;
        for(int weight : weights){
            if(shipWeight + weight <= mid){
                shipWeight += weight;
            } else {
                shipWeight = weight;
                costday ++;
            }
        }

        return costday;
    }
}
```

---

# Vocabulary

{{< alert info >}}
**conveyor** [kənˋveɚ] = conveyer

n. 運輸工具 ex: conveyor belt

{{< /alert >}}

---