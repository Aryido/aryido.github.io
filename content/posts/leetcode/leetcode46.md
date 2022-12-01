---
title: 46. Permutations

author: Aryido

date: 2022-11-30T23:38:55+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 是一道經典的全排列問題，這邊使用 backtrack 來求解。從數學上來說，*n* 個 element 的 Permutation 一共有 *n!* 種排序，思考起來算蠻簡單的，但要用程式模擬這個過程，是有點難度的，被歸類在 Medium 等級。

<!--more-->

---

## 思路
backtrack 類型的題目，都算是**Search**類型的題目，都類似 Top Down DFS 的寫法，故蠻推薦把 Recursion Search Tree 畫出來幫助思考。

{{< image classes="fancybox fig-100" src="/images/leetcode/permutation.jpg" >}}

這邊使用一個比較巧妙的方法，**用交換 num 裡面的兩個數字**的方式，來做 DFS ，經過遞迴可以產出所有的排列情況。

最高層的問題就是我們的原始 input，而 State 設計，用位置 index 做為參數。第一層做完就會確定第一個位置的數字，有 ```nums.length``` 種可能性；第二層做完就會確定第二個位置的數字，有 ```nums.length - 1``` 種可能性，以此類推...

{{< alert info >}}
這種**交換 num 裡面的兩個數字**的方式也給他的下個進階題 *47. Permutations II* 做了巧妙鋪墊。
{{< /alert >}}

# 解答
```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> permute(int[] nums) {
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

        for(int i = index ; i < nums.length ; i++){
            swap(nums, index, i);
            dfs(nums, index + 1);
            swap(nums, i, index);
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
一般這種 Top Down DFS 問題都是 return void 的；botton up DFS 才會有 return 具體的 type。
{{< /alert >}}

{{< alert warning >}}
因為 java object 是存地址，故要把當前答案做一個 copy
```java
List<Integer> res = new ArrayList<>();
for(int num : nums){
    res.add(num);
    }
ans.add(res);
```
目前 java 沒有天然提供 ```int[]``` 轉 ```List<Integer>``` 的 util。
{{< /alert >}}

{{< alert warning >}}
```java
swap(nums, index, i);
dfs(nums, index + 1);
swap(nums, i, index);
```
代表狀態的 backtrack，讓其他分支狀態不互相干擾。
{{< /alert >}}

---