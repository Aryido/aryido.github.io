---
title: 200. Number of Islands

author: Aryido

date: 2022-09-07T20:55:05+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- BFS
- DFS
- graph

comment: false

reward: false
---
<!--BODY-->
剛開始刷題時就覺得這題很有趣，有 game 的感覺。可以用來複習DFS、BFS。

<!--more-->
## DFS思路
對我來說 DFS 解比較直觀，就是把所有 `1` 連起來看作是一個獨立 graph。

這題還有些小訣竅:
- 可以定義4個方向的 directions 幫助解題
- 一般來說 DFS 題目會需要定義一個 Set 或 Metrix 來記錄已經訪問過的部分。但這題有訣竅是可以把遍歷過的位置直接換成0，大大簡化空間複雜度。
# DFS解答
```java
// use dfs to trivel matrix T:O(m*n) C:O(1)
class Solution {
    int[][] directions = {{1,0}, {-1,0}, {0,1}, {0,-1}};

    public int numIslands(char[][] grid) {
        int ans = 0;
        for(int i = 0 ; i < grid.length ; i++){
            for(int j = 0 ; j < grid[0].length ; j++){
                if(grid[i][j] == '1'){
                    ans++;
                    dfs(grid, i, j);
                }
            }
        }
        return ans;
    }

    private void dfs(char[][] grid, int i, int j){
        grid[i][j] = '0';
        for(int[] direction : directions){
            int x = i + direction[0];
            int y = j + direction[1];
            if(x >= 0 && x < grid.length && y >= 0 && y < grid[0].length && grid[x][y] == '1'){
                dfs( grid, x, y);
            }
        }
    }
}

```

## BFS思路
BFS解法是按 **層** 的概念進行的演算法，故須利用到 Queue ，紀錄需要被展開的位置(先紀錄的先展開)。

` Queue<int[]> queue = new LinkedList<>();`

但使用BFS，會發現有些位置，會被重複紀錄進 Queue 裡。雖然判斷式在遇到重複位置時，會因為`grid[i][j] = '0'`而不會繼續展開，但會花時間在 if 判斷，速度一定慢很多。

# BFS解答
```java
class Solution {
    int[][] directions = {{1,0},{-1,0},{0,1},{0,-1}};

    public int numIslands(char[][] grid) {
        int ans = 0;
        for(int i = 0 ; i < grid.length ; i++){
            for(int j = 0 ; j < grid[0].length; j++){
                if(grid[i][j] == '1'){
                    Queue<int[]> queue = new LinkedList<>();
                    queue.offer(new int[]{i,j});

                    while(!queue.isEmpty()){
                        int size = queue.size();
                        for(int k = 0 ; k < size ; k++){
                            int[] position = queue.poll();

                            // 若不判斷會超時
                            if(grid[position[0]][position[1]] != '0'){
                                grid[position[0]][position[1]] = '0';
                            }else{
                                continue;
                            }

                            for(int[] direction : directions){
                                int x = position[0] + direction[0];
                                int y = position[1] + direction[1];
                                if(x >= 0 && x < grid.length && y >= 0 && y < grid[0].length && grid[x][y] == '1'){
                                    queue.offer(new int[]{x,y});
                                }
                            }
                        }
                    }
                    ans++;
                }
            }
        }
        return ans;
    }

}
```

---