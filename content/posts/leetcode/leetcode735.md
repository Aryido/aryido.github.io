---
title: 735. Asteroid Collision

author: Aryido

date: 2022-09-05T20:42:24+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- java

tags:
- java
- LeetCode
- stack

comment: false

reward: false
---
<!--BODY-->
不定時練習LeetCode紀錄...
這題蠻好玩的，小行星碰撞 Asteroid Collision。

<!--more-->
## 思路
這題雖然好玩但我寫起來真的BUG滿天飛呢...
一開始最好是從 asteroid > 0 開始區分。
接下來開始 While 小行星碰撞，看看 stack 裡的行星會死到哪裡(XD)，這邊特別注意 `stack.peek() < -asteroid` ，並沒有等號喔。

故跳出 while 的狀態，會是當前 asteroid 遇到跟它一樣大的，或者是 stack 空了或撞不到東西。
- 若為遇到跟它一樣大的，則 stack.pop()，並繼續最外層 for loop。
- 若 stack 空了或者撞不到東西，則直接 push 進去。

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