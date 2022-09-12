---
title: 57. Insert Interval

author: Aryido

date: 2022-09-01T11:17:02+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- java

tags:
- java
- LeetCode

comment: false
reward: false
---
不定時練習LeetCode紀錄...

<!--more-->

## 思路
- **因為要回傳答案的 intervals 的數量不確定，故使用了ArrarList**
    ```java
    List<int[]> newIntervals = new ArrayList<>();
    ```
    但最後答案回傳型別為`int[][]`，故必須把 List of array 轉成 2D-array。

- **while分成三部分，注意比較範圍**
  - intervals[i][1] < newInterval[0]
  - intervals[i][0] <= newInterval[1]
  - newInterval[1] < intervals[i][0]

# 解答
```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        if(intervals.length == 0){
            return new int[][]{{newInterval[0],newInterval[1]}};
        }

        List<int[]> newIntervals = new ArrayList<>(); // unknow ans's length

        int i = 0;
        // interval's position is on left hand side of newInterval
        while(i < intervals.length && intervals[i][1] < newInterval[0]){
            newIntervals.add(intervals[i++]);
        }
        // deal with overlapping
        while(i < intervals.length && intervals[i][0] <= newInterval[1]){
            newInterval[0] = Math.min(intervals[i][0], newInterval[0]);
            newInterval[1] = Math.max(intervals[i][1], newInterval[1]);
            i++;
        }
        newIntervals.add(newInterval);
        // interval's position is on right hand side of newInterval
        while(i < intervals.length && newInterval[1] < intervals[i][0]){
            newIntervals.add(intervals[i++]);
        }

        return newIntervals.toArray(new int[newIntervals.size()][]);
    }
}

```
---