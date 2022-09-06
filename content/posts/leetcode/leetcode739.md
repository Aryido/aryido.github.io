---
title: 739. Daily Temperatures by Java

author: Aryido

date: 2022-09-05T10:59:38+08:00

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

<!--more-->
## 思路
使用 Stack 來解題。 Stack 容器內放的資料為 temperatures 的 index ， 藉由比較當前溫度 temperatures[i] 和 stack top 溫度 temperatures[stack.peek()] 來建構答案。

- 注意，題意中的 warmer ，是`嚴格大於`，`在溫度相等不算 warmer` ，故要**注意重複溫度**!
  - ex: [40,40,40,40]，希望的解答是[0,0,0,0]，而不是[1,1,1,0]
- 可以順序或反序遍歷 temperatures
- **順序和反序遍歷，在code中判斷溫度時，要不要加等號很容易出錯XDD，要多想下...**

# 解答
```java
// 順序遍歷
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        if(temperatures.length == 1){
            return new int[]{0};
        }

        int[] ans = new int[temperatures.length];
        Deque<Integer> stack = new ArrayDeque<>();

        for(int i = 0 ; i < temperatures.length ; i++){
            if(stack.isEmpty() || temperatures[i] < temperatures[stack.peek()] ){
                stack.push(i);
            }else{
                while(!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]){ // strictly greater
                    int index = stack.pop();
                    ans[index] = i - index;
                }
                stack.push(i);
            }
        }

        return ans;
    }
}
```


```java
// 反序遍歷
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        if(temperatures.length == 1){
            return new int[]{0};
        }

        int[] ans = new int[temperatures.length];
        Deque<Integer> stack = new ArrayDeque<>();

        for(int i = temperatures.length - 1; i >= 0  ; i--){
            while(!stack.isEmpty() && temperatures[i] >= temperatures[stack.peek()]){ // need equality
               stack.pop();
            }
            if(stack.isEmpty()){
                ans[i] = 0;
            }else{
                ans[i] = stack.peek() - i;
            }
            stack.push(i);
        }

        return ans;
    }
}
```
---