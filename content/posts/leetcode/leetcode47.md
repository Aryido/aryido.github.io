---
title: 47. Permutations II

author: Aryido

date: 2022-12-04T18:41:26+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- backtrack

comment: false

reward: false
---
<!--BODY-->
> 是經典的 46. Permutations 的進階，現在有 duplicate 數字，一樣使用 backtrack 來求解。從數學上來說，*n* 個 element ，且將相同的事物歸為一組, 可歸成*k*組, 且每組有*m_i*個，其 Permutation 一共有 *n!/(m_1!m_2!...m_k!)* 種排序，高中數學題需要思考下的基礎題，要用程式模擬這個過程也有點難度，被歸類在 Medium 等級。
<!--more-->

---

## 思路
承前面　[46. Permutations](https://leetcode.com/problems/permutations/)　解法，**用交換 num 裡面的兩個數字**的方式來做 DFS ，基本上可以讓 code 幾乎一樣，但須要引入茶重複機制。
{{< image classes="fancybox fig-100" src="/images/leetcode/permutation2.jpg" >}}

思考上面的圖可能初次會覺得 *[2,1,2]* 和 *[2,2,1]* 狀態不一樣，但看下層的狀態，*[1,2]* 和 *[2,1]* 的全排列其實是一樣的。
所以在嘗試所有可能性時，利用 hashset 來把**發現已經嘗試過的元素而產生的 branch 給剪除掉**。

# 解答
```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();

    public List<List<Integer>> permuteUnique(int[] nums) {
        dfs(nums, 0);
        return ans;
    }

    private void dfs(int[] nums, int index){
        if(index >= nums.length){
            List<Integer> res = new ArrayList<>();
            for(int num : nums){
                res.add(num);
            }
            ans.add(res);
            return;
        }
        Set<Integer> set = new HashSet<>();
        for(int i = index ; i < nums.length ; i++){
            if(set.add(nums[i])){
                swap(nums, i, index);
                dfs(nums, index+1);
                swap(nums, index, i);
            }
        }
    }

    private void swap(int[] nums, int i, int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}

```
{{< alert info >}}
java 的小技巧， **set 的 add 函數有 bool 回傳值**，可把 code 做簡化。

推薦這種寫法:
```
if(set.add(nums[i])){
    swap(nums, i, index);
    dfs(nums, index+1);
    swap(nums, index, i);
}
```

攏長寫法:
```
if(set.contains(nums[i])){
    continue;
}
set.add(nums[i]);
swap(nums, i, index);
dfs(nums, index+1);
swap(nums, index, i);

```
{{< /alert >}}

---