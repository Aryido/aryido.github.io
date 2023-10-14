---
title: Backtrack 基礎介紹

author: Aryido

date: 2023-10-04T20:10:05+08:00

thumbnailImage: /images/algorithm/dp-logo.jpg

categories:
- algorithm

tags:
- dp
- backtrack

comment: false

reward: false

---
<!--BODY-->
> Backtrack 是 DFS 的一種形式，基本寫法類似於 TOP DOWN DFS，處理方式就是所謂的窮舉法，將所有可能的結果都找出來；每一個結果都實際看看這樣。換個角度來說，其實這個過程就如同在樹上遍歷 (Tree Traversal) ，而普通的 DFS 是不需要回溯狀態的。 Backtrack 強調了狀態回溯。

<!--more-->
---

# Backtrack
在思考 Backtrack 類型的題目時，都可以嘗試著把 recursion tree 畫出來。每次搜索其中一個分支時，都會要記錄當前  recursion tree node 的狀態，當嘗試完某一分支之後，要把狀態回溯到嘗試之前的的狀態，之後再去嘗試另一個分支。

{{< alert info >}}
Backtrack 時間複雜度，會由 recursion tree 的 Node 個數和 Node 行為決定。
{{< /alert >}}

為甚麼要回溯呢 ?

> 如果不回溯，可能會使得其他 branch 的狀態被互相影響。例如 A 分支的狀態可能會被帶到 B 分支，這會影響結果，是不正確的。正常來說分支必須是相互獨立的。


---

# 操作框架

回溯法適合以遞迴 recursion 來解，其核心概念就是將選擇清單裡的選項都逐一執行（窮舉），以下框架透過 for loop 來將選擇清單中的每個可能性都逐一執行，**並一定要再在遞迴之後撤銷選擇**。模板如下:

- Base Case
- For each possibility p
   - Memorize current state
   - backtrack(next_state)
   - Restore current state

---
# 範例詳解
如下圖範例，該如何用得到所有可能的數字排列```[10,20,30]```，這就是一個簡單的 backtracking 範例，步驟會詳細解釋 :
{{< image classes="fancybox fig-100" src="/images/algorithm/backtrack-example.jpg" >}}

- 用 for loop 來遍歷 array ，而決定是否要選擇遍歷到的數字的條件，看是數字有沒有已經選擇過了來決定。如何知道已經選擇過了還是還沒選呢 ? **用一個 boolean array 去紀錄有無選過**，例如 ```visited[i]``` 代表我們有無選過```nums[i]```。

- 用 inner list 來裝，**到現在為止所做的所有選擇**，如果 inner list 的長度是滿的，就代表是一種答案，故把它放進 ans 列表內。

- acktracking 的流程就是 : **不停地做選擇，再撤回選擇，並回到選擇前的狀態，之後再換其他選擇**。這範例撤回選擇，並回到選擇前的狀態的作法，是把 inner list 最後的數字拿掉，並且 visited 要改成回 false。

詳細看圖說故事步驟如下，舉例其中一條 branch 的詳細過程:

initial 時，什麼選擇都還沒開始時，這時 ```inner list: []```

for loop 來做第一層選擇: 選了 ```10```，紀錄在 visited 中並放到 inner list 內， ```inner list: [10]```
  - for loop 來做第二層選擇 :

      - 無法選 ```10```，因為 visited 中有紀錄了，故路徑上打 **X**
      - ```20``` 選擇沒有問題，紀錄在 visited 中並放到 inner list 內，這時 ```inner list: [10, 20]```
        - for loop 來做第三層選擇:
           > - 無法選 ```10```，因為 visited 中有紀錄了，故路徑上打 **X**
           > - 也無法選 ```20```，因為 visited 中有紀錄了，故路徑上打 **X**
           > - ```30``` 選擇沒有問題，紀錄在 visited 中並放到 inner list 內，這時 ```inner list: [10, 20, 30]```
           >
           >   - > 發現 inner list 的長度是滿的，故把   ```inner list: [10, 20, 30]``` 複製一份新的並存到 ans 列表中，返回上一層
           > - 因為返回了，所以需要 backtrack，方式是把 inner list 最尾端數字移除並把 visited 改成 false。這時 ```inner list: [10, 20]```
           > - for loop 跑完了第三層選擇，結束這層，故返回上一層

        - 因為返回了，所以需要 backtrack，方式是把 inner list 最尾端數字移除並把 visited 改成 false。這時 ```inner list: [10]```
      - 只剩下```30```，且選擇沒有問題，故紀錄在 visited 中並放到 inner list 內，這時 ```inner list: [10, 30]```

        - for loop 來做第三層選擇:
          > - 無法選 ```10```，因為 visited 中有紀錄了，故路徑上打 **X**
          > - ```20``` 選擇沒有問題，故紀錄在 visited 中並放到 inner list 內，這時 ```inner list: [10, 30, 20]```
          >    - > 發現 inner list 的長度是滿的，故把  ```inner list: [10, 30, 20]``` 複製一份新的並存到 ans 列表中，返回上一層
          > - 因為返回了，所以需要 backtrack，方式是把 inner list 最尾端數字移除並把 visited 改成 false。這時 ```inner list: [10, 30]```
          > - 無法選 ```30```，因為 visited 中有紀錄了，故路徑上打 **X**
          > - for loop 跑完了第三層選擇，結束這層，故返回上一層
        - 因為返回了，所以需要 backtrack，方式是把 inner list 最尾端數字移除並把 visited 改成 false。這時 ```inner list: [10]```
     - for loop 跑完了第二層選擇，結束這層，故返回上一層
  - 因為返回了，所以需要 backtrack，方式是把 inner list 最尾端數字移除並把 visited 改成 false。這時 ```inner list: []```

接下來的過程都類似，就不再贅述了。

---

### 參考資料

- [Backtracking回溯解题](https://www.youtube.com/watch?v=xqidNhvwKzI&list=PLV5qT67glKSErHD66rKTfqerMYz9OaTOs&index=18)
- [Leetcode — 47. Permutations II (中文)](https://anj910.medium.com/leetcode-47-permutations-ii-%E4%B8%AD%E6%96%87-a1c62414901e)
- [Leetcode — 39. Combination Sum (中文)](https://anj910.medium.com/leetcode-39-combination-sum-%E4%B8%AD%E6%96%87-c8577ed9a00b)
