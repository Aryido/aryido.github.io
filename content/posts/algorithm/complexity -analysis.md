---
title: "Complexity Analysis"

author: Aryido

date: 2023-08-30T00:06:02+08:00

thumbnailImage: "/images/java/java-bean-logo5.jpg"

categories:
- algorithm

comment: false

reward: false
---
<!--BODY-->
> 在刷 leetcode 時，都會需要分析和解釋一下自己寫的程式的演算法複雜度，基本上主要是解釋 :
> - 時間複雜度
> - 空間複雜度
>
> 從以上兩個面相去分析 code 效率、品質、trade off，本文記錄一些分析要點。另外建議面試的時候，當對題目有想法時，先不用急著直接實作，而是先估出複雜度，並和面試人討論，確認不會 TLE ；空間是否需要優化；符不符合題目要求等等，最後再開始寫。

<!--more-->
---

**Complexity** 是用來形容某個函數和它的參數間的關係，最常用的表示方法是 **Big-O notation**，代表演算法函數的 Upper bound 。 其值有一種簡單的估算方法 :
- 函數除了最高次項以外都去掉
- 然後再把最高次項的係數去掉
- Log 底數忽略不計

得到的結果，就是演算法的複雜度答案了。

{{< alert info >}}
大寫的英文字母 ```O``` 函數，代表演算法執行步驟數目上限
{{< /alert >}}


---

# 時間複雜度(Time Complexity)
想要描述一個演算法執行速度有多快，直覺的方式是測量時間。但是由於執行時間會受「機器規格」、「coding 語言」與「實作方式」影響，目前比較公平的分析是**統計演算法步驟數**。

這裡比較要提醒的是 **Recursion tree** 的時間複雜度，主要注意**節點的單一子問題代價**，是指每次函數執行中，除去遞歸調用以外的代價。明確了單一代價，整個遞歸樹的總代價就可以計算了，就是**樹中所有節點的代價和**。

在 LeetCode、HackerRank 等等平台上解決問題時，有時候會先嘗試用暴力破解法，但即使邏輯正確，仍可能因 TLE（時間超過）錯誤而失敗。這裡有個經驗法則：
{{< image classes="fancybox fig-100" src="/images/algorithm/complexity/estimate-complexity.jpg" >}}

看圖大概可以知道，若解法超過 `10^8` 運算次數，基本上都會超時失敗 ; 若是 `10^5`，那解法必須在大約 `O(n)` 或 `O(n log n)` 的時間內執行，超過 `O(n^2)` 甚至沒有優化 `O(n^2)`大概也會失敗。

{{< alert info >}}
演算法競賽和刷題社群常用的粗略估計 Python 每秒約 `10^7` 次簡單運算（C++ 通常估 `10^8～10^9`）
{{< /alert >}}

---

# 空間複雜度(Space Complexity)

計算演算法執行時所花費的記憶體空間成本，也就是**stack、heap、memory**。空間複雜度可分為幾個部份來分析：
- **Fixed part**：對於一些程式常數、基本變數所需要的儲存空間。
- ~~**Input Output part**：存儲 input data 所需的空間和最後答案的 data，通常這些部分**不包括**在空間複雜度的計算中，因為這是解決問題所必需的。~~
- **Variable part**：演算法執行過程中額外使用的空間。
- **Recursion tree stack**：如果使用recursion，則還需要考慮遞迴調用所使用的棧空間。

在 leetcode 中演算法討論中，空間複雜度時，專注在 **Variable part** 和 **recursion tree stack** 就可以了

---

## Example

假設 n 為足夠大時 : ```
1 < logn < n < nlogn < n^2 < 2^n < n!```
{{< image classes="fancybox fig-100" src="/images/algorithm/complexity/big-o-1.jpg" >}}
如上圖所示，可以知道 Big-O 複雜度越高，隨著數據規模增長，效率就會越來越差，因此 n 越大時，寫好演算法就越重要。

- **Leetcode 509 Fibonacci**

    ```java
    public static int fibonacci(int n) {
        if (n <= 1) {
            return n;
        }
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
    ```
  可以參考自己的 leetcode 509. 有詳細分析

    - 時間複雜度: ```O(((1+(根號5))/2)^n)```

    - 雖然樹中至多有 ```2^n``` 個節點，但任何給定時間頂多只會有到最大深度 n 個節點，故空間複雜度是 ```O(n)```

- **LeetCode 206 Reverse Linked List**

  - **Fixed part**：遞迴解法中，我們用到了一個局部變數 p 來存儲遞迴調用 reverseList(head.next) 的結果。這是常數空間  ```O(1)```

  - **input output part**：輸入是一個鏈表，通常這部分不包括在空間複雜度的計算中。

  - **Variable part**：演算法沒有使用任何額外的結構（如陣列或字典等）來儲存，所以輔助空間是 ```O(1)```

  - **recursion stack**：演算法的遞迴深度和鏈表的長度 n 成正比。深度會是```O(n)```

{{< alert success >}}
一般來說​「時間複雜度」與「空間複雜度」之間是可以相互 **trade off** 的！例如說 :
- Bubble Sort 就不需要額外的記憶體空間，但是做起來比較久
- Bucket Sort 就需要額外的記憶體空間，但是做起來比較快。
{{< /alert >}}

---

## Big O Notation 表格

{{< image classes="fancybox fig-100" src="/images/algorithm/complexity/big-o-2.jpg" >}}

| Big O Notation | 別名             | 常見演算法           |
| -------------- | ---------------- | -------------------- |
| ```O(1)```     | 常數             | Array 讀取             |
| ```O(n)```     | 線性             | Linear search        |
| ```O(log n)``` | Logarithmic| 二分搜尋法           |
| ```O(n log n)```| 線性對數        | 堆積排序，合併排序   |
| ```O(n^2)```   | 平方             | 氣泡排序，插入排序   |
| ```O(n^3)```   | 三次方           | 三重迴圈 |
| ```O(2^n)```   | 指數             | 費氏數列             |
| ```O(n!)```    | 階層             | 排列組合，八皇后問題 |

---

### 參考資料

- [[資料結構] CH2. Performance Analysis](https://hackmd.io/@Zero871015/SyAI_mfsX?type=view)

- [複雜度分析](https://hackmd.io/@wiwiho/cp-note/%2F%40wiwiho%2FCPN-complexity)

- [Hank Guo's Blog](https://hankguo93.com/2021/11/05/%E7%A9%BA%E9%96%93%E8%A4%87%E9%9B%9C%E5%BA%A6-space-complexity/)

- [Algorithm Analysis](https://web.ntnu.edu.tw/~algo/AlgorithmAnalysis.html)

- [How to Avoid TLE (Time Limit Exceeded) Errors in Coding Problems](https://medium.com/@bpratikshya30/how-to-not-get-a-tle-8159bb06c3bd)
