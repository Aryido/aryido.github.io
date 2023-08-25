---
title: "Multiple points 題型簡單介紹"

author: Aryido

date: 2023-08-23T22:50:25+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- algorithm

tags:
- multiple-pointers
- array
- linked-list

comment: false

reward: false
---
<!--BODY-->
> Multiple points 的操作，經常用在 array 或 linkedList 上，有幾點事情可以在刷題時特別注意:
> 1. 指標會把 list 切成幾個部分，特別注意每一部份的定義
>
> 2. list 是否有排序或可以排序
>
> 3. 指標移動是使用**快慢指標**還是**左右指標**
>
> 4. 會不會改變原本的 list
>
> 其中第一點，**把 array 切成幾個部分**，每個部份的**定義**，是最重要的思想，可以多思考。

<!--more-->
---

## 區間定義
針對 Two points 的操作，會把目標切成幾個部分，可以由下圖知道，這裡舉例 ```i, j``` :
{{< image classes="fancybox fig-100" src="/images/algorithm/two-points.jpg" >}}
可以知道會分成三個部分，再來由指標**前進方向**來分類，分成的三個部分會有一些定義差別 :

- 同向
    - ```[0,i)``` : **處理好**
    - ```[i,j)``` : **不需要**
    - ```[j,length)``` : **還沒處理過的**
- 反向
    - ```[0,i)``` : **處理好**
    - ```[i,j]``` : **還沒處理過的**
    - ```(j,length)``` : **處理好**

{{< alert warning >}}
區間的**開閉**，會需要依照題目要求定義，但注意 ! 每個區間連接處(如 *i* 點位置)，**不能有兩種意思** !
{{< /alert >}}

## 快慢指標
多用於 Linked-List 的問題 :
- 找到 Linked-List 中心點；慢指標一次走一步，快指標一次走兩步，當 fast pointer 抵達尾端的時候， slow pointer 就會是在中間的位置。
- 找到 Linked-List 倒數第 K 個節點；先讓 fast pointer 先走 K 步，然後 Slow and Fast 兩個指標再使用一樣的速率前進，當 fast pointer 抵達尾端的時候， slow pointer 就會是在倒數第 K 個節點的位置
- 可以判斷 linked list 是否有環，若是兩個指標相遇，代表有環

{{< alert success >}}
該如何理解兩指標相遇就代表環存在？

想成在操場上跑步，跑的快的人最終都會超過跑的慢的人，兩個指標如果都進入到環中，最終跑的快的人會追上跑的慢的人。
{{< /alert >}}


## 左右指標
主要處理對稱相關的題目，例如 :
- 反轉 array
- 檢查 **palindrome 回文**



---

### 參考資料

- [[LeetCode]5. Longest Palindromic Substring 中文](https://www.youtube.com/watch?v=ZnzvU03HtYk)

- [Manacher's Algorithm](https://www.cnblogs.com/grandyang/p/4464476.html)

- [Leetcode 刷題 pattern - Two Pointer](https://blog.techbridge.cc/2019/08/30/leetcode-pattern-two-pointer/)

- [super_baba 第十天 - Two-pointer ](https://ithelp.ithome.com.tw/articles/10262277)