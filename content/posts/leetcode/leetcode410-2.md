---
title: 410. Split Array Largest Sum - binary search

author: Aryido

date: 2022-12-11T18:32:09+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- binary-search

comment: false

reward: false
---
<!--BODY-->
> 前面有介紹用 dp 方式把這題給解了，但看一下 Related Topics 發現也可以用 Binary Search 求解，上網參考大神們的解法，感覺特別巧妙。因為這題可用 dp 和  Binary Search，也變成是一道高頻難題。
> 這邊記錄一下大神們的想法。
<!--more-->

---
為甚麼可用 Binary Search 呢 ? 首先思考
- **m 為 1**

    那麼整個 nums 數組就是一組，返回 nums 所有數字的和

- **m 和 nums 個數相等**

    那每個數組都是一組，所以返回 nums 中最大的數字即可

所以對於其他有效的 m 值，返回值必定在上面兩個值之間 >> **Binary Search**

---

舉個簡單例子說明 :
{{< alert success >}}
這邊使用
- 判斷式:  **left < right**
- **right = mid**
- **left = mid + 1**
{{< /alert >}}

```nums = [1, 2, 3, 4, 5], m = 3```
- left 設為數組中的最大值 5
- right 設為數字之和 15
- 然後算出中間數為 10

接下來要做的是找出和最大且**小於等於** 10 的子數組的個數。

{{< alert danger >}}
為甚麼要 **小於等於**  呢 ? 還要在思考下...
{{< /alert >}}

- ```[1, 2, 3, 4], [5]```

    可以看到無法分為 3 組，說明 mid 偏大，所以讓 **right = mid**，然後再次進行二分查找，算出 *mid = (10 + 5)/2= 7*，再次找出和最大且小於等於 7 的子數組的個數。
- ```[1,2,3], [4], [5]```

    成功的找出了三組，說明 mid 還可以進一步降低，讓 **right = mid**，再次進行二分查找，算出 *mid = 6*，再次找出和最大且小於等於 6 的子數組的個數
- ```[1,2,3], [4], [5]```

  成功的找出了三組，嘗試著繼續降低 mid，讓 **right = mid**，再次進行二分查找，算出 *mid = 5*，再次找出和最大且小於等於 5 的子數組的個數
- ```[1,2], [3], [4], [5]```

    發現有4組，此時的 mid 太小了，應該增大 mid，讓 **left = mid + 1**，此時 **left=6 & right=6**，退出循環，返回 **right** 。

## 思路
這邊換個方式
{{< alert success >}}
使用
- 判斷式是 **left <= right**
- **right = mid - 1**
- **left = mid + 1**
{{< /alert >}}

```nums = [7,2,5,10,8], m = 2```
- left 設為數組中的最大值 10
- right 設為 nums 和 32
- 然後算出 mid 為 21

接下來要做的是找出和最大且**小於等於** 21 的 group 的個數。
{{< alert danger >}}
為甚麼要 **小於等於**  呢 ? 還要在思考下...
{{< /alert >}}
- ```[7,2,5], [10, 8]```

    成功的找出了 2 組，說明 mid 可能可以進一步降低，讓 **right = mid - 1 = 21 - 1 = 20**，再次進行二分查找，算出 *mid  = (20 + 10)/2 = 15*

接下來要做的是找出和最大且小於等於 15 的 group 的個數
- ```[7,2,5], [10], [8]```

  成功的找出了 3 組，但 3 組已經比 m 還多，讓 **left = mid + 1 = 15 + 1 = 16**，再次進行二分查找，算出 *mid = (20 + 16)/2 = 18*

接下來要做的是找出和最大且小於等於 18 的 group 的個數
- ```[7,2,5], [10,8]```

    發現有 2 組，說明 mid 可能可以進一步降低，讓 **right = mid - 1 = 20 - 1 = 19**，再次進行二分查找，算出 *mid = (19 + 18)/2 = 18*

接下來要做的是找出和最大且小於 18 的 group 的個數
- ```[7,2,5], [10,8]```

    發現有 2 組，說明 mid 可能可以進一步降低，讓 **right = mid - 1 = 19 - 1 = 18**，再次進行二分查找，算出 *mid = (18 + 18)/2 = 18*

接下來要做的是找出和最大且小於 18 的 group 的個數
- ```[7,2,5], [10,8]```

    發現有 2 組，說明 mid 可能可以進一步降低，讓 **right = mid - 1 = 18 - 1 = 17**，此時發現 *left > right* ，無法再次進行二分查找，返回 left 。

# 解答
```java
class Solution {
    public int splitArray(int[] nums, int k) {
        int left = 0;
        int right = 0;
        for(int num : nums){
            if(num > left){
                left = num;
            }
            right += num;
        }

        while(left <= right){
            int mid = left + (right-left)/2;
            if(isValid(nums, k, mid)){
                right = mid - 1;
            }else{
                left = mid + 1;
            }
        }

        return left;
    }

    private boolean isValid(int[] nums, int m, int val){
        int groupNeed = 0, curSum = 0;
        for(int num: nums){
            curSum += num;
            if(curSum > val){
                ++groupNeed;
                curSum = num;
            }
        }
        if(curSum > 0){
            ++groupNeed;
        }
        return groupNeed <= m;
    }
}
```
{{< alert danger >}}
*binary search* 最難的部分 :
- 判斷式的範圍到底有沒有等號...
- right 和 left 的減一加一

多寫多看多想吧...QQ
{{< /alert >}}

---