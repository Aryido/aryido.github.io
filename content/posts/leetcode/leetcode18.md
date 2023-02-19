---
title: "18. 4Sum"

author: Aryido

date: 2023-02-15T22:17:40+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> leetcode 幾個數字題，15、16、18，基本上套路都是一樣的(~~甚至可以預期可能還會出 5 Sum...~~)，整體的解法都差不多可以一起看，重點仍然是 :
> 1. 排序 array
> 2. 避免的重複項
>
> 這邊只是 3Sum 以此基礎上，再加了一個循環而已。 這連續幾題 sum 算是給了一個不錯的練習呢 !

<!--more-->

---

# 解答
```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> ans = new ArrayList<>();
        if(nums.length == 4){
            long sum = 0;
            for(int num : nums){
                sum = sum + num;
            }
            if(sum == target){
                List<Integer> list = List.of(nums[0],nums[1],nums[2],nums[3]);
                ans.add(list);
            }
            return ans;
        }

        Arrays.sort(nums);
        for(int i = 0 ; i < nums.length - 3 ; i++){
            for(int j = i+1 ; j < nums.length - 2 ; j ++){
                int k = j + 1;
                int l = nums.length - 1;
                while(k < l){
                    if(nums[i] + nums[j] + nums[k] + nums[l] == target){
                        List<Integer> list = List.of(nums[i],nums[j],nums[k],nums[l]);
                        ans.add(list);

                        while(j < l && nums[j] == nums[j+1]) j++;
                        while(k < l && nums[k] == nums[k+1]) k++;
                        while(k < l && nums[l] == nums[l-1]) l--;
                        k++;
                        l--;
                    }else if(nums[i] + nums[j] + nums[k] + nums[l] > target){
                        l--;
                    }else {
                        k++;
                    }
                }
                while(i < l && nums[i]==nums[i+1]) i++;
            }
        }
        return ans;
    }
}
```

{{< alert warning >}}
特別思考下 *for loop* 的終止條件 ```nums.length - 3```。

從圖像上去思考的話，在 *j*、 *k*、 *l* 到達最尾端時，剛剛好佔了三格。所以 *i* 最多只需要到*倒數第四格*，也就是不超過 ```nums.length - 3```。

基本上以此模板，可以推論 :
- ```N SUM : nums.length - (N-1)```

{{< /alert >}}

---