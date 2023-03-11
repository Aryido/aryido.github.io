---
title: "76. Minimum Window Substring"

author: Aryido

date: 2023-03-11T12:13:17+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 題目給了我們一個字符 *s*，還有一個目標字符 *t*，要在 *s* 中找到一個 **minimum window substring** 使得其包含了 *t* 中的所有的字母。整體看起來題目難在 :
> - 限制了時間複雜度為 *O(n + m)*
> - 第一次要寫出 bug free 有點困難
>
> 故備標註為 **hard** ，但整體思路上並不算太難，值得品味一下 !
>

<!--more-->

---
題目有些可以釐清的部分，以下同時列出和說明 :

- 題目只要求 **minimum window substring** 包含了 *t* 中的所有的字母，故:
  - **字母順序不用考慮**
  - **字母數量要一樣**

- The testcases will be generated such that the answer is **unique**.
    {{< alert success >}}
上面敘述是題目的給的假設，確定給的範例不會找到兩個不一樣字串，但是都是 **minimum window substring**。

但這邊可以馬上想到，題目也可以改成找出所有符合 **minimum window substring** 的字串，很有機會遇到這種考題的。
    {{< /alert >}}
- *s* and *t* consist of uppercase and lowercase English letters.
    {{< alert info >}}
這個是題目 Constraints ，老套路了! 看到這個就知道有機會用 Array 來取代 Map !
    {{< /alert >}}

- 因為有限制了時間複雜度為 *O(n + m)* ，故基本上要在一次遍歷中完成任務。

## 思路
有限制了時間複雜度為 *O(n + m)*，故 *Brute Force* 肯定不能用，因為遍歷所有的子串的時間複雜度是 *O(n^2)*。作法簡述流程是 :
- 建立一個 *tMap* 表示 *t* 字串每個字母和出現次數
- 建立 *start* 、 *i* 兩指針
- 指針 *i* 逐漸前進，看看 ```s.charAt(i)``` 有沒有在 *tMap* 內，且要順便做一個 *sMap* 統計 ```s.charAt(i)``` 數量
- 承前， *sMap* 數量不能超過 *tMap*  對應數量，若發現超過且 ***start* 指針指向的字母和 *i* 指針指向的字母一樣**，則要開始往前移動 *start* 指針。

{{< alert success >}}
擴大右邊界，但滿足某些條件會縮小左邊界，很像一個不停滑動的窗口，這類操作手法稱:
**Sliding Window**，能解很多子串，子數組，子序列等等的問題，必須要熟練掌握。
{{< /alert >}}

---

# 解答
```java
class Solution {
    public String minWindow(String s, String t) {
        String result = "";

        Map<Character, Integer> tMap = new HashMap<>();
        for(int i = 0 ; i < t.length(); i++){
            Character tChar = t.charAt(i);
            tMap.compute(tChar, (k,v) -> v == null ? 1 : v+1);
        }

        int count = 0;
        int startIndex = 0;
        Map<Character, Integer> sMap = new HashMap<>();
        for(int i = 0 ; i < s.length(); i++){
            char sChar = s.charAt(i);
            if(tMap.containsKey(sChar)){
                sMap.compute(sChar, (k,v) -> v == null ? 1 : v+1);
                if(tMap.get(sChar) >= sMap.get(sChar)){
                    count++;
                }
            }
            if(count == t.length()){
                while(tMap.getOrDefault(s.charAt(startIndex),-1) < sMap.getOrDefault(s.charAt(startIndex),-1) || tMap.getOrDefault(s.charAt(startIndex),-1) == -1){
                    if( tMap.getOrDefault(s.charAt(startIndex),-1) == -1){
                        startIndex++;
                        continue;
                    }

                    if(tMap.get(s.charAt(startIndex)) < sMap.get(s.charAt(startIndex))){
                        sMap.compute(s.charAt(startIndex), (k,v) -> v-1);
                        startIndex++;
                    }
                }
                if(result.equals("") ||  i - startIndex + 1 <= result.length()){
                    result = s.substring(startIndex, i+1);
                }
            }
        }

        return result;
    }
}
```
上面的 code 是我第一次解的，還蠻髒的..，因為對於 *map structure*， *key* 存在與否嚴重影響這題的寫法，所以 while loop 縮小 *startIndex* 這邊寫的很亂...

---
# 優化解
因為 *s* and *t* consist of uppercase and lowercase English letters，故可不用 HashMap，直接用 *int array*來代替，

-  使用 Map
    {{< image classes="fancybox fig-100" src="/images/leetcode/76-map.jpg" >}}
-  使用 Array
    {{< image classes="fancybox fig-100" src="/images/leetcode/76-array.jpg" >}}

其餘部分完全相同。雖然只改了 **data structure**，但是運行速度提高超多...，array 還是比 HashMap 快很多的。

{{< alert info >}}
ASCII 只有256個字符，因此這題可用個大小為 256 的 int 數組。但由於題目只有小寫字母所以也可以只用 128 的 int 數組
{{< /alert >}}

另外也因為使用了 ```int[]```，他有 **default value 0**，故整個判斷式變得比較乾淨。

```java
class Solution {
    public String minWindow(String s, String t) {
        String result = "";
        int[] tArray = new int[128];
        for(int i = 0 ; i < t.length(); i++){
            int amount = tArray[(int)t.charAt(i)];
            tArray[(int)t.charAt(i)] = amount + 1;
        }

        int count = 0;
        int startIndex = 0;
        int[] sArray = new int[128];
        for(int i = 0 ; i < s.length(); i++){
            if(tArray[(int)s.charAt(i)] != 0){
                int amount = sArray[(int)s.charAt(i)];
                sArray[(int)s.charAt(i)] = amount + 1;
                if(tArray[(int)s.charAt(i)] >= sArray[(int)s.charAt(i)]){
                    count++;
                }
            }
            if(count == t.length()){
                while(tArray[(int)s.charAt(startIndex)] < sArray[(int)s.charAt(startIndex)]
                    || tArray[(int)s.charAt(startIndex)] == 0
                ){
                    if(tArray[(int)s.charAt(startIndex)] < sArray[(int)s.charAt(startIndex)]){
                        int amount = sArray[(int)s.charAt(startIndex)];
                        sArray[(int)s.charAt(startIndex)] = amount - 1;
                    }
                    startIndex++;
                }
                if(result.equals("") ||  i - startIndex + 1 <= result.length()){
                    result = s.substring(startIndex, i+1);
                }
            }
        }

        return result;
    }
}
```

---