---
title: Leetcode83

author: Aryido

date: 2022-12-18T21:50:23+08:00

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
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null){
            return head;
        }

        ListNode cur = head;
        ListNode next = head.next;

        while(next != null){
            if(next.val == cur.val){
                next = next.next;
                cur.next = next;
            }else{
                 next = next.next;
                 cur = cur.next;
            }
        }
        return head;
    }
}
```

# Vocabulary

[字典連結](https://tw.dictionary.search.yahoo.com/search;_ylt=AwrtXGs1MCVj1V8AZAh9rolQ;_ylc=X1MDMTM1MTIwMDM4MQRfcgMyBGZyAwRmcjIDc2ItdG9wBGdwcmlkA3VHbnhCdFdPUnBlU3k0a1ZuS1A0VUEEbl9yc2x0AzAEbl9zdWdnAzQEb3JpZ2luA3R3LmRpY3Rpb25hcnkuc2VhcmNoLnlhaG9vLmNvbQRwb3MDMARwcXN0cgMEcHFzdHJsAzAEcXN0cmwDMTAEcXVlcnkDZGVwYXJ0dXJlJTIwBHRfc3RtcAMxNjYzMzgxODE3?p=departure+&fr2=sb-top)

{{< alert info >}}
**xxxx** [KK] : (注意事項)

{{< /alert >}}
---