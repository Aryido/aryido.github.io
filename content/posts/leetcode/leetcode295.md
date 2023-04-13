---
title: "295. Find Median from Data Stream"

author: Aryido

date: 2023-04-08T19:56:55+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- heap

comment: false

reward: false
---
<!--BODY-->
> 這題為一個設計題，給了一個 Data Stream，希望設計一個 class 能夠支援連續的 operation，並找出該 Stream 目前的中位數。注意 Data Stream 中的 Data 是無序的且可以為負數，所以我們要做的第一件事是讓**每一次 data 輸入進來，都要讓其有序**。這裡介紹的解法十分巧妙，**使用 maxHeap 和 minHeap 來解決問題**，這樣中位數的計算便只要看 maxHeap 的最大值和 minHeap 的最小值來判斷。
<!--more-->

---

暴力解就是每次進來的時候都使用 sorting ，但一次的 sorting 結果就是 ```O(NlogN)```...

## 思路
使用 maxHeap 和 minHeap ，其中滿足 :
- maxHeap 的最大值小於等於 minHeap 的最小值
- minHeap 的 size 最多比 maxHeap 的 size 多 1

根據以上原則，這樣整個 Data Stream 就被分為兩部分。接下來是思考 data 要怎麼放法。首先 :

- 第一個 data 一律先放到 minHeap
- 接下來所有 data 要跟 minHeap 的最小值比 :
  - 小於 minHeap 的最小值: data 放到 maxHeap
  - 大於 minHeap 的最小值: data 放到 minHeap
- 接下來看 Heap size 是否符合上述定義，可能的情況有 :
  - 若 maxHeap 的 size 比 minHeap 的 size 多，則把 maxHeap 的最大值放到 minHeap 內
  - 若 minHeap 的 size 比 maxHeap 的 size **多超過 1** ，則把 minHeap 最小值移動到 maxHeap 內

- 最後 findMedian 方法，只要判斷兩個 Heap 的 size()
  - 若兩個 Heap 的 size() 一樣，則**中位數取最大值和最小值的平均**
  - 若 minHeap 的 size 比 maxHeap 的 size 多，則**中位數就是 minHeap 的最小值**。

{{< alert info >}}
時間複雜度 :
每次 heap 在加入新的元素時，都會重新排列，時間為 ```O(logn)```，取出中位數的時間為 ```O(1)```，故總時間複雜度為 ```O(nlogn)```。

空間複雜度 :
需要額外 ```O(n)``` 空間儲存。
{{< /alert >}}

# 解答

```java
class MedianFinder {
    private Queue<Integer> maxHeap = new PriorityQueue<>((a,b) -> b-a);
    private Queue<Integer> minHeap = new PriorityQueue<>();

    public MedianFinder() {

    }

    public void addNum(int num) {
        if(minHeap.isEmpty() || minHeap.peek() <= num){
            minHeap.offer(num);
        } else {
            maxHeap.offer(num);
        }

        if(maxHeap.size() > minHeap.size() + 1 ){
            minHeap.offer(maxHeap.poll());
        }

        if(maxHeap.size() + 1 < minHeap.size()){
            maxHeap.offer(minHeap.poll());
        }
    }

    public double findMedian() {
        if(maxHeap.size() == minHeap.size()){
           return (maxHeap.peek() + minHeap.peek())/2d;
        } else {
           return minHeap.peek() / 1d;
        }
    }
}
```


蠻大一個重點是第一次你要把 data 放到 maxHeap 還是 minHeap ，上面解法是放到 minHeap，以下解法是把第一個 data 放到 maxHeap 的code，這裡可以特別看一下 findMedian ，要判斷的點變得比較多，因為會要處理兩種的情況 :
- 只放一個 data
- stream 到一半

```java
class MedianFinder {
    private Queue<Integer> maxHeap = new PriorityQueue<>((a,b) -> b-a);
    private Queue<Integer> minHeap = new PriorityQueue<>();

    public MedianFinder() {

    }

    public void addNum(int num) {
        if(maxHeap.isEmpty() || maxHeap.peek() > num){
            maxHeap.offer(num);
        } else {
            minHeap.offer(num);
        }

        if(maxHeap.size() == minHeap.size() + 2){
            minHeap.offer(maxHeap.poll());
        }

        if(maxHeap.size() + 2 == minHeap.size()){
            maxHeap.offer(minHeap.poll());
        }
    }

    public double findMedian() {
        if(maxHeap.size() == minHeap.size()){
           return (maxHeap.peek() + minHeap.peek())/2d;
        } else {
           return maxHeap.size() > minHeap.size() ? maxHeap.peek() / 1d : minHeap.peek() / 1d;
        }
    }
}
```

---

# Follow up:
- If all integer numbers from the stream are in the range [0, 100], how would you optimize your solution?

所有數字都在 [0, 100] 內的情況，可以開一個大小爲 101 的 array，對於每個出現的數字，就在相應位置的值上加 1 。最後，從 array 中計算中位數即可。

---
