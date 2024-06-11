---
title: Leetcode130

author: Aryido

date: 2022-10-23T14:27:33+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->

<!--more-->

---

## 思路

# 解答
```java
class Solution {

    int[][] directions = {{1,0}, {-1,0}, {0,1}, {0,-1}};

    public void solve(char[][] board) {

        // left
        for(int i = 0 ; i < board.length ; i++){
            if(board[i][0] == 'O' ){
                board[i][0] = 'A';
                dfs(i, 0, board);
            }
        }

        // right
        for(int i = 0 ; i < board.length ; i++){
            if(board[i][board[0].length - 1] == 'O' ){
                board[i][board[0].length -1] = 'A';
                dfs(i, board.length - 1, board);
            }
        }

        // top
        for(int i = 1 ; i < board[0].length - 1 ; i++){
            if(board[0][i] == 'O' ){
                board[0][i] = 'A';
                dfs(0, i, board);
            }
        }

        // down
        for(int i = 1 ; i < board[0].length - 1 ; i++){
            if(board[board.length - 1][i] == 'O' ){
                board[board.length - 1][i] = 'A';
                dfs(board.length - 1, i, board);
            }
        }

        for(int i = 0 ; i < board.length ; i++){
            for(int j = 0 ; j < board[0].length ; j++){
                if(board[i][j] == 'O'){
                    board[i][j] = 'X';
                }

                if(board[i][j] == 'A'){
                    board[i][j] = 'O';
                }

            }
        }

    }

    public void dfs(int x, int y, char[][] board){
        for(int[] direction : directions){
            int newX = x + direction[0];
            int newY = y + direction[1];
            if(0 <= newX && newX < board.length && 0 <= newY && newY < board[0].length && board[newX][newY] == 'O'){
                board[newX][newY] = 'A';
                dfs(newX, newY, board);
            }
        }
    }
}
```

# Vocabulary

[字典連結](https://tw.dictionary.search.yahoo.com/search;_ylt=AwrtXGs1MCVj1V8AZAh9rolQ;_ylc=X1MDMTM1MTIwMDM4MQRfcgMyBGZyAwRmcjIDc2ItdG9wBGdwcmlkA3VHbnhCdFdPUnBlU3k0a1ZuS1A0VUEEbl9yc2x0AzAEbl9zdWdnAzQEb3JpZ2luA3R3LmRpY3Rpb25hcnkuc2VhcmNoLnlhaG9vLmNvbQRwb3MDMARwcXN0cgMEcHFzdHJsAzAEcXN0cmwDMTAEcXVlcnkDZGVwYXJ0dXJlJTIwBHRfc3RtcAMxNjYzMzgxODE3?p=departure+&fr2=sb-top)

{{< alert info >}}
**xxxx** [KK] : (注意事項)

{{< /alert >}}
---