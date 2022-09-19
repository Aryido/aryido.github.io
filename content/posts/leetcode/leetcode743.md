---
title: 743. Network Delay Time

author: Aryido

date: 2022-09-19T21:09:18+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- java

tags:
- java
- LeetCode
- best first search

comment: false

reward: false
---
<!--BODY-->
> 可以抽象成，計算從**初始節點**到**最遠節點**的最優路徑，很標準的 *best first search*。 題目常用在水管滲透，或是網路流通，求出初始節點到每一個點到最短時間，然後取其中最大的一個就是需要的時間了。

<!--more-->

## 思路
給定 n 個節點組成的網絡，有times[i] = (ui, vi, wi) 的旅行時間列表，其中 ui 是 source  ，vi 是 target ，wi 是所需的時間。 給定一節點發送一個信號。 返回所有節點走完所需的最短時間。 如果不可能所有的n個節點都接收到信號，則返回-1。

- 這題使用到 Dijkstra 算法這種類貪心算法的機制，無法處理有負權重的最短距離，還好這道題的權重都是正數。
- 注意 while 循環裡的第一層 for 循環，這保證了每一層的結點先被處理完，才會進入進入下一層
- 為了防止重複比較，需要使用 visited 數組來記錄已訪問過的結點

## 解題步驟
1. 建一個**Adjacency List Map**，其中為了直觀點，可以建造一個Cell class，記得要 override compareTo 方法，因為會需要 Heap 排序。

(未完待續)

# 解答
```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        Map<Integer, List<Edge>> map = new HashMap<>();
        for(int[] time : times){
           map.computeIfAbsent(time[0], key -> new ArrayList<Edge>()).add(new Edge(time[1],time[2]));
        }

        Queue<Edge> pq = new PriorityQueue<>();
        boolean[] visited = new boolean[n+1];
        pq.offer(new Edge(k, 0));
        int res = 0;

        while(!pq.isEmpty()){
            Edge edge = pq.poll();
            if(visited[edge.targetNode]){
                continue;
            }
            visited[edge.targetNode] = true;
            res = edge.time;
            n--;
            if(map.containsKey(edge.targetNode)){
                for(Edge e : map.get(edge.targetNode)){
                    pq.offer(new Edge(e.targetNode, e.time + edge.time));
                }
            }
        }

        return n == 0 ? res : -1;
    }

    class Edge implements Comparable<Edge> {
        int targetNode;
        int time;

        public Edge(int targetNode, int time){
            this.targetNode = targetNode;
            this.time = time;
        }

        @Override
        public int compareTo(Edge edge){
            return this.time - edge.time;
        }

    }
}
```

---