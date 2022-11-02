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
> Dynamic Programming 大概算是 leetcode 裡面平均難度最高的章節了，還蠻需要練習的。但在講 DP 之前，我們可以先講 Search，因為 Dynamic Programming 其實就是 Search + Memoization。(未完成)

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
答案是在 leaf node 的就要用 Top Down 模式。相反如果答案是在 root ，就要用 Bottom up
- Define **State** of sub-problem
- Initialize initial state
- Call dfs(init_state)

dfs(state):
- Base case check
- For each sub-problem
    - 1. Update state = next_state
    - 2. Call dfs(next_state)
    - 3. Restore state

{{< alert warning >}}
如果沒有先檢查 Base case ，會死循環。
{{< /alert >}}
{{< alert warning >}}
每個子問題，要更新為下個狀態並遞迴傳給 dfs
{{< /alert >}}
{{< alert danger >}}
最重要的，**要把狀態變回原來進到這層的初始狀態**，因為同 level ，不能有相關性，只會跟 parent 有關係。 parent 呼叫 children ，要保證 parent 丟給每一children 的狀態要一樣。
{{< /alert >}}

---
# Exercise
[78. Subsets](https://leetcode.com/problems/subsets/)
