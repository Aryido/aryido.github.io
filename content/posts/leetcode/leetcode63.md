---
title: 63. Unique Paths II

author: Aryido

date: 2022-11-13T15:22:47+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
>這題是 62. Unique Paths 的延伸，在路徑中加了一些 obstacle ，用 Dynamic Programming 二維的 dp 數組來解題
<!--more-->

---

## 思路
可以先構造一棵 recursion tree 幫助思考，如下圖 :
```
                       [3,3]
                   /          \
                  /            \
                 /              \
              [2,3]             [3,2]
            /       \           /    \
           /         \         /      \
        [1,3]       [2,2]    [2,2]    [3,1]

```
因為每個位置只能由其 top 和 left 的位置移動而來，所以 *[3,3]* 的子問題有 *[2,3]*, *[3,2]*；也由圖知道有 *[2,2]* 這樣的重複子問題，可以把它紀錄下來，實現優化。另外有 obstacle 位置，遇到就不用往下看子問題了。

所以可以得到 :
```
State(i,j) 就是 Number of unique paths from (0,0) to (i,j)
```
# 解答
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
沒有錯位寫法如下，會需要分開 *i=0* 和 *j=0* 的情況:
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
# Vocabulary

{{< alert info >}}
**obstacle** [ˋɑbstək!]
n.障礙（物）；妨礙[C][（+to）]
{{< /alert >}}

---