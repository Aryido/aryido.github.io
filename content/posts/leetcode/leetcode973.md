---
title: 973. K Closest Points to Origin

author: Aryido

date: 2022-09-12T20:12:28+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- java

tags:
- java
- LeetCode
- heap

comment: false

reward: false
---
<!--BODY-->
> 類似這種 top k 問題且非樹結構，都可以直接用 Heap 來解題。

<!--more-->
## 思路
這裡使用 Min Heap 來做的。 重點在於 comparator 的寫法。
定義好 Min Heap 之後，就 for loop 把資料 offer 進 Heap。最後 poll k 次。

{{< alert warning >}}
- 注意解答2D-array 的初始化:  *int[][] ans = new int[k][2]*
- comparator:
  - (a,b) -> a-b : 代表由小到大
  - (a,b) -> b-a : 代表由大到小
{{< /alert >}}

# 解答
```java
class Solution {
    public int[][] kClosest(int[][] points, int k) {
        Queue<int[]> minHeap = new PriorityQueue<>(
            (array1, array2) ->
                (array1[0]*array1[0]+array1[1]*array1[1]) - (array2[0]*array2[0]+array2[1]*array2[1])
        );

        for(int[] point : points){
            minHeap.offer(point);
        }

        int[][] ans = new int[k][2];
        for(int i = 0 ; i < k ; i++){
            ans[i] = minHeap.poll();
        }
        return ans;
    }
}
```
---