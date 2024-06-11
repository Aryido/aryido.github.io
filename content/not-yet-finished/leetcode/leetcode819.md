---
title: Leetcode819

author: Aryido

date: 2022-10-10T15:31:46+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->

<!--more-->

---

## 思路

# 解答
```java
class Solution {
    public String mostCommonWord(String paragraph, String[] banned) {
        String[] words =  paragraph.replaceAll("\\pP"," ").toLowerCase().split("\\s+");
        List<String> wordList = Arrays.asList(words);

        HashMap<String, Integer> map = new HashMap<>();
        wordList.stream().map(str -> str.trim()).forEach(str -> map.compute(str, (k, v) -> v == null ? 1 : v + 1));

        for(String bw : banned){
            if(map.containsKey(bw)){
                map.remove(bw);
            }
        }

        String ans = "";
        int max = 0;
        for(var kvp : map.entrySet()){
            if(kvp.getValue() > max){
                ans = kvp.getKey();
                max = kvp.getValue();
            }
        }

        return ans;
    }
}
```

# Vocabulary

[字典連結](https://tw.dictionary.search.yahoo.com/search;_ylt=AwrtXGs1MCVj1V8AZAh9rolQ;_ylc=X1MDMTM1MTIwMDM4MQRfcgMyBGZyAwRmcjIDc2ItdG9wBGdwcmlkA3VHbnhCdFdPUnBlU3k0a1ZuS1A0VUEEbl9yc2x0AzAEbl9zdWdnAzQEb3JpZ2luA3R3LmRpY3Rpb25hcnkuc2VhcmNoLnlhaG9vLmNvbQRwb3MDMARwcXN0cgMEcHFzdHJsAzAEcXN0cmwDMTAEcXVlcnkDZGVwYXJ0dXJlJTIwBHRfc3RtcAMxNjYzMzgxODE3?p=departure+&fr2=sb-top)

{{< alert info >}}
**xxxx** [KK] : (注意事項)

{{< /alert >}}
---