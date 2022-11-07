---
title: 127. Word Ladder

author: Aryido

date: 2022-10-09T23:23:53+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- BFS
- graph
- english

comment: false

reward: false
---
<!--BODY-->
> 這題第一眼其實看不太出來是 graph 題，但仔細分析會發現是一個單詞，然後能 reach 到的是換一個字母的單詞，就是鄰居；然後要找最短路徑。 難就是難在一開始要把問題轉化成一個 graph!

<!--more-->

---

先回憶一下迷宮遍歷，迷宮中位置有上下左右**4個方向**可以走，而這裡每個單字的 index 有26個字母，就是二十六個方向可以走，總共可以走的方向是 **26 * 單字長度**，其實本質完全一樣！至於要用 BFS 還是 DFS 呢 ? DFS的話是一條路走到底，走的那條道不一定是最短的；而 BFS 一層一層慢慢擴大，故選用 Breadth First Search 來解題。

## 思路
以下解法會超時，看看那些部份可以改進吧

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        // edge case
        if(!wordList.contains(endWord)){
            return 0;
        }

        // init: BFS ; visited set
        Queue<String> queue = new LinkedList<>();
        Set<String> visited = new HashSet<>();
        queue.offer(beginWord);
        int level = 0;

        // processing
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i = 0 ; i < size ; i++){
                String cur = queue.poll();

                if(cur.equals(endWord)){
                    return ++level;
                }

                // main alg
                char[] charArray = cur.toCharArray();
                for(int j = 0 ; j < charArray.length ; j ++){
                    char tmp = charArray[j];
                    for(char c = 'a' ; c <= 'z' ; c++){
                        array[j] = c;
                        String str = new String(array);
                        if(wordList.contains(str) && !visited.contains(str)){
                            queue.offer(str);
                            visited.add(str);
                        }
                    }
                    charArray[j] = tmp;
                }
            }
            level++;
        }

        return 0;
    }
}
```
首先針對**查重複**部分做優化，一開始是在 **wordList** 和 **visited** 來判斷，但看起來超時 case 是因為 **wordList** 太長，故在這邊我換個想法，把所有 **wordList** 轉成 set 。
{{< alert info >}}
這邊可以把 wordList 轉成 set，主要是因為題目條件有說 wordList 不重複，轉成 set 可以快速查找!
{{< /alert >}}

瞬間就通過測試了...，再看看還有甚麼可以優化的吧 ! 想想把 wordList 轉成 set 然後我再做一個 visited set 其實有點多餘，兩者可以合併。看新造的單字有沒有在 set 內，如果有的話才會把新造的單字加到 queue 裡，並且把 set 裡該單字移除。藉由移除單字使得重複的單字不會繼續 BFS 導致死循環。

另外原本寫法會修改到 char array ，所以我會記錄修改的字母，然後最後把字母回復原樣，但感覺有點不好看，這邊我會用 **copyOf** ，來讓 code 更好看。
{{< alert info >}}
**copyOf** 會讓速度減慢，但是 code 會比較好看，且避免修改 **reference** 造成錯誤
{{< /alert >}}


# 解答
```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        if(!wordList.contains(endWord)){
            return 0;
        }

        Queue<String> queue = new LinkedList<>();
        Set<String> visited = new HashSet<>(wordList);
        queue.offer(beginWord);
        int level = 0;

        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i = 0 ; i < size ; i++){
                String cur = queue.poll();

                if(cur.equals(endWord)){
                    return level + 1;
                }

                char[] charArray = cur.toCharArray();
                for(int j = 0 ; j < charArray.length ; j ++){
                    char[] tmpArray = Arrays.copyOf(charArray, charArray.length);
                    for(char c = 'a' ; c <= 'z' ; c++){
                        tmpArray[j] = c;
                        String str = new String(tmpArray);
                        if(visited.contains(str)){
                            queue.offer(str);
                            visited.remove(str);
                        }
                    }
                }
            }
            level++;
        }

        return 0;
    }
}
```

{{< alert info >}}
Queue 是 interface ，提醒常用的實作是 LinkedList
```java
Queue<String> queue = new LinkedList<>();
```

{{< /alert >}}

{{< alert warning >}}
因為題目有說字母都是小寫英文單字，故:
實用小技巧
```java
for(char c = 'a' ; c <= 'z' ; c++){
}

```
{{< /alert >}}

---

# Follow up
想想上面解法比較特別部分，是因為題目有說字母都是小寫英文單字，故用了
```java
for(char c = 'a' ; c <= 'z' ; c++){
}

```
來簡化，那如果現在單字沒有限制了呢應該怎麼寫呢 ?

## 思路
首先先來構建 graph 。因為每個單字長度一樣，故兩兩比較每個單字，若只差一個字母不一樣，我們就說他們可以 reach ，明顯這是一個**無向圖**，構建完時間空間複雜度都是 **O(n^2)**。

整個 graph 都構建完之後，接下來就是尋找最短路徑就好了。

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        if(!wordList.contains(endWord)){
            return 0;
        }

        if(!beginWord.contains(endWord)){
            wordList.add(beginWord);
        }

        Map<String,List<String>> graph = generateGraph(wordList);

        Set<String> visited = new HashSet<String>();
        Queue<String> queue = new LinkedList<String>();
        visited.add(beginWord);
        queue.add(beginWord);

        int ans = 1;
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i = 0 ; i < size ; i++){
                String cur = queue.poll();
                if(cur.equals(endWord)){
                    return ans;
                }
                for(String str : graph.getOrDefault(cur, new ArrayList<String>())){
                    if(!visited.contains(str)){
                        visited.add(str);
                        queue.offer(str);
                    }
                }
            }
            ans++;
        }
        return 0;
    }

    private Map<String,List<String>> generateGraph(List<String> wordList){
        Map<String,List<String>> map = new HashMap<String,List<String>>();

        for(int i = 0 ; i < wordList.size() - 1 ; i++){
            for(int j = i+1 ; j < wordList.size() ; j++ ){
                String w1 = wordList.get(i);
                String w2 = wordList.get(j);
                if(determind(w1, w2)){
                    map.computeIfAbsent(w1, k->new ArrayList<String>()).add(w2);
                    map.computeIfAbsent(w2, k->new ArrayList<String>()).add(w1);
                }
            }

        }
        return map;
    }

    private boolean determind(String w1, String w2){
        int diff =  0;
        char[] char1 = w1.toCharArray();
        char[] char2 = w2.toCharArray();

        for(int i = 0 ; i < char1.length ; i++){
            if(char1[i] == char2[i]){
                continue;
            }
            diff++;
        }

        return diff == 1;
    }
}
```

{{< alert info >}}
有用到 **computeIfAbsent** 簡化建 graph 的 code
{{< /alert >}}

{{< alert warning >}}
在 generateGraph 時，要把 beginWord 也放進去。
{{< /alert >}}


---
# Vocabulary

{{< alert info >}}
**ladder** [ˋlædɚ]

n.[C]梯子；階梯；途徑

{{< /alert >}}

{{< alert info >}}
**adjacent** [əˋdʒesənt] : 注意 d 不發音

a. 鄰接的；前後相接的

{{< /alert >}}


---