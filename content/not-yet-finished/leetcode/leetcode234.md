---
title: "234. Palindrome Linked List
"

author: Aryido

date: 2023-08-26T01:13:14+08:00

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
    public boolean isPalindrome(ListNode head) {
        if(head.next == null){
            return true;
        }

        ListNode pre = new ListNode();
        pre.next = head;
        ListNode faster = pre;
        ListNode slower = pre;
        while(faster != null && faster.next != null ){
            faster = faster.next.next;
            slower = slower.next;
        }
        ListNode mid = slower;

        pre = null;
        ListNode curr = mid.next;
        while(curr != null){
            ListNode next = curr.next;
            curr.next = pre;
            pre = curr;
            curr = next;
        }

        while( pre != null ){
            if(head.val != pre.val){
                return false;
            }
            head = head.next;
            pre = pre.next;
        }

        return true;
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