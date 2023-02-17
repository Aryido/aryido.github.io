---
title: "16. 3Sum Closest"

author: Aryido

date: 2023-02-08T21:56:12+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 這題跟 15 題非常相似，又增加了些許難度。題目敘述一樣也簡單，求 nums 內最接近 *target* 值的三數和。優化關鍵點一樣是，*把 nums 排序*，這樣就可以確定指針滑動方向。
<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/leetcode/16.jpg" >}}

## 思路
因為是求最接近 *target* 值的三數和，而沒有管其 *index*，故可以考慮將 *nums* 排序。排序有甚麼好處呢 ? 計算第 *i* 值、第 *j* 值、第 *k* 值加總後，會有三種情況 :
- 等於 target：直接返回 target 值即可
- 小於 target：代表加總值不夠大，**因為排序**，固可將 *j* 點往右移動，尋更大的值
- 大於 target：代表加總值太大，**因為排序**，固可將 *k* 點往左移動，尋找更小的值

---

# 解答
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        if(nums.length == 3){
            int sum = 0;
            for(int num : nums){
                sum = sum + num;
            }
            return sum;
        }
        Arrays.sort(nums);
        int result = 0;
        int closeTarget = 100000;
        for(int i = 0 ; i < nums.length - 2 ; i++){
            int j = i + 1;
            int k = nums.length - 1;
            while(j < k){
                int sum = nums[i] + nums[j] + nums[k];
                int diff = Math.abs(sum - target);
                if(diff < closeTarget){
                    result = sum;
                    closeTarget = diff;
                }
                if(sum == target){
                    return target;
                }else if(sum < target){
                    j++;
                }else{
                    k--;
                }
            }
        }
        return result;
    }
}
```

{{< alert info >}}
**Java** 有些常用的**lib**或語法，記得的話其實都可以幫助寫出簡潔的 code。
```
## array 的 Stream api 求和
Arrays.stream(nums).sum();

## array 的 sort
Arrays.sort(nums);

## int java最大值
int closeTarget = Integer.MAX_VALUE;

```
{{< /alert >}}


{{< alert info >}}
會寫 **int closeTarget = 100000;** ，其實是因為看到題目的 *target* 條件 ```-10^4 <= target <= 10^4```，所以就寫了個 **10^5**，保證比 **10^4 * 3** 還要大。但還是推薦記下```Integer.MAX_VALUE```是最好的
{{< /alert >}}

{{< alert warning >}}
特別思考下 *for loop* 的終止條件，原本我是寫 ```nums.length - 1```，但其實可以使用 ```nums.length - 2```。


從程式方面去想，如果 *i* 終止條件是```nums.length - 1```，則在 *i* = ```nums.length - 1```時，會發現 *while loop* 根本進不去，其實不需要這一步。

從圖像上去思考的話，在 *j*、 *k* 到達最尾端時，剛剛好佔了兩格。所以 *i* 最多只需要到*倒數第三格*，也就是不超過 ```nums.length - 2``` !

{{< /alert >}}

---

## Refactor
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        if(nums.length == 3){
            return Arrays.stream(nums).sum();
        }
        Arrays.sort(nums);
        int result = 0;
        int closeTarget = Integer.MAX_VALUE;
        for(int i = 0 ; i < nums.length - 2 ; i++){
            int j = i + 1;
            int k = nums.length - 1;
            while(j < k){
                int sum = nums[i] + nums[j] + nums[k];
                int diff = Math.abs(sum - target);
                if(diff < closeTarget){
                    result = sum;
                    closeTarget = diff;
                }

                if(sum == target){
                    return target;
                }else if(sum < target){
                    while(j < k && nums[j] == nums[j+1]) j++;
                    j++;
                }else{
                    while(j < k && nums[k] == nums[k-1]) k--;
                    k--;
                }
            }
        }
        return result;
    }
}
```

{{< alert danger >}}
在移動指針的時候，多加 *while* 迴圈判斷，但看起來時間反而花更多了...
```
else if(sum < target){
    while(j < k && nums[j] == nums[j+1]) j++;
    j++;
}else{
    while(j < k && nums[k] == nums[k-1]) k--;
    k--;
}
```
{{< /alert >}}