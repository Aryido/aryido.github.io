---
title: "90. Subsets II"

author: Aryido

first draft: 2022-11-15T22:43:17+08:00

date: 2023-10-17T23:01:42+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- dfs
- backtrack

comment: false

reward: false
---
<!--BODY-->
> 是經典的 78. Subsets 的進階版，現在數字會有重複(duplicate) 。
<!--more-->

---

## 思路
很樸質的雙層迴圈解法，還有點小 work-around ，因為不能包含  duplicate subsets ，所以用了``` Arrays.sort``` 和 ``` Set<String> set = new HashSet<>();```
# 解答
```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Set<String> set = new HashSet<>();
        List<List<Integer>> ans = new ArrayList<>();
        Arrays.sort(nums);

        List<Integer> initArray =  new ArrayList<>();
        ans.add(initArray);
        set.add(initArray.toString());

        for(int num : nums){
            int size = ans.size();
            for(int i = 0 ; i < size ; i++){
                List<Integer> list = ans.get(i);
                List<Integer> newList =  new ArrayList<>(list);
                newList.add(num);
                if(set.add(newList.toString())){
                    ans.add(newList);
                }
            }
        }

        return ans;
    }
}
```

---

# 時間空間複雜度
假設 nums 有 N 個元素:
### 時間複雜度: ```O(N*2^N)```

Backtrack 時間複雜度，會由 recursion tree 的 **Node 個數**和 **Node 行為**決定。
- Node 個數即為**所有 subset 個數**:```2^N```
- Node 行為主要影響為複製 List ，這邊就以可能複製的最長的 List，取花費時間的上限:```O(N)```

最後總體看一下，假設每個 Node 都花費最長```O(N)```時間，則時間複雜度:```O(N*2^N)```

### 空間複雜度：```O(N)```
recursion tree 會產生一個 recursion stack ，其最深度剛好就是 N 。再來 code 沒有創建額外的存儲空間，，故每個 Node 還是常數空間複雜度。

總結以上，故空間複雜度為(深度 * 常數空間): ```O(N*1)=O(N)```

---
### 參考資料

- [[LeetCode] 78. Subsets 子集合](https://www.cnblogs.com/grandyang/p/4309345.html)