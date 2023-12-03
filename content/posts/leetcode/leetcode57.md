---
title: "57. Insert Interval"

author: Aryido

first draft: 2022-09-01T11:17:02+08:00

date: 2023-12-02T17:28:05+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- array

comment: false

reward: false

---
<!--BODY-->
> 此題給定一個沒有 overlapping 且按照起始座標排序的 interval 集合，然後再給一個 newInterval 放到該集合內，要合併 interval 使它們都不會重疊，最後返回新的 **non-overlapping intervals**。
> 本題的生活化的模擬，可以把題目想成自修室租借，然後插入自己的使用時段(~~亂入別人的使用時間...~~)，最後返回整個自修室被租借的時段。

<!--more-->

---

# 思路
用一個變數 ```i``` 代表遍歷到第幾個interval。
- 如果當前第 ```i```的 interval **結束位置**小於 newInterval  的**起始位置**的話，說明沒有重疊，則將第```i```的 interval 加入答案中，然後 ```i``` 自增 1，直到有 interval 越界或有重疊while 迴圈退出。
- 再來用一個 while 迴圈處理所有重疊的區間，每次用兩個 interval
  - **起始位置的較小值**，
  - **結束位置的較大值**

  來更新 newInterval ，然後 ```i``` 自增 1。 直到 ```i``` 越界或沒有重疊時 while 迴圈退出。之後將更新好的 newInterval 加入答案中。

- 最後後將 ```i``` 之後剩下的 interval 加入答案中，便完成這題。

{{< alert info >}}
**因為要回傳答案的 intervals 的數量不確定，故先使用了ArrayList**

但最後答案回傳型別為`int[][]`，故必須把 List of array 轉成 2D-array。
```java
newIntervals.toArray(new int[newIntervals.size()][])
```
{{< /alert >}}

---

# 解答
```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        if(intervals.length == 0){
            return new int[][]{{newInterval[0],newInterval[1]}};
        }

        List<int[]> ans = new ArrayList<>();

        int i = 0;
        while(i < intervals.length && intervals[i][1] < newInterval[0]){
            ans.add(intervals[i++]);
        }
        // deal with overlapping
        while(i < intervals.length && intervals[i][0] <= newInterval[1]){
            newInterval[0] = Math.min(intervals[i][0], newInterval[0]);
            newInterval[1] = Math.max(intervals[i][1], newInterval[1]);
            i++;
        }
        ans.add(newInterval);

        while(i < intervals.length && newInterval[1] < intervals[i][0]){
            ans.add(intervals[i++]);
        }

        return ans.toArray(new int[ans.size()][]);
    }
}

```
{{< alert warning >}}
**while分成三部分，注意比較範圍**
- ```intervals[i][1] < newInterval[0]```
- ```intervals[i][0] <= newInterval[1]```
- ```newInterval[1] < intervals[i][0]```
{{< /alert >}}

{{< alert info >}}
找出有重疊的區間，判斷法有兩種:
- ```intervals[i]```的開始時間小於  newInterval 的結束時間
- ```intervals[i]```的結束時間大於  newInterval 的開始時間
{{< /alert >}}

---

# 時間空間複雜度

假設 intervals 內有 N 個 interval:

### 時間複雜度: ```O(N)```
基本上就是遍歷一次 intervals ，就可以得出解答，故為 ```O(N)```。


### 空間複雜度：```O(1)```
首先因為解答長度不一定，故有造出一個 ans 這個 ArrayList 來存放解答。
但基本上認為這是屬於解答集部分，故不考慮至空間複雜度內。

在把有 overlapping 部分的 interval 合併時，只是更動原本的 newInterval ，故沒有用到額外空間，所以得到 ```O(1)```。


---

### 參考資料

- [[LeetCode]57. Insert Interval 中文](https://www.youtube.com/watch?v=E9IYRG_WYcM&t=55s)

- [[LeetCode] 57. Insert Interval 插入区间](https://www.cnblogs.com/grandyang/p/4367569.html)