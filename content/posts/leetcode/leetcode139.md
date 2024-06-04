---
title: 139. Word Break

author: Aryido

date: 2022-11-12T22:10:19+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dp

comment: false

reward: false
---
<!--BODY-->
>一道很經典的題目，是給定一 string ，能不能分被拆分成 wordDict 裡面的單詞。注意這題，wordDict 裡面的單詞可以重複使用，即單詞使用沒有次數限制，所以 string 可以分成任意段，這就增加了題目的難度。解法蠻多種的，可先從 brute force 下手，再加上暫存優化後，就是蠻標準的 dp 解了，來解一下吧。
<!--more-->

---

## 思路
從頭開始切 string，如果切到位置可分拆成 wordDict 內的字時，再把剩餘 string再切一次。
```
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        return find(s, new HashSet<>(wordDict));
    }

    private boolean find(String s, Set<String> set){
        if(set.contains(s)){
            return true;
        }

        for(int i = 0 ; i < s.length() ; i++ ){
            String left = s.substring(0, i);
            String right = s.substring(i);
            if (set.contains(left) && find(right, set)) {
                return true;
            }
        }
        return false;
    }
}
```
{{< alert warning >}}
不希望每次查找都需要遍歷 wordDict，因為單詞使用沒有次數限制，故把 wordDict 轉成 HashSet 吧，可加速單詞比對效率。
{{< /alert >}}
但上述寫法會 Time Limit Exceeded。看到會超時的 case 如下 :
```
"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab"
["a","aa","aaa","aaaa","aaaaa","aaaaaa","aaaaaaa","aaaaaaaa","aaaaaaaaa","aaaaaaaaaa"]
```
是在哪裡導致超時的原因呢 ? 思考一下上述寫法，例如 *i = 1* 時，切完右邊剩下的 string 若處理完，會發現剛好也處理完成 *i = 2* 會遇到的 case。但因為是在```for loop```中呼叫遞迴，故如果沒有把*i = 1* 時，處理過的 case 儲存下來的話，*i = 2* 就只能再處理一遍，會因檢查太多次一樣的 substring 導致超時。

{{< alert success >}}
搭配一個 HashMap 來紀錄 substring 的檢查結果
{{< /alert >}}

# 解答
```java
class Solution {
    Map<String, Boolean> map = new HashMap<>();

    public boolean wordBreak(String s, List<String> wordDict) {
        return find(s, new HashSet<>(wordDict));
    }

    private boolean find(String s, Set<String> set){
        if (map.containsKey(s)) {
            return map.get(s);
        }

        if(set.contains(s)){
            return true;
        }

        for(int i = 0 ; i < s.length() ; i++ ){
            String left = s.substring(0, i);
            String right = s.substring(i);
            if (set.contains(left) && find(right, set)) {
                return map.compute(s, (k,v) -> true);
            }
        }
         return map.compute(s, (k,v) -> false);
    }
}
```
{{< alert success >}}
```map.compute(s, (k,v) -> false)```會把資料存入 map 中並返回最新的 value。 等同於 :
```
map.put(s, true);
return true;
```
{{< /alert >}}

{{< alert info >}}
Time: O(n^2)
{{< /alert >}}

---
# 補充
[還有一個雙迴圈版本可以看一下](https://blog.jiebu-lang.com/leetcode-139-word-break/)，蠻厲害的。
