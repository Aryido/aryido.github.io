---
title: "763. Partition Labels"

author: Aryido

date: 2023-03-05T22:48:40+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 這道題給了一個 string，然後要盡可能將 string 切割越多塊 sub-string 越好( as many parts as possible)，其中條件是**每個 char，最多只能出現在自己的 sub-string 中**。 即 :
>
> - 分割字串使字串中的每個字母在該分割段落中出現達到最多次。
>
> 題目理解是和自己想解法是比較花時間的，看過解答後都可以很快寫出來。
<!--more-->

---

## 思路
用範例來說明吧，首先要遍歷 string ，然後要將字母最後一次出現的位置記錄起來。 觀察一下```ababcbacadefegdehijhklij```，
- *a* : 8
- *b* : 5
- *c* : 7
- ...

*a* 這個字母，透過前面 map 紀錄，知道 *a* 最後出現 index 是 8。 然後使用一個 end 變數，來儲存當前出現過的所有字母中最後一個出現的位置。
這代表直到我們掃到第 8 個字母之前，我們都不能分割任何段落。

再看 *b* 這個字母，，透過前面 map 紀錄，知道 *b* 最後出現 index 是 5。但 5 比 8 小，代表 *b* 字母都在 *a* 之前，故不用更新 end 的值。

{{< alert info >}}
這邊假設一下，若在中途，碰到了一個字母 *x* 且其最後出現的位置在第 *y* 比 8 大，那 end 的值就要更新。
{{< /alert >}}

經由以上分析，做完 map 後，重新遍歷 string ，並使用一個 end 變數，來儲存**當前出現過的所有字母中最後一個出現的位置，和 end ，這兩個選一個大的**。那什麼時候可以切割呢？**當 end == i 時就可以切割了**。因為這表示目前出現過的所有 **char** 都不會在之後出現，可以將這個 sub-string 長度紀錄到答案中，並將 *start* 設為 *end + 1*，繼續開始直到 string 遍歷完畢。

{{< alert success >}}
兩次分開 for-loop string，故時間複雜度為 O(n)；

因為題目 Constraints 有 ```s consists of lowercase English letters('a' to 'z') only.```
故只需要儲存 26 個字母，以及每個 sub-string 的長度，所以空間複雜度可以視為 O(1)。
{{< /alert >}}

# 解答
```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer> ans = new ArrayList<>();
        Map<Character, Integer> map = new HashMap<>();
        for(int i = 0 ; i < s.length() ; i++){
            map.put(s.charAt(i), i);
        }
        int start = 0;
        int end  = 0;
        for(int i = 0 ; i < s.length() ; i++){
            end = Math.max(end, map.get(s.charAt(i)));
            if(end == i){
                ans.add(end - start + 1 );
                start = end + 1;
            }
        }
        return ans;
    }
}
```

---
# 優化解法
特別注意，因為只需要儲存 26 個字母，所以可以進一步使用 *array* 來儲存 *char* !

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        int lastIndex[] = new int[128];
        for (int i = 0; i < s.length(); ++i){
            lastIndex[(int)s.charAt(i)] = i;
        }
        List<Integer> ans = new ArrayList<>();
        int start = 0;
        int end = 0;
        for (int i = 0; i < s.length(); ++i) {
            end = Math.max(end, lastIndex[(int)s.charAt(i)]);
            if (i == end) {
                ans.add(end - start + 1);
                start = end + 1;
            }
        }
        return ans;
    }
}
```

---

# Brute Force
```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer> ans = new ArrayList<>();
        int start = 0;
        int end  = 0;
        for(int i = 0 ; i < s.length() ; i++){
            end = Math.max(end, s.lastIndexOf(s.charAt(i)));
            if(end == i){
                ans.add(end - start + 1 );
                start = end + 1;
            }
        }
        return ans;
    }
}
```

{{< alert success >}}
for-loop string 內有 lastIndexOf()方法，故 Time complexity: O(n^2)

Space complexity: 一樣O(1)
{{< /alert >}}

---