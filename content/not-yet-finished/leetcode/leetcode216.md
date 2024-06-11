---
title: "Leetcode216"

author: Aryido

date: 2023-10-17T23:29:29+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
>
<!--more-->

---

# 思路

---

# 解答
```java
class Solution {
     List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> combinationSum3(int k, int n) {
       dfs(k, n, new ArrayList<>(), 1);
       return ans;
    }

    public void dfs(int k, int target, List<Integer> inner, int level){
        if(target == 0 && inner.size() == k){
            ans.add(new ArrayList<>(inner));
            return;
        }

        if( target < 0 || inner.size() > k){
            return;
        }

        for(int i = level ; i <= 9 ; i++){
            inner.add(i);
            dfs(k, target - i, inner, i+1);
            inner.remove(inner.size() - 1);
        }

    }
}
```

---

# 時間空間複雜度

### 時間複雜度 : ```O(N^2)```
兩層迴圈

### 空間複雜度 : ```O(N^2)```
2-D陣列

---

{{< alert info >}}
example
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/leetcode/logo.jpg" >}}

---

# Vocabulary

[字典連結](https://tw.dictionary.search.yahoo.com/search;_ylt=AwrtXGs1MCVj1V8AZAh9rolQ;_ylc=X1MDMTM1MTIwMDM4MQRfcgMyBGZyAwRmcjIDc2ItdG9wBGdwcmlkA3VHbnhCdFdPUnBlU3k0a1ZuS1A0VUEEbl9yc2x0AzAEbl9zdWdnAzQEb3JpZ2luA3R3LmRpY3Rpb25hcnkuc2VhcmNoLnlhaG9vLmNvbQRwb3MDMARwcXN0cgMEcHFzdHJsAzAEcXN0cmwDMTAEcXVlcnkDZGVwYXJ0dXJlJTIwBHRfc3RtcAMxNjYzMzgxODE3?p=departure+&fr2=sb-top)

{{< alert info >}}
**xxxx** [KK] : (注意事項)

{{< /alert >}}

---
### 參考資料

- [[LeetCode]5. Longest Palindromic Substring 中文](https://www.youtube.com/watch?v=ZnzvU03HtYk)