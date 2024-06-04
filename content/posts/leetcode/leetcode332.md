---
title: 332. Reconstruct Itinerary

author: Aryido

date: 2022-09-17T10:03:29+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- graph
- dfs

comment: false

reward: false
---
<!--BODY-->
> 這種飛航問題基本上都是屬於 Graph 題，題目敘述也很生活化(~~根本旅行必備知識~~)。 因為所有的路徑有且只會被用一次，故是一個 **Euler Circuit**。
>
> 進一步抽象，可說這題是屬於 **Post-order traversal on Edges** 問題。 從入口做 **post-order** ，會是出口先被紀錄，然後再往回 **backtracking** 回入口，把路上的所有 node 都記下來。 老實說技巧性有點太強，且還是高頻...。 另外注意英文閱讀，有些單字很重要例如 *lexical order*，沒注意到可能會出現錯誤。

<!--more-->
## 思路
題目上有幾個條件要注意
1. 如果有兩個正確路徑，要回傳 **smallest lexical order**
2. 這題有特別給假設，至少一定有一條正確路徑
3. 每張票只能使用一次
4. 這題敘述和條件已知是 Euler Circuit，求其路徑。

使用 **Post-order** 的原因，是因為要走完後，知道訊息才能判斷這路徑正不正確。 **Pre-Order** 的 graph 概念上是先做再說，故在這題，若使用**Pre-Order**，可能會遇到飛往下一個地點後，發覺沒機票可以用了被卡死，所以要用 **Post-order**。

## 解題步驟
1. 建一個連結圖Heap map，稱**Adjacency Heap Map**，heap 主要是針對 lexical order 自然排序

2. Call dfs。既然是 DFS ，其中參數一定包含狀態，此題狀態參數為 current airport

3. 有用到 **getOrDefault()**，是因為有可能 **Map.get(key)** 是 **null**。 這個意思是代表當前地點從一開始就沒有任何機票，所以他一定是終點。 故直接加入解答裡。 另外補充如果是 **Map.get(key)** 有 Queue 但是空的，代表當前地點飛機票一開始有但用完了。

4. 最後特別注意，因為是 **Post-order**，故解答順序要反過來才是答案。

5. 時間複雜度: O(E*logE)

# 解答
```java
class Solution {
    Map<String, Queue<String>> adjacencyMap = new HashMap<>();
    List<String> ans = new LinkedList<>();

    public List<String> findItinerary(List<List<String>> tickets) {
        if(tickets.size() == 1){
            return tickets.get(0);
        }

        //init adjacencyMap
        for(List<String> ticket : tickets){
            adjacencyMap.computeIfAbsent(ticket.get(0), k -> new PriorityQueue<String>()).offer(ticket.get(1));
        }

        dfs("JFK");

        return ans;

    }

    public void dfs(String current){
        Queue<String> heap = adjacencyMap.getOrDefault(current, new PriorityQueue<String>());
        while(!heap.isEmpty()){
            String point = heap.poll();
            dfs(point);
        }
        ans.add(0,current);
    }
}
```

# Vocabulary
{{< alert info >}}
**Itinerary**[aɪˋtɪnə͵rɛrɪ] : 注意念法

n.[C] 旅程；路線；旅行計畫

adj. 旅行的；旅程的；路線的

{{< /alert >}}

{{< alert info >}}
**departure**[dɪˋpɑrtʃɚ]: 飛航常用單字

n.
1. 離開；出發，起程[C][U][（+for）]
2. 背離，違背，變更[C][（+from）]
{{< /alert >}}

{{< alert info >}}
**lexical**[ˋlɛksɪk!]

smallest lexical order:

'LGA' has a smaller lexical order than 'LGB'.

{{< /alert >}}


---