---
title: "63. Unique Paths II"

author: Aryido

date: 2022-11-13T15:22:47+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dp

comment: false

reward: false

---
<!--BODY-->
> 這題是 62. Unique Paths 的延伸，能選擇往下或往右走直至終點為止，要求出有多少種可能走法，但多了一個限制，會在路徑中加了一些 obstacle 擋住了某些路徑。是一道典型的 Dynamic Programming - 2D matrix 類型的題目。和爬樓梯等都屬於動態規劃中常見題目，因此也經常會被用於面試之中。
<!--more-->

---

# 思路

首先定義**狀態方程式**為一個 2D-matrix。由於只能右移動和下移動， 因此可以定義第```[i, j]```格子的值，一定是和左邊或上面格子的值有關，即可以由 ```[i - 1, j] + [i, j -1]``` 來表示。 故狀態方程式:

> ```state[i][j] = state[i-1][j] + state[i][j-1]```

再來看題目範例來說明 : 從起點到 ```[3,3]``` ，承前**狀態方程式**，知道會有子問題 ```[2, 3]``` 、 ```[3,2]```，這兩個位置，再往上一層會發現有重複子問題 ```[2,2]```，故可以把計算結果 memo 下來，實現優化。其 memo 2D dp matrix ，每個元素代表 :
> Number of unique paths from ```(0,0)``` to ```(i,j)```


在 Unique Paths II 因為有加上 obstacle ，故還要多判斷一下。遇到 obstacle 的格子要要回傳 0 ，因為 obstacle 是不能通過的點，不能產生實際路徑數。

{{< alert warning >}}
這題還有個特別陷阱，當起點直接放 obstacle 時，代表沒有路可以走，這個需要提前判斷。
```
if (obstacleGrid[0][0] == 1) {
    return 0;
}
```
{{< /alert >}}

---

# 解答
首先使用反向 dp 來解題，從最終答案往回看子問題，當得到最基本子問題答案之後一步步回填解答。

```java
class Solution {
    int xSize ;
    int ySize ;
    Integer[][] memo;

    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        xSize = obstacleGrid.length;
        ySize = obstacleGrid[0].length;
        memo = new Integer[xSize][ySize];

        if(obstacleGrid[0][0] == 1){
            return 0;
        }

        return dfs(xSize - 1, ySize - 1, obstacleGrid);
    }

    private int dfs(int x, int y, int[][] obstacleGrid){
        if(x == 0 && y == 0){
            return 1;
        }

        if(x < 0 || x >= xSize || y < 0 || y >= ySize || obstacleGrid[x][y] == 1){
            return 0;
        }

        if(memo[x][y] != null){
            return memo[x][y];
        }

        int res = 0;
        res += dfs( x - 1,  y, obstacleGrid);
        res += dfs( x,  y - 1, obstacleGrid);

        return  memo[x][y] = res;
    }
}
```

---

接下來版本是正向 dp，從最小子問題慢慢求解至最終問題。

# 解答
```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        int[][] memo = new int[m+1][n+1];

        if(obstacleGrid[0][0] == 1){
            return 0;
        }

        memo[1][1] = 1;
        for(int i = 1 ; i <= m ; i++){
            for(int j = 1 ; j <= n ; j++){
                if(obstacleGrid[i-1][j-1] == 1){
                    memo[i][j] = 0;
                }else{
                     memo[i][j] += memo[i-1][j] + memo[i][j-1];
                }
            }
        }

        return memo[m][n];
    }
}
```

{{< alert warning >}}
小技巧: memo 有用 padding 技巧，可處理 out of bound的部分。但要注意這樣導致 memo 和原本 matrix 會有錯位。
{{< /alert >}}
沒有錯位寫法如下，會需要分開 ```i = 0``` 和 ```j = 0``` 的情況:
```java
class Solution {
    int xSize ;
    int ySize ;
    int[][] memo;

    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        xSize = obstacleGrid.length;
        ySize = obstacleGrid[0].length;
        memo = new int[xSize][ySize];

        if(obstacleGrid[0][0] == 1){
            return 0;
        }

        for(int i = 0 ; i < xSize ; i++){
            for(int j = 0 ; j < ySize ; j++){
                if(i == 0 && j == 0){
                    memo[i][j] = 1;
                    continue;
                }
                if(obstacleGrid[i][j] == 1){
                    memo[i][j] = 0;
                    continue;
                }
                if( i == 0 ){
                    memo[i][j] = memo[i][j - 1];
                    continue;
                }
                if( j == 0 ){
                    memo[i][j] = memo[i - 1][j];
                    continue;
                }
                memo[i][j] = memo[i][j - 1] + memo[i - 1][j];
            }
        }
        return memo[xSize - 1][ySize - 1];
    }
}
```

---

# 時間空間複雜度
假設地圖 Grid 為 ```M*N``` 大小 :

### 時間複雜度: ```O(M*N)```
從正向 DP 中，發現演算法中有兩個 for-loop ，外層循環遍歷 row，內層循環遍歷 column 。 因此，總循環次數即 ```M*N```，故時間複雜度為: ```O(M*N)```。

從反向 DP 中，發現演算法中發現:
- Base case處理 : ```O(1)```
- 例外處理 : ```O(1)```
- memorization 紀錄: ```O(1)```
- 構造當前層答案 : ```O(1)```

故時間複雜度為: ```O(節點個數)```，那麼有多少個節點呢 ? 因為主要是要填滿 DP 矩陣，故要填 M*N 個，故時間複雜度還是為: ```O(M*N)```。


### 空間複雜度：```O(M*N)```
因為使用 DP 矩陣來儲存運算結果，而矩陣大小即為空間複雜度:  ```O(M*N)```。

---
# Vocabulary

{{< alert info >}}
**obstacle** [ˋɑbstək!]
n.障礙（物）；妨礙[C][（+to）]
{{< /alert >}}

---
### 參考資料

- [DynamicProgramming2D解题套路【LeetCode刷题套路教程15】](https://www.youtube.com/watch?v=ZyNWj0-34o0&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=15)

- [63.unique-paths-ii.md](https://github.com/azl397985856/leetcode/blob/master/problems/63.unique-paths-ii.md)

- [[LeetCode] 63. Unique Paths II 不同的路径之二](https://www.cnblogs.com/grandyang/p/4353680.html)

- [Day 2: 按照規律的方式填表可以解決大部分的問題！](https://ithelp.ithome.com.tw/articles/10215365)