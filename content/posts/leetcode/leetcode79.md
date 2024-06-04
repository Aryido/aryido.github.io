---
title: "79. Word Search"

author: Aryido

date: 2023-03-20T22:45:06+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dfs

comment: false

reward: false
---
<!--BODY-->
> 題目給定一個 board 以及 一個 word ，我們要判斷的 board 上是否可以連線出 word。這題是蠻典型的 graph 類題目，用  BFS 或 DFS 解題都行，但用深度優先 DFS 來解題會比較好一些(可以先思考一下為什麼)。解題流程還蠻制式化的，是熟練 graph 類型的練習好題目 XD。
<!--more-->

---

## 思路

**為什麼使用 DFS 來解題會比較好一些呢 ?**

以時間複雜度來說，BFS 或 DFS 相差不大，但空間複雜度 :
- BFS 會需要把相鄰**所有可能**的解部分，加到 queue 內
- DFS 是用遞迴，故是在 memory 中用 stack 來儲存 ; 若途中發現不對，就會返回上一層並 pop 掉。

故整體來說 DFS 來解題會比較好一些。並且題目有說， *The same letter cell may **not** be used more than once* ，故需要有防止遍歷重複走過位置。這邊可以想到要建立一個和 board 一樣大的 visited 2D-array，但其實也可以直接對 board 進行修改 *(以下解題方式是將遍歷過的位置改為 0 )*，但**記得遞歸調用完後，需要恢復之前的狀態**。

對其上下左右分別調用 DFS ，只要有一個返回 true，就表示找到了對應的字符串 *(故以下解題方式是在 for-loop 內直接 return true)*。

# 解答
```java
class Solution {
    int[][] directions = new int[][] {{1,0}, {-1,0}, {0,1}, {0,-1}};

    public boolean exist(char[][] board, String word) {
        for(int i = 0 ; i < board.length ; i++){
            for(int j = 0 ; j < board[0].length ; j++){
                if(dfs(board, i, j, 0, word)){
                    return true;
                }
            }
        }
        return false;
    }

    public boolean dfs(char[][] board, int i, int j, int index, String word){
        if(index == word.length()){
            return true;
        }
        if( i < 0 || i >= board.length ||
            j < 0 || j >= board[0].length ||
            word.charAt(index) != board[i][j]
        ){
            return false;
        }
        char temp = board[i][j] ;
        for(int[] direction : directions){
            board[i][j] = '0';
            int x = i + direction[0];
            int y = j + direction[1];
            if(dfs(board, x, y, index + 1, word)){
                return true;
            }
            board[i][j] = temp;
        }
        return false;
    }
}
```

{{< alert success >}}
對上下左右四個方向進行操作，故可以先定義 :
```
int[][] directions = new int[][] {{1,0}, {-1,0}, {0,1}, {0,-1}};
```
另外特別注意，**是用 { , } 來表示 array**。
{{< /alert >}}

# Vocabulary

{{< alert info >}}
**adjacent** [əˋdʒesənt] : (d不發音，不要念錯了)

adj. 相鄰的，前後相接的

{{< /alert >}}

{{< alert info >}}
**adjoin** [əˋdʒɔɪn] : (d不發音，不要念錯了)

vt. 貼近，相連

example:
- New Jersey adjoins New York. 新澤西與紐約相連。

{{< /alert >}}

---