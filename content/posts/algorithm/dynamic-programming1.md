---
title: Dynamic Programming 基礎介紹

author: Aryido

date: 2022-11-02T21:52:00+08:00

thumbnailImage: /images/algorithm/dp-logo.jpg

categories:
- algorithm

tags:
- dp

comment: false

reward: false
---
<!--BODY-->
> Dynamic Programming 大概算是 leetcode 裡面平均難度最高的章節了，還蠻需要練習的。但在講 DP 之前，我們可以先講 Search，因為 Dynamic Programming 其實就是 Search + Memoization。

<!--more-->
---
# Search
原理是透過把原問題分解為相對簡單的子問題的方式，來求解複雜問題的方法，這個把大問題分解成小問題的過程，我們就稱為 Search 。

Search Space 基本可以用 Tree 的形式展現出來，時間複雜度取決於 Tree 的**深度**和**每個 node 的 children 個數**

{{< alert info >}}
Search 最重要的就是定義好**狀態**，保證每個子問題都能用一個狀態來描述。
{{< /alert >}}

---

# Top Down 模板
答案是在 leaf node 的就要用 Top Down 模式。
- Define **State** of sub-problem
- Initialize initial state
- Call dfs(init_state)

dfs(state):
1. Base case check
2. For each sub-problem
    - a. Update state = next_state
    - b. Call dfs(next_state)
    - c. Restore state

{{< alert warning >}}
如果沒有先檢查 Base case ，會死循環。
{{< /alert >}}
{{< alert warning >}}
每個子問題，要更新為下個狀態，並傳給 dfs 遞迴
{{< /alert >}}
{{< alert danger >}}
最重要的，**要把狀態變回原來進到這層的初始狀態**，因為同 level ，不會有相關性，同 level 只會跟 parent 有關係。 parent 呼叫 children ，要保證 parent 丟給每一children 的狀態要一樣。
{{< /alert >}}

---
# Exercise
[78. Subsets](https://leetcode.com/problems/subsets/)

---

# Dynamic Programming
如果 Search Space 把子問題的結果記錄下來，使之不用重複計算多次的話，這就稱為 Dynamic Programming ，其時間複雜度取決於子問題的個數。

{{< alert success >}}
Dynamic Programming = Search + Memoization
{{< /alert >}}

**所有 DP 問題都可以寫成 Bottom up DFS 的形式**，所以 DP 問題最重要的，其實還是定義好**狀態**，小技巧是從一個中間狀態去思考遞迴的規則，來定義狀態。

---
# Bottom Up 模板
答案是在 root ，就用 Bottom up。

- Define **State** of sub-problem
- Initialize 一個紀錄子問題答案的資料結構，例如 matrix 或 set 等等
- Return dfs(top_level_answer_state)

dfs(state):
1. Base case check
2. **如果當前答案有計算過了，就直接 return 答案**
3. For each sub-problem
    - a. 向子問題要答案 >> Call dfs(sub-problem_state)
    - b. 用回傳的 dfs(sub-problem_state) 答案，來建立當前答案**即 Transition Rule : 遞迴原則**
4. **紀錄當前答案**

{{< alert warning >}}
Dynamic Programming 對比 Search，就是多第二步和第四步，這些事和 Memoization 相關的部分。
{{< /alert >}}

---
# Exercise
[139. Word Break](https://leetcode.com/problems/word-break/)

---