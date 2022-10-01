---
title: 735. Asteroid Collision

author: Aryido

date: 2022-09-05T20:42:24+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- stack

comment: false

reward: false
---
<!--BODY-->
不定時練習LeetCode紀錄...

這題雖然好玩但我寫起來真的BUG滿天飛，小行星碰撞 Asteroid Collision。

<!--more-->
## 思路
一開始從 asteroid > 0 開始區分，即往右飛的一律都加入 stack 中。

接下來考慮往左飛的 asteroid，使用 while loop 讓小行星碰撞，看看 stack 裡的行星會死到哪裡。

這邊特別注意 `stack.peek() < -asteroid` ，並沒有等號。因為當兩一樣大的 asteroid 碰撞時，會需要移除 stack 裡的 asteroid，並且當前 asteroid 會消亡，故要新用一個 if 來判斷。

呈上，跳出 while 的狀態，會是當前 asteroid 遇到跟它一樣大的，或者是 stack 空了或撞不到東西。
- 若為遇到跟它一樣大的，則 stack.pop()，並繼續最外層 for loop。
- **若 stack 空了或者撞不到東西，則直接 push 進去。**

# 解答
```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {

        Deque<Integer> stack = new LinkedList<>();

        for(int asteroid : asteroids){
            if(asteroid > 0){
                stack.push(asteroid);
            }else{
                while(!stack.isEmpty() && stack.peek() > 0 && stack.peek() < -asteroid){
                    stack.pop();
                }

                if(!stack.isEmpty() && stack.peek() == -asteroid){
                    stack.pop();
                    continue;
                }

                if(stack.isEmpty() || stack.peek() < 0){
                    stack.push(asteroid);
                }
            }
        }

        int[] ans = new int[stack.size()];
        for(int i = stack.size() - 1 ; i >=0 ; i-- ){
            ans[i] = stack.pop();
        }
        return ans;
    }
}
```
---